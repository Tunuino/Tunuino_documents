# Tunuino Console — USB, DFU e gravação de firmware

**[Descarregar o kit aprovado: Console Windows + firmware F427](https://github.com/Tunuino/Tunuino/actions/runs/39823286596/artifacts/12444449828)**. A [pipeline final](https://github.com/Tunuino/Tunuino/actions/runs/39823286596) passou em 1 min 39 s, incluindo testes, compilação ARM, empacotamento Windows, abertura dos controlos JavaFX e criação do kit sincronizado. Não houve ensaio físico.

Implementação inicial: [89724f6](https://github.com/Tunuino/Tunuino/commit/89724f6f09236078c839630d7098db8a393ba5c2). Revisão atual: [67ddd5a](https://github.com/Tunuino/Tunuino/commit/67ddd5aa02a30afd631fc2f3e32f5255577c94b8), diretamente na master.

A interface tem os separadores **Dados e mapas** e **Firmware e USB**, com tamanho inicial adaptado ao ecrã.

## Implementado

- Atualização periódica das portas COM disponíveis, preservando a seleção quando a porta continua presente.
- Pesquisa independente de dispositivos USB STM32 DFU pelo hardware ID `0483:DF11`, mesmo sem qualquer porta COM.
- Apresentação dos erros de driver do Windows; erro de pesquisa não é apresentado como ausência de dispositivo.
- Seleção da ferramenta ST instalada, ou da pasta `console/STM32_Programmer_CLI/bin/STM32_Programmer_CLI.exe` de um pacote rusEFI existente. O caminho fica guardado para utilização offline.
- Seleção do dispositivo DFU por número de série, reenumeração antes de gravar e verificação do grupo STM32F42x/F43x (`0x419`). Este ID é partilhado; não prova sozinho que o chip é F427 ou que a revisão da placa é v0.7. A interface pede confirmação explícita da placa antes de gravar.
- Seleção de firmware com manifesto de alvo, modo, endereço, SHA-256 e declaração de substituição do bootloader. Validação dos vetores de arranque e dos limites da imagem.
- Gravação DFU na Console através da ferramenta ST, com verificação obrigatória. Só indica sucesso com saída positiva da ferramenta e confirmação de verificação.
- Registo da saída do programador, limites de tempo, tratamento de falhas, bloqueio de ligação COM e de fecho da janela durante programação.
- Instalação dos drivers ST a partir do `silent_st_drivers2.exe` do pacote rusEFI, usando o fluxo de extração e o script Windows fornecido nesse instalador. O Windows pode pedir autorização de administrador.
- Caminho manual OpenBLT através da ferramenta oficial `BootCommander.exe`, transporte XCP serial a 115200, para imagens `.srec` próprias com manifesto e endereços fora da área reservada ao bootloader.

## Primeiro ensaio com a Proteus v0.7 STM32F427

1. Descarregar o kit da execução CI correspondente a este commit e extrair tudo. Os kits anteriores não contêm o novo manifesto de gravação.
2. Guardar a instalação atual da ECU antes de a substituir. O ensaio continua a ser de bancada, com atuadores desligados.
3. Executar `console/TunuinoConsole.exe` e abrir o separador **Firmware e USB**.
4. A Console procura uma instalação normal do STM32CubeProgrammer. Se não a encontrar, usar **Ferramenta ST…** e selecionar o executável que já existe no pacote rusEFI. Manter todas as DLLs e a pasta `Data_Base` da ferramenta.
5. Entrar em DFU através de BOOT0 e reset, conforme a placa. A pesquisa DFU é independente da lista COM. O dispositivo deve aparecer na lista **Dispositivo USB DFU**.
6. Se o Windows indicar erro de driver, usar **Instalar drivers…** e selecionar `drivers/silent_st_drivers2.exe` do pacote rusEFI. Também pode instalar os drivers fornecidos com o CubeProgrammer oficial.
7. Usar **Selecionar firmware** e escolher `firmware/tunuino-proteus-v07-f427-usb-bench.bin`. Manter o ficheiro `.bin.properties` ao lado. A Console valida ambos.
8. Encerrar qualquer ligação COM antes de gravar, confirmar o dispositivo USB e usar **Gravar por DFU**. A confirmação mostra que a imagem substitui o bootloader na flash.
9. Depois da verificação positiva, repor BOOT0 em arranque normal e reiniciar a ECU. Selecionar a COM, ligar e identificar a ECU.

Não há apagamento integral, remoção de proteções, alteração de option bytes ou arranque forçado em endereço base neste fluxo.

## OpenBLT: limites atuais

O backend manual está implementado, mas **não existe ainda uma imagem Tunuino OpenBLT validada para esta placa**. A imagem USB de bancada atual é exclusivamente DFU e começa em `0x08000000`; a Console recusa usá-la no modo OpenBLT.

Para esse modo é necessário ter o bootloader já instalado e ativo, a ferramenta oficial BootCommander com as respetivas DLLs, e firmware `.srec` compilado para os endereços da aplicação desse bootloader. Não converter a imagem DFU atual em `.srec` para tentar contornar esta diferença.

A deteção automática da assinatura OpenBLT, a mudança automática do firmware para DFU/OpenBLT e a instalação de um bootloader Tunuino não estão implementadas nesta entrega. O nome de uma porta COM não prova que está em OpenBLT. Não foi copiado para a Tunuino o comando de reboot rusEFI: o firmware Tunuino tem protocolo próprio e precisa de implementar esse comando antes de o usar.

## Pacote Windows e ferramentas externas

O pacote inclui Java, JavaFX, biblioteca serial, os novos controlos e instruções offline em `tools/PROGRAMMERS.txt`. A compilação padrão **não inclui os executáveis ST, BootCommander ou o instalador de drivers**: usa a instalação local ou o pacote rusEFI que o utilizador já possui.

O empacotador `tools/package_programmer_tools.py` pode incluir distribuições completas fornecidas durante a compilação:

- `TUNUINO_ST_TOOLS`: pasta `STM32_Programmer_CLI` com `bin/STM32_Programmer_CLI.exe`, DLLs, bases de dispositivos e avisos do fornecedor.
- `TUNUINO_OPENBLT_TOOLS`: pasta com `BootCommander.exe`, bibliotecas e avisos do fornecedor.
- `TUNUINO_ST_DRIVERS`: caminho para `silent_st_drivers2.exe`.

O empacotador regista hashes dos ficheiros incluídos e não descarrega instaladores na máquina do utilizador. A Console também reconhece as ferramentas colocadas no layout portátil indicado no ficheiro de instruções.

## Manifestos

A compilação do firmware de bancada gera automaticamente o `.bin.properties` e inclui o seu hash no `manifest.json`. O empacotamento do kit verifica esse ficheiro juntamente com o firmware, mantendo Console e firmware no mesmo commit.

O manifesto é um contrato local de alvo e integridade, não uma assinatura digital do fabricante nem uma deteção física do hardware.

## Validação

- Suite local C/Java/KiCad aprovada, incluindo os testes de comunicação com o firmware C e callbacks USB simulados.
- Compilação local de todas as classes Java, incluindo JavaFX e os novos controlos, aprovada.
- Testes novos com ferramentas simuladas: deteção/erro de driver, múltiplos dispositivos, reatribuição de USB1 para USB2, seleção por número de série, MCU errado, dispositivo desligado, falha de verificação, código de saída de erro, integridade do ficheiro e cópia temporária imutável.
- Validação de checksum, sobreposição e limites de S-record, bloqueio da imagem DFU no modo OpenBLT, argumentos sem interpolação pelo shell, timeout e limites de saída do processo.
- CI Windows aprovada na revisão 67ddd5a: compilou e empacotou Java 21, repetiu os testes de manutenção e abriu os controlos JavaFX do pacote, enumerando COM e executando a pesquisa DFU do Windows. O runner não tem uma ECU ligada.
- Kit final aprovado: firmware, manifesto de gravação e Console do mesmo commit; hashes validados. Verificação local adicional do empacotador: ausência de ferramentas opcionais, inclusão de distribuição e rejeição de ferramenta alterada.

**Não foi ligada ou gravada uma ECU física.** Os testes simulados não validam o controlador USB, a instalação dos drivers numa máquina real, a gravação em flash, o BootCommander com esta ECU ou funcionamento de um motor.
