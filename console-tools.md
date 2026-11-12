# Ferramentas da Tunuino Console

O menu Tools conserva a ordem da referência fornecida. A tabela distingue funções operacionais de dependências ainda ausentes. Uma entrada cinzenta não representa uma função implementada: o respetivo tooltip explica a dependência.

| Entrada | Estado nesta versão |
|---|---|
| Device password | Não implementado: falta autenticação e armazenamento protegido na ECU |
| DBW calibration tool | Não implementado: falta controlo DBW, sensores redundantes e driver de ponte H no firmware |
| Calibrate WBO circuit | Não implementado: falta o driver de calibração do circuito WBO; calibração da entrada Lambda externa é uma função diferente |
| Test outputs | Pulsos físicos por protocolo nativo, com limites e verificações descritos abaixo; falta ensaio físico |
| Diff. oil bleed | Não implementado: falta o serviço da bomba/diferencial do veículo |
| Autotune | Recolha real, proposta limitada e aplicação ao editor local; limites abaixo |
| Logbook | Estatísticas retidas na ECU, leitura, checkpoint e reset; campos ainda sem aquisição aparecem indisponíveis |
| User defined CAN stream | Não implementado: faltam transporte CAN e configuração persistente de frames |
| Set EDL-1 datalogger time | Não implementado: falta comunicação com o datalogger |
| Show assigned outputs | Lista real de funções, contactos e conflitos da configuração sincronizada |
| Show assigned inputs | Lista real de funções, contactos e conflitos da configuração sincronizada |
| User names... | Nomes de entradas/saídas, até 16 caracteres, guardados no PC por identidade da ECU e usados nas listas de atribuição; ainda não são incluídos nos logs ou no projeto |
| Scope | Captura de eventos reais de cambota/cames, abrir/guardar, zoom e visualização congelada; exige o novo firmware |
| Graph Log | Disponível, F6 |
| Tune Display | Disponível, F7 |
| Customize keys | Edição das teclas dos menus, deteção de conflitos, reposição das teclas predefinidas e persistência no PC |
| General options | Fonte da interface e modo claro/escuro; restantes opções do manual ainda não estão implementadas |
| Available strategies | Mostrar/ocultar grupos na navegação, com persistência; não ativa estratégias da ECU |
| Decode VW coding... | Não implementado: faltam transporte e definições de codificação CAN do veículo |
| Vehicle VIN... | Leitura/gravação de VIN dedicado na ECU; resposta de VIN por CAN/OBD ainda não implementada |
| Project comment... | Comentário de até 64 caracteres Unicode na ECU e exportação de snapshot .tnproj; restauração ainda não disponível |

## Atribuições e nomes

As listas usam uma leitura completa e limpa da mesma ECU/geração. Verde significa atribuído; amarelo, livre; vermelho, conflito. Uma bobina partilhada por um par de cilindros válido é apresentada como uma função, sem falso conflito. Os contactos J101/J102/J103 foram conferidos no PCB Proteus v0.7 revisão 383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46. Atualizações inalteradas não reconstroem as listas.

Os nomes guardados no computador são identificados como tal. Não são apresentados como se tivessem sido gravados na ECU. Alterar nomes, teclas, aspeto ou grupos visíveis não modifica a calibração ou as saídas.

## Test outputs

Selecionar a saída e contacto, número de ciclos, tempo ligado e tempo desligado. Os limites do driver Tunuino são 20 ms para LS e 5 ms para IGN; intervalo desligado mínimo de 150 ms permite terminar a autorização temporária do teste anterior. IGN são sinais lógicos para um estágio externo.

Antes de cada pulso, confirmar identidade, runtime válido da mesma geração, alimentação externa, RPM zero, ausência de erro de trigger, configuração limpa/válida e saídas inativas. A confirmação é explícita. Não são enviados ARM ou RUN. Cada pulso é limitado pelo temporizador do firmware, não pelo temporizador do Windows. A resposta perdida/refusada não provoca repetição. Cancelar impede os ciclos seguintes; o pulso já aceite termina no seu prazo na ECU. No final, a console volta a verificar ausência de saídas ativas ou pendentes.

## Scope

Abre em Tools → Scope / F5 e Log → Scope. O bit de capacidade 16 identifica o suporte do firmware. Firmware anterior continua compatível com as restantes operações; sem essa capacidade a Console explica que a captura não está disponível.

**Não exige sincronização do motor, perfil de cilindros nem tabelas de combustível/ignição/VVT carregadas.** Usa a configuração real e limpa da mesma ECU/geração (cabeçalho e página auxiliar), já lida antes dos mapas. A escrita e as ferramentas de acionamento/Autotune mantêm as verificações completas próprias.

As mensagens distinguem configuração ainda não carregada, firmware sem Scope, operação ECU ocupada, captura de entradas desativada e resposta sem eventos. Com captura desativada na ECU, habilitar Enable trigger capture em Primary trigger e gravar esse bloco. Isso não arma injetores ou bobinas. Uma ECU parada e desligada do motor pode responder sem qualquer evento; esse resultado é apresentado como ausência de pulsos, não como necessidade de sincronização do motor.

O firmware conserva os últimos 256 flancos ascendentes consumidos pela aquisição. Uma consulta congela o conjunto; as leituras seguintes usam token e blocos até 32 eventos, com tempos relativos em microssegundos. Geração, token, configuração e sequência são conferidos. Não se alteram interrupções, configuração ou saídas para obter uma captura; falhas de aquisição continuam a impedir uma captura válida.

A vista tem zoom e ficheiros .tnscope e representa eventos digitais, não tensão analógica. Abrir ficheiros offline não cria evidência de captura atual no assistente. As operações USB continuam serializadas. Não se fabricam pulsos para preencher uma vista vazia.

A correção foi testada na Console completa, com QSerialPort/pseudoporta e dispatcher C real no computador, sem perfil e sem alimentação externa, incluindo falhas de leitura em mapas independentes. Scope e revisão mantiveram-se acessíveis com as verificações próprias, sem escrita/Burn. Os 17 testes locais passaram e as janelas foram inspecionadas. Referência: fotografias recebidas e guia fornecido de configuração de trigger, páginas 3–4, que mostra a observação dos sinais durante arranque.

A [compilação Windows, os testes, o instalador e a publicação Beta 0.1.3.294](https://github.com/Tunuino/Tunuino/actions/runs/42932229516) foram aprovados.

A confirmação física de aquisição na Proteus permanece pendente. A USB serve para comunicar; sem movimento/sinais de cambota ou cames não há pulsos para mostrar. Não é necessária nova gravação de firmware para esta alteração da Console.

## Autotune

Abre em Tools → Autotune e Log → Autotune. A recolha usa exclusivamente runtime recebido da mesma ECU, geração e quatro páginas base. Não importa um CSV sem proveniência como se fosse uma recolha dessa ECU.

A implementação atual exige alvo lambda habilitado na ECU e ausência de estratégias de malha fechada, transientes, trims e correções de combustível que não estejam compensadas no cálculo. Os intervalos de RPM/MAP/TPS são editáveis. CLT deve estar entre 70 e 130 °C; são rejeitadas amostras inválidas, fora dos eixos, intervalos superiores a dois segundos e transientes de RPM, TPS ou MAP. Exige dois segundos estáveis e pelo menos cinco amostras por célula. Amostras repetidas a menos de 100 ms não aumentam a contagem.

A proposta aplica 50% da correção lambda medida/alvo, limitada a ±10% por revisão. Zero VE não é inventado como base válida. A aplicação é permitida após parar a recolha, confirmar identidade/geração/páginas e verificar que a Fuel Table local não recebeu outras edições. Só modifica o editor local. Para tornar a alteração permanente, usar separadamente Write to ECU, com motor parado e saídas desarmadas; esse percurso preserva a cópia anterior e verifica as quatro páginas.

## VIN, comentário e Logbook

Exigem o novo firmware com capacidade 32 (perfil completo 63). Tools abre a página correspondente na janela de informação da ECU. A console congela a leitura por token, confere identidade/geração e verifica o valor depois de guardar. Revisões evitam que uma leitura antiga substitua dados entretanto alterados. Uma resposta perdida não repete a gravação.

O VIN tem 17 caracteres alfanuméricos, sem I/O/Q; vazio apaga-o. É independente dos mapas e excluído dos ficheiros de projeto. Não há ainda resposta de VIN por CAN/OBD. O comentário admite até 64 caracteres Unicode imprimíveis. A exportação .tnproj contém as quatro páginas completas sincronizadas, comentário, alvo, UID/geração de origem, CRC32 e SHA-256; não inclui VIN ou Logbook. Esta janela ainda não abre nem restaura esses snapshots.

O Logbook recolhe máximos de RPM/MAP/CLT/IAT, tempo acumulado acima de 95% TPS com RPM válido, maior contador observado de erros de trigger e bits retidos de falhas das saídas. EGT, velocidade, contador de reinícios watchdog e calibração do circuito WBO permanecem indisponíveis. O tempo indicado é o uptime da sessão em que se guardou, não uma data/hora nem horas totais de funcionamento.

Os dados são recolhidos também sem a console ligada. O checkpoint automático funciona apenas sem USB enumerado e exige alimentação externa válida, motor parado, ausência de flancos recentes, saídas desarmadas e configuração limpa durante dois segundos; fica limitado a uma gravação por minuto. Com USB ativo, guardar pelo botão Checkpoint ou por Write to ECU; não se inicia um erase/program automático durante uma transferência. Desligar abruptamente a alimentação pode perder observações posteriores ao último checkpoint. A janela mostra se há observações novas apenas em RAM. Reset exige confirmação e preserva VIN, comentário e mapas.

A flash usa registos atómicos com CRC, marcador final e migração dos formatos anteriores. Guardar estes dados nunca torna uma calibração não gravada permanente; Write to ECU preserva os metadados. Uma falha de confirmação na flash bloqueia novas gravações até reiniciar e reler a ECU.

## Validação e limites

Compilação local completa da console Qt e nove testes CTest, incluindo ToolsTest para ordem do menu, atribuições, conflitos, bobinas partilhadas, leitura Scope, verificações de pulsos e proveniência/limites de Autotune. Os ensaios de configuração existentes continuam a usar o dispatcher e os módulos C reais com persistência simulada.

O novo teste C de Scope cobre ordenação, rotação do buffer, imutabilidade da captura congelada, blocos, token/geração, limites e despacho do protocolo completo, incluindo compatibilidade com o perfil anterior. Passou com AddressSanitizer/UndefinedBehaviorSanitizer no computador; deteção de fugas desativada neste ambiente por limitação do LeakSanitizer. O workflow de firmware executa-o com GCC e Clang.

Estes resultados não são validação física. O Scope precisa da versão de firmware que publica a capacidade 16. Testes físicos de saídas, aquisição USB e Autotune na Proteus continuam pendentes. Esta publicação não declara concluídas as sete funções marcadas como não implementadas, nem todos os detalhes de General options/User names do manual.

A implementação das ferramentas passou a [compilação/instalador Windows](https://github.com/Tunuino/Tunuino/actions/runs/42281614256), os [testes C com GCC e Clang](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42281610313) e a [compilação ARM do firmware](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42281610496). O reforço posterior de ligação da captura Scope à identidade/geração sincronizada passou no ToolsTest local e tem nova execução de CI.

VIN/comentário/Logbook: os nove testes Qt passaram localmente. ServiceDataTest usa transporte sintético e verifica leitura por blocos, independência dos campos, releitura, resposta perdida, estado incerto da flash e exclusão do VIN do projeto. O teste C de persistência usa os módulos reais com flash simulada e cobre migração, interrupção em cada palavra do registo, revisão desatualizada, Unicode, máximos retidos e resposta USB após gravação demorada; passou com ASan/UBSan, com a mesma limitação local de LeakSanitizer. A versão final passou os [testes GCC/Clang](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42289802874), a [compilação ARM](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/42289803755) e a [compilação, testes, instalador e publicação de atualização Windows](https://github.com/Tunuino/Tunuino/actions/runs/42288893308). Validação física na Proteus continua pendente.
