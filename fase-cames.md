# Fase da árvore de cames na Proteus

O firmware Proteus v0.7/F427 passa a medir o ângulo da aresta selecionada da árvore de cames a partir das capturas de cambota e cames. A Console apresenta a medida em **Estado do motor**, durante a monitorização USB, com a idade da última aresta confirmada.

Atualizar **Console e firmware**. A roda e a janela de cames continuam a ser as do perfil do motor. A leitura exige cambota sincronizada e dois impulsos de cames coerentes por 720°. Sem sinal ou confirmação, não surge uma medida. Firmware anterior continua a comunicar; a Console indica leitura indisponível.

O valor é expresso em graus de **cambota**, de 0 a 719,99°, desde o primeiro dente após a falha, incluindo a paridade configurada. Não representa avanço VVT ou posição relativamente ao PMS. O ângulo fica associado à última aresta; não acompanha a rotação entre impulsos. A resolução apresentada não garante essa precisão no motor.

Esta etapa integra a aquisição necessária para desenvolver VVT. **Ainda não comanda uma válvula VVT.** Não altera mapas, ligações, configuração persistente ou o estado armado das saídas. O modelo suporta uma aresta selecionada por 720°; outros padrões e múltiplas árvores continuam pendentes.

Os testes automáticos usam capturas sintéticas: rodas 36-1/60-2, duas paridades, interpolação entre dentes e na falha, contador além de 32 bits, perda/duplicação, filas pendentes e protocolo C/Java. Não constituem validação física. Falta ensaiar o sinal real na Proteus, a variação com RPM/aceleração e a referência mecânica do motor.

[Formato e limites técnicos](https://github.com/Tunuino/Tunuino/blob/master/docs/cam-phase-protocol.md).
