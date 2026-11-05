# Graph Log da Tunuino Console

O Graph Log substitui a página de registo que tinha o botão desativado. Abre em Log > Graph Log, Tools > Graph Log ou F6, numa janela independente da área de trabalho.

## Funcionalidades implementadas

- Start logging / Stop logging: recolha das respostas reais de telemetria da ECU, com tempo monotónico de receção no PC.
- Gráfico temporal com seleção de canal, escala vertical automática, linhas ou pontos, zoom, vista completa e navegação para momentos anteriores ou posteriores com a visualização pausada.
- Pause display / Resume display: pausa a visualização, mantendo a recolha quando o registo está ativo. Ao retomar, mostra os dados entretanto recebidos.
- Clique ou arrastamento no gráfico pausado: cursor temporal e consulta dos valores dos canais nessa amostra.
- Save CSV / Open log: guardar os canais e abrir registos Tunuino offline. Campos vazios representam canais indisponíveis; não são convertidos em zero. CSV com cabeçalho fixo `time_s,RPM,TPS,MAP,Lambda,CLT,IAT,Battery`.
- Validação integral do CSV antes de substituir o registo atual: cabeçalho, oito colunas, números finitos, tempos não negativos e estritamente crescentes, máximo de 16 MB e 100000 amostras.
- Confirmação antes de substituir ou limpar um registo não guardado. A desconexão ou falha de comunicação termina a recolha, preservando o registo para guardar. O limite de 100000 amostras termina a recolha sem descartar as amostras existentes.

## Canais e limites atuais

O protocolo runtime atual permite RPM, MAP, Lambda e CLT, respeitando os indicadores de validade de cada resposta. RPM zero só é aceite quando validado pelo estado reportado pela ECU. TPS, IAT e Battery não têm campos calibrados neste protocolo e ficam indisponíveis.

A consulta nominal é de 500 ms, aproximadamente 2 Hz; os tempos do CSV refletem a receção efetiva no PC. Não está implementada aquisição de alta frequência. Intervalos superiores a dois segundos e amostras inválidas interrompem a linha do gráfico, para não apresentar continuidade artificial.

Esta é a primeira parte do Graph Log. Ainda não inclui vários canais simultâneos no mesmo gráfico, doze separadores configuráveis, presets, seleção e recorte de intervalos, condições automáticas, anexação de ficheiros nem outros formatos de log. Fechar a janela conserva o registo na sessão da aplicação; guardar CSV antes de sair da console.

## Validação

- Compilação local da console completa e arranque offscreen durante três segundos: passaram, com Qt 6.4.2 em Linux. Para esta verificação local foi omitida apenas a geração do script de distribuição da cópia temporária do CMake, cuja sintaxe no projeto corresponde ao Qt utilizado no Windows.
- FrameCodecTest: passou.
- LogDataTest: passou; inclui ida e volta CSV, campos indisponíveis, RPM zero, temperatura negativa, CRLF e rejeição atómica de ficheiros malformados.
- GraphLogWidgetTest: passou com dados sintéticos; verifica início bloqueado sem ligação, recolha, pausa com recolha em segundo plano, retoma, desconexão e renderização.
- Os dois novos testes estão incluídos no CTest do workflow Windows existente. A compilação e o instalador Windows dependem desse workflow.
- Sem validação física de recolha na Proteus nesta alteração. Os testes sintéticos não demonstram o funcionamento físico dos sensores ou a precisão da aquisição.

As tabelas e os seus redimensionamentos não foram alterados nesta implementação.
