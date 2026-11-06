# Gravação de mapas na Tunuino Console

## Função atual na Console Qt

**ECU → Write to ECU**, com atalho **F2**, guarda o mapa ativo na ECU. Estão disponíveis **Fuel Table** e **Ignition Table**, incluindo quando o respetivo gráfico 3D é a janela ativa. MAP, CLT, Lambda e TPS também podem ser guardados com a respetiva página de calibração ativa; consultar [Sensores e calibrações](console-calibracoes-estabilidade.md). AFR Target ainda não tem gravação ligada nesta interface.

A console deve estar ONLINE e ter lido o mapa da mesma ECU e geração. Editar uma tabela não envia dados automaticamente. Ao escolher Write to ECU, a confirmação identifica qual das duas tabelas será guardada.

Os eixos são guardados com as 256 células. RPM aceita valores inteiros de 0 a 20000; MAP aceita 0 a 500 kPa, com até duas casas decimais. Os dois eixos devem aumentar estritamente. Fuel aceita 0 a 300 e Ignition -90 a 90 graus, com até duas casas decimais. A disposição visual MAP de baixo para cima é convertida para a ordem de armazenamento do firmware.

## Sequência de gravação

1. Confirmar a identidade completa da ECU, a geração, a ausência de alterações temporárias e a validação da configuração.
2. Confirmar motor parado, alimentação externa válida e saídas desarmadas. A operação não envia comandos ARM, RUN, STOP nem testes de saídas.
3. Ler as quatro páginas da configuração e confirmar que o mapa base coincide com o mapa anteriormente lido pela console.
4. Preservar automaticamente uma cópia interna das quatro páginas e da identidade em AppLocalDataLocation/ecu-history. Não é pedido um ficheiro ao utilizador. Esta cópia é da configuração e não reintroduz a função de backup de firmware removida do DFU.
5. Preparar apenas a página do mapa escolhido, em blocos de 64 bytes.
6. Reler todas as páginas. O mapa deve coincidir com a edição e as restantes páginas devem permanecer iguais à base.
7. Revalidar configuração, geração, identidade, alimentação, RPM e estado das saídas. Só então enviar uma única ordem de gravação permanente.
8. Reler novamente as quatro páginas e confirmar configuração válida, persistência, geração seguinte e saídas desarmadas.

Durante a operação, a edição e os ficheiros dos dois mapas ficam bloqueados, bem como o fecho normal da aplicação e a instalação de uma atualização. No final, a console sincroniza novamente os dados.

## Falhas e limites

Uma recusa antes de começar a escrita não grava mapas e mantém a ligação quando a identidade já foi confirmada. Uma base ou geração diferente exige nova leitura. Não se substitui silenciosamente uma configuração diferente.

Uma falha depois de preparar blocos pode deixar dados temporários na ECU, sem os tornar permanentes. A console assinala alterações temporárias e bloqueia novas escritas; com o motor parado, reiniciar a ECU e aguardar nova leitura antes de tentar outra vez. Desligar apenas a ignição pode não reiniciar o processador enquanto houver alimentação USB.

Se a resposta à ordem de gravação permanente se perder, o resultado fica incerto: a console orienta para releitura e nunca repete essa ordem automaticamente. Só uma nova ação explícita do utilizador pode iniciar outra transação, depois de recuperar uma base limpa.

A gravação não permite afinação com o motor em funcionamento. Preserva sensores, perfil do motor, política autónoma e restantes mapas. Não ativa funções nem arma saídas.

## Validação

- Compilação local completa da console com Qt 6.4.2 em Linux; a cópia temporária de CMake usada no ensaio omite apenas a geração do script de distribuição da versão Qt do Windows.
- Seis testes locais: FrameCodecTest, LogDataTest, AutoConnectionTest, MapWriteTransactionTest, SensorCalibrationTest e GraphLogWidgetTest.
- MapWriteTransactionTest cobre Fuel e Ignition, células negativas de ignição, valores/eixos inválidos, identidade, base e geração antigas, alterações temporárias prévias, saídas armadas, CRC, falha de cópia anterior, rejeição de blocos, mudança de ECU, arranque/perda de alimentação antes da gravação, diferenças em páginas não editadas, falha de persistência e perda de resposta sem repetir Burn.
- O mesmo teste foi executado contra o dispatcher e os módulos C reais de Tunuino_Firmware, revisão fac3da2214abdea9bbeda3d0be42f4ae87bfb48e, para Fuel e Ignition. A persistência e as saídas desse ensaio são simuladas no computador; não usa o driver físico de flash, USB nem GPIO.
- O harness opcional encontra-se em qt_console/tests/MapWriteFirmwareHarness.c; pode ser compilado com os ficheiros C de firmware/src e o include firmware/include. Definir TUNUINO_FIRMWARE_HARNESS com o caminho do executável ao correr MapWriteTransactionTest.
- O novo teste integra o CTest do workflow Windows existente. Ainda falta validar gravação permanente e releitura numa Proteus física.

Os redimensionamentos das tabelas permanecem em pausa. A documentação de arranque/aquecimento continua em [Arranque e aquecimento](arranque-aquecimento.md); a disponibilidade dos respetivos controlos na Console Qt é distinta do suporte no firmware.
