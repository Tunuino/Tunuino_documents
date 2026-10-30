# Tunuino Console — USB, DFU e gravação de firmware

A Console disponibiliza as funções de ligação USB, deteção DFU e gravação de firmware. O aspeto da interface pode variar entre versões.

Para instalação, utilizar o Bundle `Tunuino-F427-Bundle.zip` da versão aplicável, quando disponível, com Console, firmware e INI compatíveis.

A confirmação física anterior do utilizador refere-se ao firmware **0.0.2**, no **build 65472d5**, com USB e identificação Proteus/F4 visíveis na fotografia. A versão 0.0.3 e a ligação pela aplicação TunerStudio ainda precisam de ensaio físico.


A distribuição publicada tem os separadores **Dados e mapas** e **Firmware e USB**, com tamanho inicial adaptado ao ecrã. A revisão visual substitui-os pela navegação lateral descrita acima.

## Implementado

- Atualização periódica das portas COM disponíveis, preservando a seleção quando a porta continua presente.
- Pesquisa independente de dispositivos USB STM32 DFU pelo hardware ID `0483:DF11`, mesmo sem qualquer porta COM.
- Apresentação dos erros de driver do Windows; erro de pesquisa não é apresentado como ausência de dispositivo.
- Seleção da ferramenta ST instalada, ou da pasta `console/STM32_Programmer_CLI/bin/STM32_Programmer_CLI.exe` de uma distribuição ST existente. O caminho fica guardado para utilização offline.
- Seleção do dispositivo DFU por número de série, reenumeração antes de gravar e verificação do grupo STM32F42x/F43x (`0x419` ou `0x0419`, aceitando zeros iniciais). Este ID é partilhado; não prova sozinho que o chip é F427 ou que a revisão da placa é v0.7. A interface pede confirmação explícita da placa antes de gravar.
- Seleção de firmware com manifesto de alvo, modo, endereço, SHA-256 e declaração de substituição do bootloader. Validação dos vetores de arranque e dos limites da imagem.
- Gravação DFU na Console através da ferramenta ST, com verificação obrigatória. Só indica sucesso com saída positiva da ferramenta e confirmação de verificação.
- Registo da saída do programador, limites de tempo, tratamento de falhas, bloqueio de ligação COM e de fecho da janela durante programação.
- Instalação dos drivers ST a partir do `silent_st_drivers2.exe` de uma distribuição ST, usando o fluxo de extração e o script Windows fornecido nesse instalador. O Windows pode pedir autorização de administrador.
- Caminho manual OpenBLT através da ferramenta oficial `BootCommander.exe`, transporte XCP serial a 115200, para imagens `.srec` próprias com manifesto e endereços fora da área reservada ao bootloader.

## Primeiro ensaio com a Proteus v0.7 STM32F427

1. Extrair o Bundle compatível para uma pasta nova. Confirmar que contém a Console, o firmware e o manifesto correspondente antes de iniciar a gravação.
2. Guardar a instalação atual da ECU antes de a substituir. O ensaio continua a ser de bancada, com atuadores desligados.
3. Executar `console/TunuinoConsole.exe` e abrir o separador **Firmware e USB**.
4. A Console procura uma instalação normal do STM32CubeProgrammer. Se não a encontrar, usar **Ferramenta ST…** e selecionar o executável da instalação STM32CubeProgrammer. Manter todas as DLLs e a pasta `Data_Base` da ferramenta.
5. Entrar em DFU através de BOOT0 e reset, conforme a placa. A pesquisa DFU é independente da lista COM. O dispositivo deve aparecer na lista **Dispositivo USB DFU**.
6. Se o Windows indicar erro de driver, usar **Instalar drivers…** e selecionar `drivers/silent_st_drivers2.exe` de uma distribuição ST existente. Também pode instalar os drivers fornecidos com o CubeProgrammer oficial.
7. Usar **Selecionar firmware** e escolher `firmware/tunuino-proteus-v07-f427-usb-bench.bin`. Manter o ficheiro `.bin.properties` ao lado. A Console valida ambos.
8. Encerrar qualquer ligação COM antes de gravar, confirmar o dispositivo USB e usar **Gravar por DFU**. A confirmação mostra que a imagem substitui o bootloader na flash.
9. Depois da verificação positiva, repor BOOT0 em arranque normal e reiniciar a ECU. A aplicação esperada é **Proteus v0.7 ChibiOS USB bench**, com COM CDC. Selecionar a COM, ligar e identificar a ECU. O Bundle atual responde à identidade **0.0.3**; o tamanho da imagem consta de `firmware/size.txt`.

Não há apagamento integral, remoção de proteções, alteração de option bytes ou arranque forçado em endereço base neste fluxo.

## OpenBLT: limites atuais

O backend manual está implementado, mas **não existe ainda uma imagem Tunuino OpenBLT validada para esta placa**. A imagem USB de bancada atual é exclusivamente DFU e começa em `0x08000000`; a Console recusa usá-la no modo OpenBLT.

Para esse modo é necessário ter o bootloader já instalado e ativo, a ferramenta oficial BootCommander com as respetivas DLLs, e firmware `.srec` compilado para os endereços da aplicação desse bootloader. Não converter a imagem DFU atual em `.srec` para tentar contornar esta diferença.

A deteção automática da assinatura OpenBLT, a mudança automática do firmware para DFU/OpenBLT e a instalação de um bootloader Tunuino não estão implementadas nesta entrega. O nome de uma porta COM não prova que está em OpenBLT. A mudança de modo exige um comando próprio no firmware Tunuino e ainda não está implementada.

## Pacote Windows e ferramentas externas

O pacote inclui Java, JavaFX, biblioteca serial, os novos controlos e instruções offline em `tools/PROGRAMMERS.txt`. A compilação padrão **não inclui os executáveis ST, BootCommander ou o instalador de drivers**: usa a instalação local das ferramentas e dos drivers ST.

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

**Os testes automáticos e locais não usam uma ECU física.** O utilizador forneceu posteriormente fotografias de gravação verificada e, no build 65472d5, de ligação USB com identidade recebida da ECU. Os testes simulados não validam o controlador USB, a instalação dos drivers numa máquina real, a gravação em flash, o BootCommander com esta ECU ou funcionamento de um motor.

## Correção do bloqueio antes da gravação — 22 de outubro de 2026

A fotografia enviada pelo utilizador mostra uma ECU real detetada pelo CubeProgrammer v2.19.0, com `Device ID : 0x0419` e grupo `STM32F42xxx/F43xxx`. A Console anterior procurava literalmente `0x419` e rejeitava o zero inicial, embora o valor hexadecimal seja o mesmo. O bloqueio ocorria antes do comando de escrita; aquela tentativa não iniciou a gravação.

A validação DFU aceita zeros iniciais e continua a exigir a linha de identificação completa e código de saída zero. Os testes reproduziram primeiro a falha anterior e passaram após a correção com `0x0419`, `0X000419` e saída CRLF do Windows. Identificadores diferentes, prefixos como `0x04190`, identificadores ausentes/inválidos e falhas da ferramenta continuam a impedir qualquer escrita. Estes ensaios usam ferramentas simuladas.


## USB sem porta COM após reset — configuração VBUS do F427

O utilizador relatou ausência de porta COM depois de reiniciar em modo normal. A fotografia do Gestor de Dispositivos não mostra uma porta serial; mostra um dispositivo USB composto cuja identidade não está estabelecida. Isto não permite concluir, por si só, que o processador não executa código, que a ECU está avariada ou que falta um driver.

Na revisão anterior faltava configurar `USB_OTG_FS->GCCFG` para a operação sem deteção externa de VBUS. O STM32F427 usa `NOVBUSSENS` no bit 21, com `VBUSASEN` e `VBUSBSEN` desativados nos bits 18 e 19. A versão TinyUSB fixada no projeto não faz esta configuração específica da placa; o seu driver tenta forçar B-valid em GOTGCTL, mecanismo usado noutros núcleos. A configuração segue também `USB_DevInit` sem VBUS sensing do [driver oficial ST](https://github.com/STMicroelectronics/stm32f4xx-hal-driver/blob/master/Src/stm32f4xx_ll_usb.c).

A correção aplica esta configuração depois do reset do periférico e antes da inicialização/conexão TinyUSB. Preserva os restantes bits e não configura pinos adicionais. O esquema Proteus consultado declara cristal de 8 MHz e os pinos de dados usados continuam PA11/PA12, AF10.

Verificação: suite completa C/Java/KiCad local, ensaios de registos USB com estado de reset, cada bit inicial, preservação dos bits alheios e repetição da inicialização; posições dos bits comparadas com o cabeçalho CMSIS F427 da revisão fixada. Os testes nos dois compiladores e os builds reais ARM/Windows passaram no GitHub. Estes resultados não provam a enumeração física USB; a omissão é confirmada no código, mas a relação com o sintoma nesta ECU ainda é uma hipótese a testar.

Para ensaiar: extrair o ZIP conjunto desta revisão para uma pasta nova, gravar o `.bin` com o respetivo `.bin.properties`, confirmar a mensagem de verificação positiva, repor BOOT0 em modo normal e cortar e restabelecer toda a alimentação. Verificar se surge uma porta COM. Se não surgir, registar o final da gravação e as propriedades/IDs de hardware do dispositivo USB que aparece ou desaparece ao ligar/desligar a ECU.

## Mesmo erro de identificação voltou a ser reportado

A nova fotografia volta a mostrar `Device ID : 0x0419` com recusa antes da escrita. Foi descarregado e inspecionado o Bundle da revisão 16caace: checksum do ZIP e do JAR corretos, manifesto da Console a apontar para 16caace e classe compilada a conter a correção de zeros iniciais. Isto confirma o conteúdo publicado; não permite identificar o executável aberto no computador do utilizador.

O diagnóstico anterior juntava duas condições diferentes numa única mensagem: saída não zero do CubeProgrammer e ausência do ID esperado. O diagnóstico de gravação passa a registar o build, a ferramenta, o firmware escolhido e o **código de saída ST**. A recusa por saída não zero mostra esse código explicitamente. A recusa por ID inválido, diferente ou contraditório tem mensagem própria. Não foi removido o bloqueio de escrita quando a ferramenta falha.

O parser compara o valor hexadecimal numérico e aceita zeros iniciais, saída CRLF, cores ANSI e preenchimento NUL. IDs diferentes, linhas inválidas e identificações contraditórias continuam a ser recusados. A suite verifica estas condições com ferramentas simuladas. A pipeline executou a mesma suite contra o JAR compilado e confirmou que o seu manifesto identifica o commit publicado.

Extrair a nova pasta e verificar **build 65472d5** no título antes de repetir a tentativa. Se continuar bloqueada, guardar a mensagem completa e o código de saída apresentado. Não se pode concluir, com a fotografia anterior, se a causa restante é uma versão antiga em execução, a saída da ferramenta ou uma diferença ainda não observada no texto devolvido. Naquele momento não havia confirmação de gravação física bem-sucedida; a fotografia posterior descrita abaixo já mostra verificação positiva.


## Startup e USB com o repositório Tunuino/ChibiOS

Antes da migração, uma fotografia mostrava **Download verified successfully** para a imagem anterior de cerca de 16 KiB, mas o utilizador relatava ausência de COM e STM32 bootloader depois do reset. A causa exata dessa falha anterior não foi isolada; o resultado posterior com a imagem ChibiOS está registado abaixo.

A implementação ChibiOS substitui a imagem de bancada padrão por **ChibiOS HAL USB/SerialUSB** do próprio [Tunuino/ChibiOS](https://github.com/Tunuino/ChibiOS/tree/fd2e59c34878b1ef93ef5358f11fb5cc830027ff), fixado no commit indicado. O startup, kernel, relógio e driver CDC vêm dessa base. A sequência de inicialização é: inicializar HAL/kernel e serial USB, desligar o USB por 250 ms, iniciar o controlador e voltar a ligar. Usa PA11/PA12, HSE de 8 MHz, SYSCLK de 168 MHz e USB de 48 MHz, sem deteção externa de VBUS.

A Console continua a gravar pela ferramenta ST e a exigir verificação. A imagem continua a começar em **0x08000000** e substitui um bootloader existente na flash. Depois da escrita, repor BOOT0 e reiniciar a placa.



Extrair tudo para uma pasta nova e confirmar **build 65472d5** no título e **ChibiOS USB bench** na descrição do firmware. Após gravação verificada, repor BOOT0 e reiniciar. A nova COM e a leitura de identidade já foram confirmadas pelo utilizador neste build. Se continuar sem dispositivo USB, o próximo diagnóstico útil é ler por ST-Link o PC, os registos de falha e o símbolo tunuino_bench_fault, cujo endereço consta do manifesto do firmware.

O primeiro ensaio físico de USB/identificação desta imagem foi confirmado pelo utilizador, conforme o registo seguinte. [Detalhes técnicos](https://github.com/Tunuino/Tunuino/blob/65472d5ec942eee82f8b8be3fb4be7c5d37c0d4a/docs/proteus-chibios-usb.md).


## Primeiro ensaio físico USB/identificação confirmado — 22 de outubro de 2026

Em 22 de outubro de 2026, o utilizador confirmou **«Já funciona»** e enviou uma fotografia da Console **build 65472d5** ligada à ECU. A interface apresenta **Proteus · STM32 F4 · revisão 7 · firmware 0.0.2**, um UID e a mensagem **«ECU identificada · firmware sem dados em tempo real»**. Este ensaio confirma execução da aplicação, porta COM/ligação USB e resposta de identificação nesta unidade. Não foi um teste físico executado pelo agente.

A ausência de dados em tempo real é esperada nesta imagem de bancada, que só anuncia identificação. Ainda não foram demonstrados sensores, RPM, mapas, controlo de motor, estabilidade prolongada ou repetibilidade de vários ciclos de alimentação/religação USB.

O Bundle e os manifestos originais mantêm os resultados e hashes registados no momento da compilação. Este resultado físico posterior é documentado aqui; não se altera o ZIP já publicado nem se declara validação integral da ECU.
