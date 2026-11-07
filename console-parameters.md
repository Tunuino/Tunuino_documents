# Janelas de Fueling, Ignition e sensores

A navegação principal abre cada bloco numa janela MDI independente, com fecho/reabertura sem perder a edição. Foi removida a janela Calibration & Setup e a sua segunda árvore. A pesquisa percorre os subgrupos. A seleção usa azul; os valores dos parâmetros usam verde.

Os 20 blocos nativos têm duas colunas e quatro comandos: Open parameters, Save parameters, Restore to defaults e Help. Ficheiros .tnparams são específicos do bloco e validados antes de alterar a edição. Abrir ou repor valores não grava na ECU. Write to ECU guarda apenas o bloco da janela, não as edições pendentes noutras janelas.

| Área | Blocos ligados ao protocolo |
|---|---|
| Fueling | General, Fuel cut, Lambda guard, Acceleration, Warmup, Cylinders trim, Short term trim, Target lambda e Flex fuel |
| Injectors | Atribuições e dead time |
| Coils | Atribuições e dwell |
| Compensação de tensão | Curva partilhada de dead time/dwell |
| Ignition | Firing order, IAT, trims por cilindro, Rev limiters, Primary trigger e Phase |

TPS, MAP, CLT, IAT, Lambda e Battery mantêm calibração, captura ADC real e gravação existentes, agora em janelas separadas. As atribuições mostram o contacto J102/J103 e a lista completa fica em Customise. Os mapas e gráficos mantêm janelas próprias.

A escrita confere identidade/geração e base limpa, exige motor parado, alimentação externa e saídas desarmadas, cria cópia das quatro páginas, prepara/rele as alterações, executa uma gravação permanente e verifica todas as páginas. A máscara do bloco preserva os restantes bytes e bits partilhados. Nunca altera o modo autónomo nem envia ARM/RUN.

## Perfil de motor e acabamento das janelas

Firing order tem número de cilindros editável, ordem, slots None e Invert phase. A secção Advanced começa fechada e reúne modo, fuel/dwell/dead time/limite de injeção, geometria de trigger e matriz por cilindro. Criar um perfil usa referências editáveis e deixa todas as saídas por atribuir. Redimensionar preserva as atribuições existentes; novos cilindros exigem escolha explícita de contactos. A atribuição recusa uma saída ocupada por outra função.

Ao reordenar um perfil, conservam-se os ângulos de evento existentes, incluindo espaçamentos não uniformes. Alterar o número de cilindros propõe espaçamento uniforme de 720 graus, indicado na janela; os ângulos TDC continuam editáveis e devem ser conferidos para o motor real.

Há cinco modos nativos: sequencial, semi-sequencial e batch com bobinas individuais, e semi-sequencial/batch com bobinas partilhadas. Bobinas partilhadas exigem número par, pares a 360 graus e trims de ignição iguais no par. Mover uma bobina partilhada atualiza os dois cilindros; Coils apresenta uma função por par. Nenhuma destas edições arma saídas ou altera o modo autónomo.

Restore to defaults em Firing order repõe apenas a ordem de referência e a fase; preserva fuel, dwell, cablagem, modo e geometria. Ficheiros do bloco podem transportar o perfil em edição. Uma mudança externa do perfil bloqueia uma edição antiga até descartar os dados locais; alterações auxiliares independentes podem ser relidas sem perder a edição.

As janelas têm cabeçalho de secção cinzento, toolbar escura, valores/índices verdes e caixas ciano, sem cabeçalhos redundantes Parameter/Value. A altura inicial acompanha o número de linhas; fechar/reabrir preserva o tamanho escolhido. Advanced tem deslocamento próprio. Os redimensionamentos dos mapas continuam fora desta alteração.

## Trigger wizard

Ignition - Trigger wizard abre uma janela MDI independente com três etapas: roda primária, fase e revisão em duas colunas. Inclui presets 36-1, 36-2, 60-2, 24-1 e 12-1, geometria personalizada de dentes em falta, captura e limite máximo de RPM. Com perfil existente permite ângulo do dente zero, janela de cam e Invert phase. Não cria nem altera número de cilindros, ordem, modo ou contactos; liga às janelas Firing order e Scope.

A revisão mostra valores guardados e propostos, UID/geração da leitura e a última captura real do Scope compatível com essa configuração. Abrir um ficheiro de Scope não conta como captura da ECU. Uma captura anterior descreve a geometria guardada, não valida alterações locais. Mudanças de identidade/geração invalidam essa evidência; alterações externas dos campos em edição bloqueiam a escrita até descartar o rascunho. Write to ECU só fica disponível na revisão válida e usa a mesma cópia, releitura e gravação permanente única dos restantes blocos. Cancelar a confirmação não inicia a escrita; falhas não repetem automaticamente a operação.

O hardware nativo usa flancos ascendentes condicionados e rodas de dentes em falta. A janela de cam refere o último dente físico observado, com zero no primeiro dente após a falha. O ângulo do dente zero é uma referência do ciclo nativo e não uma medição automática do PMS. O assistente não altera o modo autónomo nem arma saídas. A sequência segue o guia PDF, páginas 78–88, apenas nos passos suportados: configuração, fase e inspeção do Scope. Bloqueio do avanço para pistola de ponto, deteção automática de PMS, outros padrões e seleção de flancos/tipo elétrico continuam indisponíveis; não é equivalência completa com o assistente da referência.

## Lambda guard

Fueling - Lambda guard abre uma janela de parâmetros independente, com duas colunas, quatro comandos e Write to ECU. A proteção nativa está desativada por defeito. Define limite absoluto de lambda (0,650–2,000), atraso de deteção (20–5000 ms), mínimos de RPM, MAP, temperatura e TPS, e atraso após arranque. As referências iniciais ficam inativas e não representam calibração validada para um motor.

Com as condições cumpridas, lambda acima do limite ou inválida durante o atraso dispara a falha Protection existente: para todas as saídas, mantém a falha e exige rearme explícito. TPS inválido sob as restantes condições também conta como falha. Não é um controlo de correção de combustível. Arranque, RPM/carga/temperatura/TPS abaixo dos mínimos, cortes de combustível, launch, enriquecimento de aceleração e respetivo holdoff reiniciam a deteção. O cálculo tem resolução de amostragem de 20 ms; pausas longas não contam integralmente para o atraso.

Ativar exige perfil de motor e calibrações MAP, Lambda, CLT e TPS explícitas. A edição de sensores recusa remover uma entrada necessária enquanto a proteção está ativa. A escrita preserva mapas, cablagem, restantes flags e modo autónomo. A aplicação nunca arma saídas ao abrir ou gravar.

Esta função exige firmware com capacidade Lambda guard (bit 64, total nativo 127). Firmware anterior continua utilizável para os outros blocos, mas a gravação deste bloco fica bloqueada. A extensão versionada ocupa os bytes 240–255 da página 4; flags anteriores e páginas mantêm o formato. Guardar estes parâmetros, mesmo desativados, cria uma configuração que requer o novo firmware. O firmware anterior recusa essa extensão; uma regressão exige recuperar uma configuração anterior compatível.

O PDF define a organização e o formato das janelas, mas não descreve os parâmetros individuais de Lambda guard. Esta é uma proteção nativa com limite absoluto e paragem retida, sem alegar equivalência completa com os critérios ou opções da referência.

## Limites que continuam por resolver

Não é ainda equivalência completa com todas as funções da referência. Secondary injectors continuam indisponíveis. Trigger wizard cobre agora o subconjunto nativo descrito abaixo. Firing order já permite criar/redimensionar o perfil e escolher 1–12 cilindros. General, atribuições, dwell e fase usam esse perfil. Os restantes campos e estratégias ainda não têm equivalência completa com a referência.

Flex fuel usa entrada analógica; não implementa sensor de frequência. Rev limiters usa corte de combustível; não implementa padrões de corte de ignição. IAT, compensação de tensão e trims têm flags partilhadas entre combustível/ignição, identificadas na ajuda. O alvo lambda nativo é uma curva MAP de três pontos; a grelha AFR Target permanece offline. Não foram adicionados parâmetros sem suporte como se fossem estratégias operacionais.

O guia PDF fornecido, páginas 21–22, define o formato das janelas e comandos, mas remete os parâmetros individuais para a ajuda interna do programa. Faltam referências detalhadas para reproduzir todos os submenus e campos exatamente.

## Verificação

Compilação local completa e doze testes CTest aprovados. ParameterTest executou 20 gravações com dispatcher/validação C reais e flash/saídas simuladas. EngineProfileTest acrescentou sete gravações cobrindo criação, redimensionamento, os cinco modos, mudança de par e inversão de fase. Cada operação verificou uma gravação permanente, cópia anterior e preservação das restantes páginas. Também foram verificados conflitos, ângulos não uniformes, JSON, releitura e defaults sem alterar fuel/dwell.

TriggerWizardTest acrescentou duas gravações com dispatcher C real e flash/saídas simuladas: geometria sem perfil e geometria/fase com perfil. Verificou cópia, uma gravação permanente e preservação dos mapas, auxiliares, contactos e modo autónomo. Também passou validação de limites, dependência do soft limiter, conflitos, confirmação cancelada, falha sem repetição, identidade/geração da captura e callback apenas após captura completa do Scope.

Lambda guard passou testes C locais com ASan/UBSan, incluindo limites de formato, requisitos de perfil/sensores, atraso, cancelamento por condições de carga, lambda/TPS inválidos, wrap do relógio, pausa longa e falha retida através do controlador real. Os testes de Scope, serviço/persistência/USB e dispatcher também passaram. A janela bloqueou gravação para firmware antigo e manteve a proteção desativada ao inicializar os parâmetros. Os testes de UI e gravação usam a simulação de flash/saídas; não há validação física.

A janela Lambda guard, o assistente e o bloco isolado foram renderizados com o tema da aplicação nas vistas normal e Advanced. O encaminhamento e a reabertura MDI de parâmetros, sensores e Trigger wizard passaram na verificação local; a captura completa via QWidget::grab falhou nesse Qt local, pelo que não se declara validação visual completa da janela principal.

A [compilação Windows, os doze testes, arranque, instalador e publicação automática](https://github.com/Tunuino/Tunuino/actions/runs/42373358208) foram aprovados. O [firmware ARM Proteus](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42373355866) compilou e gerou o pacote. Os [testes nativos do firmware com GCC e Clang](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42373355855), incluindo Lambda guard e persistência/USB, passaram. Os ensaios são simulações no computador, sem validação física na Proteus. Esta etapa acrescenta suporte no firmware; não foi gravada automaticamente nenhuma ECU.

## Guias adicionais recebidos

Referências de trabalho: How-to_Configure_Trigger_in_EMU_BLACK.pdf (versão 1.0, 10 páginas) e EMU_PRO_Software_Guide.pdf (versão 261.2.2, 168 páginas). Complementam o guia BLACK V3 anterior e as fotografias aprovadas. A organização visual BLACK V3 pedida mantém prioridade; o Smart Grid do PRO não substitui automaticamente as janelas independentes.

O guia de trigger distingue expressamente V2 e V3: em V3 a ordem está em Ignition / Firing order e a cablagem por cilindro em Ignition / Coils / Ignition Outputs; a configuração dos sensores está em Sensors and inputs / Digital inputs. Invert phase pertence a Firing order. A calibração física usa Ignition Angle Lock e Locked Angle, seguidos de verificação a outro avanço e desativação do bloqueio no fim. Os valores de exemplo não devem ser aplicados automaticamente à Tunuino. First Trigger Tooth e Trigger Angle da referência não são equivalentes diretos ao ângulo absoluto tooth-zero do motor nativo.

A lacuna prioritária de ignição continua a ser o bloqueio real do avanço no firmware e a respetiva janela, com testes que confirmem o comportamento perante correções e trims. Seleção elétrica de Hall/VR, polaridade, filtros e outros padrões depende de suporte real do hardware/firmware; não basta copiar campos. A aquisição Scope não deve ser confundida com a autorização de saídas.

O guia PRO acrescenta referências concretas para Tune Display (páginas 69–70: canais, propriedades, grelha e F7), Scope, Graph Log, APS/DBW Tuner, VE Tuner e Fuel Film (páginas 133–135). Fuel Film é um modelo de película por cilindro, distinto do enriquecimento TPS atual, e não deve ser apresentado como já implementado. O guia também remete grande parte das estratégias individuais para a ajuda interna; não resolve por si só todos os campos de Lambda guard ou secondary injectors.

Esta revisão atualiza referências e lacunas; não altera código, firmware, calibrações ou funcionalidades publicadas.
