# Extensão do contador de microssegundos

Implementação portátil: [commit 0dfc4b7](https://github.com/Tunuino/Tunuino/commit/0dfc4b7241492e30b9752d4f76137d4454ce4db7).

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

## Pendente

Não existe ainda configuração de timer STM32, interrupção de overflow, associação de capturas atrasadas, gestão de prioridades de interrupções ou ensaio físico. O módulo não escolhe timers nem pinos para Proteus ou uaEFI. A validação da frequência real, do serviço periódico e da integração com a aquisição continua pendente.
