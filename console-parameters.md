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

## Limites que continuam por resolver

Não é ainda equivalência completa com todas as funções da referência. Lambda guard, secondary injectors e trigger wizard continuam indisponíveis. Firing order reordena um perfil existente, preservando os ângulos TDC e a cablagem; o número de cilindros é apenas apresentado. Criar/redimensionar o perfil ainda falta. General, atribuições, dwell e fase precisam de perfil já configurado.

Flex fuel usa entrada analógica; não implementa sensor de frequência. Rev limiters usa corte de combustível; não implementa padrões de corte de ignição. IAT, compensação de tensão e trims têm flags partilhadas entre combustível/ignição, identificadas na ajuda. O alvo lambda nativo é uma curva MAP de três pontos; a grelha AFR Target permanece offline. Não foram adicionados parâmetros sem suporte como se fossem estratégias operacionais.

O guia PDF fornecido, páginas 21–22, define o formato das janelas e comandos, mas remete os parâmetros individuais para a ajuda interna do programa. Faltam referências detalhadas para reproduzir todos os submenus e campos exatamente.

## Verificação

Compilação local completa e dez testes CTest aprovados. ParameterTest executou 19 gravações com dispatcher/validação C reais e flash/saídas simuladas, verificando uma gravação permanente por operação, preservação das quatro páginas, bits partilhados e separação entre firing order e cablagem. A verificação local da janela principal confirmou encaminhamento, abertura simultânea e reabertura; a captura completa via QWidget::grab falhou no Qt local, pelo que não se declara validação visual completa da janela principal. O bloco de parâmetros isolado foi renderizado e inspecionado.

Compilação/testes/instalador Windows em curso. Estes ensaios não são validação física na Proteus. Esta alteração da console usa o firmware atual; não foi gerado outro firmware nesta etapa.
