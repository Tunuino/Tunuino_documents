# Proteus v0.7 F427 — ensaio USB com ChibiOS

**[Descarregar o Bundle com Console e firmware](https://github.com/Tunuino/Tunuino/releases/download/nightly-2026-10-22-39842387200/tunuino_bundle_proteus_v07_f427_usb_bench.zip)** · [Release](https://github.com/Tunuino/Tunuino/releases/tag/nightly-2026-10-22-39842387200) · build **65472d5**.

A imagem atual usa o próprio [Tunuino/ChibiOS](https://github.com/Tunuino/ChibiOS/tree/fd2e59c34878b1ef93ef5358f11fb5cc830027ff), fixado nesse commit, para startup, relógio, ChibiOS/RT e USB CDC. Segue o ciclo USB da rusEFI, com desconexão de 250 ms antes da ligação. [Implementação e configuração](https://github.com/Tunuino/Tunuino/blob/65472d5ec942eee82f8b8be3fb4be7c5d37c0d4a/docs/proteus-chibios-usb.md).

## Estado confirmado

O alvo indicado pelo utilizador é **Proteus v0.7 com STM32F427**. Em 22 de outubro de 2026, o utilizador confirmou **«Já funciona»** e enviou uma fotografia da Console **build 65472d5** ligada à ECU. A interface apresenta **Proteus · STM32 F4 · revisão 7 · firmware 0.0.2**, um UID e a mensagem **«ECU identificada · firmware sem dados em tempo real»**. Este ensaio confirma execução da aplicação, porta COM/ligação USB e resposta de identificação nesta unidade. Não foi um teste físico executado pelo agente.

A imagem ChibiOS atual tem **18 872 bytes** e firmware de identificação **0.0.2**. Compilação ARM e testes de software passaram; o primeiro ensaio físico de USB e identificação foi agora confirmado pelo utilizador. Não é firmware para funcionamento de um motor.

A ausência de dados em tempo real é esperada nesta imagem de bancada, que só anuncia identificação. Ainda não foram demonstrados sensores, RPM, mapas, controlo de motor, estabilidade prolongada ou repetibilidade de vários ciclos de alimentação/religação USB.

## Configuração da imagem

- Flash em **0x08000000**: substitui o início da instalação e um eventual bootloader na flash. O bootloader ST de sistema está na ROM.
- HSE de 8 MHz; SYSCLK de **168 MHz** e USB de **48 MHz**.
- USB OTG FS em PA11/PA12 AF10, sem deteção externa de VBUS em PA9.
- Dispositivo esperado: **Proteus v0.7 ChibiOS USB bench**, com uma COM CDC. UID serial real do STM32; VID/PID de desenvolvimento CAFE:4001.
- Identidade Tunuino: Proteus, F4, revisão numérica 7, versão 0.0.2 e UID real de 96 bits.
- Capacidades de telemetria/mapas a zero. Não inicializa injeção, ignição, bomba, ventoinha, ADC ou captura de cambota/cames.

O código próprio só configura os pinos USB. Isto não certifica os níveis elétricos dos drivers durante reset/DFU; fazer o ensaio em bancada com atuadores desligados e alimentação correta da ECU.

## Gravar pela Console

1. Extrair o **Bundle completo** para uma pasta nova. Abrir console/TunuinoConsole.exe e confirmar **build 65472d5** no título.
2. Guardar a instalação e option bytes existentes, ou dispor da imagem original e do procedimento de recuperação. O utilizador tem ST-Link para diagnóstico/recuperação; não é necessário para comunicação USB normal.
3. Abrir **Firmware e USB**. Selecionar a ferramenta ST instalada ou o STM32_Programmer_CLI.exe do pacote rusEFI existente, mantendo DLLs e Data_Base.
4. Colocar BOOT0 em modo de sistema e reiniciar conforme a placa. Pesquisar DFU. A lista DFU é independente das portas COM.
5. Selecionar firmware/tunuino-proteus-v07-f427-usb-bench.bin, mantendo o respetivo .bin.properties. A descrição deve indicar **ChibiOS USB bench**; o binário tem **18 872 bytes**.
6. Confirmar placa/dispositivo e gravar por DFU. A Console só dá sucesso após verificação positiva da ferramenta ST.
7. Repor BOOT0 em arranque normal e reiniciar a ECU. Para um ciclo de alimentação completo, retirar também qualquer alimentação pelo USB e voltar a ligar a alimentação normal da bancada e o cabo de dados.
8. Procurar a nova COM CDC. Em **Dados e mapas**, selecionar Proteus e a porta, ligar e identificar. Confirmar UID consistente e versão 0.0.2. Os controlos de sensores/mapas permanecem indisponíveis.

[Detalhes da Console, drivers e ferramentas](console-firmware-usb.md). O Bundle inclui Java; não exige Maven/Java instalados. As ferramentas ST/BootCommander e os drivers de terceiros não são automaticamente redistribuídos.

Também é possível gravar o HEX/ELF pelo CubeProgrammer. Para BIN, o endereço é **0x08000000**. Não modificar proteções ou option bytes para experimentar esta imagem. A imagem não é OpenBLT, F767 ou uaEFI.

## Se continuar sem COM

A verificação de flash confirma os bytes escritos, mas não que a aplicação arrancou. A ausência de DFU com BOOT0 normal também não prova que a aplicação está a executar. Não atribuir a causa a um driver apenas pela ausência da COM.

Confirmar primeiro o build e o tamanho/descrição da imagem acima. Se desaparecer todo o dispositivo USB depois do reset, o ST-Link permite observar PC, CFSR/HFSR e tunuino_bench_fault. O endereço do símbolo está em firmware/manifest.json; os valores próprios são 10 (identidade inválida), 11 (serial USB não iniciou), 12 (controlador USB não iniciou) e 20 (sessão/protocolo interrompido). Um zero não prova que o USB enumera; asserts ChibiOS também podem parar a execução por outro caminho.

Após timeout, CRC inválido ou alteração da configuração USB durante uma sessão ativa, a imagem para o controlador e requer reset. A aceitação de dados pela fila não é confundida com transmissão concluída.

## Verificação e ficheiros

A [pipeline 39842387200](https://github.com/Tunuino/Tunuino/actions/runs/39842387200) passou nos sete jobs: C/Java/KiCad com GCC e Clang, compilação ARM, vetores e descritores CDC ligados, JAR e runtime Windows distribuído. O main ChibiOS é testado com USB simulado para pedidos juntos, conclusão atrasada, timeout, CRC, desconexão e reconfiguração com conclusão antiga. Estes testes automáticos são distintos do ensaio físico de USB/identificação confirmado pelo utilizador acima; não validam controlo de motor.

O único ZIP anexo contém console/, firmware/, kit-manifest.json, START-HERE.txt e SHA256SUMS.txt. A pasta firmware inclui ELF/HEX/BIN, propriedades, manifesto, mapa/layout e tamanho, licenças e corresponding-source.tar.gz. O firmware ligado inclui ChibiOS/RT GPLv3; o código próprio e os componentes adaptados mantêm os seus avisos MIT/Apache-2.0. O código-fonte correspondente segue dentro do mesmo Bundle.

## Histórico

As imagens anteriores de cerca de 16 KiB usavam startup próprio e TinyUSB. A [primeira imagem aprovada](https://github.com/Tunuino/Tunuino/actions/runs/39791962001/artifacts/12430804009) e o [primeiro kit sincronizado](https://github.com/Tunuino/Tunuino/actions/runs/39798031342/artifacts/12435280193) ficam como histórico, sujeitos à retenção dos artefactos. Não são a imagem ChibiOS atual.

A [AN2606 da ST](https://www.st.com/resource/en/application_note/an2606-introduction-to-system-memory-boot-mode-on-stm32-mcus-stmicroelectronics.pdf) documenta o DFU ROM dos STM32F42xxx/43xxx. A referência de hardware consultada é o [esquema MCU Proteus](https://github.com/Tunuino/proteus/blob/383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46/mcu.kicad_sch). A pipeline e a política de um único Bundle estão descritas em [workflows.md](workflows.md).
