# Tune Display da Tunuino Console

Abre em **Tools → Tune Display**, com **F7**, ou na navegação **Gauges → Tune Display**. É uma janela independente na área de trabalho, com fecho, minimizar/maximizar, posição e tamanho ajustáveis. Fechar e voltar a abrir mantém os canais escolhidos.

## Apresentação

- Duas barras horizontais: **LAM.**, lambda medida, e **TRGT**, alvo lambda da ECU. Gradiente azul, marcadores amarelos e escala comum de 0,60 a 1,40. Uma leitura fora desta escala conserva o número e limita a posição do marcador à extremidade.
- Nove mostradores numa grelha 3×3, com cabeçalhos cinzentos, fundo preto e números verdes.
- Cada campo começa em **None**. Duplo clique, botão direito ou Enter/Espaço no campo focado abre a escolha: None, RPM, TPS, MAP, Lambda, CLT, IAT, Battery e Target lambda. Canais podem repetir-se.
- As nove escolhas ficam guardadas entre sessões. Uma configuração guardada inválida é rejeitada por inteiro, recuperando None nos nove campos.

## Dados e limites

As leituras usam as respostas reais de runtime já usadas no estado do motor e no Graph Log. Os indicadores de validade são respeitados; os dados de TPS, IAT e bateria usam as calibrações sincronizadas da mesma ECU e geração. Dados indisponíveis aparecem como **---**. Desligar a ECU limpa imediatamente números e marcadores; mais de dois segundos sem uma nova amostra também limpa o painel.

TRGT é calculado exclusivamente pela curva alvo habilitada na página auxiliar lida da ECU, usando a carga MAP válida da resposta atual. A interpolação usa as unidades inteiras do firmware e os extremos da curva. Sem curva habilitada, base sincronizada ou MAP válido, aparece ---. O editor local AFR Target não é apresentado como se fosse o alvo ativo na ECU.

Esta versão mostra dados em tempo real. Ainda não alterna para os valores do cursor de um registo histórico. Não altera mapas, calibrações ou saídas da ECU.

## Validação

- Compilação local completa Qt 6.4.2 em Linux; apenas a geração do script de distribuição foi omitida na cópia temporária de CMake usada no computador.
- Sete testes CTest: FrameCodecTest, LogDataTest, AutoConnectionTest, MapWriteTransactionTest, SensorCalibrationTest, TuneDisplayTest e GraphLogWidgetTest.
- TuneDisplayTest verifica os nove campos, escolha por menu, persistência, rejeição de definições inválidas, alvo e limites da curva, dados offline/inválidos/antigos, fecho/reabertura MDI e desenho em tamanhos distintos.
- O teste visual usa dados sintéticos. Ainda falta validar a apresentação de dados numa Proteus física.
- A compilação e o instalador Windows são verificados pelo workflow Windows existente.
