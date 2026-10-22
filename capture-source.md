# Origem de capturas e propagação de falhas

Implementação portátil: [commit 9fac0d4](https://github.com/Tunuino/Tunuino/commit/9fac0d458e48fc3f38752087e0d84845657c39d2).

O novo módulo C `capture_source.h/c` reúne a extensão do relógio de 32 bits, a resolução das capturas atrasadas e a publicação na fila do processador de cambota/cames.

## Integração prevista

Um único produtor serializado executa as operações do relógio e da origem. O consumidor usa apenas `source.processor` nas chamadas de processamento, leitura e montagem da telemetria. Os campos do relógio e o estado da origem pertencem ao produtor e não podem ser consultados concorrentemente pelo consumidor.

1. Inicializar a origem e o processador com os intervenientes e o hardware parados.
2. O driver retém o registo de captura e verifica se houve perda de eventos.
3. Observar um contador atual coerente do mesmo timer, através de `tunuino_capture_source_observe`.
4. Publicar os registos retidos com `tunuino_capture_source_push`, por ordem cronológica; em empate, cambota antes de cames.
5. O consumidor processa a fila e monta uma nova telemetria.

A observação periódica do contador continua mesmo sem impulsos do motor. Os limites e responsabilidades descritos em [clock32.md](clock32.md) mantêm-se: timer de 32 bits a 1 MHz, limite real entre observações e limite real de idade das capturas. O módulo não lê registos físicos, não reordena eventos e não deteta voltas completas perdidas apenas pelos bits brutos.

O driver chama `tunuino_capture_source_lost` se detetar overcapture, reset, suspensão ou incumprimento do contrato temporal. Não publica um evento para substituir os que se perderam.

## Propagação de falhas

A fila ganhou `tunuino_capture_abort`, uma operação exclusiva do produtor que retém SOURCE_FAULT e preserva uma falha anterior. Usa a sinalização atómica já existente na fila; não altera diretamente os dados privados do consumidor.

| Situação | Estado da origem | Resultado na fila |
| --- | --- | --- |
| Observação do relógio falha | CLOCK | SOURCE_FAULT |
| Captura recusada pelo relógio | CAPTURE | SOURCE_FAULT |
| Driver declara perda de eventos | LOST | SOURCE_FAULT |
| Fila cheia, ordem inválida ou tipo de evento inválido | QUEUE | Preserva a causa original |

As falhas permanecem retidas. Pedidos seguintes não publicam eventos e não limpam a causa. Na leitura seguinte, o processador deteta a falha da fila e deixa de expor RPM/fase válidas. Uma nova montagem de telemetria devolve NOT_READY mesmo que os sensores analógicos continuem válidos. Eventos ainda pendentes na fila deixam de ser entregues.

Isto não cancela uma resposta já construída nem altera automaticamente uma cópia de telemetria publicada anteriormente. A integração do firmware deve continuar a montar e publicar snapshots atuais, respeitando a separação entre produtor e consumidor.

A recuperação exige reinicialização explícita com ambos os lados e o hardware parados, descarte das capturas antigas pelo driver e nova confirmação de sincronização. Uma inicialização inválida não permite reutilizar as RPM antigas.

## Verificação

A suite completa `python3 tools/check_core.py` passou. Os novos testes cobrem:

- relógio, captura atrasada e fila, incluindo uma volta do contador entre o impulso e a observação;
- confirmação de RPM pelo descodificador real do núcleo;
- perda declarada com eventos pendentes, salto do relógio, captura demasiado antiga ou futura, ordem inválida, tipo inválido e overflow da fila;
- preservação da primeira falha, bloqueio de novos eventos e de entregas pendentes;
- invalidação de RPM e fase no processador e NOT_READY na montagem de telemetria;
- reinicialização explícita, nova sincronização e funcionamento do relógio com o motor parado;
- argumentos inválidos e inicialização inválida após sincronização.

A execução específica com AddressSanitizer e UndefinedBehaviorSanitizer também passou. Estes novos cenários são sequenciais e simulados; não constituem ensaio de interrupções reais. Os testes concorrentes existentes da fila continuam a passar.

## Pendente

Faltam a configuração dos timers STM32, leitura coerente dos registos, deteção física de overcapture, prioridades de interrupções, publicação de snapshots para USB e ensaio nas placas Proteus e uaEFI. Não foram escolhidos novos pinos nem anunciadas capacidades de hardware.
