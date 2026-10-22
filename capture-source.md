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

A observação periódica do contador continua mesmo sem impulsos do motor. Os limites e responsabilidades descritos em [clock32.md](clock32.md) mantêm-se: timer de 32 bits a 1 MHz, limite real entre observações e limite real de idade das capturas. O módulo não lê registos físicos nem deteta voltas completas perdidas apenas pelos bits brutos. A API individual push exige eventos já ordenados; a API sample descrita abaixo ordena os dois canais de uma observação.

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

## Capturas conjuntas do timer

Implementado no commit 94d47c3a70135eb44760b375b4e7e1578dcb781f: `tunuino_capture_source_sample` recebe zero, uma ou duas capturas brutas e uma leitura atual do mesmo timer. Há no máximo uma captura por canal. O driver deve reter todos os registos pendentes antes de ler o contador atual, detetar overcapture/reset/perda temporal e garantir que não deixa uma captura mais antiga para a chamada seguinte.

A função valida os dois timestamps antes de publicar qualquer evento e ordena-os pelo instante estendido de 64 bits, mesmo na passagem do contador por zero. Em empate publica cambota antes de cames. Sem capturas, a mesma API faz a observação periódica necessária com o motor parado.

Perda declarada, captura inválida, canal inválido/duplicado, argumentos de lote inválidos, falha do relógio ou da fila bloqueiam a aquisição. A saída temporal fica a zero em erro. Um ponteiro de saída nulo é erro de argumento não fatal. A recuperação continua a exigir reinicialização com os dois lados e o hardware parados.

A publicação não é atómica como lote: se existir apenas um lugar livre, a primeira captura pode entrar e a segunda causar overflow. O consumidor deve manter as verificações existentes de falha da fila e frescura; isto não cancela respostas já construídas.

A suite C/Java/KiCad completa passou localmente. Os testes novos verificam ordem através do wrap, cames realmente anteriores à cambota, empate entre canais, captura futura sem publicação parcial durante a validação, observação sem impulsos, perda declarada, duplicação de canal, argumentos inválidos, falha temporal e overflow após publicação parcial, incluindo invalidação das RPM. São testes simulados, sem leitura de registos STM32 ou validação física nas ECUs.

Os workflows deste commit também concluíram com sucesso no GitHub: Core checks com GCC/Clang e Java 21, e compilação da consola JavaFX no Windows. Execuções: https://github.com/Tunuino/Tunuino/actions/runs/39772884906 e https://github.com/Tunuino/Tunuino/actions/runs/39772884937.

## Capturas até ao CSV Java

Integração de teste no [commit cd04251](https://github.com/Tunuino/Tunuino/commit/cd042510b4ac15a5767ae5c31f9efb58014f55b7).

O processo C usado pelos testes Java deixou de injetar eventos diretamente no processador. Os cenários de cambota passam agora pela API `tunuino_capture_source_sample`, com capturas brutas de 32 bits, extensão do relógio, fila, descodificação, telemetria, dispatcher e comunicação entre processos, até aos clientes e ao gravador CSV reais da Console.

Os novos cenários verificam, para as identidades simuladas Proteus e uaEFI:

- 45 amostras com RPM calculadas e timestamps contínuos antes e depois da volta do contador de 32 bits, incluindo a falha de dentes;
- perda declarada de capturas depois da primeira amostra válida, propagada até à resposta NOT_READY;
- recusa de capturas posteriores enquanto a falha permanece retida, sem recuperação automática;
- ausência de novas linhas no CSV após a falha e preservação da linha anterior, que continua legível pelo leitor offline;
- ligação de protocolo ainda aberta perante indisponibilidade da aquisição;
- observações periódicas do relógio durante a paragem simulada do motor.

A suite completa `python3 tools/check_core.py` passou localmente, incluindo C, Java, KiCad e os testes entre processos. Os testes específicos existentes da origem passaram com AddressSanitizer e UndefinedBehaviorSanitizer; a deteção de fugas foi desativada porque o LeakSanitizer não consegue inspecionar os processos neste ambiente. Este resultado não é uma verificação de fugas.

Esta alteração integra e alarga o ambiente de teste; não acrescenta um driver físico nem modifica o protocolo ou as funcionalidades de produção. Os sensores e os impulsos continuam simulados. Continuam pendentes a captura dos registos STM32, a deteção física de overcapture, USB real e validação nas placas. O README principal mantém-se sem publicações de progresso.

## Telemetria coerente entre aquisição e USB

Implementação portátil no [commit 6a2233f](https://github.com/Tunuino/Tunuino/commit/6a2233f8ceef138d27d567d8c5225b1e1962b82e): `live_mailbox.h/c`.

A aquisição pode agora publicar uma amostra completa enquanto a tarefa USB lê outra, através de três buffers com transferência atómica de propriedade. Há exatamente um produtor serializado e um consumidor serializado. A inicialização, reinicialização e qualquer alteração da configuração exigem os dois lados parados; o objeto não pode ser copiado nem escrito por DMA. A inicialização recusa plataformas nas quais o índice atómico não é lock-free.

Este mecanismo guarda a publicação mais recente; não é uma fila de datalog e pode substituir publicações ainda não lidas. Não usa alocação dinâmica, locks explícitos ou ciclos de repetição no código de publicação/leitura. Isto não demonstra wait-freedom, aptidão para ISR ou um limite de execução no STM32.

A leitura copia para um snapshot pertencente à tarefa USB. Esse snapshot deve permanecer estável durante o dispatcher e a construção da resposta. Nunca se entrega ao dispatcher um ponteiro para os buffers internos nem para dados escritos concorrentemente pela aquisição.

### Frescura e falhas

- Sem amostra, máscara incompleta/desconhecida, timestamp futuro ou idade maior ou igual ao limite: leitura NOT_READY, com saída a zero.
- O instante de leitura deve ser monotónico e usar o mesmo domínio temporal em milissegundos da aquisição; o limite de idade deve ser positivo.
- Uma publicação válida pode ser relida enquanto estiver fresca; se o produtor parar de atualizar, expira.
- A aquisição deve continuar a publicar snapshots inválidos quando perde sinais. A publicação de uma falha substitui a leitura válida anterior; não cancela respostas já construídas.
- A recuperação dos sinais pode ser publicada, mas isto não limpa falhas retidas na origem de capturas. A reinicialização da origem continua a exigir paragem explícita.

O processo C dos testes integrados passa agora pela mailbox antes de atualizar o snapshot usado pelo protocolo. Mantêm-se os cenários até ao CSV Java de wrap, perda de capturas, paragem do motor e falha/recuperação de sensores para ambas as identidades simuladas.

### Verificação desta etapa

A suite completa `python3 tools/check_core.py` passou localmente. Os testes próprios cobrem cópia independente da entrada, substituição de amostras pendentes, releitura, expiração no limite exato, timestamps futuros e UINT64_MAX, máscaras inválidas, recuperação e reinicialização.

O teste concorrente publicou 200 mil snapshots com campos correlacionados, verificou integridade e leituras sem recuo e forçou mais de cem trocas de propriedade através de pontos de sincronização no teste. AddressSanitizer e UndefinedBehaviorSanitizer passaram; a deteção de fugas ficou desativada. ThreadSanitizer abortou internamente neste ambiente; o mesmo erro foi reproduzido num programa mínimo independente, com duas threads e um contador atómico. Não se declara aprovação por ThreadSanitizer.

Os workflows do commit anterior cd042510b4ac15a5767ae5c31f9efb58014f55b7 concluíram com sucesso: Core checks e Windows console build. Os workflows deste novo commit são lançados pela publicação e devem ser consultados para o resultado remoto.

Esta etapa acrescenta o mecanismo portátil e a sua integração no ambiente simulado. Continuam pendentes as tarefas e interrupções físicas STM32, a origem comum dos relógios, a frequência de publicação e a validação USB nas Proteus/uaEFI. O README principal mantém apenas a apresentação curta e o logótipo.

## Verificação de frescura no pedido completo

Implementado no [commit 864d55d](https://github.com/Tunuino/Tunuino/commit/864d55def5d1232b41cada4dddfc8245742a447c): nova inicialização opcional `tunuino_link_init_mailbox`.

O ciclo de comunicação pode agora ser ligado diretamente à mailbox de aquisição. Antes de executar o dispatcher de cada pedido completo, lê a amostra mais recente para um snapshot interno e verifica a idade com o instante atual da comunicação. Um pacote iniciado com a amostra ainda fresca e concluído no limite de idade recebe NOT_READY para o pedido de telemetria. O limite de idade é positivo e configurável; o relógio da comunicação tem de ser monotónico e pertencer ao mesmo domínio temporal da aquisição.

Nesta modalidade, o campo `context.live` fornecido à inicialização é ignorado. Identidade, mapas e capacidades mantêm o significado anterior. O ciclo torna-se o único consumidor da mailbox; a tarefa USB deve serializar todas as operações do ciclo e não chamar o leitor da mailbox por outro caminho. Providers e mailbox devem permanecer válidos durante a ligação. A inicialização ordinária continua disponível e remove a associação anterior à mailbox, respeitando a paragem do transporte exigida para reinicializar.

A cópia usada para construir a resposta pertence ao ciclo. Uma publicação posterior não modifica os bytes já preparados para transmissão. Se a aquisição parar de publicar, os pedidos seguintes detetam a expiração, sem depender de uma nova publicação para invalidar a cópia anterior.

### Resultados

A suite completa C/Java/KiCad passou localmente. Os testes C verificaram mailbox vazia, expiração no instante exato em que termina um pedido fragmentado, publicação inválida durante uma resposta já construída, recuperação, timestamp futuro, expiração sem novas publicações, identificação e metadados dos mapas ainda disponíveis, argumentos/configuração inválidos e reinicialização na modalidade anterior.

O teste entre processos, para as duas identidades simuladas, mantém as capturas de cambota ativas enquanto suspende intencionalmente a publicação. O primeiro pedido recebe dados válidos; os dois seguintes recebem NOT_READY; a publicação retoma e o quarto pedido recebe uma amostra nova. O gravador CSV preserva as duas amostras válidas e o intervalo de 21 ms entre elas, sem gravar as duas respostas expiradas nem reconectar.

Os testes do ciclo passaram também com AddressSanitizer e UndefinedBehaviorSanitizer, com deteção de fugas desativada. Os workflows do commit anterior 6a2233f8ceef138d27d567d8c5225b1e1962b82e concluíram com sucesso: Core checks e Windows console build. Os resultados remotos desta alteração devem ser consultados nos workflows do novo commit.

Esta é uma integração no núcleo portátil, exercitada com relógio e capturas simulados. Continuam pendentes o escalonamento real das tarefas STM32, os relógios físicos, os drivers USB e a validação nas Proteus/uaEFI. Não foi acrescentado controlo de injeção ou ignição.
