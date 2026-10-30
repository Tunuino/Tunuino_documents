# Calibração de leituras brutas de sensores


## Implementado

O módulo C `analog.h/c` converte uma leitura bruta de 16 bits por interpolação linear entre 2 e 32 pontos. Os pontos brutos têm de estar em ordem estritamente crescente. Os valores calibrados podem crescer, diminuir ou mudar de direção; isso permite tabelas por troços, sem impor um modelo de sensor.

O chamador fornece a curva e os limites físicos já validados para o sensor e circuito reais. Todos os pontos têm de respeitar esses limites. Não existem curvas predefinidas de MAP, sonda lambda ou temperatura.

A conversão usa inteiros de 64 bits nos cálculos intermédios, devolve um inteiro de 32 bits e arredonda ao inteiro mais próximo, com empates afastados de zero. Rejeita leituras fora dos extremos da curva, sem extrapolar ou limitar silenciosamente ao extremo.

## Ligação à telemetria

`tunuino_analog_update` atualiza um canal do snapshot `tunuino_telemetry_sensors`, incluindo instante da aquisição e validade:

| Canal | Unidade calibrada | Limite de configuração |
| --- | --- | --- |
| MAP | milésimos de kPa | mínimo não negativo |
| Lambda | milionésimos de lambda | mínimo estritamente positivo |
| Temperatura do líquido | milésimos de grau Celsius | inteiro com sinal |

MAP e lambda ficam limitados a INT32_MAX nesta conversão, embora o formato de telemetria use campos sem sinal.

Uma leitura marcada como inválida pelo driver, uma leitura fora da curva ou uma configuração inválida apaga o valor e a validade do canal escolhido. Os outros canais são preservados. Um canal desconhecido é recusado sem alterar o snapshot. As funções distinguem sucesso, amostra inválida, leitura fora da gama e configuração inválida.

A validade fornecida pelo driver deve refletir a aquisição real. O módulo não deteta por si só uma falha elétrica que produza um valor dentro da gama.

A atualização é executada pelo consumidor, serializada com a montagem da telemetria. A curva e os seus pontos permanecem imutáveis durante a chamada. O instante deve ser o da aquisição, no mesmo domínio temporal das capturas de cambota/cames. A verificação de idade continua em `tunuino_telemetry_build`; não se deve renovar o instante apenas porque a mesma amostra voltou a ser lida.

## Verificação

`python3 tools/check_core.py` passou, incluindo os testes existentes C/Java e os novos testes de calibração.

Os novos testes cobrem todos os valores de uma curva sintética por troços, extremos inclusivos, leituras fora da gama, validade fornecida pelo driver, arredondamento positivo/negativo, uma curva decrescente entre INT32_MAX e INT32_MIN, configuração inválida, 32 pontos e atualização/invalidação independente dos canais. A execução com AddressSanitizer e UndefinedBehaviorSanitizer também passou.

## Integração simulada com o datalog


O peer C inclui agora o cenário `sensor-recover`: leituras brutas passam por `tunuino_analog_update`, montagem da telemetria, dispatcher, enquadramento binário, clientes Java e gravador CSV. As RPM provêm do descodificador de capturas de dentes simulados. MAP, lambda e temperatura resultam da interpolação de curvas sintéticas, incluindo uma curva de temperatura decrescente.

O cenário percorre oito pedidos de dados:

| Pedido | Condição | Resultado |
| --- | --- | --- |
| 1 | Todos os sinais válidos | Amostra disponível |
| 2 | MAP bruto acima do extremo da curva | NOT_READY |
| 3 | Driver assinala leitura lambda inválida | NOT_READY |
| 4 | Curva de temperatura com número de pontos inválido | NOT_READY |
| 5 | Novas leituras válidas | Amostra disponível |
| 6 | MAP com idade igual ao prazo máximo | NOT_READY |
| 7 | Timestamp lambda no futuro | NOT_READY |
| 8 | Novas leituras válidas | Amostra disponível |

Os testes Java verificam cada estado nas duas famílias sintéticas, mantêm a sessão aberta e confirmam os valores e timestamps após recuperação. Uma segunda execução usa o gravador real: oito pedidos produzem três linhas, com intervalos de aquisição de 4 ms e 3 ms entre as linhas, preservando a ausência de dados durante as falhas. Os cinco estados indisponíveis não geram zeros substitutos no CSV.

A suite completa `python3 tools/check_core.py` passou. O peer C também passou com AddressSanitizer e UndefinedBehaviorSanitizer, recebendo os pedidos binários e verificando independentemente a sequência de estados e os CRCs em Python.

As curvas, leituras, relógio e identidades são sintéticos. Estes testes não validam ADC, USB físico, temporização real ou compatibilidade elétrica.

## Pendente

Esta parte não implementa ADC, DMA, referência de tensão, filtragem, diagnóstico elétrico, drivers de sonda lambda ou curvas reais. Ainda não foi ensaiada fisicamente em Proteus ou uaEFI. A integração simulada dos resultados calibrados com capturas e clientes Java está testada acima. A aquisição e a comunicação física continuam por fazer.
