# Janelas de Fueling, Ignition e sensores

A navegação principal abre cada bloco numa janela MDI independente, com fecho/reabertura sem perder a edição. Foi removida a janela Calibration & Setup e a sua segunda árvore. A pesquisa percorre os subgrupos. A seleção usa azul; os valores dos parâmetros usam verde.

Os 19 blocos nativos têm duas colunas e quatro comandos: Open parameters, Save parameters, Restore to defaults e Help. Ficheiros .tnparams são específicos do bloco e validados antes de alterar a edição. Abrir ou repor valores não grava na ECU. Write to ECU guarda apenas o bloco da janela, não as edições pendentes noutras janelas.

| Área | Blocos ligados ao protocolo |
|---|---|
| Fueling | General, Fuel cut, Acceleration, Warmup, Cylinders trim, Short term trim, Target lambda e Flex fuel |
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

## Limites que continuam por resolver

Não é ainda equivalência completa com todas as funções da referência. Lambda guard e secondary injectors continuam indisponíveis. Trigger wizard cobre agora o subconjunto nativo descrito abaixo. Firing order já permite criar/redimensionar o perfil e escolher 1–12 cilindros. General, atribuições, dwell e fase usam esse perfil. Os restantes campos e estratégias ainda não têm equivalência completa com a referência.

Flex fuel usa entrada analógica; não implementa sensor de frequência. Rev limiters usa corte de combustível; não implementa padrões de corte de ignição. IAT, compensação de tensão e trims têm flags partilhadas entre combustível/ignição, identificadas na ajuda. O alvo lambda nativo é uma curva MAP de três pontos; a grelha AFR Target permanece offline. Não foram adicionados parâmetros sem suporte como se fossem estratégias operacionais.

O guia PDF fornecido, páginas 21–22, define o formato das janelas e comandos, mas remete os parâmetros individuais para a ajuda interna do programa. Faltam referências detalhadas para reproduzir todos os submenus e campos exatamente.

## Verificação

Compilação local completa e doze testes CTest aprovados. ParameterTest executou 19 gravações com dispatcher/validação C reais e flash/saídas simuladas. EngineProfileTest acrescentou sete gravações cobrindo criação, redimensionamento, os cinco modos, mudança de par e inversão de fase. Cada operação verificou uma gravação permanente, cópia anterior e preservação das restantes páginas. Também foram verificados conflitos, ângulos não uniformes, JSON, releitura e defaults sem alterar fuel/dwell.

TriggerWizardTest acrescentou duas gravações com dispatcher C real e flash/saídas simuladas: geometria sem perfil e geometria/fase com perfil. Verificou cópia, uma gravação permanente e preservação dos mapas, auxiliares, contactos e modo autónomo. Também passou validação de limites, dependência do soft limiter, conflitos, confirmação cancelada, falha sem repetição, identidade/geração da captura e callback apenas após captura completa do Scope.

O assistente e o bloco isolado foram renderizados com o tema da aplicação nas vistas normal e Advanced. O encaminhamento e a reabertura MDI de parâmetros, sensores e Trigger wizard passaram na verificação local; a captura completa via QWidget::grab falhou nesse Qt local, pelo que não se declara validação visual completa da janela principal.

A revisão anterior passou compilação Windows, onze testes, arranque e instalador. A compilação Windows desta alteração está em curso. Os ensaios são simulações no computador, sem validação física na Proteus. Esta etapa usa o firmware já disponível; não altera o firmware.
