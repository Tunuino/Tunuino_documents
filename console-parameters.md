# Janelas de Fueling, Ignition e sensores

A navegação principal abre cada bloco numa janela MDI independente, com fecho/reabertura sem perder a edição. Foi removida a janela Calibration & Setup e a sua segunda árvore. A pesquisa percorre os subgrupos. A seleção usa azul; as cores dos parâmetros distinguem números, listas, atribuições e caixas.

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
