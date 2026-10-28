# Console: Português / English, mapas iniciais e calibrações

Atualização de **28 de outubro de 2026** para a Console Windows e Proteus v0.7 STM32F427. Alterações de apresentação e aquisição de calibrações; o formato do firmware permanece compatível.

## Língua da interface

A caixa **Português / English** fica no canto superior direito. A mudança é imediata e fica guardada para o próximo arranque. Os nomes técnicos **Dead time**, **Dwell**, **TPS**, **MAP** e **Autotune** mantêm-se reconhecíveis nas duas línguas.

A tradução atua sobre os textos apresentados, incluindo navegação, controlos, separadores, tabelas, ajudas e estados. Não altera os valores dos seletores, as ligações dos sensores, os mapas, os bytes da configuração ou os identificadores do protocolo. Os textos de origem são preservados para voltar ao português sem reconstruir páginas nem perder edições. As janelas de mapas e os avisos usam o mesmo mecanismo.

## Tabelas preenchidas ao abrir

O painel, Fuel Tables e Ignition Tables abrem com os dois mapas 16×16 preenchidos pela referência genérica aspirada existente, entre 200 e 7000 rpm e 20–105 kPa absolutos. A ignição secundária e a superfície 3D também usam esses valores.

A legenda distingue **BASE GENÉRICA · POR AFINAR** de **DADOS DA ECU** por tipo de tabela. A leitura de VE não transforma a ignição local em dados da ECU. Ler um mapa substitui a referência desse mapa; mudar de separador ou uma atualização dos controlos preserva as cópias lidas na mesma sessão. Desligar ou trocar de ECU remove as cópias da sessão anterior e repõe bases locais claramente identificadas.

**Editar tabela** ou duplo clique permite abrir a referência já preenchida para edição offline. Esse editor local não oferece gravação na ECU. Para escrever uma tabela real mantém-se o percurso explícito de leitura, edição e gravação verificada; os controlos de criação de bases continuam disponíveis para preparar outras cópias. Abrir a Console não envia mapas nem arma saídas. Os mostradores de sensores sem medição continuam sem valores; mapas base não são telemetria simulada. O destaque da célula ativa só usa mapas efetivamente lidos da ECU.

A base aspirada não cobre pressão de turbo e não constitui afinação validada para um motor específico.

## Autocalibração assistida

Os assistentes ficam em **Sensores e calibrações**, junto dos campos e referências existentes. O fluxo normal é ler a configuração, selecionar a ligação, capturar cada condição indicada, usar a calibração na edição e guardar pelo botão existente. TPS, IAT e tensão mantêm a gravação própria que preserva modos e parâmetros alheios ao sensor.

| Sensor | Passos e referências iniciais |
|---|---|
| TPS | Borboleta fechada = 0%; totalmente aberta = 100%. A Console mede os extremos, incluindo sensores de sentido decrescente. |
| MAP | Dois pontos de pressão absoluta conhecida; exemplos editáveis de 50 e 100 kPa. |
| Lambda | Dois valores conhecidos da saída analógica do controlador wideband; exemplos editáveis de 0,8 e 1,2 lambda. Não calibra a célula da sonda. |
| CLT e IAT | Três temperaturas conhecidas com termómetro; exemplos editáveis de 20, 60 e 100 °C. Mantêm os modelos matemáticos NTC 2,2 kΩ/B3950 e 10 kΩ/B3950 com pull-up editável. |
| Bateria | Dois pontos medidos com multímetro; exemplos editáveis de 12 e 14 V, incluindo o divisor da entrada. |

Cada captura recolhe oito leituras reais espaçadas de 40 ms, usa a mediana e exige ADC válido, relógio crescente, geração estável e dispersão até 20 contagens. Rejeita saturação em 0/4095 e pontos separados por menos de 32 contagens. As curvas medidas mantêm os valores físicos associados ao ordenar por ADC. A alteração de ECU, ligação ou snapshot invalida os pontos pendentes. A recolha e a aplicação local não escrevem na flash nem ativam compensações.

Um sensor alimentado a 5 V não implica extremos de sinal exatamente a 0 e 5 V. O TPS é calibrado pelos extremos efetivamente medidos depois do condicionamento da entrada. Para os restantes sensores, valores pré-preenchidos são exemplos para orientar o procedimento; a pressão, temperatura, tensão ou saída do controlador precisa de uma referência conhecida. O circuito ADC não é uma entrada direta universal de 5 V ou 12 V.

## Injector dead time em milissegundos

A configuração de injeção apresenta a base fixa numa pequena tabela **Referência / Dead time (ms)**. A referência local inicial é 1,000 ms; a leitura de um perfil preserva o valor guardado. A base fixa é usada quando a compensação por tensão está desativada.

Em **Combustível e proteções → Tensão** há a tabela editável:

| Tensão da bateria | Dead time de referência |
|---|---|
| 8 V | 1,500 ms |
| 12 V | 1,000 ms |
| 16 V | 0,700 ms |

São os valores genéricos que já existiam no esquema da Console, agora visíveis em tabela e recuperáveis por **Usar referência inicial**. Não identificam um injetor nem substituem a curva do fabricante à pressão diferencial instalada. A tabela aceita vírgula ou ponto decimal sem arredondar silenciosamente. A mesma coluna de tensão define os três pontos de dwell no formato atual; a interface explica essa associação. Mostrar ou editar a referência não ativa a compensação por tensão.

## Verificação

Passaram localmente os testes do núcleo C/Java e da aplicação JavaFX real em modo headless Linux. Cobrem mapas preenchidos offline, proveniência independente VE/ignição, substituição por leitura, invalidação ao mudar de ECU, captura estável/instável, geração e relógio, TPS invertido, ordenação de NTC, ausência de aplicação prematura, bloqueio offline/ocupado e mudança de ligação. Verificam edição exata do Dead time em ms e alternância Português/English sem alteração de mapas ou bytes da configuração.

A compilação e a verificação do pacote Windows são executadas pelo workflow Windows Console. O teste local não utiliza ECU física e não valida eletricamente sensores, injetores ou curvas de calibração. Os ensaios físicos na Proteus continuam necessários.

Implementação: [de46fe1](https://github.com/Tunuino/Tunuino/commit/de46fe1919182c558b5f7fa6d1a13145f089def7). [Verificações e capturas Windows](https://github.com/Tunuino/Tunuino/actions/workflows/build-console.yml).
