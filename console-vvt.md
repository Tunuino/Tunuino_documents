# VVT: CAM 1 Parameters, PID e duas tabelas de ângulo

## Organização da Console

A navegação acompanha as fotografias fornecidas em 10/11/2026:

- **VVT → VTEC Control**: controlo discreto do solenoide VTEC, preservado.
- **VVT → CAM 1 → Parameters**.
- **VVT → CAM 1 → PID**.
- **VVT → CAM 1 → CAM1 angle 1 / CAM1 angle 2**.
- **VVT → CAM 2**: estrutura visível, com opções indisponíveis enquanto não existir feedback de fase nativo.
- **VVT → VVT**: monitor separado da configuração. O monitor VTEC também permanece separado.

Os quatro blocos anteriores Position, Solenoid and PID, Activation e Target source foram reunidos em Parameters e PID. A importação migra os campos e converte o antigo tempo em milissegundos para a apresentação em segundos. Mantém os limites implícitos e a ausência de deadband dos controladores anteriores, em vez de lhes aplicar os novos valores de referência.

## Parameters

A ordem principal é a das imagens: **CAM offset, Maximum retard/advance, Control type, Invert output, Output frequency, Min/Max solenoid DC, Override DC, Overridden DC, Override target, Overridden target, Steady pos DC, Higher DC, Min RPM, Min coolant temp, Min oil temp e Start delay**.

As cores e a toolbar Open / Save / Defaults / Help seguem o painel de referência. Os valores de override ficam editáveis apenas com a respetiva opção selecionada. O menu de contexto permite copiar a aresta real de CAM 1 como referência; exige VVT desativado na configuração lida, identidade/geração e medição atuais, e não escreve nem arma saídas.

**Advanced / assignments and protections** conserva a escolha explícita de uma saída LS livre pelo contacto J102, limite inferior do curso, idade do feedback, histereses, pressão de óleo opcional, proteção de seguimento, alvo fixo de calibração e seleção da tabela. Não se atribui uma saída automaticamente.

Control type disponibiliza **None** e **VVTi - single solenoid**. Double Vanos aparece indisponível: o driver de duplo solenoide não foi implementado. Min oil temp está visível mas desativado em −40 °C, com explicação: falta um canal nativo calibrado de temperatura do óleo; esse valor não é uma leitura nem uma proteção funcional.

CAM offset continua a ser a referência medida da aresta sem comando, em graus de cambota no ciclo de 720°. O avanço medido é referência menos posição da aresta; não modifica o TDC da ignição. O limite inferior do curso está em Advanced, permitindo preservar configurações anteriores e definir cursos de atraso. O curso completo deve permanecer na janela de sincronização existente e na mesma revolução.

A referência fica associada à roda fónica, dentes em falta, janela/fase CAM e filtros. Alterar essa geometria exige rever Parameters antes de ativar o controlo. Copiar uma aresta não prova que a came está no batente mecânico; essa condição exige verificação física.

## PID

Janela própria com os oito campos das imagens:

| Campo | Função nativa |
|---|---|
| Proportional gain | Correção proporcional ao erro de ângulo |
| Integral gain | Integração por tempo entre novas arestas |
| Derivative gain | Derivada da medição |
| Integral term limit min/max | Limites assinados do termo integral |
| Output min/max | Limites assinados da correção PID, somada a Steady pos DC |
| Deadband | Anula P/D e mantém a integral dentro da banda de ângulo |

Os novos valores de referência são P=1, I=0, D=0, integral ±5%, correção ±15% e deadband 0,5°. São referências inativas; não uma afinação para um motor. Configurações anteriores conservam o comportamento anterior até edição explícita.

A soma final também respeita Min/Max solenoid DC. A integral tem proteção contra acumulação em saturação. Repetir a mesma aresta não integra novamente nem recalcula a derivada; pausas excessivas não viram saltos de integral.

## CAM1 angle 1 e CAM1 angle 2

Cada tabela tem **16×16 células independentes**, tabela e gráfico 3D na mesma janela, separador ajustável, toolbar, edição por contexto e vistas só tabela / só gráfico / divisão horizontal ou vertical. RPM permanece em baixo e MAP kPa à esquerda, aumentando de baixo para cima, como pedido anteriormente. Tipografia e breakpoints acompanham a largura disponível.

O gráfico usa os dados da tabela, pode rodar e ampliar e tem moldura/eixos identificados. O zero tem uma referência de cor estável nas tabelas CAM; não passa a valor máximo apenas por existir uma célula negativa.

As páginas nativas são 7 para angle 1 e 8 para angle 2: eixos RPM inteiros 0–20000, MAP 0–500 kPa com duas casas e células assinadas −90° a +90° com duas casas. A interpolação é bilinear; fora dos eixos usa a borda, sem extrapolar.

A seleção é explícita em Advanced → Target table: alvo fixo de calibração, CAM1 angle 1 ou CAM1 angle 2. Não há comutação automática inventada. Primeiro grava-se a tabela escolhida; todas as suas células devem estar dentro do curso configurado e o MAP deve estar calibrado. Abrir janelas/projetos não seleciona tabelas nem arma saídas.

## Overrides e proteções

Override DC aplica o duty solicitado dentro dos limites de solenoide. Override target substitui o alvo pedido dentro do curso. A inversão PWM aplica-se durante controlo ativo; quando inibido/desativado, o comando físico continua zero.

Os overrides não dispensam saídas armadas explicitamente, motor em funcionamento, gates de RPM/CLT/tempo/óleo e feedback CAM fresco e dentro do curso. Override DC suspende o temporizador de erro de seguimento, pois o PID deixa de comandar a posição; as restantes proteções permanecem.

Sem alvo válido no modo de tabela, o comando é zero; não se usa silenciosamente o alvo fixo. Perda de alvo/feedback ou óleo enquanto ativo, curso excedido e erro persistente de seguimento retêm o corte até desarmar/rearmar. Override target, quando selecionado, é a fonte explícita do alvo.

## Gravação e compatibilidade

A extensão VVT da página 6 passa à versão 3; o prefixo VTEC mantém-se. Bytes 124–141 acrescentam inversão, overrides, limites PID e deadband. Uma capacidade própria anuncia estas funções e a página 8; firmware anterior mantém o conjunto de páginas que suporta.

A escrita mantém motor parado, saídas desarmadas, alimentação válida, identidade/geração e base limpa atuais, cópia anterior, preparação/releitura das **oito páginas**, um Burn e releitura final. Não há repetição automática de Burn incerto. Saídas ocupadas por VTEC, injetores, bomba, ventoinha, relé, idle ou boost são rejeitadas.

O projeto .tnproj e o comparador conservam os dois editores e páginas. Importar dados apenas altera edições locais. O registo de persistência é **9**, com leitura/migração dos formatos **2–8** e preservação do banco anterior em interrupções. Firmware antigo não lê os registos novos; conservar uma cópia compatível antes de um downgrade.

## Referência e validação

Referências: três fotografias fornecidas nesta correção (navegação, Parameters/tabelas e PID) e o PDF fornecido **EMU_PRO_Software_Guide.pdf**, Appendix D, páginas **113–119**: medição/janela, duty, ativação, PID e mapas interpolados. O guia público **EMU Black V3 Software Guide**, introdução, remete os detalhes das estratégias para o Help da aplicação; não descreve todos os campos das fotografias. Não se atribuem esses detalhes ao PDF.

Foram comparadas as capturas nativas de Parameters, PID e tabela/gráfico, incluindo disposição, cores, toolbar e redimensionamento. Há diferenças explícitas de hardware: CAM 2 de fase, temperatura do óleo e duplo solenoide estão indisponíveis.

- **17 testes locais da Console aprovados**. Incluem migração de projetos, preservação das edições, conflitos, dois mapas independentes, preservação VTEC e gravação/releitura de oito páginas contra o dispatcher C real no computador. Também se mantém a comunicação com os conjuntos anteriores de seis/sete páginas.
- **14 testes nativos locais com ASan/UBSan aprovados**; a deteção de fugas só foi desativada localmente devido ao ptrace do ambiente. Verificados limites P/I, deadband, overrides, inversão/desligamento, perda de feedback, seleção da segunda tabela e migração/interrupção da persistência.
- **GCC e Clang aprovados** no [workflow nativo](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42811200164).
- **Compilação ARM STM32F427 aprovada** no [workflow](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42811200209).

**Windows aprovado:** compilação, 17 testes, arranque automático, instalador e publicação **Beta 0.1.3.290** no [workflow](https://github.com/Tunuino/Tunuino/actions/runs/42811461825). A atualização da Console não grava o firmware na ECU; as novas funções exigem o firmware correspondente.

**Validação física na Proteus/solenoide/motor pendente**. O monitor continua perto de 5 Hz; não é o registo de 100 Hz indicado para avaliar resposta rápida. Não foi demonstrada resposta de 200–300 ms num motor. Cames adicionais, CAM multi-aresta, afinador Relay/AMIGO e controlo do motor DBW continuam pendentes.

Código: [Console](https://github.com/Tunuino/Tunuino/commit/1e8958fe59ec5aec2f714f966ee1e2a8f737328b) e [firmware](https://github.com/Tunuino/Tunuino_Firmware/commit/85e853741ba2382196e5c64a5242f9f3bff1ea1f).
