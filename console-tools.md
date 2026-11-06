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
| Logbook | Não implementado: faltam estatísticas retidas na memória da ECU |
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
| Vehicle VIN... | Não implementado: falta memória dedicada e transporte de diagnóstico na ECU |
| Project comment... | Não implementado: falta guardar o comentário na ECU e no projeto |

## Atribuições e nomes

As listas usam uma leitura completa e limpa da mesma ECU/geração. Verde significa atribuído; amarelo, livre; vermelho, conflito. Uma bobina partilhada por um par de cilindros válido é apresentada como uma função, sem falso conflito. Os contactos J101/J102/J103 foram conferidos no PCB Proteus v0.7 revisão 383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46. Atualizações inalteradas não reconstroem as listas.

Os nomes guardados no computador são identificados como tal. Não são apresentados como se tivessem sido gravados na ECU. Alterar nomes, teclas, aspeto ou grupos visíveis não modifica a calibração ou as saídas.

## Test outputs

Selecionar a saída e contacto, número de ciclos, tempo ligado e tempo desligado. Os limites do driver Tunuino são 20 ms para LS e 5 ms para IGN; intervalo desligado mínimo de 150 ms permite terminar a autorização temporária do teste anterior. IGN são sinais lógicos para um estágio externo.

Antes de cada pulso, confirmar identidade, runtime válido da mesma geração, alimentação externa, RPM zero, ausência de erro de trigger, configuração limpa/válida e saídas inativas. A confirmação é explícita. Não são enviados ARM ou RUN. Cada pulso é limitado pelo temporizador do firmware, não pelo temporizador do Windows. A resposta perdida/refusada não provoca repetição. Cancelar impede os ciclos seguintes; o pulso já aceite termina no seu prazo na ECU. No final, a console volta a verificar ausência de saídas ativas ou pendentes.

## Scope

Abre em Tools → Scope / F5 e Log → Scope. O novo bit de capacidade 16 identifica o suporte. Firmware anterior continua compatível com as restantes operações.

O firmware conserva os últimos 256 eventos de flanco ascendente efetivamente consumidos pela aquisição de cambota e cames. Uma consulta congela o conjunto; as leituras seguintes usam token e blocos de até 32 eventos, com tempos relativos em microssegundos. Geração, token e origem temporal são conferidos durante a transferência. Erros de aquisição invalidam o acesso. Não se alteram as interrupções, a configuração ou as saídas para obter uma captura.

A apresentação mostra os eventos no tempo, com zoom e ficheiros .tnscope. Estes dados não são uma medição de tensão analógica. Capturas offline estão identificadas como offline. A transmissão é serializada com as outras operações USB.

## Autotune

Abre em Tools → Autotune e Log → Autotune. A recolha usa exclusivamente runtime recebido da mesma ECU, geração e quatro páginas base. Não importa um CSV sem proveniência como se fosse uma recolha dessa ECU.

A implementação atual exige alvo lambda habilitado na ECU e ausência de estratégias de malha fechada, transientes, trims e correções de combustível que não estejam compensadas no cálculo. Os intervalos de RPM/MAP/TPS são editáveis. CLT deve estar entre 70 e 130 °C; são rejeitadas amostras inválidas, fora dos eixos, intervalos superiores a dois segundos e transientes de RPM, TPS ou MAP. Exige dois segundos estáveis e pelo menos cinco amostras por célula. Amostras repetidas a menos de 100 ms não aumentam a contagem.

A proposta aplica 50% da correção lambda medida/alvo, limitada a ±10% por revisão. Zero VE não é inventado como base válida. A aplicação é permitida após parar a recolha, confirmar identidade/geração/páginas e verificar que a Fuel Table local não recebeu outras edições. Só modifica o editor local. Para tornar a alteração permanente, usar separadamente Write to ECU, com motor parado e saídas desarmadas; esse percurso preserva a cópia anterior e verifica as quatro páginas.

## Validação e limites

Compilação local completa da console Qt e oito testes CTest, incluindo ToolsTest para ordem do menu, atribuições, conflitos, bobinas partilhadas, leitura Scope, verificações de pulsos e proveniência/limites de Autotune. Os ensaios de configuração existentes continuam a usar o dispatcher e os módulos C reais com persistência simulada.

O novo teste C de Scope cobre ordenação, rotação do buffer, imutabilidade da captura congelada, blocos, token/geração, limites e despacho do protocolo completo, incluindo compatibilidade com o perfil anterior. Passou com AddressSanitizer/UndefinedBehaviorSanitizer no computador; deteção de fugas desativada neste ambiente por limitação do LeakSanitizer. O workflow de firmware executa-o com GCC e Clang.

Estes resultados não são validação física. O Scope precisa da versão de firmware que publica a capacidade 16. Testes físicos de saídas, aquisição USB e Autotune na Proteus continuam pendentes. Esta publicação não declara concluídas as dez funções marcadas como não implementadas, nem todos os detalhes de General options/User names do manual.
