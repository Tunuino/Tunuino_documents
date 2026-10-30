# Extensão do contador de microssegundos


O módulo C `clock32.h/c` transforma observações cronológicas de um contador livre de 32 bits, a 1 MHz, em timestamps de 64 bits. Mantém um tempo inicial explícito e soma a diferença modular entre observações, incluindo a passagem de UINT32_MAX para zero. O tempo inicial pode ter uma origem diferente do valor bruto.

## Contrato com o driver

- Um único chamador serializado observa o contador. O objeto não é um relógio atómico partilhado entre tarefas ou interrupções.
- O driver garante um contador livre de 32 bits, a exatamente 1 MHz, e um intervalo real entre observações nunca superior ao limite configurado.
- O limite aceite é de 1 a INT32_MAX microssegundos. O limite é inclusivo; observações no mesmo tick são aceites.
- A observação periódica continua quando não existem impulsos do motor. A 1 MHz, uma volta completa demora cerca de 71,6 minutos; o limite configurado não pode exceder cerca de 35,8 minutos.
- Não se misturam leituras atuais do contador com valores antigos dos registos de captura. A associação de capturas atrasadas à época correta ainda exige integração específica no driver.
- Reinício, suspensão ou reconfiguração do contador exige reinicialização com os intervenientes parados e invalidação da aquisição anterior.

O valor bruto, por si só, não permite detetar uma ou mais voltas completas perdidas. Um reset do contador também pode parecer um avanço válido. Estes casos não são resolvidos pelo módulo: o driver deve prevenir ou sinalizar a perda do contrato temporal.

## Falhas e integração

Uma diferença observada superior ao limite ou uma soma que excederia UINT64_MAX retém FAULT até uma reinicialização explícita. As chamadas com erro devolvem timestamp zero; esse zero nunca deve ser usado como uma captura válida.

Argumentos inválidos são recusados. Uma inicialização inválida deixa o objeto sem configuração. Um ponteiro de saída nulo não avança o relógio.

O chamador deve invalidar a aquisição e os dados publicados quando houver FAULT. Este módulo não altera automaticamente a fila, o descodificador ou a telemetria e não publica timestamps de outro objeto.

## Verificação

A suite completa `python3 tools/check_core.py` passou. Os testes específicos verificam:

- igualdade de ticks, extremos do intervalo permitido e uma passagem explícita de UINT32_MAX para zero;
- recuo observado, salto excessivo e retenção da falha;
- chegada a UINT64_MAX e recusa do incremento seguinte;
- 20 000 observações pseudoaleatórias comparadas com uma referência de tempo de 64 bits, atravessando várias voltas;
- envio dos timestamps estendidos pela fila e pelo processador de cambota, com voltas do contador durante a aquisição inicial, junto da falha da roda e após sincronização;
- manutenção das RPM calculadas e expiração por falta de novas capturas.

Os testes também passaram com AddressSanitizer e UndefinedBehaviorSanitizer.

## Capturas atrasadas


`tunuino_clock32_capture` associa o valor bruto de um registo de captura ao tempo de 64 bits da última observação do contador. Não avança nem recua o relógio. Calcula a idade modular e subtrai-a ao tempo observado; assim, uma captura anterior à volta do contador conserva o instante do impulso quando é lida depois da volta.

O chamador primeiro retém a captura e depois observa um contador atual coerente do mesmo timer. Ambas as operações usam o mesmo chamador serializado. Não se deve fornecer o registo antigo a `observe`.

O limite de idade aceite é de 1 a INT32_MAX microssegundos, inclusivo. Capturas cuja diferença indique idade acima do limite, um instante futuro ou tempo anterior à inicialização são recusadas com REJECTED e saída zero. Uma captura no mesmo tick é aceite. Uma recusa não altera nem bloqueia o relógio. Uma falha já retida no relógio continua a devolver FAULT.

A comparação com a origem de inicialização impede timestamps anteriores ao início da época de software, mesmo quando essa origem tem um valor elevado. As chamadas inválidas não alteram o relógio.

O driver continua responsável por comprovar que a captura é recente, que não houve reset/overcapture e que não se perderam voltas completas. Os bits brutos não distinguem uma captura atual de outra com o mesmo valor de uma volta anterior. Este módulo não reordena eventos nem invalida automaticamente a fila; o chamador não deve publicar capturas recusadas.

### Testes acrescentados

- Captura antes da volta do contador e observação depois, com verificação do timestamp exato.
- Extremos de idade, futuro, captura anterior à inicialização, argumentos inválidos e falha retida.
- Resolução junto de UINT64_MAX e 20 000 capturas com idade variável, comparadas com referência de 64 bits.
- Oito cenários de cambota com a volta do contador em diferentes momentos, incluindo leitura atrasada de 0, 100 e 200 microssegundos. As RPM continuam a usar o instante do impulso, sem variar com esse atraso.
- Suite completa `python3 tools/check_core.py` e execução específica com AddressSanitizer e UndefinedBehaviorSanitizer: passaram.

## Pendente

A resolução portátil de capturas atrasadas está implementada e testada acima. Não existe ainda configuração de timer STM32, leitura coerente dos registos físicos, tratamento de overcapture, interrupção de overflow, gestão de prioridades de interrupções ou ensaio físico. O módulo não escolhe timers nem pinos para Proteus ou uaEFI. A validação da frequência real, do serviço periódico e da integração com a aquisição continua pendente.
