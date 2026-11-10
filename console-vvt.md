# VVT de CAM 1: alvo fixo e tabela RPM/MAP

A Console tem janelas separadas em **VVT → Continuous cam timing - CAM 1**: Position, Solenoid and PID, Activation, Target source, Base target table e VVT monitor. O firmware controla uma came por um solenoide low-side PWM. VTEC continua uma função discreta separada.

## Alvo RPM/MAP

- Base target table tem 16×16 células em graus de cambota, com RPM em baixo e MAP kPa à esquerda, aumentando de baixo para cima. Usa os mesmos controlos de edição, cores, toolbar, gráfico 3D separado e menu de contexto das restantes tabelas. Os eixos são editáveis offline; o cursor acompanha RPM/MAP e os breakpoints do editor.
- A tabela é uma página nativa da ECU: eixos RPM inteiros 0–20000, MAP 0–500 kPa com duas casas, células assinadas −90° a +90° com duas casas. O controlador faz interpolação bilinear, incluindo valores negativos. Fora dos eixos usa o valor da borda; não extrapola.
- **Target source** mantém o alvo fixo para calibração e permite selecionar explicitamente RPM / MAP base table. Primeiro escreve-se a tabela; depois seleciona-se a fonte. Abrir a janela, ler a ECU ou importar um projeto não seleciona a tabela nem arma saídas.
- Quando a fonte de tabela está selecionada, todas as células têm de estar dentro do curso configurado de CAM 1 e o MAP tem de estar calibrado. A Console verifica a base lida e o firmware repete a validação antes de gravar. Valores fora do curso são rejeitados; não se recortam silenciosamente ao limite.
- Sem MAP válido ou sem tabela válida, o VVT não utiliza o alvo fixo como substituição. O duty é zero; se o controlo estava ativo, a falha fica retida até desarmar/rearmar explicitamente.
- A tabela inicial de zeros é uma referência inativa, não um mapa afinado para um motor. Um projeto antigo sem essa tabela recebe um editor de referência e mantém a fonte fixa. Uma página vazia na ECU significa tabela não configurada.
- O monitor distingue alvo fixo e RPM/MAP, mostra o alvo efetivamente pedido e assinala alvo indisponível. Mantém posição, duty, motivo de inibição, tempo, idade do feedback, pressão e retenção. Respostas inválidas, antigas ou de outra ECU/geração são rejeitadas.

## Controlador e posição

CAM 1 utiliza uma aresta selecionada por ciclo de 720°, calculada pelo descodificador de cambota/CAM. A posição é avanço relativo à referência: uma aresta anterior corresponde a avanço positivo. Não é uma nova referência de TDC da ignição.

Position configura referência, curso permitido e idade máxima do feedback. A toolbar pode copiar uma aresta atual por USB, verificando identidade, geração, geometria e antiguidade. A cópia exige VVT desativado na configuração lida, preserva edições se a ECU/janela mudar e não escreve nem arma saídas. A leitura não prova que a came está no batente mecânico sem comando; isso exige verificação física.

A referência fica associada à roda fónica, dentes em falta, janela/fase de CAM e filtros. Alterar essa configuração exige rever e escrever Position antes de ativar VVT. O curso completo deve permanecer na janela de sincronização existente e na mesma revolução; a Console não altera a janela automaticamente.

Solenoid and PID permite escolher uma saída LS livre pelo contacto J102, frequência 10–300 Hz, sentido, duty mínimo/máximo/estável e ganhos P/I/D. Há limitação da integral e proteção contra acumulação em saturação; D atua sobre a medição. O PID atualiza com uma nova aresta de CAM. Repetir a mesma medição não acumula integral nem recalcula a derivada. O alvo varia com RPM/MAP, mas o cálculo PID continua associado à medição nova.

Activation configura RPM e temperatura com histerese, tempo mínimo de funcionamento e pressão de óleo opcional. Pressão insuficiente ou perda de feedback enquanto ativo, curso excedido e erro persistente de seguimento desligam VVT com retenção. Pausas de processamento não viram saltos de integral.

## Preservação e gravação

O prefixo VTEC da página 6 mantém-se. A extensão VVT versão 1 conserva o alvo fixo; a versão 2 acrescenta a fonte no byte 123. A tabela ocupa a página 7 de 576 bytes, anunciada por uma capacidade própria. Firmware sem essa capacidade continua com o conjunto anterior e não recebe comandos de página 7.

Cada janela escreve os seus campos. Edições independentes da página 6 são preservadas; alterações em campos em edição bloqueiam a gravação até revisão/descarte. VVT e VTEC reservam saídas separadas e não podem ocupar injetores, bomba, ventoinha, relé, idle ou boost.

As **sete páginas** entram na cópia anterior, validação, preparação/releitura, um único Burn e releitura final. A gravação exige motor parado, saídas desarmadas, alimentação válida e base limpa/atual da mesma ECU e geração. Um Burn incerto não é repetido automaticamente.

O projeto .tnproj guarda a tabela e a sua página quando disponível. A importação de projetos anteriores preserva os restantes dados e não escreve na ECU. O comparador identifica as células e os eixos VVT e aplica alterações apenas localmente.

A persistência usa o registo **8**, lendo os registos **2–7**. Configurações anteriores conservam o alvo fixo e recebem a página de tabela vazia. A migração mantém o banco anterior durante gravações interrompidas. Firmware anterior não lê os novos registos; conservar uma cópia compatível antes de um downgrade.

## Limites atuais

O suporte continua limitado a **uma came, uma aresta por 720° e um solenoide low-side**. Estão pendentes cames adicionais, rodas CAM com várias arestas, CAM 2 com leitura de fase, duplo solenoide/H-bridge e afinador Relay/AMIGO. O controlo do motor DBW também continua pendente.

A monitorização visual é cerca de 5 Hz, não o registo de 100 Hz indicado na referência para avaliar respostas rápidas. Não foi demonstrada resposta de 200–300 ms num motor. Frequência, duty, curso, PID e os zeros da tabela inicial não equivalem à calibração de um motor/solenoide.

## Referência e verificação

Consultado o PDF fornecido **EMU_PRO_Software_Guide.pdf**, Appendix D, páginas 113–119. Para esta extensão, páginas **117–118**: mapas interpolados de alvo, mudança uniforme de células para observar degraus e configuração do mapa em funcionamento normal. O PDF não fornece o protocolo/driver da Proteus, uma calibração universal ou os detalhes completos de todas as janelas; a dimensão 16×16 segue o projeto Tunuino.

O editor reutiliza o desenho das tabelas já aprovado e foi inspecionado em dois tamanhos. Não se apresenta como reprodução do afinador automático da página 119.

- **17/17 testes locais da Console aprovados**; os testes afetados voltaram a passar após os ajustes finais. Incluem projetos antigos, persistência da tabela, fonte explícita, monitor, preservação VTEC e escrita/releitura das sete páginas contra o dispatcher C real no computador, com uma cópia e um Burn por gravação. Uma tabela fora do curso é rejeitada antes de criar cópia ou pedir Burn.
- **14 testes nativos locais aprovados com ASan/UBSan**; a deteção de fugas foi desativada apenas localmente devido ao ptrace do ambiente. Incluem interpolação assinada/bordas, seleção e perda do alvo, limites, PID, retenções, migração de configuração e interrupções de persistência.
- A compilação ARM STM32F427 foi aprovada no [workflow do firmware](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42803220449).
- **Validação física na Proteus/solenoide/motor pendente**. Os testes no computador e a compilação não confirmam movimento físico ou afinação.

Os testes nativos do firmware passaram com **GCC e Clang** no [workflow](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42803220069).

Windows: a compilação, os testes e a publicação Beta estão em curso no [workflow 289](https://github.com/Tunuino/Tunuino/actions/runs/42803269434). A atualização da Console não grava o firmware na ECU.

Código: [Console](https://github.com/Tunuino/Tunuino/commit/af7750666de7e3496fd6a49cb8a2452619af1f01) e [firmware](https://github.com/Tunuino/Tunuino_Firmware/commit/872fe6bd91eabee1772e77bc2498b98a995b298e).
