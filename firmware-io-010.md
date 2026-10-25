# Tunuino 0.1.0 — ensaio de entradas, sensores e mapas

Alvo: **Proteus v0.7 com STM32F427**. Esta versão acrescenta aquisição ADC,
entradas de cambota/cames, calibração de sensores e mapas persistentes à
ligação USB da Tunuino Console e do TunerStudio. É uma versão de ensaio de
entradas e configuração. Não comanda injetores, bobinas, borboleta, bomba ou
outros atuadores; os mapas guardados ainda não são usados no controlo do motor.

Usar a Console, o firmware e o INI do **mesmo Bundle**. O pacote da revisão
0.1.0 contém estes ficheiros:

Código preparado no [PR de firmware e Console #2](https://github.com/Tunuino/Tunuino/pull/2).
A [pipeline do Bundle 40195908748](https://github.com/Tunuino/Tunuino/actions/runs/40195908748)
verifica e prepara o pacote desta revisão. A publicação como Release depende
da integração no branch principal; não usar um Bundle 0.0.3 para este ensaio.

| Ficheiro | Utilização |
|---|---|
| `console/TunuinoConsole.exe` | Console Windows com Java incluído |
| `firmware/tunuino-proteus-v07-f427-usb-bench.bin` | Imagem F427 em 0x08000000 |
| O respetivo `.bin.properties` | Alvo, endereço e integridade para a gravação |
| `firmware/tunuino_proteus_v07_f427_io.ini` | Definição TunerStudio 0.1.0 |
| `kit-manifest.json` e `SHA256SUMS.txt` | Origem comum e hashes |

O nome do BIN mantém `usb-bench` por compatibilidade com o empacotamento. A
descrição deve indicar **Tunuino 0.1.0 IO bench**. O antigo INI
`tunuino_proteus_v07_f427_usb_bench.ini` pertence à versão 0.0.3 e não serve
para este perfil.

## Gravar e confirmar na Console

1. Extrair o Bundle completo para uma pasta nova. Guardar a instalação atual
   ou dispor da imagem original e do procedimento de recuperação. Seguir o
   [procedimento USB/DFU](proteus-v07-bringup.md) com os atuadores desligados.
2. Em **Firmware e USB**, selecionar a ferramenta ST e o BIN com as suas
   propriedades. Gravar por DFU e aguardar a verificação positiva. O endereço
   é **0x08000000**; a imagem substitui o início da flash e um eventual
   bootloader aí instalado. O bootloader ST de sistema permanece na ROM.
3. Repor BOOT0 em arranque normal, reiniciar e procurar a COM CDC. Selecionar
   Proteus e ligar na Console. Confirmar **F4**, revisão **7**, firmware
   **0.1.0**, UID consistente e capacidades **15**.
4. Usar **Ver dados em tempo real** e depois **Entradas ADC**. As 16 entradas
   apresentam contagens ADC e tensão no pino do MCU. A aquisição analógica
   pode ser observada antes de configurar MAP, lambda e temperatura.

Inicialmente, RPM e sensores calibrados estão desativados e aparecem como
**—**. Isto é esperado. O firmware não inventa curvas de sensores para
preencher os indicadores. Configurar os sensores no TunerStudio e voltar à
Console permite testar os valores calibrados. Os erros de captura e o estado
da configuração aparecem na janela ADC.

## Ligar no TunerStudio

1. Desligar a ligação USB da Console. Apenas uma aplicação pode ocupar a COM
   de cada vez.
2. Criar um projeto e selecionar **Other/Browse** para carregar
   `firmware/tunuino_proteus_v07_f427_io.ini` do mesmo Bundle.
3. Escolher a COM da ECU, **115200 baud** e **Firmware Default**. A assinatura
   esperada é **`Tunuino Proteus v0.7 F427 IO v1`**. Se a assinatura divergir,
   confirmar o BIN e o INI usados.
4. Abrir o menu **Tunuino**. Estão disponíveis configuração de entradas/roda,
   calibrações MAP/lambda, curva de temperatura, entradas ADC e tabelas VE e
   ignição.

O painel usa o tempo da ECU e uma máscara de validade independente para os
quatro canais. Sinais desativados ou inválidos não são medições de zero. A
definição representa-os como indisponíveis; a apresentação exata no
TunerStudio precisa de ser confirmada na aplicação.

## Calibrar antes de ativar

As entradas MAP e lambda podem escolher **AV1..AV11**; a temperatura pode
escolher **AT1..AT4**. Não existe uma atribuição predefinida a sensores.
Usar os circuitos de entrada da placa e os dados medidos nesse conjunto.

- **MAP:** introduzir dois pontos de contagens ADC crescentes e os valores
  correspondentes em kPa. Confirmar os valores com a especificação do sensor
  e uma referência de pressão conhecida.
- **Lambda:** introduzir dois pontos ADC crescentes e os valores lambda da
  saída analógica do controlador wideband. O sinal esperado é a saída do
  controlador, não a ligação direta de uma sonda de oxigénio.
- **Temperatura:** preencher os três pontos da curva com contagens ADC
  estritamente crescentes e temperaturas conhecidas. As temperaturas podem
  diminuir com o aumento da contagem, conforme o circuito e o sensor.

Preencher a curva antes de selecionar a entrada correspondente e usar
**Burn/Gravar**. A ECU valida o conjunto, grava-o e só depois aplica as
alterações. Curvas vazias, eixos repetidos e valores fora dos limites são
recusados. Se o Burn falhar, corrigir os campos; alterações ainda por aplicar
ficam em RAM e são descartadas ao reiniciar sem novo Burn.

**AV1..AV11, 12 V dividido, AT1..AT4** correspondem a 16 canais ADC. Os volts
mostrados são os volts no pino ADC do MCU, calculados com a referência
configurada; os divisores externos não são compensados. Em particular,
**“12 V dividido (MCU)” não é uma leitura de tensão da bateria**. Não aplicar
tensão de bateria diretamente a um pino ADC. A identificação de conectores e
a validação elétrica da unidade continuam a exigir o esquema da placa.

## Ensaiar cambota e cames

O perfil usa **VR1_OUT → PE7** para cambota e **VR2_OUT → PE8** para cames,
com flanco ascendente nas saídas condicionadas. Os GPIO ficam como entradas,
usando os pull-ups externos da placa. Não se deve ligar um sinal VR bruto
diretamente ao MCU.

Em **Inputs and wheel**, configurar o total de posições da roda **incluindo
os dentes em falta**, o número de dentes em falta e o limite de RPM. Por
exemplo, só selecionar 36/1 se o gerador/roda de ensaio for realmente 36-1.
Ativar a aquisição e fazer Burn. A descodificação só publica RPM depois de
confirmar a sincronização. Perda de sinal, sobrecarga da fila ou erro temporal
invalidam a leitura. Um reinício ou reaplicação da configuração ressincroniza
a aquisição.

Os timestamps são recolhidos no atendimento da interrupção, a partir de um
temporizador de 1 MHz. A precisão, a polaridade física e a leitura nos limites
de RPM ainda precisam de ensaio. A entrada de cames não estabelece uma
referência PMS nem habilita controlo de motor nesta versão.

## Mapas e persistência

Os mapas VE e ignição têm **8 × 8** células, com eixos RPM/MAP crescentes e
**duas casas decimais** na ECU. Começam com células a zero e não constituem
uma afinação para motor.

No TunerStudio, editar as tabelas e usar Burn. Na Console, ler o mapa da ECU,
editar e usar **Guardar e verificar na ECU**. A Console envia o mapa completo,
grava-o e volta a lê-lo; só confirma sucesso se os bytes coincidirem. Um
ficheiro offline `.tnmap` pode ter três casas decimais, mas valores que não
caibam exatamente nas duas casas da ECU são recusados, sem arredondamento.
Alterações pendentes já existentes na ECU precisam de ser aplicadas ou
descartadas antes de enviar outro mapa pela Console.

A configuração e os dois mapas são guardados em conjunto num diário com CRC,
dois setores e marcador de conclusão escrito no fim. Os setores F427 **5 e
6**, a partir de **0x08020000** e **0x08040000**, ficam reservados para estes
dados, fora dos 128 KiB destinados à aplicação. Não devem coexistir aí dados
de outro firmware/bootloader. O primeiro ensaio de persistência consiste em
gravar, desligar toda a alimentação, voltar a ligar e ler os mesmos parâmetros
e mapas. Esta validação física ainda está pendente.

Durante gravações de flash, a aquisição pode ficar temporariamente
indisponível. Após erro terminal de transporte, o firmware volta a enumerar
o USB; atualizar portas e reabrir a COM. As definições guardadas sobrevivem
a essa recuperação.

## Datalog e validação

A Console grava CSV com
`time_ms,rpm,map_kpa,lambda,coolant_c,valid_mask`. Colunas indisponíveis ficam
vazias e o gráfico interrompe a linha. A máscara usa RPM=1, MAP=2, lambda=4 e
temperatura=8. Os CSV antigos de cinco colunas continuam a abrir.

Foram executados testes C/Java com um modelo de entradas sintéticas, incluindo
calibração, sensores parciais, escrita e leitura completa de mapas VE e
ignição, CSV, erros de protocolo e recuperação USB simulada. O diário foi
testado com falhas em 153 passos de programação, corrupção e mudança de
setor. O INI é comparado com os offsets, unidades, comandos e CRC do firmware.
A interface JavaFX foi executada e a imagem ARM compilada.

**Estes testes não substituem o ensaio nesta Proteus nem executaram a aplicação
TunerStudio.** O USB/identificação 0.0.2 anteriormente confirmado pelo
utilizador não valida automaticamente ADC, captura, flash ou o perfil 0.1.0.
Autotune, controlo de injeção/ignição e funcionamento num motor continuam por
implementar e validar.

[Implementação técnica](https://github.com/Tunuino/Tunuino/blob/5f59a7ced7e385c31c1c4eb3ff3fa9c53fcc19f5/docs/proteus-io.md)
· [Mapa de entradas e origem](https://github.com/Tunuino/Tunuino/blob/5f59a7ced7e385c31c1c4eb3ff3fa9c53fcc19f5/hardware/review/proteus-v07-inputs.json)
· [Bundle e workflows](workflows.md).
