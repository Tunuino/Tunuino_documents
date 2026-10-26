# Tunuino Console — interface ECU de 16×16

Atualizado em **26 de outubro de 2026**. Referência visual: painel escuro com logótipo Tunuino azul, instrumentos, tabelas e representação 3D. Implementação: JavaFX 21 na aplicação Windows, não uma imagem estática.

## Páginas de trabalho

| Página | Função |
|---|---|
| Dashboard | Instrumentos de dados reais, tabela VE/ignição 16×16, destaque de célula por RPM/MAP, tabela secundária de ignição e superfície 3D, comunicação e eventos |
| Tuning | Abertura e edição de mapas `.tnmap`, leitura/validação/gravação de páginas 16×16 da ECU, pré-visualização matemática de VE/ignição e temporização de bancada |
| Fuel Tables | Mapa VE nativo 16×16, heatmap, leitura explícita, edição por célula e visualização 3D; não significa controlo de injetores |
| Ignition Tables | Mapa de avanço nativo 16×16, leitura explícita, edição por célula e visualização 3D; não significa comando das bobinas |
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

## Logótipo no cabeçalho

O logótipo original passou de 105 para **270 px de largura** na janela normal, de 84 para **246 px** na janela compacta e de 68 para **228 px** nas janelas de menor altura. A proporção original é preservada. A alteração está no commit [85dc050](https://github.com/Tunuino/Tunuino/commit/85dc0503a75f4bda28043dea029859b6473500c9). A compilação limpa e os testes JavaFX locais passaram, incluindo os tamanhos compactos; a verificação do pacote Windows decorre no CI.

## Edição a partir do painel

No Dashboard, Fuel Tables e Ignition Tables, lê primeiro a tabela da ECU. Usa **Editar tabela** ou faz duplo clique numa das 256 células: o editor abre com a linha MAP e a coluna RPM escolhidas. A abertura e a edição são locais; **Guardar e verificar na ECU** é a ação explícita que escreve, persiste e compara a leitura completa com o mapa enviado. Após sucesso, as três páginas atualizam-se com o mapa verificado.

A edição a partir do painel exige uma tabela lida na ligação identificada atual e a porta livre. Parar ou retomar uma operação USB preserva a cópia da mesma sessão. Desligar, perder a identificação ou mudar de ECU limpa os mapas apresentados e exige nova leitura. Um editor já aberto conserva a cópia local, mas não pode escrever depois de mudar a sessão ou a identificação. Se uma tentativa de gravação falhar, os mapas do painel são invalidados até nova leitura.

Os seis mostradores ajustam a largura para ocupar uma linha em janelas de desktop com espaço suficiente; em janelas estreitas distribuem-se em duas linhas de três. Os cartões secundários também ajustam a largura. Os controlos das tabelas passam à linha seguinte quando necessário.

## Testes e limites

A alteração [7b53fbd](https://github.com/Tunuino/Tunuino/commit/7b53fbd9cec0668f94cb0776357fcbc1e26c3621) passou os testes locais do núcleo C/Java e os testes reais de interface JavaFX em Linux, com dados sintéticos. Foram verificados: botão de edição, duplo clique na célula 16/16, exclusão dos eixos, seleção da célula no editor, bloqueio durante operações USB, limpeza ao desligar/trocar de ECU e seis mostradores numa linha. Estes testes não ligaram uma ECU física. A compilação e a verificação do pacote Windows com o Java incluído passaram. Também passaram os workflows Firmware (imagem F427 e bibliotecas ARM), Unit tests, Desktop, TunerStudio, Bundle e Publish Release.

Os workflows Windows Console e Desktop geram capturas reais JavaFX, verificam a navegação e a largura reduzida, mantêm o Autotune inativo e testam que os indicadores offline não exibem valores falsos. A disponibilidade de imagens e o sucesso de CI **não equivalem a ensaio na ECU física nem a funcionamento do TunerStudio Windows**.

O firmware distribuído não controla injetores, ignição, bomba ou atuadores. A implementação completa desses sistemas exige hardware validado, limites de corrente e tempo, proteções, calibrações reais e ensaios em bancada.

[Bundle Windows + firmware F427 + INI](https://github.com/Tunuino/Tunuino/releases/tag/nightly-2026-10-26-40301671104): publicado em 26 de outubro de 2026, um único ZIP, commit `7b53fbd`. A publicação verifica a origem comum dos componentes e os checksums; a validação física continua pendente.

[Código da Console](https://github.com/Tunuino/Tunuino/tree/master/java_console) · [Releases](https://github.com/Tunuino/Tunuino/releases) · [Documentação do firmware](firmware-f427-progresso.md).
