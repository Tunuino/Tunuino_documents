# Proteus v0.7 F427 — primeiro ensaio por USB

## Alvo confirmado e estado

O utilizador confirmou **Proteus v0.7 com STM32F427**, tem ST-Link e prefere gravar/comunicar por USB. A F767 fica como variante opcional separada.

Foi criado um alvo **experimental de bancada**, com startup STM32F427, linker, relógio e USB CDC. A primeira [compilação completa](https://github.com/Tunuino/Tunuino/actions/runs/39791962001) passou e produziu uma imagem de **16 416 bytes**, sem símbolos por resolver e com os vetores iniciais verificados. [Implementação](https://github.com/Tunuino/Tunuino/commit/926e63057beb20560df6cb295e83acc23a492b32). Os [testes de callbacks e o empacotamento Windows](https://github.com/Tunuino/Tunuino/commit/29a589fd373888b2b5d88e5a6c928840f3fde962) foram acrescentados no bloco seguinte.

**Ainda não houve ensaio físico.** Compilar e ligar não demonstra que o USB enumera nesta placa nem que todas as funções da ECU trabalham.

## O que esta imagem faz

- Arranca diretamente da flash em `0x08000000`.
- Usa o cristal de 8 MHz indicado em Y1501 no esquema MCU da Proteus consultado; configura SYSCLK e USB a 48 MHz.
- Configura PA11/PA12 como USB OTG FS, abre uma interface CDC e responde ao protocolo de identificação da Console.
- Devolve família Proteus, série F4, UID real de 96 bits e firmware 0.0.1. O campo numérico de revisão é 7, correspondendo a v0.7 neste alvo.
- Anuncia capacidades de telemetria/mapas a zero. Não fornece sensores simulados como se fossem medições físicas.
- Não inicializa injeção, ignição, bomba, ventoinha, aquisições ADC ou captura de cambota/cames.

O código próprio só configura GPIO de USB. Isto não certifica os níveis elétricos dos drivers da placa durante reset/DFU: **fazer o ensaio na bancada com bobinas, injetores e restantes atuadores desligados**.

Esta imagem não é para F767, uaEFI ou funcionamento de um motor.

## Ficheiros

A primeira compilação aprovada está disponível em [USB-BENCH-EXPERIMENTAL](https://github.com/Tunuino/Tunuino/actions/runs/39791962001/artifacts/12430804009). É preciso iniciar sessão no GitHub para descarregar artefactos; têm retenção de 30 dias. A pipeline **Tunuino CI** passou a gerar a imagem de bancada quando os respetivos ficheiros mudam e os testes passam. A execução manual gera também a Console e um kit sincronizado.

O ZIP contém:

- `tunuino-proteus-v07-f427-usb-bench.elf`, `.hex` e `.bin`;
- `manifest.json` com alvo, endereços, hashes SHA-256, compilador e commits das dependências;
- mapa de ligação, tamanho, atributos/layout ELF e licenças do projeto e das dependências.

Confirmar o hash do ficheiro escolhido contra o manifesto. Este alvo é diferente das bibliotecas ARM com NOT-FLASHABLE no nome: essas bibliotecas não são firmware.

## Antes da gravação

Esta imagem começa em `0x08000000` e **substitui o início da instalação existente, incluindo um eventual bootloader na flash**. Não preserva esse bootloader. O bootloader de sistema da ST fica em ROM e não faz parte da imagem.

Antes de apagar/gravar, guardar uma cópia da flash atual e registar as option bytes, ou dispor da imagem original e do procedimento de recuperação. O ST-Link pode ser usado para esta cópia/recuperação; não é obrigatório para a comunicação USB normal. Se uma proteção impedir a leitura, não remover a proteção para obter a cópia: a remoção pode apagar a flash. Não alterar RDP, BFB2 ou outras option bytes para experimentar este alvo.

A alimentação correta da ECU continua necessária; não se assume que o cabo USB sozinho alimente todos os circuitos da Proteus. Usar um cabo USB de dados.

## Entrar em DFU por USB

1. Instalar [STM32CubeProgrammer para Windows](https://www.st.com/en/development-tools/stm32cubeprog) e o driver DFU fornecido pela ST.
2. Com atuadores desligados, colocar a Proteus em arranque de sistema através do BOOT0 da placa e efetuar reset, conforme os controlos físicos da unidade. A seleção depende também de BOOT1 e BFB2; não alterar option bytes se o dispositivo não aparecer.
3. Ligar o USB. No CubeProgrammer, selecionar **USB**, atualizar a lista e procurar o dispositivo DFU, normalmente USB1.
4. Nesta primeira passagem, confirmar que o dispositivo é detetado e que se consegue ler a identificação. Não apagar nem alterar proteções para conseguir a deteção.

O F427 possui DFU na ROM; não depende de o firmware Tunuino já estar instalado. Os requisitos e limitações estão em [AN2606, secção STM32F42xxx/43xxx](https://www.st.com/resource/en/application_note/an2606-introduction-to-system-memory-boot-mode-on-stm32-mcus-stmicroelectronics.pdf). O esquema usado como referência é [mcu.kicad_sch](https://github.com/Tunuino/proteus/blob/383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46/mcu.kicad_sch); não se atribuiu um número de botão sem confirmar a unidade física.

## Gravar e testar a identificação

Depois de guardar a instalação anterior:

1. No CubeProgrammer ligado por USB DFU, abrir o `.hex` ou `.elf` do alvo acima. Estes formatos transportam os endereços. Se usar o `.bin`, o endereço é **0x08000000**.
2. Programar os setores necessários e verificar a gravação. Não selecionar mass erase nem modificar option bytes para este ensaio.
3. Devolver BOOT0 à posição de arranque normal e fazer reset da placa. Fechar a ligação DFU do CubeProgrammer.
4. A aplicação deve aparecer como **Proteus v0.7 USB bench**, com uma porta COM CDC. Não aplicar o driver DFU à interface CDC da aplicação.
5. Na Tunuino Console para Windows, selecionar Proteus, atualizar portas, escolher a COM, clicar **Ligar USB** e depois **Identificar ECU**.
6. Confirmar família Proteus, série F4 e UID consistente em pedidos repetidos. Os controlos de datalog/mapas devem permanecer indisponíveis nesta imagem.

A [Console portátil para Windows x64](https://github.com/Tunuino/Tunuino/actions/runs/39792846378/artifacts/12431777696) já foi gerada, com runtime Java e dependências incluídos (ZIP de aproximadamente 36,5 MB). A verificação do executável empacotado passou no Windows: abriu uma janela JavaFX, carregou o logótipo e enumerou portas. Não foi ligada uma ECU física. Extrair o ZIP inteiro e executar `TunuinoConsole.exe`, mantendo as pastas app/runtime ao lado do executável. Não é preciso instalar Java ou Maven para este pacote.

Após fechar a sessão serial ou perante timeout/CRC inválido, esta primeira versão exige reset da placa e nova abertura da porta. O bloqueio é intencional para não reutilizar respostas antigas de uma sessão interrompida. A transmissão avança apenas no callback de conclusão do endpoint; cada transferência é um pacote curto, para evitar confundir a confirmação de dados com um ZLP.

## Verificação feita

- Compilação e ligação reais ARM com GCC para STM32F427, incluindo startup e TinyUSB.
- Inspeção de tamanho, vetores, memória e ausência de símbolos indefinidos.
- Suite completa C/Java/KiCad aprovada localmente após acrescentar os testes do ciclo de bancada.
- Testes com callbacks USB simulados: dois pedidos juntos, primeiro pedido já recebido com DTR, aceitação parcial de cinco bytes, confirmação de envio, CRC inválido, timeout e bloqueio após fechar/reabrir DTR.

Os callbacks simulados não exercitam registos STM32 nem os controladores físicos. Não se realizou gravação, enumeração USB, medição elétrica ou ensaio do motor.

TinyUSB, CMSIS STM32F4 e CMSIS Core estão fixados por commit em `dependencies.json`; o pacote inclui as suas licenças. VID/PID 0xCAFE/0x4001 são valores de exemplo/desenvolvimento usados nesta bancada, sem alegação de atribuição USB de produção.

## Depois do primeiro resultado

O primeiro resultado útil é: **DFU detetado → gravação verificada → aplicação CDC enumerada → identidade real recebida**. Só depois se avança para sensores e RPM reais. O sucesso deste ensaio não valida injeção, ignição ou funcionamento integral da ECU.

## Pacotes aprovados para este primeiro ensaio

- Firmware: commit 926e63057beb20560df6cb295e83acc23a492b32, workflow Proteus F427 USB bench aprovado e artefacto 12430804009.
- Console portátil: commit 29a589fd373888b2b5d88e5a6c928840f3fde962, compilação, empacotamento e verificação do runtime aprovados no Windows; artefacto 12431777696.
- Os testes simulados do ciclo de bancada foram executados localmente com o mesmo main.c presente na imagem aprovada. A execução da imagem de bancada do segundo commit foi cancelada depois de 1193 segundos no passo de instalação da toolchain. A imagem aprovada do primeiro commit continua disponível; não se declara que o segundo build produziu uma imagem.

A JavaFX em classpath emitiu um aviso de carregamento em unnamed module durante a verificação do pacote. A abertura da janela, recursos e biblioteca serial concluiu com sucesso; isto não constitui validação integral da interface ou do USB físico.

## Pipeline atual e kit sincronizado

Os quatro workflows independentes foram substituídos por [Tunuino CI](https://github.com/Tunuino/Tunuino/actions/workflows/ci.yml). Testes primeiro, compilação ARM/Windows depois e kit final apenas com componentes da mesma execução e do mesmo commit. Ver [workflows.md](workflows.md) para seleção por ficheiros alterados, cache e resultados medidos. Para reconstruir um kit completo, usar a execução manual da pipeline. Os links de pacotes já aprovados acima permanecem válidos até expirarem.
