# Graph Log da Tunuino Console

O Graph Log substitui a página de registo que tinha o botão desativado. Abre em Log > Graph Log, Tools > Graph Log ou F6, numa janela independente da área de trabalho.

## Funcionalidades implementadas

- Start logging / Stop logging: recolha das respostas reais de telemetria da ECU, com tempo monotónico de receção no PC.
- Doze separadores configuráveis, com nomes próprios e entre um e quatro gráficos em cada separador.
- Até dois canais por gráfico, com cores e escalas independentes à esquerda e à direita. Cada eixo permite seleção do canal, ocultação, escala automática ou limites mínimo/máximo manuais.
- Gráficos temporais com linhas ou pontos por separador, zoom, vista completa e navegação para momentos anteriores ou posteriores com a visualização pausada. O cursor e o intervalo temporal são comuns aos gráficos, permitindo comparar as mesmas amostras.
- Barra de ferramentas compacta, com atalhos Q/A para zoom e Shift+S para alternar linhas/pontos no separador atual.
- Nomes, gráficos, canais, estilo e limites dos eixos ficam guardados entre sessões. Configurações guardadas inválidas são rejeitadas integralmente e a console usa a configuração de apresentação predefinida; não altera dados da ECU.
- Pause display / Resume display: pausa a visualização, mantendo a recolha quando o registo está ativo. Ao retomar, mostra os dados entretanto recebidos.
- Clique ou arrastamento no gráfico pausado: cursor temporal e consulta dos valores dos canais nessa amostra.
- Save CSV / Open log: guardar os canais e abrir registos Tunuino offline. Campos vazios representam canais indisponíveis; não são convertidos em zero. CSV com cabeçalho fixo `time_s,RPM,TPS,MAP,Lambda,CLT,IAT,Battery`.
- Validação integral do CSV antes de substituir o registo atual: cabeçalho, oito colunas, números finitos, tempos não negativos e estritamente crescentes, máximo de 16 MB e 100000 amostras.
- Confirmação antes de substituir ou limpar um registo não guardado. A desconexão ou falha de comunicação termina a recolha, preservando o registo para guardar. O limite de 100000 amostras termina a recolha sem descartar as amostras existentes.

## Canais e limites atuais

O protocolo runtime atual permite RPM, MAP, Lambda e CLT, respeitando os indicadores de validade de cada resposta. RPM zero só é aceite quando validado pelo estado reportado pela ECU. TPS é calculado a partir do ADC real da entrada selecionada e da calibração TPS lida da mesma ECU e geração. Só fica disponível com ADC e curso válidos. IAT e Battery continuam indisponíveis nesta interface.

A consulta nominal é de 500 ms, aproximadamente 2 Hz; os tempos do CSV refletem a receção efetiva no PC. Não está implementada aquisição de alta frequência. Intervalos superiores a dois segundos e amostras inválidas interrompem a linha do gráfico, para não apresentar continuidade artificial.

Ainda não inclui gestão de presets, seleção e recorte de intervalos, condições automáticas, anexação de ficheiros nem outros formatos de log. Os separadores partilham um único registo de amostras: trocar canais ou escalas modifica a apresentação, não a recolha. Fechar a janela conserva o registo na sessão da aplicação; guardar CSV antes de sair da console.

## Configurar os gráficos

- Clique direito no separador, ou duplo clique no seu nome: renomear. O menu do separador permite acrescentar gráficos até ao limite de quatro.
- Clique direito num gráfico: configurar os seus dois canais/eixos, acrescentar ou remover gráficos, alternar linhas/pontos e ajustar a vista ao registo completo. Permanece pelo menos um gráfico por separador.
- Duplo clique no gráfico: abrir a configuração dos seus canais. Na barra de ferramentas, Channels config abre a configuração do primeiro gráfico do separador atual.
- Em Channels config, selecionar Left axis / Right axis, o canal ou Hidden e Autoscale. Desmarcar Autoscale para editar Min value e Max value. A console rejeita limites invertidos e o mesmo canal repetido nos dois eixos.
- Pausar a visualização e clicar num gráfico: todos os gráficos e a lista de valores passam a mostrar o mesmo instante. A recolha continua enquanto o registo estiver ativo.

## Validação

- Compilação local da console completa e arranque offscreen durante três segundos: passaram, com Qt 6.4.2 em Linux. Para esta verificação local foi omitida apenas a geração do script de distribuição da cópia temporária do CMake, cuja sintaxe no projeto corresponde ao Qt utilizado no Windows.
- FrameCodecTest: passou.
- LogDataTest: passou; inclui ida e volta CSV, campos indisponíveis, RPM zero, temperatura negativa, CRLF e rejeição atómica de ficheiros malformados.
- GraphLogWidgetTest: passou com dados sintéticos; verifica início bloqueado sem ligação, recolha, pausa com recolha em segundo plano, retoma, desconexão, doze separadores, configuração de dois canais e eixos, renomeação, persistência entre instâncias, rejeição atómica de configurações inválidas, recuperação de definições corrompidas, cursor partilhado, adição/remoção de gráficos sem perder amostras e renderização.
- Revisão visual local dos gráficos com canais e escalas distintos: concluída com dados sintéticos. Não é uma captura de dados de uma ECU física.
- AutoConnectionTest: passou.
- Os quatro testes estão incluídos no CTest do workflow Windows existente. A compilação e o instalador Windows dependem desse workflow.
- Sem validação física de recolha na Proteus nesta alteração. Os testes sintéticos não demonstram o funcionamento físico dos sensores ou a precisão da aquisição.

As tabelas e os seus redimensionamentos não foram alterados nesta implementação.
