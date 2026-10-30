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
| Inputs / Outputs | Sensores/ADC e configuração, perfil sequencial por cilindro, consulta de estado, pulsos de bancada, armação e paragem de saídas físicas |
| Diagnostics | Leitura da validação de configuração, estado da flash, ADC, sincronização da cambota, dente e contador de erros |
| Firmware | Identificação, USB CDC, ferramentas de DFU/programação e verificação do firmware |
| Settings | Identidade da aplicação e informação de compatibilidade |

## Telemetria: distinguir medição de apresentação

A versão **IO 0.2.0 STM32F427 / Proteus v0.7** envia RPM, MAP, lambda e temperatura de refrigeração como canais calibrados, cada um com bit de validade. Os quatro mostradores respetivos atualizam-se exclusivamente durante monitorização USB. Quando a medição falta ou a monitorização pára, o valor regressa a `—`.

O painel inclui locais para TPS, IAT, bateria, pressão de combustível, pressão de óleo, temperatura de óleo, temperatura de combustível, segunda lambda e velocidade. **Estes canais não são inventados:** mostram `—` e indicação de ausência de telemetria enquanto não existir suporte correspondente no firmware. Estados de bomba, relé, ventoinhas e atuadores também não são simulados como `ON` ou `OFF`.

## Tabelas e protocolos

As tabelas VE e ignição têm **16 pontos RPM × 16 pontos MAP** no firmware e na Console. São obtidas por `TUNUINO:M` com snapshot de geração e CRC32. A leitura é explícita e só ocorre com porta USB disponível, sem monitorização ou programação simultânea. As tabelas abrem preenchidas com a base genérica aspirada local, explicitamente identificada como **BASE GENÉRICA · POR AFINAR**. A leitura validada substitui apenas a tabela correspondente pelos dados reais da ECU. O destaque de célula ativa exige uma tabela efetivamente lida da ECU. O dashboard destaca o quadrado de interpolação RPM/MAP a partir das últimas amostras válidas, sem modificar os mapas nem enviar escritas automáticas.

O editor de mapas utiliza um fluxo diferente: uma gravação explícita valida, persiste e confirma o conteúdo por leitura completa. O ficheiro 8×8 só é aceite como **importação histórica para conversão local**; não é um formato de funcionamento da ECU IO 0.2.0. A versão nova usa páginas 576/576 e esquema de flash 2. Deve-se guardar uma cópia dos mapas/configuração antigos antes de atualizar o firmware.

## Logótipo no cabeçalho


## Edição a partir do painel

No Dashboard, Fuel Tables e Ignition Tables, as referências base 16×16 já podem ser abertas offline por **Editar tabela** ou duplo clique. São locais e o editor aberto desta forma não oferece escrita na ECU. Para editar o mapa da centralina, lê primeiro essa tabela; o editor abre com a linha MAP e a coluna RPM escolhidas. A abertura e a edição são locais; **Guardar e verificar na ECU** é a ação explícita que escreve, persiste e compara a leitura completa com o mapa enviado. Após sucesso, as três páginas atualizam-se com o mapa verificado.

A edição a partir do painel exige uma tabela lida na ligação identificada atual e a porta livre. Parar ou retomar uma operação USB preserva a cópia da mesma sessão. Desligar, perder a identificação ou mudar de ECU remove as cópias da sessão anterior e volta a mostrar as referências locais, identificadas como genéricas. Os dados da ECU exigem nova leitura. Um editor já aberto conserva a cópia local, mas não pode escrever depois de mudar a sessão ou a identificação. Se uma tentativa de gravação falhar, os mapas do painel são invalidados até nova leitura.

Os seis mostradores ajustam a largura para ocupar uma linha em janelas de desktop com espaço suficiente; em janelas estreitas distribuem-se em duas linhas de três. Os cartões secundários também ajustam a largura. Os controlos das tabelas passam à linha seguinte quando necessário.

## Comando das saídas

Em Inputs / Outputs, define o número de cilindros e os canais LS/IGN, PMS, janela de fase, ReqFuel, dwell, tempo morto e auxiliares. Não é selecionado nenhum motor automaticamente. Enviar o perfil mantém as saídas desarmadas; armar exige calibração persistida, confirmação explícita e correspondência entre o perfil visível e o enviado. Uma edição local invalida essa correspondência. Existe uma paragem de todas as saídas e um teste de pulso isolado com motor parado. O perfil é volátil e exige consulta do estado pela Console; ainda não permite funcionamento autónomo ou datalog separado em simultâneo com armação.

## Testes e limites


Os workflows Windows Console e Desktop geram capturas reais JavaFX, verificam a navegação e a largura reduzida, mantêm o Autotune inativo e testam que os indicadores offline não exibem valores falsos. A disponibilidade de imagens e o sucesso de CI **não equivalem a ensaio na ECU física nem a funcionamento do TunerStudio Windows**.

O firmware novo inclui comando experimental de injetores, sinais de ignição, bomba, relé e ventoinha, com perfil configurável em Inputs / Outputs e armação explícita. A validação física continua pendente. O estado apresentado indica comandos lógicos do MCU e não mede corrente ou confirmação elétrica da carga. [Configuração, ensaio e limitações](controlo-motor.md).


O novo controlo experimental passou igualmente os testes C/Java, compilação F427, pacote Windows com teste de interface, Desktop, TunerStudio, Bundle e publicação. Estes são testes automatizados, sem ensaio físico das novas saídas.

[Código da Console](https://github.com/Tunuino/Tunuino/tree/master/java_console) · [Releases](https://github.com/Tunuino/Tunuino/releases) · [Documentação do firmware](firmware-f427-progresso.md).
