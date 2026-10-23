# Proteus v0.7 F427 — ensaio USB com ChibiOS

**[Descarregar o Bundle com Console, firmware e INI](https://github.com/Tunuino/Tunuino/releases/download/nightly-2026-10-23-39927868860/tunuino_bundle_proteus_v07_f427_usb_bench.zip)** · [Release](https://github.com/Tunuino/Tunuino/releases/tag/nightly-2026-10-23-39927868860) · build **f105ed0**.

O firmware **0.0.3** acrescenta identificação de bancada pelo TunerStudio, mantendo o protocolo da Tunuino Console. O ZIP inclui `firmware/tunuino_proteus_v07_f427_usb_bench.ini`, correspondente ao firmware distribuído. [Ligar no TunerStudio](tunerstudio.md).

## Estado confirmado

O alvo é **Proteus v0.7 com STM32F427**. Em 22 de outubro de 2026, o utilizador confirmou «Já funciona» e enviou uma fotografia da Console build **65472d5**, com Proteus/F4, revisão 7, firmware **0.0.2**, UID e identificação recebida da ECU. Isto confirma execução da aplicação, COM USB e identificação naquela unidade e versão. Não foi um teste físico executado pelo agente.

A nova imagem **0.0.3** passou nos testes de software e compilação. **A ligação com a aplicação TunerStudio e o novo firmware ainda precisam de ensaio físico.** A confirmação anterior não valida automaticamente esta atualização.

Este firmware continua a ser de bancada. Não adquire sensores, não produz RPM, não disponibiliza mapas e não controla um motor. As capacidades de telemetria e mapas da Console permanecem a zero. No TunerStudio, os campos disponíveis são identidade, revisão, versão e UID; o perfil é apenas de leitura.

## Configuração da imagem

- Startup, kernel e USB CDC do [Tunuino/ChibiOS](https://github.com/Tunuino/ChibiOS/tree/fd2e59c34878b1ef93ef5358f11fb5cc830027ff), fixado nesse commit.
- Flash em **0x08000000**, substituindo o início da instalação e um eventual bootloader na flash. O bootloader ST de sistema fica na ROM.
- HSE de 8 MHz; SYSCLK de **168 MHz** e USB de **48 MHz**.
- USB OTG FS em PA11/PA12 AF10, sem deteção externa de VBUS em PA9; desconexão de 250 ms antes de iniciar e ligar o controlador.
- Dispositivo esperado: **Proteus v0.7 ChibiOS USB bench**, com COM CDC. Serial USB a partir do UID real do STM32; VID/PID de desenvolvimento CAFE:4001.
- Identidade Tunuino: Proteus, série F4, revisão numérica 7, firmware **0.0.3** e UID de 96 bits.
- Não inicializa injeção, ignição, bomba, ventoinha, ADC ou captura de cambota/cames.

O código próprio só configura os pinos USB. Isto não certifica os níveis elétricos dos drivers durante reset/DFU; o ensaio é em bancada, com atuadores desligados e alimentação correta da ECU.

## Gravar pela Console

1. Extrair o **Bundle completo** para uma pasta nova. Abrir `console/TunuinoConsole.exe` e confirmar **build f105ed0** no título.
2. Guardar a instalação e option bytes existentes, ou dispor da imagem original e do procedimento de recuperação. O utilizador tem ST-Link para diagnóstico/recuperação; não é necessário para comunicação USB normal.
3. Abrir **Firmware e USB**. Selecionar a ferramenta ST instalada, mantendo DLLs e a base de dispositivos junto do executável.
4. Colocar BOOT0 em modo de sistema e reiniciar conforme a placa. Pesquisar DFU. A lista DFU é independente das portas COM.
5. Selecionar `firmware/tunuino-proteus-v07-f427-usb-bench.bin`, mantendo o respetivo `.bin.properties`. A descrição deve indicar **ChibiOS USB bench**.
6. Confirmar placa/dispositivo e gravar por DFU. A Console só dá sucesso após verificação positiva da ferramenta ST.
7. Repor BOOT0 em arranque normal e reiniciar a ECU. Para um ciclo de alimentação completo, retirar também qualquer alimentação pelo USB e voltar a ligar a alimentação normal da bancada e o cabo de dados.
8. Procurar a nova COM CDC. Em **Dados e mapas**, selecionar Proteus e a porta, ligar e identificar. Confirmar UID consistente e versão **0.0.3**. Sensores e mapas continuam indisponíveis.
9. Para usar o TunerStudio, desligar primeiro a ligação COM da Console e seguir o [guia do INI](tunerstudio.md). Selecionar o INI do mesmo Bundle, 115200 baud e **Firmware Default**.

[Detalhes da Console, drivers e ferramentas](console-firmware-usb.md). O Bundle inclui Java; não exige Maven/Java instalados. Ferramentas ST/BootCommander e drivers de terceiros não são automaticamente redistribuídos.

Também é possível gravar HEX/ELF pelo CubeProgrammer. Para BIN, o endereço é **0x08000000**. Não modificar proteções ou option bytes para experimentar esta imagem. A imagem não é OpenBLT, F767 ou uaEFI.

## Diagnóstico

A verificação de flash confirma os bytes escritos, mas não que a aplicação arrancou. A ausência de DFU com BOOT0 normal também não prova que a aplicação está a executar. Não atribuir a causa a um driver apenas pela ausência da COM.

Confirmar o build e a descrição da imagem. Se desaparecer todo o dispositivo USB depois do reset, o ST-Link permite observar PC, CFSR/HFSR e `tunuino_bench_fault`. O endereço está em `firmware/manifest.json`; os valores próprios são 10 (identidade inválida), 11 (serial USB não iniciou), 12 (controlador USB não iniciou) e 20 (sessão/protocolo interrompido). Zero não prova que o USB enumera; asserts ChibiOS também podem parar a execução.

Após timeout de um pedido/transmissão, CRC inválido ou alteração da configuração USB durante uma sessão ativa, a imagem para o controlador e requer reset. Tempo sem pedidos, por si só, não é erro. A aceitação pela fila USB não é confundida com transmissão concluída.

## Verificação e ficheiros

A [pipeline 39927868860](https://github.com/Tunuino/Tunuino/actions/runs/39927868860) passou nos sete jobs: C/Java/KiCad com GCC e Clang, compilação ARM, vetores e descritores CDC ligados, JAR/runtime Windows, kit sincronizado e publicação. O `main` ChibiOS é testado com USB simulado para identificação nativa, descoberta TunerStudio, pedidos simples/framed juntos, conclusão atrasada, timeout, CRC inválido, desconexão e reconfiguração. Não foi executada a aplicação TunerStudio nem uma ECU física nestes ensaios.

O único ZIP anexo contém `console/`, `firmware/`, `kit-manifest.json`, `START-HERE.txt` e `SHA256SUMS.txt`. Em `firmware/` estão ELF/HEX/BIN, propriedades, **INI**, manifesto, mapa/layout/tamanho, licenças e `corresponding-source.tar.gz`. O manifesto inclui o hash do INI e a assinatura verificada no binário. O código-fonte correspondente, incluindo o INI, segue no mesmo Bundle. A imagem ligada inclui ChibiOS/RT GPLv3; os componentes mantêm os seus avisos MIT/Apache-2.0.

[Implementação TunerStudio](https://github.com/Tunuino/Tunuino/blob/f105ed0b7ff90b5dd37cceb49a32883202adcb84/docs/tunerstudio-bench.md) · [Configuração USB](https://github.com/Tunuino/Tunuino/blob/f105ed0b7ff90b5dd37cceb49a32883202adcb84/docs/proteus-chibios-usb.md) · [Política de Bundle único](workflows.md).

## Histórico

A imagem ChibiOS **0.0.2**, de **18 872 bytes**, foi a primeira com USB/identificação confirmados pelo utilizador, no [build 65472d5](https://github.com/Tunuino/Tunuino/releases/tag/nightly-2026-10-22-39842387200). Não implementa os comandos deste INI. As imagens anteriores de cerca de 16 KiB usavam startup próprio e TinyUSB e não são a imagem atual.

A [AN2606 da ST](https://www.st.com/resource/en/application_note/an2606-introduction-to-system-memory-boot-mode-on-stm32-mcus-stmicroelectronics.pdf) documenta o DFU ROM dos STM32F42xxx/43xxx. A referência de hardware é o [esquema MCU Proteus](https://github.com/Tunuino/proteus/blob/383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46/mcu.kicad_sch).
