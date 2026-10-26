# Tunuino Console — interface ECU de 16×16

Atualizado em **26 de outubro de 2026**. Referência visual: painel escuro com logótipo Tunuino azul, instrumentos, tabelas e representação 3D. Implementação: JavaFX 21 na aplicação Windows, não uma imagem estática.

## Páginas de trabalho

| Página | Função |
|---|---|
| Dashboard | Instrumentos de dados reais, tabela VE/ignição 16×16, destaque de célula por RPM/MAP, tabela secundária de ignição e superfície 3D, comunicação e eventos |
| Tuning | Abertura e edição de mapas `.tnmap`, leitura/validação/gravação de páginas 16×16 da ECU, pré-visualização matemática de VE/ignição e temporização de bancada |
| Fuel Tables | Mapa VE nativo 16×16, heatmap, leitura explícita e visualização 3D; não significa controlo de injetores |
| Ignition Tables | Mapa de avanço nativo 16×16, leitura explícita e visualização 3D; não significa comando das bobinas |
| Autotune | Entrada visível mas desativada; o firmware de bancada não oferece afinação em malha fechada |
| Data Log | Monitorização USB com histórico, cursores, reprodução CSV e gravação de amostras reais |
| Inputs / Outputs | Configuração dos sensores, ADC, captura, validade e persistência; as saídas físicas ainda não estão controladas |
| Diagnostics | Leitura da validação de configuração, estado da flash, ADC, sincronização da cambota, dente e contador de erros |
| Firmware | Identificação, USB CDC, ferramentas de DFU/programação e verificação do firmware |
| Settings | Identidade da aplicação e informação de compatibilidade |

## Telemetria: distinguir medição de apresentação

A versão **IO 0.2.0 STM32F427 / Proteus v0.7** envia RPM, MAP, lambda e temperatura de refrigeração como canais calibrados, cada um com bit de validade. Os quatro mostradores respetivos atualizam-se exclusivamente durante monitorização USB. Quando a medição falta ou a monitorização pára, o valor regressa a `—`.

O painel inclui locais para TPS, IAT, bateria, pressão de combustível, pressão de óleo, temperatura de óleo, temperatura de combustível, segunda lambda e velocidade. **Estes canais não são inventados:** mostram `—` e indicação de ausência de telemetria enquanto não existir suporte correspondente no firmware. Estados de bomba, relé, ventoinhas e atuadores também não são simulados como `ON` ou `OFF`.

## Tabelas e protocolos

As tabelas VE e ignição têm **16 pontos RPM × 16 pontos MAP** no firmware e na Console. São obtidas por `TUNUINO:M` com snapshot de geração e CRC32. A leitura é explícita e só ocorre com porta USB disponível, sem monitorização ou programação simultânea. As tabelas têm valores reais apenas depois de uma leitura validada. O dashboard destaca o quadrado de interpolação RPM/MAP a partir das últimas amostras válidas, sem modificar os mapas nem enviar escritas automáticas.

O editor de mapas utiliza um fluxo diferente: uma gravação explícita valida, persiste e confirma o conteúdo por leitura completa. O ficheiro 8×8 só é aceite como **importação histórica para conversão local**; não é um formato de funcionamento da ECU IO 0.2.0. A versão nova usa páginas 576/576 e esquema de flash 2. Deve-se guardar uma cópia dos mapas/configuração antigos antes de atualizar o firmware.

## Testes e limites

Os workflows Windows Console e Desktop geram capturas reais JavaFX, verificam a navegação e a largura reduzida, mantêm o Autotune inativo e testam que os indicadores offline não exibem valores falsos. A disponibilidade de imagens e o sucesso de CI **não equivalem a ensaio na ECU física nem a funcionamento do TunerStudio Windows**.

O firmware distribuído não controla injetores, ignição, bomba ou atuadores. A implementação completa desses sistemas exige hardware validado, limites de corrente e tempo, proteções, calibrações reais e ensaios em bancada.

[Código da Console](https://github.com/Tunuino/Tunuino/tree/master/java_console) · [Releases](https://github.com/Tunuino/Tunuino/releases) · [Documentação do firmware](firmware-f427-progresso.md).
