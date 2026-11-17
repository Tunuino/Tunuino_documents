# ETB 1 e ETB 2

A Console tem um grupo **ETB**, com **Accelerator pedal** partilhado e dois controladores independentes: **ETB 1** e **ETB 2**. Cada controlador contém **Parameters**, **TPS calibration**, **PID** e **Autotune**. O controlo do motor e as verificações dos sensores executam no firmware da Proteus, não na interface do computador.

## Configuração

1. Em **Inputs and outputs**, atribuir os dois sinais do pedal, os sinais principal/verificação de cada borboleta e uma ponte H para cada motor utilizado. O TPS principal existente é também o **TPS / ETB 1 main**: existe uma única atribuição física. As restantes janelas não repetem seletores de entradas ou saídas.
2. Gravar essas atribuições com **Write to ECU**. Em Accelerator pedal e TPS calibration, capturar as tensões reais de repouso e abertura total, ou introduzir a calibração medida. A captura exige ECU atual, ADC recente, motor parado e saídas desarmadas. Não há movimento automático para medir os batentes.
3. Rever os parâmetros e proteções de cada controlador e gravar. Os valores iniciais são referências desativadas, não uma calibração universal de pedal ou borboleta. As duas pistas podem ter sentidos opostos; cada uma exige amplitude mínima de 1 V e endpoints entre 0,125 e 4,750 V.
4. Rever o PID de cada ETB ou executar o seu Autotune. Abrir janelas, ler a ECU ou abrir um projeto não arma as saídas.

O pedal é partilhado, mas os sensores TPS, PID, limites e estado de falha são independentes. Uma falha de ETB durante controlo armado corta as pontes H e as saídas do motor. As verificações incluem divergência entre pistas, intervalo de amostragem, alimentação, erro de seguimento e retorno de pedal ao repouso antes de permitir atuação. O firmware usa ADC sem o filtro configurável das entradas AV para esta regulação. O estado de falha é retido até novo rearme explícito. Um temporizador independente desliga ambas as pontes H se o ciclo principal deixar de renovar o comando durante 20 ms.

Cada painel usa a gravação existente, com cópia anterior, validação, uma gravação persistente e releitura. O vermelho indica alterações locais pendentes; o Autotune nunca grava automaticamente. A ligação COM mantém-se aberta; a transação de gravação suspende brevemente as consultas de diagnóstico para evitar misturar respostas.

## Autotune PID

Requer os sinais redundantes e as calibrações já gravados, alimentação do motor da borboleta, motor do veículo parado, saídas do motor desarmadas e pedal em repouso. **Start Autotune** movimenta apenas a borboleta selecionada em torno de 40%, mede oscilações reais e calcula uma proposta P/I/D. Há limites de comando, tempo de aproximação, período de oscilação e duração total; falta de movimento ou sinais inválidos termina sem proposta.

**Abort** desliga o ensaio. Fechar a janela pede cancelamento. O ensaio exige renovação explícita a cada 500 ms; perda de comunicação deixa expirar essa autorização e desliga o motor. Consultas de diagnóstico normais não renovam o ensaio. Não existe reinício automático.

O gráfico apresenta alvo e posição recebidos da ECU. **Apply PID proposal** prepara apenas os ganhos do ETB selecionado no editor local. Rever e usar **Write to ECU** para gravar. Uma proposta perde validade quando muda a ECU, a geração ou a configuração. **Discard proposal** elimina a proposta local. Isto é afinação PID com endpoints previamente medidos; não é calibração automática dos batentes, da corrente ou de curvas não lineares.

## Hardware confirmado

Referência KiCad e PCB Proteus v0.7, revisão `383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46`:

| Motor | Contactos | PWM | DIR | DIS |
| --- | --- | --- | --- | --- |
| Ponte H ETB 1 | J101 7 (+), 6 (−) | PD12 / TIM4 CH1 | PD10 | PD11 |
| Ponte H ETB 2 | J101 15 (+), 8 (−) | PD13 / TIM4 CH2 | PD9 | PD8 |

Os motores usam **12V_MR, J101 23**. Alimentar apenas a ECU por USB não alimenta os motores. As pontes usam o mesmo temporizador: quando ambas estão atribuídas, têm de usar a mesma frequência PWM. A Console e o firmware rejeitam frequências incompatíveis e atribuições repetidas.

As duas pontes TLE9201SG não têm SPI/SO ligados no PCB. Não há medição de corrente nem diagnóstico SPI do driver disponíveis para a Console, nem um limite de corrente ajustável apresentado como funcional. As proteções internas fixas do circuito não constituem telemetria da aplicação. A verificação por tensão da ECU não mede separadamente a presença de 12V_MR; sem alimentação do motor, o ensaio deve falhar por ausência de movimento.

## Protocolo e compatibilidade

O firmware acrescenta a página 12 de 256 bytes para os dois ETB e as seis calibrações, mantendo a atribuição TPS principal na página auxiliar existente. A identidade anuncia capacidade ETB no bit 21. A Console lê as 12 páginas automaticamente e inclui a página ETB nos projetos e nas cópias anteriores. Firmware anterior mantém o seu número de páginas e não recebe comandos ETB.

O journal persistente passa para formato 11 e migra configurações dos formatos 2 a 10 com ETB inicialmente desativado. Projetos anteriores recebem os novos editores desativados. São preservadas as configurações e mapas de combustível, ignição, VTEC e VVT das duas cames.

Consultas e ensaio usam operações TUNUINO:X 12 (estado), 13 (iniciar), 14 (cancelar) e 15 (renovar), com geração e canal. Nenhum destes comandos grava configuração ou arma as saídas de injeção/ignição.

## Referências e verificação

Consultado o PDF fornecido **EMU_PRO_Software_Guide.pdf**, Appendix C, páginas **97–103**, para sensores redundantes, calibração, dois corpos de borboleta, PID e ensaio de afinação. A organização foi comparada com essa referência; a implementação não reproduz a calibração automática completa descrita no manual. O pinout e o driver foram verificados no KiCad/PCB e no [datasheet oficial TLE9201SG](https://www.infineon.com/assets/row/public/documents/10/49/infineon-tle9201sg-ds-en.pdf), páginas 4, 8 e 19–20.

Testes locais: **17/17 da Console**, **16 testes nativos com ASan/UBSan**, compilação ARM e verificação do orçamento de stack. Incluem controlador e falhas, afinação com modelo simulado de borboleta, cancelamento, independência dos dois PID, propostas sem gravação automática, escrita/releitura das 12 páginas, preservação de projetos e migração persistente. A integração Qt/porta série com dispatcher nativo também passou a leitura inicial de 12 páginas e falhas injetadas em leituras parciais. As janelas foram inspecionadas com o tema da aplicação.

Estes testes são execução em computador e compilação; **a atuação de uma borboleta física na Proteus ainda não foi validada**. Não foi gravado firmware numa ECU durante este trabalho.

## Código e compilação Windows

- [Console na master](https://github.com/Tunuino/Tunuino/commit/5c3bd3c6b1d57276d2add22a43cb0a1e0067fd01).
- [Firmware ETB na master](https://github.com/Tunuino/Tunuino_Firmware/commit/cb2dfd3217d547b4f2e6f1c5ad5f8937b8d0ec8c).
- [Compilação e instalador Windows](https://github.com/Tunuino/Tunuino/actions/runs/44191580072).
- [Compilação ARM aprovada](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/44191200468).
- [Testes nativos GCC e Clang aprovados](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/44191200443).

O suporte de atuação ETB exige este firmware ou posterior. Uma atualização apenas da Console não acrescenta o controlador a um firmware anterior. A atualização de firmware permanece uma operação explícita no menu ECU.
