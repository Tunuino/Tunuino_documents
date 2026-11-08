# Janelas de Fueling, Ignition e sensores

A navegação principal abre cada bloco numa janela MDI independente, com fecho/reabertura sem perder a edição. Foi removida a janela Calibration & Setup e a sua segunda árvore. A pesquisa percorre os subgrupos. A seleção usa azul; as cores dos parâmetros distinguem números, listas, atribuições e caixas.

Os 27 blocos nativos têm duas colunas e quatro comandos: Open parameters, Save parameters, Restore to defaults e Help. Ficheiros .tnparams são específicos do bloco e validados antes de alterar a edição. Abrir ou repor valores não grava na ECU. Write to ECU guarda apenas o bloco da janela, não as edições pendentes noutras janelas.

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

Firing order tem número de cilindros editável, ordem, slots None e Invert phase. A secção Advanced começa fechada, acessível pelo menu de contexto Advanced engine profile, e reúne modo, fuel/dwell/dead time/limite de injeção, geometria de trigger e matriz por cilindro. Criar um perfil usa referências editáveis e deixa todas as saídas por atribuir. Redimensionar preserva as atribuições existentes; novos cilindros exigem escolha explícita de contactos. A atribuição recusa uma saída ocupada por outra função.

Ao reordenar um perfil, conservam-se os ângulos de evento existentes, incluindo espaçamentos não uniformes. Alterar o número de cilindros propõe espaçamento uniforme de 720 graus, indicado na janela; os ângulos TDC continuam editáveis e devem ser conferidos para o motor real.

Há cinco modos nativos: sequencial, semi-sequencial e batch com bobinas individuais, e semi-sequencial/batch com bobinas partilhadas. Bobinas partilhadas exigem número par, pares a 360 graus e trims de ignição iguais no par. Mover uma bobina partilhada atualiza os dois cilindros; Coils apresenta uma função por par. Nenhuma destas edições arma saídas ou altera o modo autónomo.

Restore to defaults em Firing order repõe apenas a ordem de referência e a fase; preserva fuel, dwell, cablagem, modo e geometria. Ficheiros do bloco podem transportar o perfil em edição. Uma mudança externa do perfil bloqueia uma edição antiga até descartar os dados locais; alterações auxiliares independentes podem ser relidas sem perder a edição.

As janelas têm cabeçalho de secção cinzento, toolbar escura e cores por tipo de campo, sem cabeçalhos redundantes Parameter/Value. A altura inicial acompanha o número de linhas; fechar/reabrir preserva o tamanho escolhido. Advanced tem deslocamento próprio. Os redimensionamentos dos mapas continuam fora desta alteração.

## Configuração manual de trigger

O menu de contexto do Trigger wizard abre Manual trigger setup, uma janela MDI independente com três etapas: roda primária, fase e revisão em duas colunas. Inclui presets 36-1, 36-2, 60-2, 24-1 e 12-1, geometria personalizada de dentes em falta, captura e limite máximo de RPM. Com perfil existente permite ângulo do dente zero, janela de cam e Invert phase. Não cria nem altera número de cilindros, ordem, modo ou contactos; liga às janelas Firing order e Scope.

A revisão mostra valores guardados e propostos, UID/geração da leitura e a última captura real do Scope compatível com essa configuração. Abrir um ficheiro de Scope não conta como captura da ECU. Uma captura anterior descreve a geometria guardada, não valida alterações locais. Mudanças de identidade/geração invalidam essa evidência; alterações externas dos campos em edição bloqueiam a escrita até descartar o rascunho. Write to ECU só fica disponível na revisão válida e usa a mesma cópia, releitura e gravação permanente única dos restantes blocos. Cancelar a confirmação não inicia a escrita; falhas não repetem automaticamente a operação.

O hardware nativo usa flancos ascendentes condicionados e rodas de dentes em falta. A janela de cam refere o último dente físico observado, com zero no primeiro dente após a falha. O ângulo do dente zero é uma referência do ciclo nativo e não uma medição automática do PMS. O assistente não altera o modo autónomo nem arma saídas. A sequência segue o guia PDF, páginas 78–88, apenas nos passos suportados: configuração, fase e inspeção do Scope. O bloqueio do avanço para pistola de ponto está agora disponível em Primary trigger. Deteção automática de PMS, outros padrões e seleção de flancos/tipo elétrico continuam indisponíveis; não é equivalência completa com o assistente da referência.

## Lambda guard

Fueling - Lambda guard abre uma janela de parâmetros independente, com duas colunas, quatro comandos e Write to ECU. A proteção nativa está desativada por defeito. Define limite absoluto de lambda (0,650–2,000), atraso de deteção (20–5000 ms), mínimos de RPM, MAP, temperatura e TPS, e atraso após arranque. As referências iniciais ficam inativas e não representam calibração validada para um motor.

Com as condições cumpridas, lambda acima do limite ou inválida durante o atraso dispara a falha Protection existente: para todas as saídas, mantém a falha e exige rearme explícito. TPS inválido sob as restantes condições também conta como falha. Não é um controlo de correção de combustível. Arranque, RPM/carga/temperatura/TPS abaixo dos mínimos, cortes de combustível, launch, enriquecimento de aceleração e respetivo holdoff reiniciam a deteção. O cálculo tem resolução de amostragem de 20 ms; pausas longas não contam integralmente para o atraso.

Ativar exige perfil de motor e calibrações MAP, Lambda, CLT e TPS explícitas. A edição de sensores recusa remover uma entrada necessária enquanto a proteção está ativa. A escrita preserva mapas, cablagem, restantes flags e modo autónomo. A aplicação nunca arma saídas ao abrir ou gravar.

Esta função exige firmware com capacidade Lambda guard (bit 64; firmware desta etapa anuncia também bit 128 para bloqueio de avanço, total nativo 255). Firmware anterior continua utilizável para os outros blocos, mas a gravação deste bloco fica bloqueada. A extensão versionada ocupa os bytes 240–255 da página 4; flags anteriores e páginas mantêm o formato. Guardar estes parâmetros, mesmo desativados, cria uma configuração que requer o novo firmware. O firmware anterior recusa essa extensão; uma regressão exige recuperar uma configuração anterior compatível.

O PDF define a organização e o formato das janelas, mas não descreve os parâmetros individuais de Lambda guard. Esta é uma proteção nativa com limite absoluto e paragem retida, sem alegar equivalência completa com os critérios ou opções da referência.

## Bloqueio de avanço para calibração

Ignition / Triggers / Primary trigger inclui Ignition angle lock e Locked angle, nas duas colunas existentes, com caixa ciano e ângulo verde. A janela mostra lembrete quando o bloqueio está selecionado e tem altura inicial suficiente para os seis campos. A referência visual e de organização continua a ser BLACK V3, o mais próxima possível das fotografias; a capacidade funcional acompanha a Proteus e os 1–12 cilindros do perfil nativo.

O bloqueio é desativado por defeito, com referência de 0 graus. A faixa nativa é -10 a 60 graus, em passos de 0,01 grau. Quando ativo, substitui o avanço final depois do mapa, avanço de arranque, IAT, launch e trims de ignição por cilindro. Não altera correções de combustível, dwell, cablagem, modo ou proteções. A configuração não arma saídas. Após calibrar com pistola de ponto, desmarcar e gravar novamente para voltar ao avanço normal.

A ativação exige perfil válido e captura de trigger ligada. O firmware anuncia capacidade 128; os restantes controlos de Primary trigger mantêm-se utilizáveis em firmware antigo. Edições/importações dos novos campos são recusadas nesses firmwares, também na transação de escrita. Mudança externa dos parâmetros ou do perfil durante a edição bloqueia a gravação; o menu de contexto permite descartar a edição e usar a leitura atual.

Os cinco bytes 27–31 da página 1 passam a conter versão opcional, enable, reservado e ângulo S16 em centésimos de grau. Cinco zeros continuam a ser a configuração antiga desativada. A extensão é criada apenas ao guardar enable ou ângulo diferente de zero; uma vez presente é preservada mesmo desativada. Configurações com a extensão requerem o novo firmware; regressão exige configuração anterior compatível. Os restantes bytes e páginas são preservados.

O fluxo de gravação continua a exigir motor parado, saídas desarmadas e cópia/releitura. Não há ajuste do bloqueio em tempo real durante o arranque: parar, editar/gravar e voltar a verificar fisicamente. Isto é uma diferença funcional em relação ao procedimento de referência. O Scope e os mapas de pré-visualização não medem nem certificam o ponto físico. O wizard indica quando a leitura guardada tem bloqueio ativo e remete a Primary trigger; não altera o bloqueio ao gravar geometria.

Testes C com ASan/UBSan verificaram formato antigo, limites e campos inválidos, impedimento de mudar a configuração armada, todos os 12 slots nos cinco modos e a precedência final do bloqueio. Pulsos reais do controlador sobre saídas simuladas confirmaram 0, 10, -10 e 60 graus a 1000 RPM, além de 10 graus a 300 RPM com avanço de arranque, IAT, launch e trim ativos. A tolerância do ensaio é de 150 microssegundos pela amostragem simulada de 100 microssegundos. Testes de UI/transação cobriram firmware antigo, conflitos, ativação e desativação, uma gravação permanente por operação e preservação das restantes páginas. A janela isolada foi renderizada e revista. Não é validação com osciloscópio ou motor real.

## Limites que continuam por resolver

Não é ainda equivalência completa com todas as funções da referência. Secondary injectors continuam indisponíveis. Trigger wizard cobre agora o subconjunto nativo descrito abaixo. Firing order já permite criar/redimensionar o perfil e escolher 1–12 cilindros. General, atribuições, dwell e fase usam esse perfil. Os restantes campos e estratégias ainda não têm equivalência completa com a referência.

Flex fuel usa entrada analógica; não implementa sensor de frequência. Rev limiters usa corte de combustível; não implementa padrões de corte de ignição. IAT, compensação de tensão e trims têm flags partilhadas entre combustível/ignição, identificadas na ajuda. O alvo lambda nativo é uma curva MAP de três pontos; a grelha AFR Target permanece offline. Não foram adicionados parâmetros sem suporte como se fossem estratégias operacionais.

O guia PDF fornecido, páginas 21–22, define o formato das janelas e comandos, mas remete os parâmetros individuais para a ajuda interna do programa. Faltam referências detalhadas para reproduzir todos os submenus e campos exatamente.

## Verificação

Compilação local completa e doze testes CTest aprovados. ParameterTest executou 21 gravações com dispatcher/validação C reais e flash/saídas simuladas. EngineProfileTest acrescentou sete gravações cobrindo criação, redimensionamento, os cinco modos, mudança de par e inversão de fase. Cada operação verificou uma gravação permanente, cópia anterior e preservação das restantes páginas. Também foram verificados conflitos, ângulos não uniformes, JSON, releitura e defaults sem alterar fuel/dwell.

TriggerWizardTest acrescentou duas gravações com dispatcher C real e flash/saídas simuladas: geometria sem perfil e geometria/fase com perfil. Verificou cópia, uma gravação permanente e preservação dos mapas, auxiliares, contactos e modo autónomo. Também passou validação de limites, dependência do soft limiter, conflitos, confirmação cancelada, falha sem repetição, identidade/geração da captura e callback apenas após captura completa do Scope.

Lambda guard passou testes C locais com ASan/UBSan, incluindo limites de formato, requisitos de perfil/sensores, atraso, cancelamento por condições de carga, lambda/TPS inválidos, wrap do relógio, pausa longa e falha retida através do controlador real. Os testes de Scope, serviço/persistência/USB e dispatcher também passaram. A janela bloqueou gravação para firmware antigo e manteve a proteção desativada ao inicializar os parâmetros. Os testes de UI e gravação usam a simulação de flash/saídas; não há validação física.

A janela Lambda guard, o assistente e o bloco isolado foram renderizados com o tema da aplicação nas vistas normal e Advanced. O encaminhamento e a reabertura MDI de parâmetros, sensores e Trigger wizard passaram na verificação local; a captura completa via QWidget::grab falhou nesse Qt local, pelo que não se declara validação visual completa da janela principal.

A [compilação Windows, os doze testes, arranque, instalador e publicação automática](https://github.com/Tunuino/Tunuino/actions/runs/42376210981) foram aprovados. O [firmware ARM Proteus](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42376174726) compilou e gerou o pacote. Os [testes do firmware com GCC e Clang](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42376174761), incluindo os testes novos de pulsos com bloqueio de avanço, passaram. Os ensaios locais são simulações no computador, sem validação física na Proteus. Esta etapa acrescenta suporte no firmware; não foi gravada automaticamente nenhuma ECU.

## Guias adicionais recebidos

Referências de trabalho: How-to_Configure_Trigger_in_EMU_BLACK.pdf (versão 1.0, 10 páginas) e EMU_PRO_Software_Guide.pdf (versão 261.2.2, 168 páginas). Complementam o guia BLACK V3 anterior e as fotografias aprovadas. A organização visual BLACK V3 pedida mantém prioridade; o Smart Grid do PRO não substitui automaticamente as janelas independentes.

O guia de trigger distingue expressamente V2 e V3: em V3 a ordem está em Ignition / Firing order e a cablagem por cilindro em Ignition / Coils / Ignition Outputs; a configuração dos sensores está em Sensors and inputs / Digital inputs. Invert phase pertence a Firing order. A calibração física usa Ignition Angle Lock e Locked Angle, seguidos de verificação a outro avanço e desativação do bloqueio no fim. Os valores de exemplo não devem ser aplicados automaticamente à Tunuino. First Trigger Tooth e Trigger Angle da referência não são equivalentes diretos ao ângulo absoluto tooth-zero do motor nativo.

O bloqueio real do avanço e a respetiva janela foram implementados, com os testes descritos acima. Seleção elétrica de Hall/VR, polaridade, filtros e outros padrões depende de suporte real do hardware/firmware; não basta copiar campos. A aquisição Scope não deve ser confundida com a autorização de saídas.

O guia PRO acrescenta referências concretas para Tune Display (páginas 69–70: canais, propriedades, grelha e F7), Scope, Graph Log, APS/DBW Tuner, VE Tuner e Fuel Film (páginas 133–135). Fuel Film é um modelo de película por cilindro, distinto do enriquecimento TPS atual, e não deve ser apresentado como já implementado. O guia também remete grande parte das estratégias individuais para a ajuda interna; não resolve por si só todos os campos de Lambda guard ou secondary injectors.

Esta revisão atualiza referências e lacunas; não altera código, firmware, calibrações ou funcionalidades publicadas.

## File / Save Project as e Open Project

File / Save Project as (Ctrl+S) guarda agora o mapa local completo em .tnproj, independentemente da janela ativa. Deixou de chamar a exportação da Fuel Table como fallback. O diálogo apresenta Tunuino project (*.tnproj), começa em Documentos/Tunuino/Maps e recorda a pasta escolhida. A extensão é acrescentada quando necessário. A gravação usa substituição atómica para não destruir o ficheiro anterior se falhar.

O formato tunuino-project-v2 contém alvo/schema, as quatro páginas da configuração de base, as tabelas Fuel, Ignition e AFR Target com eixos/células atuais, todos os editores de parâmetros, sensores, firing order, atribuições/ângulos por cilindro, estado em edição do trigger e comentário. Guarda também trabalho offline e edições incompletas, incluindo perfis ainda sem saídas atribuídas. Sem leitura anterior de uma ECU, as páginas de base são referências inativas; não são apresentadas como uma leitura física. As páginas são a base preservada e os editores representam as alterações locais; o ficheiro não é apresentado como uma imagem pronta para Burn. As edições independentes de várias janelas são preservadas, não fundidas silenciosamente quando se sobrepõem.

File / Open Project (Ctrl+O) verifica formato, alvo, dimensões, campos, CRC32 e SHA-256 antes de alterar qualquer editor. Um ficheiro inválido deixa a edição atual intacta. A abertura repõe o conjunto local, sem enviar comandos de escrita ou armar saídas. A leitura automática posterior da ECU mantém os mapas abertos e as edições importadas, usando a nova leitura real como base das futuras transações explícitas. Dados do ficheiro nunca substituem identidade/geração ou snapshots de confiança do transporte.

A gravação para a ECU continua a ser a operação explícita existente da janela/bloco/tabela. Não foi acrescentada restauração global do ficheiro para a ECU nesta correção. A validação física e os requisitos de configuração completa continuam a ser exigidos nessa operação; guardar trabalho offline incompleto é permitido. Leituras ADC e evidência do Scope não são restauradas como medições reais. VIN e Logbook continuam a ser dados do dispositivo, fora do mapa.

A exportação/importação JSON de uma tabela permanece apenas nos comandos da própria tabela, identificados como Import/Export table. Os comandos de parâmetros .tnparams mantêm o seu âmbito de bloco. O botão de guardar no comentário do projeto passa a abrir o mesmo Save Project as completo. Os antigos JSON de tabela e snapshots v1 não são aceites pelo novo Open Project como projetos v2 completos.

Compilação local e 14 testes CTest aprovados. CompleteMapSmoke usa a janela real e o menu File, com Firing order ativa e um perfil de seis cilindros ainda sem atribuições completas: alterou as três tabelas e um parâmetro, guardou/reabriu o conjunto e confirmou igualdade dos dados, comentário e eixos. Verificou o filtro/sufixo do diálogo real e recusa atómica de um sensor inválido. MapProjectTest verificou checksum, páginas, todos os blocos/sensores, perfis offline, ficheiros inválidos e falha de gravação. Os testes existentes de escrita com dispatcher C e flash simulada também passaram. A [compilação Windows, os 14 testes incluindo CompleteMapSmoke, arranque, instalador e publicação automática](https://github.com/Tunuino/Tunuino/actions/runs/42410841069) foram aprovados. Não há alteração de firmware nem ensaio físico nesta etapa.

Nomenclatura corrigida no menu e diálogos: Open Project… (Ctrl+O), Save Project as… (Ctrl+S) e filtro Tunuino project (*.tnproj). O teste da janela real passou a verificar os textos dos comandos. Verificação local de CompleteMapSmoke e MapProjectTest aprovada. A [compilação Windows, os 14 testes, arranque, instalador e publicação automática](https://github.com/Tunuino/Tunuino/actions/runs/42411595608) passaram. Esta correção altera apresentação, sem alterar o formato ou conteúdo dos projetos.

## Aspeto das janelas conforme as fotografias

A moldura MDI comum usa barra de título azul clara/cinzenta, contorno ciano, ícone de parâmetros e botões de minimizar, maximizar/restaurar e fechar. Aplica-se aos mapas, gráficos, sensores, parâmetros, ferramentas e Tune Display. Arrastar, redimensionar e os comandos de janela continuam a usar o comportamento MDI do Qt.

Os blocos de parâmetros têm toolbar escura com quatro ícones vetoriais próprios: abrir, guardar, repor e ajuda; cabeçalho cinzento; grelha compacta de duas colunas e linhas de 30 px. Números usam ciano, listas rosa, atribuições/ordem verde e bloqueio de avanço amarelo. A edição focada usa fundo claro e texto escuro. A navegação usa ícones de bloco e grupos, fonte Segoe UI e espaçamento compacto. Dwell passa a Coil dwell time.

Firing order mostra número de cilindros, pelo menos oito posições com None nos slots não usados, e Invert phase. Adapta-se até aos 12 cilindros nativos. Advanced engine profile passou para o menu de contexto, sem remover a edição avançada ou os dados. Write to ECU fica no menu ECU/F2 e no contexto da grelha; o botão de rodapé deixou de ocupar o formulário. Indicações de rotina passam para a tooltip; erros, conflitos e avisos de bloqueio continuam visíveis.

Esta alteração corrige apresentação e não completa todas as estratégias da fotografia. Continuam em falta o assistente de dwell, correções dwell por RPM/MAP, custom corrections e três tabelas, segunda tabela de ignição e os limitadores adicionais por temperatura/etanol/custom. O trigger primário mantém os campos nativos: First Trigger Tooth/Trigger Angle não são substituídos pelo ângulo absoluto tooth-zero. A fase nativa não equivale à janela completa de Secondary trigger; CAM 2 não tem aquisição implementada. Não foram acrescentados controlos fictícios para essas funções.

Compilação local completa e 14 testes CTest aprovados. CompleteMapSmoke verifica os cliques reais de maximizar, restaurar, minimizar, fechar e reabrir, mantendo o perfil de seis cilindros. A geometria dos botões também é usada nas consultas sem widget que o Qt faz para distinguir a barra de arrasto. Firing order e Primary trigger foram renderizadas e revistas com o tema real. Os ensaios são locais/simulados; não há alteração de firmware ou validação física nesta etapa.

## Arranque com área de trabalho limpa

A Console inicia sem Fuel Table e sem os painéis TUNUINO | Configuration e Engine Status. Nenhuma tabela ou gráfico abre automaticamente; a navegação à esquerda mantém-se disponível. Fuel Table abre pela navegação e os dois painéis laterais pelo menu View. A mudança limita-se à visibilidade inicial e preserva os dados e os comandos existentes.

Compilação local da aplicação e CompleteMapSmoke aprovadas. Este ensaio usa a janela real, abertura pela navegação e fecho/reabertura, sem ligação física à ECU. A revisão visual anterior passou a compilação Windows, 14 testes, instalador e publicação automática: https://github.com/Tunuino/Tunuino/actions/runs/42415113581.

## Trigger wizard por motor predefinido

Ignition / Trigger wizard abre um diálogo modal compacto com título Trigger wizard, linha Predefined triggers, oito opções Apply to e OK/Cancel, seguindo as fotografias fornecidas. A ordem das opções é primary trigger, secondary trigger, CAM#2 trigger, firing order, ignition outputs, injector phase, coil dwell time e VVTi CAMs offsets. Seleção azul, grupos disponíveis verdes, indisponíveis cinzentos e caixas com visto.

O catálogo foi ampliado para 25 entradas: 18 têm geometria primária e ordem confirmadas em documentação técnica; seis têm apenas ordem; o 1G-FE permanece indisponível. Isto descreve os dados que o diálogo aplica e não certifica funcionamento completo de 24 motores com a Proteus. Todos os presets continuam sem configurar automaticamente cam, CAM2, referência absoluta de ponto, atribuições de saídas, fase dos injetores, dwell ou offsets de VVT. Os seis grupos correspondentes ficam desativados. A descrição mostra os dentes, número/ordem de cilindros, variante abrangida e lacunas; as fontes constam também da tooltip da entrada.

| Motor / variante | Geometria primária aplicada | Ordem aplicada | Fontes técnicas |
|---|---|---|---|
| 2JZ VVTi | 36-2 | 1-5-3-6-2-4 | [1](https://support.haltech.com/portal/en/kb/articles/2jz-1jz-engine) · [2](https://support.haltech.com/portal/en/kb/articles/2jz-vvt-i-trigger) |
| BMW E36 M52B20 | 60-2 | 1-5-3-6-2-4 | [1](https://support.haltech.com/portal/en/kb/articles/m52-engine) |
| BMW E46 N42B18 | Não aplicada | 1-3-4-2 | [1](https://workshop-manuals.com/bmw/3_series_e46/316i_n42_tour/3_technical_data/12__engine_electrical_system_%28n42%29/12__ignition_wires_spark_plugs/2_td__ignition_harness_spark_plugs_e46/) |
| BMW E46 M54B30 | 60-2 | 1-5-3-6-2-4 | [1](https://manuals.plus/m/076f9e1e05f1686beecceb8cfdf2d6b35d0ab37bfb5ed089ae94c8918a37aef6.pdf) |
| FORD FOCUS MK2 ST225 B5254 | Não aplicada | 1-2-4-5-3 | [1](https://workshop-manuals.com/ford/focus_2004.75_07.2004/mechanical_repairs/3_powertrain/303_engine/303-01d_engine_2.5l_duratec-st_%28vi5%29/specifications/) |
| LEXUS IS200 1G-FE | Não aplicada | Indisponível | Por confirmar |
| LOTUS ELISE CUP 2ZR-FE | 36-2 | 1-3-4-2 | [1](https://pdfcoffee.com/fsm-workshop-manualpdf-pdf-free.html) · [2](https://charm.li/Toyota/2010/Corolla%20L4-1.8L%20%282ZR-FE%29/Repair%20and%20Diagnosis/Specifications/Mechanical%20Specifications/Firing%20Order/) |
| MAZDA MX-5 NC LF-VE | Não aplicada | 1-3-4-2 | [1](https://support.haltech.com/portal/en/kb/articles/mazda-lf-mzr-engine) |
| MINI COOPER R50 R53 | Não aplicada | 1-3-4-2 | [1](https://charm.li/Mini/2003/Cooper%20%28R50%29%20L4-1.6L%20%28W10%29/Repair%20and%20Diagnosis/Maintenance/Tune-up%20and%20Engine%20Performance%20Checks/Firing%20Order/Specifications/) · [2](https://charm.li/Mini/2002/Cooper%20S%20%28R53%29%20L4-1.6L%20SC%20%28W11%29/Repair%20and%20Diagnosis/Powertrain%20Management/Ignition%20System/Description%20and%20Operation/) |
| NISSAN VQ35DE 350Z / Skyline | Não aplicada | 1-2-3-4-5-6 | [1](https://support.haltech.com/portal/en/kb/articles/vq35de-engine) |
| VW R32 BUB BFH | Não aplicada | 1-5-3-6-2-4 | [1](https://www.billswebspace.com/R32TechnicalSpecifications.pdf) · [2](https://workshop-manuals.com/volkswagen/golf-mk5/power_unit/6-cylinder_injection_engine/technical_data/engine_data/sticky-snack.php) |
| TOYOTA 1JZ VVTi | 36-2 | 1-5-3-6-2-4 | [1](https://support.haltech.com/portal/en/kb/articles/2jz-1jz-engine) |
| TOYOTA 1NZ-FE | 36-2 | 1-3-4-2 | [1](https://support.haltech.com/portal/en/kb/articles/1nz-fe-engine) |
| TOYOTA 2NZ-FE | 36-2 | 1-3-4-2 | [1](https://support.haltech.com/portal/en/kb/articles/1nz-fe-engine) |
| TOYOTA 1AZ-FE | 36-2 | 1-3-4-2 | [1](https://support.haltech.com/portal/en/kb/articles/2az-fe-engine) |
| TOYOTA 2AZ-FE | 36-2 | 1-3-4-2 | [1](https://support.haltech.com/portal/en/kb/articles/2az-fe-engine) |
| TOYOTA 2ZZ-GE | 36-2 | 1-3-4-2 | [1](https://support.haltech.com/portal/en/kb/articles/2zz-ge-engine) |
| TOYOTA 1UZ-FE VVTi (1998-on) | 36-2 | 1-8-4-3-6-5-7-2 | [1](https://support.haltech.com/portal/en/kb/articles/1uz-fe-engine) |
| TOYOTA 2UZ-FE (1998-2005) | 36-2 | 1-8-4-3-6-5-7-2 | [1](https://support.haltech.com/portal/en/kb/articles/1uz-fe-engine) |
| TOYOTA 2UZ-FE VVTi | 36-2 | 1-8-4-3-6-5-7-2 | [1](https://support.haltech.com/portal/en/kb/articles/1uz-fe-engine) |
| TOYOTA 3UZ-FE VVTi | 36-2 | 1-8-4-3-6-5-7-2 | [1](https://support.haltech.com/portal/en/kb/articles/1uz-fe-engine) |
| FORD ZETEC-E 2.0 | 36-1 | 1-3-4-2 | [1](https://support.haltech.com/portal/en/kb/articles/duratec-zeta-2-0l-4-cyl) |
| FORD BARRA TURBO BA-BF-FG | 36-1 | 1-5-3-6-2-4 | [1](https://support.haltech.com/portal/en/kb/articles/4-0l-barra-engine) |
| GM LS2 GEN IV 58x | 60-2 | 1-8-7-2-6-5-4-3 | [1](https://support.haltech.com/portal/en/kb/articles/ls2-ls3-gen-4-engine) · [2](https://support.haltech.com/portal/en/kb/articles/ls2-ls3-trigger) |
| GM LS3 GEN IV 58x | 60-2 | 1-8-7-2-6-5-4-3 | [1](https://support.haltech.com/portal/en/kb/articles/ls2-ls3-gen-4-engine) · [2](https://support.haltech.com/portal/en/kb/articles/ls2-ls3-trigger) |

Os dados usados são factos de documentação de fabricantes/fornecedores, incluindo manuais de oficina reproduzidos nos endereços indicados. Não foram copiados ângulos, polaridades ou contactos de outra ECU. As variantes importam: LS2/LS3 são exclusivamente 58x/60-2, excluindo 24x; UZ com 36-2 exclui o primeiro 1UZ de 12 dentes; Zetec-E não significa Mazda MZR/Duratec-HE; o 2JZ corresponde a variantes com sensor de cambota 36-2 e não ao distribuidor; VQ35DE não abrange VQ35HR. NZ/AZ/ZZ são os motores FE/GE nomeados, não híbridos ou de injeção direta.

O Mazda NC tem roda OEM de múltiplas falhas 36-2-2-2; o VQ35DE também necessita de decoder específico. Os respetivos presets aplicam apenas a ordem e deixam Primary desativado. N42, Focus ST225 HYDA, MINI W10/W11 e R32 BUB/BFH também aplicam apenas a ordem nesta etapa. O M52 pode ter sensor de cam por frequência que exige adaptação; o diálogo identifica essa limitação. Não se deduz geometria de cam ou referência de ponto a partir da marca ou da cilindrada.

OK prepara apenas os editores locais selecionados. Primary modifica dentes/ausências no editor primário e no rascunho manual, preservando captura, RPM, bloqueio de avanço, referência absoluta e janela cam. Firing order modifica número/ordem, preservando os contactos existentes; novos cilindros ficam sem atribuição. Com a mesma contagem, preserva os ângulos de evento existentes, incluindo os não uniformes; ao mudar a contagem usa o espaçamento uniforme já adotado pelo editor, a conferir para o motor real. Os outros editores e os seus estados de conflito/base não são relidos ou alterados. Todos os destinos selecionados são validados antes da aplicação.

Cancelar/fechar ou apenas escolher um motor não altera a configuração. Nenhuma página base é substituída e OK não envia gravação ou ARM/RUN. As edições podem ser guardadas no projeto e gravadas depois pela operação explícita existente de cada bloco, com as suas validações. Operações ECU em curso bloqueiam a aplicação. Manual trigger setup continua acessível pelo contexto; Primary trigger conserva a sua janela direta.

A abertura da lista foi corrigida após reproduzir nomes quase brancos sobre fundo claro. O seletor usa agora uma QListView com delegate próprio e cores explícitas para linhas normais e selecionadas, independente do desenho de menu herdado da tabela. A lista mostra até 11 linhas, com deslocamento para os 25 nomes. A correção não altera presets, grupos disponíveis ou dados aplicados.

CompleteMapSmoke passou a abrir o popup real, conferir os 25 nomes no modelo, verificar texto escuro no desenho de uma linha não selecionada, clicar no segundo motor e deslocar/clicar no último. Também confirma a barra de deslocamento e o fecho da lista após seleção. A aplicação e o popup foram renderizados; os 14 testes locais passaram. Não houve ligação física à ECU. A ampliação anterior do catálogo passou a compilação Windows, testes, instalador e publicação: https://github.com/Tunuino/Tunuino/actions/runs/42418124242.

Cada entrada com dados tem fontes associadas e verificação estrutural: roda dentro dos limites nativos, ordem como permutação de 1–12 cilindros e referências HTTPS. Uma entrada inválida não disponibiliza grupos.

Compilação local e 14 testes CTest aprovados. TriggerWizardTest percorreu os 25 registos: confirmou 18 geometrias e 24 ordens, seleção independente de grupos, preservação de contactos/combustível/dwell/referência/comentário/páginas, novos cilindros sem atribuição, manutenção de eventos não uniformes e recusa de grupos sem suporte ou metadados inválidos. CompleteMapSmoke verificou a abertura modal real, motor indisponível, cancelar, aplicar primary isoladamente e aplicar firing isoladamente. Os ensaios anteriores de duas gravações pela configuração manual, com dispatcher C e flash/saídas simuladas, continuam a passar. O diálogo foi renderizado e revisto. Não há alteração de firmware nem validação física de motores nesta etapa.

A primeira versão do diálogo passou a compilação Windows, os 14 testes, instalador e publicação automática: https://github.com/Tunuino/Tunuino/actions/runs/42417089018.

## Sensors and inputs: organização e janelas

A árvore foi reorganizada segundo as duas fotografias, incluindo a continuação até VSS and Gearbox. Os grupos e entradas mantêm a ordem da referência, com ícones próprios de bloco, pasta, curva e assistente. As entradas sem implementação ficam cinzentas/desativadas, com a razão na tooltip; não abrem formulários fictícios.

| Grupo | Implementação disponível nesta etapa | Limites |
|---|---|---|
| Analog inputs | Configuração em duas colunas: resistências fixas, filtros por AV1–AV11 e pull-ups de temperatura | Resistências sem comutação; monitor de tensões separado em Tools / General options / Monitors and panels |
| Digital inputs | Primary trigger, CAM 1, CAM 2 e VSS: configuração e filtros nativos; monitores separados em Tools / General options / Monitors and panels | Comutação elétrica e curvas de sensibilidade indisponíveis. CAM 2 ainda sem descodificação de fase/VVT |
| Oxygen Sensor | Parameters abre a calibração Lambda, com os presets de controladores existentes | Apenas saída analógica externa; não acrescenta controlo interno da sonda |
| TPS, PPS | Calibração TPS de dois pontos e captura ADC | PPS e curvas independentes de plausibilidade indisponíveis |
| Switches | Organização Clutch/brake, Built-in, User, MUX, Latching e Rotary | Não há campos nativos para essas funções |
| IAT, CLT | Dois assistentes NTC e duas calibrações de três pontos | Failsafe configurável indisponível |
| MAP, BARO | Parameters e MAP calibration partilham o editor MAP existente, que contém entrada e curva | BARO separado indisponível |
| Pressure | Parameters com atribuições Fuel/Oil, assistente próprio de Fuel/Oil e calibrações; monitor separado em Tools / General options / Monitors and panels | As outras oito curvas de pressão continuam indisponíveis |
| Temperature | Assistente com alvo IAT/CLT e vista de entradas dos Standard sensors | Custom sensors e as oito curvas adicionais indisponíveis |
| Other sensors | Battery conserva a calibração existente | Parameters, EWG, turboshaft e fuel level indisponíveis |
| VSS and Gearbox | Vehicle speed: relação de velocidade e prazo de perda de sinal ligados à aquisição VSS real | Rodas, eixos, mudanças e sensores de caixa ainda indisponíveis |

As oito calibrações têm agora toolbar escura, ícones de abrir/guardar/repor/ajuda, cabeçalho cinzento, grelha de duas colunas, rótulos ciano e valores verdes. Capturas permanecem junto do ponto de tensão. Os contactos continuam identificados e Customise apresenta a escolha explícita de ligação. Os controlos conservam os mesmos modelos, máscaras e validações; as janelas continuam independentes no ambiente MDI.

Write to ECU mantém-se no menu ECU/F2 e passa também ao contexto da grelha. O botão de rodapé deixou de ocupar a janela. Abrir/guardar na toolbar refere apenas a calibração desse sensor (.tnsensor, formato tunuino-sensor-v1); Save Project as continua a guardar o projeto completo. O ficheiro do sensor é verificado quanto ao formato, sensor, dimensões e valores antes de aplicar, e a gravação usa QSaveFile. Restore last loaded calibration repõe a base carregada apenas no editor, sem gravar na ECU. Não foi apresentado como um preset universal ou como leitura ADC.

### Assistentes de curvas

IAT sensor wizard e CLT sensor wizard calculam um modelo NTC personalizado a partir de R25, Beta, resistência de pull-up, tensão de excitação e três temperaturas. R25, Beta e pull-up começam por preencher; OK só fica ativo depois de introduzir dados válidos. O assistente geral Temperature permite escolher IAT ou CLT.

O cálculo usa R(T)=R25×exp[Beta×(1/(T+273,15)−1/298,15)] e V=Vexc×R/(Rpull-up+R). Ordena os três resultados por tensão crescente e verifica valores finitos, temperaturas entre -40 e 150 °C e pontos distintos na precisão do editor. O firmware continua a interpolar linearmente entre apenas três pontos; o assistente não instala um novo algoritmo NTC. A excitação e o pull-up introduzidos descrevem o circuito real: não são programados na ECU nem apresentados como características universais da Proteus ou de um sensor.

O assistente Pressure permite selecionar MAP, Fuel pressure ou Oil pressure. Usa dois pontos crescentes de tensão e pressão; MAP usa pressão absoluta, enquanto combustível/óleo seguem a referência do sensor instalado. Limites: 0–6 V, MAP 0–500 kPa e Fuel/Oil 0–2000 kPa com pontos de 0,1 kPa. A gravação verifica a faixa ADC real e a referência lida. A escolha de alvo conserva os valores introduzidos e recusa uma curva fora dos limites.

OK aplica a curva ao editor local correspondente, preserva a entrada escolhida e não envia escrita/ARM/RUN. As edições entram no projeto completo. Cancelar não altera o editor. Se existir uma operação ECU em curso, a aplicação é recusada e o rascunho do assistente fica disponível. O assistente não substitui medições físicas por amostras sintéticas.

### Monitorização e verificação

As vistas de inputs usam exclusivamente a configuração guardada e o pacote de runtime da ECU. Conferem identidade, geração e idade da amostra; tensões exigem ADC válido, referência de 2,8–3,6 V e contagem até 4095. Valores antigos, de outra sessão/geração, desligados ou inválidos aparecem como ---; não são simulados. As funções mostradas no monitor são as guardadas na ECU, não as atribuições locais ainda por gravar. A entrada interna de alimentação não é exposta como AV disponível. A vista digital identifica o estado como engine sync partilhado, não como nível lógico medido individualmente no CAM.

Compilação local completa e 14 testes CTest aprovados. SensorCalibrationTest cobriu a fórmula NTC com ponto conhecido, ordem dos pontos, limites inválidos, assistente linear, contactos, identidade/geração/expiração das amostras, estado digital e monitor de temperaturas. Mantiveram-se as doze gravações dos seis sensores nas duas ordens, com dispatcher C, flash e saídas simuladas, uma gravação por operação e preservação de páginas. CompleteMapSmoke conferiu os onze grupos, os nós finais das duas fotografias, entradas indisponíveis, navegação, abertura modal, geração/aplicação IAT e MAP, entrada preservada, cancelar sem alterações e igualdade de parâmetros/páginas não abrangidos. As janelas de calibração e o assistente foram renderizados e revistos com o tema real.

Não há alteração de firmware nem validação física de sensores/motor nesta etapa. A correção anterior da lista de motores passou a compilação Windows, os testes, instalador e publicação automática: https://github.com/Tunuino/Tunuino/actions/runs/42419345295.


## Pressão de combustível e óleo

Pressure / Fuel pressure cal. e Oil pressure cal. abrem editores independentes com entrada AV1–AV11, contacto J103 e dois pontos tensão/pressão. Customise mantém a escolha explícita da ligação. Nenhuma entrada é ativada por abrir a janela; as referências iniciais 0–1000 kPa são inativas e não correspondem a um sensor universal. A escrita de um sensor preserva o outro e os restantes campos/páginas. Não é permitido ocupar uma AV já reservada por MAP, Lambda, TPS, Battery, Launch ou Flex fuel ativos.

Pressure / Parameters mostra as duas funções guardadas, contacto, tensão e pressão calculada com a curva guardada. Usa ADC real, identidade/geração e prazo de dois segundos; falha ADC, entrada desativada, valor fora da curva ou sessão antiga produzem ---. Não extrapola nem limita silenciosamente a pressão. A leitura da Console deriva do pacote ADC já existente; não consulta o novo comando K. Não foram acrescentadas proteção de pressão ou correção de combustível.

O firmware acrescenta capacidade 256 (0x100), total nativo 511, e uma extensão versionada nos bytes 38–63 da primeira página, anteriormente reservados entre os três pontos CLT e os seus valores. A configuração continua com quatro páginas e 1664 bytes. Bloco de 26 bytes todo a zero significa formato antigo desativado; versão 1 tem byte 38=1, registos de nove bytes Fuel em 39–47 e Oil em 48–56, e sete zeros reservados em 57–63. Cada registo contém canal U8, ADC mínimo/máximo U16 e pressão mínima/máxima U16 em décimos de kPa, em ordem de bytes alta primeiro. ADC 0–4095, pressão 0–2000 kPa, pontos estritamente crescentes. A primeira edição materializa apenas referências inativas para o outro sensor.

TUNUINO:K fornece leitura nativa só de consulta: corpo de 15 bytes com estado, versão 1, máscara Fuel/Oil, dois U32 milli-kPa e geração U32. Configuração suja, geração anterior, ADC inválido ou amostra fora da curva invalidam o valor. Publicar configuração limpa reinicia a amostra. O pacote runtime A/O e os comandos anteriores mantêm tamanho e significado.

A Console bloqueia escrita Fuel/Oil, inclusive ECU/F2 e o escritor transacional, quando o firmware não anuncia esta capacidade. Firmware anterior continua utilizável para as funções existentes, mas rejeita uma configuração com a nova extensão. Regressar ao firmware antigo exige uma cópia de configuração anterior compatível. Projetos antigos com seis sensores são abertos acrescentando apenas os dois editores desativados a partir da base; os outros editores e páginas são preservados. Projetos atuais guardam os oito sensores.

Compilação local e 14 testes CTest aprovados, incluindo 16 gravações dos oito sensores em duas ordens através do dispatcher C com flash/saídas simuladas, preservação do sensor oposto, conflitos de entrada, limite de precisão, bloqueio de firmware antigo, monitor, assistente e migração de projeto. Seis suites nativas locais passaram com GCC, AddressSanitizer e UndefinedBehaviorSanitizer (LeakSanitizer desligado no ambiente local). A compilação ARM e os testes GCC/Clang no GitHub passaram: https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42463393128 e https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42463393182. Editor e assistente renderizados/revistos com o tema real. Sem gravação física na ECU nem validação de sensor/motor. Compilação Windows em curso: https://github.com/Tunuino/Tunuino/actions/runs/42463453774.


## Analog Inputs: correção da janela de configuração

Analog inputs na navegação abre Sensors and inputs - Analog inputs, uma janela MDI de configuração com toolbar Open parameters / Save parameters / Restore to defaults / Help, cabeçalho cinzento, grelha de duas colunas e pares Pullup/Pulldown + digital filter. Cada par tem a cor da referência (verde, rosa, ciano, laranja, azul e amarelo); a sequência continua nas onze AV da Proteus. Inclui as duas linhas CLT/IAT de pull-up. A janela permite deslocamento vertical para todas as entradas. O antigo monitor de quatro colunas passa para Sensors and inputs / Analog input monitor. O assistente de pressão e as calibrações individuais mantêm as suas funções próprias.

Os filtros selecionáveis são 500Hz (no filter), 50Hz, 25Hz, 10Hz, 5Hz, 2Hz e 1Hz. No novo firmware a aquisição ADC é agendada a cada 2 ms, 500 Hz nominais; atrasos de execução podem alterar o intervalo efetivo, ainda não medido no hardware. O filtro nativo é passa-baixo de primeira ordem, com alpha=1−exp(−2π f Δt), usando o tempo real decorrido. Não se afirma equivalência do algoritmo DSP com o software da fotografia. No filter ignora este passa-baixo, mantendo o circuito RC e a média de quatro conversões já existentes. A configuração muda cada AV independentemente. Alimentação interna e AT1–AT4 não passam por estes filtros. As contagens AV transmitidas pelo runtime e usadas por MAP/Lambda/TPS/pressões/estratégias passam a incluir a filtragem selecionada; não são apresentadas como contagens anteriores ao filtro.

Há uma diferença física que a interface identifica: o PCB Proteus v0.7, revisão 383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46, tem R701–R704, R801–R804 e R901–R903 de 470 kΩ para GND nas AV1–AV11. R1101–R1104 são pull-ups de 2,7 kΩ das AT1–AT4 para 5V_SENSOR_1. Não há GPIO, relé ou comutador para trocar essas resistências. Por isso os campos mostram 470K Pulldown (fixed) e CLT/IAT pullup 2K7 (fixed), só de leitura. As duas últimas linhas descrevem o hardware AT, não atribuem um canal a CLT/IAT. As escolhas 1M Pulldown, 4K7 Pulldown e 4K7 Pullup e a comutação 2K2 da fotografia não podem ser executadas nesta placa. Não foram simuladas nem tornadas graváveis; ficheiros/projetos não podem alterar os indicadores físicos.

A configuração de filtros ocupa bytes 70–95 da primeira página: bloco de 26 bytes todo a zero significa filtros legados em bypass; versão 1 usa byte 70=1, códigos AV1–AV11 em 71–81 (0 bypass, 1=50, 2=25, 3=10, 4=5, 5=2, 6=1 Hz) e zeros reservados em 82–95. São antigos slots não utilizados de valores CLT. Os três pontos CLT reais e a extensão de pressão mantêm-se intactos. O novo firmware aceita o bloco antigo sem filtros; o firmware antigo rejeita a extensão nova. Regressar exige uma configuração anterior compatível.

Capacidade de protocolo 512 (0x200), total nativo 1023, anuncia os filtros; a Console bloqueia a escrita pela janela e pelo escritor transacional se não existir. A gravação altera apenas versão e onze códigos, com cópia, preparação, releitura, uma gravação permanente e verificação. Não exige criar perfil de motor para configurar entradas e não altera cablagem/saídas/modos. Rascunhos entram no projeto completo; projetos anteriores ganham o bloco a partir da sua base, sem alterar páginas ou gravar. Alterações externas dos filtros durante edição bloqueiam a gravação até descartar o rascunho.

O estado do filtro reinicia com falha ADC, primeira amostra, mudança de geração, relógio recuado ou intervalo superior a 100 ms. Leituras inválidas continuam inválidas; o estado antigo não atravessa uma falha ou nova configuração. Com intervalo zero, não há integração duplicada.

Verificação: compilação local completa e 14 testes CTest aprovados. ParameterTest fez 22 escritas de blocos, incluindo filtros e uma desativação adicional do bloqueio de avanço, com dispatcher C e flash/saídas simuladas: uma gravação por operação e preservação das páginas não abrangidas. Testes cobrem as sete escolhas, resistências não editáveis, ficheiros inválidos, ausência de capacidade, conflito externo e descarte. CompleteMapSmoke confirma navegação para a configuração, migração do projeto anterior e recusa de extensão desconhecida. Sete suites nativas locais passaram com GCC + AddressSanitizer/UndefinedBehaviorSanitizer (LeakSanitizer desativado localmente); o novo teste cobre seis frequências, resposta conhecida a 1 Hz, convergência, bypass, intervalo repetido, falha/relógio/geração e uso da leitura filtrada no MAP. Janela e popup renderizados/revistos com o tema real.

Compilação ARM aprovada: https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42490334607. As sete suites também passaram em GCC e Clang no GitHub: https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42490334627. Compilação Windows em curso: https://github.com/Tunuino/Tunuino/actions/runs/42490378955. Não houve gravação física na ECU nem validação do sinal/temporização num motor.


## Digital inputs / Primary trigger

A entrada Primary trigger em Sensors and inputs / Digital inputs abre Digital inputs - Primary trigger, uma janela MDI compacta com toolbar de quatro comandos, cabeçalho Primary trigger e quatro linhas em duas colunas, na ordem das cinco fotografias. Sensor type usa verde; as outras três linhas usam ciano. As listas têm texto escuro e seleção azul. O monitor anterior passa para Sensors and inputs / Digital inputs / Primary trigger monitor. A janela de geometria em Ignition / Triggers mantém as funções existentes; este bloco não substitui dentes, fase, captura ou bloqueio de avanço.

| Campo | Apresentação e funcionamento na Proteus v0.7 |
|---|---|
| Sensor type | Hardware setting (not readable). VR Sensor e Hall / Optical sensor aparecem na lista, desativados e identificados como hardware only. Não se deduz o sensor instalado. |
| Adaptive threshold | Hardware controlled. Low/High visíveis e indisponíveis; a Console não inventa um nível guardado. |
| Pullup/Pulldown | Hardware / external wiring. None, Pullup 4K7/1K/820 Ohm e Pulldown 4K7/1K/820 Ohm aparecem como opções indisponíveis por software. |
| Input filter | None, Low, Medium, High editáveis; filtragem nativa por intervalo mínimo entre flancos de cambota aceites: 0, 5, 10 e 20 µs. |

As três listas de hardware abrem para consulta; as opções alternativas estão desativadas. Importar um ficheiro/projeto não pode convertê-las em parâmetros graváveis nem afirmar que uma seleção elétrica foi aplicada. O valor do hardware permanece desconhecido, sem leitura da posição do jumper.

O circuito de referência Proteus v0.7, revisão 383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46, liga J101 / 5 (+) e 13 (-) ao condicionador MAX9924 U1203. O modo depende do circuito e do jumper físico JP1201; os respetivos sinais de seleção não chegam a saídas de controlo ou entradas de leitura do MCU. R1215 de 4,7 kΩ liga a saída lógica VR1_OUT a 3,3 V: não corresponde a um pull-up selecionável no terminal do sensor. R1205/R1208 são resistências em série, não opções de pulldown do menu. O [datasheet do fabricante](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX9924-MAX9927.pdf) descreve os modos do condicionador definidos pelos níveis elétricos de ZERO_EN/INT_THRS. Não existe suporte para trocar esses modos ou resistências pela ligação USB atual.

O filtro nativo é blanking de flancos ascendentes, aplicado no callback de aquisição antes da fila de captura: aceita o primeiro flanco e recusa os seguintes dentro do intervalo escolhido, sempre a partir do último aceite. Os flancos recusados não entram no decoder nem no Scope. CAM continua sem alteração e não avança o estado de filtragem da cambota. A subtração do contador U32 mantém o intervalo através da volta do timer. Configurar/ressincronizar a captura reinicia o estado. None preserva a ausência de filtragem anterior.

Estes tempos são uma implementação nativa explícita: as fotos não especificam o algoritmo nem os tempos da referência. Não é medição da largura de um impulso, filtro RC ou limiar adaptativo analógico, e não se afirma equivalência desses mecanismos. O flanco condicionado mantém-se ascendente. A mudança não seleciona automaticamente sensor, geometria, cablagem, ARM/RUN ou modo autónomo.

A extensão usa byte 5 da primeira página como versão e byte 9 como nível. 0/0 é o formato legado sem filtro; versão 1 aceita níveis 0–3. Outras combinações são recusadas. A capacidade 1024 (0x400), total nativo 2047, permite à Console gravar apenas estes dois bytes com o escritor existente, cópia e verificação integral. Ausência da capacidade bloqueia a escrita antes de enviar comandos. Firmware anterior rejeita a extensão; regressar exige uma configuração anterior compatível. As quatro páginas e o formato do runtime mantêm tamanho.

O rascunho fica no projeto completo. Abrir projetos anteriores acrescenta apenas o bloco Primary trigger a partir da página guardada, sem gravação ou alteração de outros editores. Extensões desconhecidas são recusadas. Alteração externa dos campos enquanto há edição bloqueia a gravação até descartar o rascunho. Configurar este filtro não exige criar um perfil de motor, mas a escrita mantém as condições existentes de motor parado e saídas desarmadas.

Verificação: compilação local completa e 14 testes CTest aprovados. ParameterTest executou 23 escritas de blocos contra o dispatcher C com flash/saídas simuladas, incluindo este filtro: uma gravação permanente por operação, cópia prévia e preservação de todas as páginas/campos não abrangidos. Confirma os quatro campos, as escolhas, as alternativas elétricas desativadas, rejeição de dados importados fictícios, bloqueio sem capacidade e conflito/descarte. CompleteMapSmoke verifica a navegação real, janela de quatro linhas, migração do projeto anterior e versão inválida. Janela e lista Input filter renderizadas/revistas com o tema real.

Oito suites nativas locais passaram com GCC + AddressSanitizer/UndefinedBehaviorSanitizer (LeakSanitizer desativado localmente). O novo teste cobre formatos, níveis, limites exatos, primeiro flanco, ruído, CAM independente, volta do contador, reinício e sequência de dentes com intervalo de 25 µs. São testes da função usada pelo callback e do protocolo em simulação, sem ensaio elétrico da placa ou do sensor. Compilação ARM e oito suites GCC/Clang aprovadas: https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42492084776 e https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42492084945. Compilação Windows em curso: https://github.com/Tunuino/Tunuino/actions/runs/42492212975.


## Consulta dos PDFs e CAM 1

A consulta dos documentos e a comparação visual antes de concluir cada bloco ficaram explícitas nas instruções do repositório. A configuração mantém-se separada da monitorização. As páginas usadas e as limitações são registadas; passar testes não é apresentado como prova de equivalência visual ou funcional.

Referências consultadas nesta alteração: EMU_BLACK_V3_Software_Guide.pdf, página 21 (parâmetros em duas colunas), páginas 15–16 (ajuda dos parâmetros) e página 79 (configuração dos sensores primário/secundário em Sensors and inputs / Digital inputs / Primary Trigger e CAM 1). EMU_PRO_Software_Guide.pdf, página 116, foi consultado para distinguir configuração de posição de came de configuração elétrica. Não se importam para a interface BLACK as listas ou ligações elétricas de outro equipamento. As secções consultadas não enumeram todas as opções das fotografias; os três campos e listas CAM 1 foram conferidos também com as imagens fornecidas, sem alegar que o PDF descreve opções que não detalha.

Digital inputs / CAM 1 abre agora uma janela compacta de configuração com Sensor type, Pullup/Pulldown e Input filter, em duas colunas, toolbar de quatro comandos e cores verde/ciano. Sensor type inclui VR Sensor, Hall / Optical sensor e VR Sensor sensitivity table, com alternativas elétricas desativadas. A entrada usa o condicionador MAX9924 U1204 em J101 / 4 (+), 12 (-); não há comando nativo para comutar o circuito, resistências ou programar uma tabela de sensibilidade VR. O estado elétrico não é inventado. O monitor anterior fica em Sensors and inputs / Digital inputs / CAM 1 monitor.

Input filter usa None / Low / Medium / High, com blanking nativo de 0/5/10/20 µs entre flancos ascendentes CAM aceites. O estado CAM é separado do da cambota; flancos num canal não reiniciam o intervalo do outro. Configurar/ressincronizar a captura reinicia ambos. Não se afirma equivalência com os tempos/algoritmo do equipamento de referência. Não modifica fase, geometria, perfil, captura, saídas ou modo autónomo.

Extensão da primeira página: byte 98 versão, byte 99 nível. 0/0 é legado sem filtro; versão 1 aceita níveis 0–3. Capacidade 2048 (0x800), total nativo 4095. A Console grava apenas esses dois bytes, mediante o procedimento existente de cópia, preparação, gravação única e releitura. Sem capacidade, a escrita fica bloqueada antes de enviar comandos. Projetos anteriores ganham apenas o rascunho CAM a partir da base; versões desconhecidas são recusadas. Firmware antigo rejeita a extensão, pelo que regressar exige uma configuração anterior compatível.

Compilação local e 14 testes CTest aprovados. ParameterTest completou 24 escritas de blocos contra o dispatcher C com flash/saídas simuladas, uma gravação por operação e preservação das quatro páginas/campos não abrangidos, incluindo o filtro primário ao gravar CAM. Cobertura de três campos, lista de sensibilidade desativada, falta de capacidade, conflito/descarte, migração do projeto e versão inválida. Janela e popup foram renderizados e comparados com as fotografias. Nove suites locais passaram com GCC + AddressSanitizer/UndefinedBehaviorSanitizer, LeakSanitizer desligado localmente. O teste CAM cobre versões, limites, ruído, contador, reinício e estados intercalados independentes. ARM e nove suites GCC/Clang aprovadas: https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42495009021 e https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42495008967. Windows em curso: https://github.com/Tunuino/Tunuino/actions/runs/42495337741. Sem ensaio físico da ECU/sensores.


## VSS e Vehicle speed

Referências consultadas: EMU_BLACK_V3_Software_Guide.pdf, página 21 (formulário de parâmetros), e [guia oficial de migração V3](https://ecumaster.com/files/EMU_BLACK/EMU_BLACK_Migration_Guide_to_V3_Software.pdf), página 11 (Digital inputs / VSS e reorganização do cálculo de velocidade). A [descrição oficial do software V3](https://community.ecumaster.com/t/emu-black-v3-software-informations/91) confirma a separação entre configuração das entradas e fontes de velocidade/rodas/eixos. Os PDFs fornecidos não enumeram o formulário completo VSS: o guia BLACK, página 70, refere o sinal em Logbook; o guia PRO, página 77, descreve velocidade no stream. Não se afirma reprodução integral de campos não documentados.

Digital inputs / VSS tem Sensor type, Pullup/Pulldown, Input filter, Input e Trigger edge, em duas colunas, toolbar de quatro comandos e cores verde/ciano. VSS and Gearbox / Vehicle speed contém Source, Speed ratio e Signal timeout. As duas janelas foram renderizadas para conferir a organização e o estilo comum. Sensors and inputs / VSS and Gearbox / Vehicle speed monitor é uma janela separada, só de leitura; não substitui configurações.

A ligação foi conferida no PCB/esquema da Proteus suportada: Digital input 1, J101 / 10, condicionador SN74LVC2G17 U1202, PC6, com pull-up físico R1401 de 2,7 kΩ para 5V_SENSOR_1. Hall / Optical sensor e Pullup 2K7 são fixos. VR exige condicionador externo; não se oferece comutação elétrica fictícia. A entrada começa desativada. Escolher Digital input 1 ativa captura de entrada, sem armar saídas. Trigger edge permite Rising/Falling. Input filter usa None/Low/Medium/High como blanking nativo 0/5/10/20 µs; não se afirma equivalência de algoritmo com a referência.

A relação km/h por Hz começa em 0 (sem calibração); aceita 0–1000, passo 0,001. A frequência pode ser mostrada antes da calibração; km/h exige relação positiva. O prazo de perda de sinal aceita 100–5000 ms, inicialmente 1000 ms. É necessária a relação real do sensor/transmissão/roda do veículo. Não se inventa uma relação universal. A falta de pulsos não distingue veículo parado de sensor desligado, pelo que após o prazo mostra indisponível, não velocidade zero confirmada. Não foram implementadas fontes CAN/GPS, agregação de rodas/eixos ou cálculo de mudança.

A captura possui contador, período e blanking independentes da cambota e CAM. O processamento valida dois pulsos, máximo de 10 kHz e 500 km/h, tempo monotónico, perda de sinal e volta do contador. O primeiro pulso após perda de sinal exige novo par. A leitura antiga não reaparece após a volta do contador de 32 bits. Publicar configuração invalida a geração anterior.

Extensão VSS na primeira página, bytes 82–95: versão em 82, entrada em 83, flanco em 84, filtro em 85, relação ×1000 U32 em 86–89, prazo U16 em 90–91 e reservados zero em 92–95. Todos os 14 bytes zero representam legado desativado; versão 1 valida todos os limites. Os filtros analógicos passam a possuir apenas o prefixo 70–81; as duas extensões são validadas e gravadas separadamente, preservando-se entre si. As quatro páginas continuam a ter 1664 bytes no total. Capacidade VSS 4096 (0x1000); total nativo 8191. Firmware antigo bloqueia a escrita antes de comandos; downgrade exige configuração anterior compatível.

TUNUINO:U fornece corpo de 28 bytes: estado/versão, flags e motivo, frequência em mHz, velocidade em milésimos de km/h, período µs, idade ms, geração e contador. Flags 0 indicam leitura inválida; 1 frequência sem calibração; 3 frequência e velocidade. Motivos: 0 válido, 1 desativado, 2 aguardar dois pulsos, 3 perda de sinal, 4 fora dos limites, 5 relógio recuou, 6 configuração/geração desatualizada e 7 frequência sem calibração. Inteiros são big-endian. O monitor usa exclusivamente respostas da ECU, verifica identidade/geração, limpa respostas antigas e suspende pedidos quando oculto.

Guardar usa a transação existente: snapshot atual, cópia, preparação, uma gravação e releitura integral. A máscara altera apenas o bloco editado e a inicialização mínima da extensão legada. Sem mudanças em filtros analógicos, CAM, cambota, perfil ou saídas. Projetos antigos recebem os dois novos rascunhos a partir da base guardada, sem modificar as páginas; formatos desconhecidos são recusados.

Validação: compilação local e 14 testes CTest aprovados; 26 escritas de blocos contra o dispatcher C real com flash e saídas simuladas, uma gravação por operação e preservação das quatro páginas. Testes incluem máscaras, falta de capacidade, migração, formato inválido, frequência sem calibração e resposta tardia de outra identidade. Dez suites nativas passaram; AddressSanitizer/UndefinedBehaviorSanitizer locais com LeakSanitizer desligado. CI do firmware: [ARM aprovado](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42497495284) e [GCC/Clang aprovados](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42497495259). Sem ensaio físico de ECU, sensor ou veículo. O novo bloco requer o firmware com capacidade VSS.


## Digital inputs / CAM 2

Referências consultadas nesta alteração: EMU_BLACK_V3_Software_Guide.pdf, página 21 (parâmetros em duas colunas), página 79 (separação da configuração elétrica dos triggers) e EMU_PRO_Software_Guide.pdf, página 116 (entrada, tipo e flanco de sensor de posição, distintos do atuador VVT). O guia BLACK consultado não enumera um formulário CAM 2 completo; não se atribuem a essas páginas campos que não mostram. O estilo de duas colunas, toolbar de quatro comandos, cabeçalho cinzento, texto verde/ciano e listas com seleção azul foi conferido com a referência comum e as fotografias de CAM 1 já fornecidas. A janela e o popup de filtro foram renderizados para inspeção.

Sensors and inputs / Digital inputs / CAM 2 abre agora configuração própria, com Sensor type, Pullup/Pulldown, Input filter, Input, Trigger edge e Signal timeout. Os três primeiros seguem a organização elétrica; entrada, flanco e prazo são os controlos necessários à aquisição nativa. Sensors and inputs / Digital inputs / CAM 2 monitor abre uma janela separada de frequência, período e pulsos aceites. As janelas mantêm edição e fecho/reabertura normais.

A ligação foi conferida no PCB e esquema: Digital input 2, J101 / 1, U1207 SN74LVC2G17 não inversor, pull-up R1403 de 2,7 kΩ para 5V_SENSOR_1, R1404/RC e U1501 pad 64, PE11. O mapeamento PE11 também foi verificado no [datasheet STM32F427](https://www.st.com/resource/en/datasheet/stm32f427vi.pdf). Esta é uma entrada lógica Hall/óptica; VR exige condicionador externo. Tipo e resistências não são comutados por software. As alternativas indisponíveis permanecem identificadas, sem controlo fictício de sensibilidade VR.

A entrada começa desativada. Selecionar Digital input 2 permite captura independente de cambota, CAM 1 e VSS, com flanco Rising/Falling. None/Low/Medium/High aplicam blanking nativo de 0/5/10/20 µs; não são uma alegação de equivalência ao filtro da referência. O prazo de perda de sinal é 100–5000 ms, inicialmente 1000 ms. Exigem-se dois pulsos e período mínimo de 100 µs. São invalidados sinais fora dos limites, relógio recuado, sinal perdido, geração antiga e configuração por gravar. Um contador que volta a zero não faz reaparecer um sinal expirado.

Este bloco implementa configuração e aquisição de pulsos CAM 2. Ainda não implementa descodificação do padrão CAM 2, ângulo de fase, sincronização do motor por CAM 2, VVT nem comando de solenoides. A sincronização existente continua a usar CAM 1. O monitor declara esta limitação; não mostra ângulos calculados fictícios. Nenhuma abertura, leitura ou gravação arma saídas.

Extensão da primeira página: bytes 57–63, versão/input/edge/filtro/prazo U16/reservado. Todos zero representam legado desativado; versão 1 aceita entrada 0/1, flanco 0/1, filtro 0–3, prazo 100–5000 e reservado zero. Pressão passa a possuir apenas o prefixo 38–56 (19 bytes); a extensão CAM 2 é validada separadamente. Inicializar ou editar pressão preserva CAM 2 e vice-versa. O total das quatro páginas permanece 1664 bytes. Capacidade CAM 2: 8192 (0x2000), total nativo 16383. Firmware sem capacidade bloqueia escrita antes de enviar comandos; software/firmware anteriores não suportam esta extensão e regressar exige configuração compatível.

TUNUINO:J fornece resposta só de leitura de 28 bytes: estado/versão, flags/motivo, frequência mHz em 4–7, zero em 8–11, período µs em 12–15, idade ms em 16–19, geração em 20–23 e contador em 24–27, inteiros big-endian. Flag 1/motivo 0 indica frequência válida; flag 0 usa motivos 1 desativado, 2 aguardar dois pulsos, 3 sinal perdido, 4 fora dos limites, 5 relógio e 6 configuração/geração inválida. A leitura nunca ocupa o comando nem os dados de CAM 1. O monitor verifica identidade/geração, rejeita respostas tardias, limpa valores antigos e suspende pedidos quando oculto.

A escrita usa a cópia prévia, preparação, uma gravação e releitura existentes. A máscara abrange apenas a extensão CAM 2. Conflitos de edição exigem descartar o rascunho; outros blocos são preservados. Projetos antigos recebem o rascunho a partir das páginas guardadas, sem alterar essas páginas. Formatos desconhecidos são recusados antes de aplicar o projeto.

Validação: compilação local e 14 testes CTest aprovados, incluindo migração, formato inválido, navegação e fecho/reabertura. ParameterTest completou 27 escritas contra o dispatcher C real com flash/saídas simuladas, uma gravação por operação e preservação integral dos restantes dados. Testes específicos verificam capacidade, máscara, conflito/descarte, coexistência com pressão, monitor a 25 Hz/40 ms e rejeição de geração antiga. Onze suites nativas passaram com GCC + AddressSanitizer/UndefinedBehaviorSanitizer (LeakSanitizer desligado localmente). CI aprovado: [ARM](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42500303716) e [GCC/Clang](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42500303649). Sem ensaio físico de sensor, ECU ou motor.


## Pressure / Parameters e separação do assistente

Pressure / Parameters abria um monitor de quatro colunas; passa agora a abrir configuração em duas colunas, com Fuel pressure input e Oil pressure input. A aquisição fica separada em Sensors and inputs / Pressure / Pressure monitor. Foram mantidos cabeçalho cinzento, toolbar escura com Open parameters / Save parameters / Restore to defaults / Help, seleção azul e cores dos parâmetros. A janela foi renderizada e inspecionada. Não se afirma paridade com campos de pressão que o protocolo ainda não implementa.

Referências consultadas: EMU_BLACK_V3_Software_Guide.pdf, página 21 (formulário de parâmetros), e EMU_PRO_Software_Guide.pdf, página 27 (assistente linear de pressão com pontos de tensão e pressão). O guia BLACK consultado não enumera todos os campos Pressure / Parameters. A árvore segue as fotografias já fornecidas. O guia PRO usa bar no exemplo; a Tunuino mantém kPa explícitos, conforme as calibrações e leitura já existentes. Não se alteraram escalas guardadas ou unidades implicitamente.

Cada função mostra apenas a entrada atual, Disabled quando diferente e Customise... para a lista completa AV1–AV11 com contactos J103. Não se propõe uma ligação universal para sensores de pressão. O circuito e os contactos continuam os já conferidos na implementação de pressão. Escolher uma entrada ocupada por MAP, Lambda, TPS, aquisição de tensão, Launch ou Flex fuel é recusado. Fuel e Oil não podem partilhar o mesmo canal. Uma troca explícita entre as duas funções é validada contra as atribuições finais; não retira entradas a outras funções.

Gravar parâmetros altera apenas os canais Fuel/Oil (bytes 39 e 48) e mantém os pontos de calibração existentes. Numa extensão legada ausente, inicializa apenas o prefixo de pressão 38–56 com referências editáveis; as entradas começam desativadas. Essas referências não são calibração verificada de um sensor instalado. As curvas devem ser conferidas em Fuel pressure cal. e Oil pressure cal. Nenhuma gravação ativa proteção de pressão ou arma saídas. Restore to defaults desativa as duas entradas, conservando curvas já existentes.

A capacidade Pressure existente continua obrigatória. Sem suporte, a janela retém a edição e bloqueia a escrita. Não se exige perfil de motor para configurar os sensores. A transação mantém cópia prévia, preparação, uma gravação e releitura. Conflitos externos nos canais/versão bloqueiam a escrita até descartar o rascunho; alterações de calibração independentes podem ser relidas sem substituir os canais em edição. Máscaras preservam CAM 2 em 57–63, filtros analógicos, VSS, CAM 1 e restantes páginas. Não foi alterado o firmware nem o formato.

O assistente anterior já permitia escolher MAP/Fuel/Oil, mas o grupo Pressure começava em MAP e usava a mesma identificação. Pressure / Sensor wizard passa a abrir um assistente próprio, com Fuel pressure e Oil pressure; MAP mantém o seu assistente independente. Os rótulos de pressão deixam de dizer absoluta para Fuel/Oil, cuja referência depende do sensor instalado. O assistente prepara apenas a curva do sensor escolhido, preserva a atribuição e não grava na ECU. Não foram inventados presets de fabricante.

Projetos anteriores recebem o rascunho inputs.pressure a partir dos canais guardados, sem alterar as páginas do projeto. O projeto completo guarda este rascunho juntamente com as calibrações; os ficheiros .tnparams deste bloco contêm apenas as duas escolhas de entrada.

Validação: compilação local e 14 testes CTest aprovados. ParameterTest completou 28 operações contra o dispatcher C real, com flash e saídas simuladas, uma gravação por operação, cópia e releitura de todas as páginas. Cobertura específica de capacidade, inicialização legada, coexistência CAM 2, máscaras, canais duplicados/ocupados, troca explícita de Fuel/Oil preservando curvas, conflito/descarte e escolhas Current/Disabled/Customise. CompleteMapSmoke verificou a janela de parâmetros, migração de projeto e o assistente a preparar Fuel/Oil, mantendo páginas e outros parâmetros. Sem ensaio físico de ECU ou sensores.


## Menus, desktops e ajuda

A barra passa a File, Edit, Desktops, Tools, Window, Help e ECU. Base Maps deixa de estar disponível na barra. ECU mantém as ações anteriores: validação, Write to ECU, Firmware / USB DFU e instruções DFU; nenhuma operação de firmware ou protocolo foi alterada. Os monitores ficam nos respetivos grupos da navegação. Seleção Beta/Stable e Check for updates ficam em Help / About. General Options contém as preferências da aplicação, sem monitores ou painéis.

Referências: quatro fotografias de Edit, Desktops, Window e Help fornecidas nesta alteração; EMU_BLACK_V3_Software_Guide.pdf, páginas 9–11 (comandos e atalhos) e 19–21 (ambientes, nomes, disposição e templates). Os quatro popups foram renderizados e comparados com os nomes, ordem, separadores e seleção azul das fotografias. A barra e os menus usam fundo cinzento e texto Segoe UI. O conteúdo File/Tools já existente foi conservado; ECU é a exceção expressamente pedida pelo utilizador.

Edit contém Undo (Ctrl+Z), Redo (Ctrl+Y), Show undo list.., separador, Show/hide Tree view panel (F9) e Show/hide Event log (Shift+F9). Undo/Redo reutiliza o histórico real da tabela numérica ativa, incluindo operações em lote; não cria uma segunda pilha. O histórico retém até 50 snapshots e mostra a quantidade de células alteradas. Fica indisponível em editores sem esse histórico; ainda não há undo global das configurações de sensores/perfil. Event log regista mensagens reais da barra de estado, até 2000 linhas; não é um log de aquisição de motor.

Desktops contém Restore desktops, Store desktops, separador, Open desktops template..., Save desktops template..., separador, Next desktop (Ctrl+Shift+P), Previous desktop (Ctrl+Shift+O), separador, Switch desktop to..., Switch Tree view panel / windows (Tab) e Collapse all items in Tree view panel (Ctrl+\\). Os primeiros dez desktops têm Ctrl+1..0. Existem inicialmente logs, cranking, triggers, ignition, fuel, idle, boost e Extra 1–11, todos vazios. O menu de contexto do separador permite renomear o desktop ativo.

Cada desktop mantém as janelas visíveis, disposição, tamanho, estado normal/minimizado/maximizado e ordem de ativação. Mudar de desktop mostra/oculta as mesmas janelas persistentes, sem copiar ou substituir os parâmetros. Store/Restore e o fecho normal conservam a disposição nas definições locais da Console. Esta versão usa o perfil local da aplicação, ainda sem seleção automática de layouts por UID de ECU. Sem disposição guardada, o arranque permanece limpo.

Templates .tnlayout usam o formato tunuino-desktops-v1: apenas nomes, identidades e retângulos das janelas, sem mapa, calibrações ou comandos de ECU. O carregamento valida o documento inteiro antes de aplicar: desktop ativo, limites de quantidade, nomes, identidades repetidas e dimensões. A gravação do ficheiro é atómica. Janelas desconhecidas são ignoradas; as páginas auxiliares de registo/firmware podem ser recriadas pelo registo existente. Não executa flash nem grava configuração na ECU. Atalhos personalizados dos desktops são preservados ao alternar entre ambientes.

Window contém Tile Horizontally, Tile Vertically, separador, Next, Previous e Close all windows (Ctrl+Shift+C). Distribui ou fecha as janelas visíveis do desktop atual; fechar mantém os editores e a sua edição disponíveis para reabrir.

Help contém General help (F1), Show pinout, Show/Hide help panel e About. A ajuda geral é integrada e explica projetos, escrita, ambientes e tabelas. O painel contextual apresenta a ajuda do bloco ativo. Show pinout abre os contactos de sinal já verificados para a Proteus v0.7: AV, AT, LS, IGN lógico, cambota, CAM 1 e entradas digitais VSS/CAM 2. Identifica-se como lista dos sinais suportados, não como esquema completo de alimentação/massas. About conserva a identificação da Tunuino.

Validação: compilação local e 14 testes CTest aprovados. CompleteMapSmoke verifica a ordem exata da barra, os menus Edit/Window, troca e restauro de desktops, geometria, manutenção de atalhos, rejeição de template inválido, disposição de janelas, fecho do desktop, Tree view/Event log, ajuda/pinout, Undo/Redo/lista e preservação do projeto completo. Os testes são locais/simulados; não exigiram ECU nem alteração de firmware.


## General Options e CAM 1 sensitivity — correção da organização

Monitors and panels foi removido de General Options. Os monitores abrem na navegação junto da sua função: Analog input monitor; Digital inputs / Primary trigger monitor, CAM 1 monitor e CAM 2 monitor; Pressure / Pressure monitor; VSS and Gearbox / Vehicle speed monitor. Gauges contém Runtime gauges e Engine Status. CAM 1 sensitivity abre a curva de configuração/referência, distinta do monitor. Atualizações e canal Beta/Stable ficam em Help / About, mantendo o menu ECU.

Referências: as três novas fotografias e EMU_BLACK_V3_Software_Guide.pdf, páginas 64–66. General Options foi refeito com duas colunas, texto verde, secções cinzentas, caixas com marcas de seleção, listas e campos numéricos. Foram renderizados e comparados o formulário e a curva. As preferências existentes são preservadas; os valores de exemplo da fotografia não substituem calibrações.

| Grupo | Funções ligadas |
|---|---|
| General Options | Appearance Standard/Dark; Units Metric/Imperial; Parameters panel size; Show toolbar; Mark modified cells; Load ECU desktops |
| 2D Tables | New 2D table size Small/Medium/Big e Tables color Red/Blue/Green/Amber nas curvas implementadas |
| 3D Tables | Color scheme Standard/Blue-Violet/Blue-Green-Red; Display square tables; New 3D table appearance Table and graph/Only table; New 3D table size; Cubic 3D graph |
| Logging | Stop logging when paused; Auto save logs; Display system time; Display min values marks; Display max values marks |
| Gauges | Default gauge size; Default gauge type Round gauge/Value/Bar; Round gauge style Standard/Deep Ocean/Amber Gold |
| Other | Don't ask at app exit, preservando os bloqueios de operações ECU em curso |

As unidades psi/°F são de apresentação em mostradores e Graph log. CSV, aquisição e editores de calibração mantêm os valores/unidades nativos explicitamente indicados. Mark modified cells desenha triângulos amarelos relativamente à referência inicial. Show toolbar controla a toolbar principal. Load ECU desktops guarda/carrega disposição pelo UID validado, conservando também o ambiente sem ECU.

O tamanho inicial aplica-se às janelas novas; a disposição guardada não é substituída. Table and graph abre tabela/gráfico lado a lado. Paleta, proporção quadrada das células e aspeto cúbico não alteram o mapa. Gauges é uma janela própria, com canais e seleção independentes de Tune Display, alimentada apenas pela telemetria validada. Dados antigos ou sem ligação removem valores e agulha.

Auto save logs grava CSV a cada 30 segundos, ao parar e ao desligar, em Documents/Tunuino/logs. Stop logging when paused interrompe também a recolha; desativado conserva o comportamento de recolha em segundo plano. System time usa a época real de início; um CSV antigo sem época absoluta continua em tempo relativo, sem inventar a hora. Os extremos são obtidos dos pontos válidos na área visível.

Load on Y axis permanece visível/desativado: o editor mantém load em linhas e RPM em colunas; a transposição visual ainda falta. RTA algorithm, RTA delay(ms) e RTA power factor(%) também estão visíveis/desativados, com explicação. Não modificam o Autotune existente, que continua baseado em recolha/registo. Estes quatro campos são lacunas reais, não comandos simulados.

CAM 1 sensitivity mostra gráfico de área, doze pontos da fotografia e tabela V/rpm. Permite editar e guardar a curva no projeto completo ou em .tncurve. Valida RPM estritamente crescentes de 0 a 20000, tensão de 0 a 5 V e precisão dos pontos. Projetos antigos sem curvas recebem a referência local. Alterar a curva preserva as quatro páginas nativas.

A curva está identificada como referência local: a Proteus suportada tem condicionamento MAX9924 fixo e não disponibiliza comando para aplicar limiares por RPM. F2 não grava esta janela na ECU. Não se apresenta a curva como sensibilidade aplicada nem como monitorização de sincronização. CAM 2 mantém a limitação do seu condicionador Hall/óptico.

Validação: compilação local e 14 testes CTest aprovados. Testes de abertura/aplicação real de General Options, toolbar, persistência, campos indisponíveis, curva separada do monitor, validação e round-trip no projeto preservando páginas; pausa de recolha e CSV automático; psi/°F na apresentação mantendo CSV em kPa/°C; renderização dos três tipos/estilos de mostrador e limpeza sem ligação. Os dois componentes foram renderizados isoladamente para comparação visual. Sem alterações de firmware ou validação física da curva.


## Correção do teste Windows de General Options

A execução 42530876478 compilou com sucesso, mas CompleteMapSmoke atingiu o limite de 30 segundos; os outros 13 testes passaram. O teste integrado passa a registar etapas com tempo decorrido e dispõe de um limite finito de 120 segundos, para distinguir lentidão de bloqueio. As verificações existentes foram mantidas.

A coloração dos mapas consultava QSettings por célula e o gráfico 3D por face/vértice. As preferências de paleta e proporção passam a ser lidas uma vez por coloração/frame, evitando essas leituras repetidas sem cache permanente: alterações de General Options continuam a refletir-se na atualização seguinte. Não altera valores de calibração, protocolo ou firmware.

Compilação local completa e 14 testes CTest aprovados. CompleteMapSmoke passou de 4,15 para 1,29 segundos no mesmo ambiente local. Isto é um ensaio de software, sem validação física. A [execução Windows 42532274246](https://github.com/Tunuino/Tunuino/actions/runs/42532274246) foi aprovada: compilação, 14 testes CTest, arranque offscreen, instalador e publicação Beta. CompleteMapSmoke passou em 2,57 segundos no Windows. O limite maior permanece como margem finita e os checkpoints permitem diagnosticar futuras falhas; a aprovação não dependeu de exceder os 30 segundos anteriores.
