# LEDs de estado — Proteus v0.7 / STM32F427

Atualizado em **26 de outubro de 2026**. Implementação no commit
[c589056](https://github.com/Tunuino/Tunuino/commit/c589056bc2d069eba7836cc0c40216c986a67566),
publicado na `master`. Os LEDs são indicadores do firmware; o LED de alimentação
da placa continua separado.

## Comportamento implementado

Os quatro LEDs de estado fazem um teste no arranque: acendem aos 100 ms e
apagam aos 200 ms. Depois mostram os estados seguintes. Um erro crítico tem
prioridade sobre o teste inicial.

| Pino | Função | Indicação |
| --- | --- | --- |
| PE3 | Erro crítico | Aceso nos erros de arranque 10, 11 ou 12 e nas paragens do sistema ChibiOS |
| PE4 | Comunicação | Muda o ritmo conforme a USB, as trocas de dados, a configuração pendente e os erros |
| PE5 | Estado do motor | Apagado sem RPM válidas, aceso durante o arranque, alterna a cada 50 ms em funcionamento |
| PE6 | Aviso | Erros recentes ou persistentes de captura do sinal e recuperação de comunicação enquanto o código 20 está presente |

O limite de indicação entre arranque e funcionamento é **400 RPM**. É apenas
um limite do LED; não altera a temporização nem os cálculos do controlo do motor.
O aviso de um novo erro de captura fica visível durante pelo menos 500 ms e
mantém-se se a captura continuar em falha.

### Ritmos do LED de comunicação

| Estado | Aceso | Apagado |
| --- | --- | --- |
| USB ainda não configurada | 33 ms | 19 ms |
| USB configurada, sem troca válida recente | 99 ms | 59 ms |
| Resposta de protocolo completamente transmitida no último segundo | 33 ms | 100 ms |
| Configuração pendente ou gravação, sem sessão ativa | 500 ms | 500 ms |
| Erro crítico | 450 ms | 50 ms |

A prioridade é: erro crítico, troca recente, configuração/gravação, estado USB.
Aparecer uma COM não conta como troca válida. O firmware só marca atividade
após a notificação de conclusão da resposta USB, não quando a resposta entra
na fila. Desligar/reconfigurar a USB limpa a atividade anterior.

Os LEDs usam temporizadores de interrupção e começam antes de carregar os
parâmetros guardados. Podem continuar a piscar se a rotina principal bloquear.
Não constituem um watchdog nem garantem comunicação correta com o TunerStudio.
Interrupções desativadas, falhas da CPU e bloqueios da flash podem impedir o
serviço dos temporizadores. O LED PE3 tem também um caminho direto no hook de
paragem do ChibiOS; isso não cobre todas as exceções possíveis do processador.

## Verificação e teste na placa

**Passaram localmente:** bateria completa de testes C/Java e USB simulado;
teste específico dos LEDs com GPIO/temporizadores simulados; compilação ARM
F427 e inspeção dos vetores, descritores USB e limites da imagem. A imagem
local tem **46.920 bytes**. O teste específico verifica ritmos e prioridades,
arranque, sessão/expiração, desligar USB, motor parado/arranque/funcionamento,
avisos, erros críticos, paragem e contador de tempo a dar a volta.

**Pendente:** confirmação elétrica e visual na Proteus física. Não houve ensaio
na ECU nem confirmação de que esta alteração resolve a falha de teste de porta
no TunerStudio. A assinatura e o formato das páginas do INI não foram alterados.

Depois de instalar um pacote gerado a partir deste commit ou de um descendente:

1. Gravar o firmware correspondente e repor o arranque normal antes do reset.
2. Observar o breve teste dos quatro LEDs de estado.
3. Com o motor parado, esperar PE5 apagado depois do teste inicial.
4. Verificar a mudança de ritmo de PE4 quando a USB é reconhecida e quando há
   respostas válidas à Console ou ao TunerStudio.
5. Fechar a ligação da Console antes de testar a mesma COM no TunerStudio.

Os pacotes anteriores não incluem esta implementação. O
[Release c589056](https://github.com/Tunuino/Tunuino/releases/tag/nightly-2026-10-26-40366536946)
foi publicado com
[instalador Windows](https://github.com/Tunuino/Tunuino/releases/download/nightly-2026-10-26-40366536946/tunuino_setup_windows_x64.msi)
e [ZIP para testes](https://github.com/Tunuino/Tunuino/releases/download/nightly-2026-10-26-40366536946/tunuino_bundle_proteus_v07_f427_usb_bench.zip).
Passaram no GitHub: Firmware, Unit tests (GCC/Clang), Desktop, TunerStudio,
Windows Console, Bundle (incluindo instalação, atualização e desinstalação do
MSI) e Publish Release. Os dois anexos foram confirmados como carregados e
associados ao commit acima; isto não substitui o ensaio físico dos LEDs.

[Implementação e contrato técnico](https://github.com/Tunuino/Tunuino/blob/c589056bc2d069eba7836cc0c40216c986a67566/docs/proteus-chibios-usb.md#status-indicators)
· [Firmware atual](firmware-f427-progresso.md) · [TunerStudio](tunerstudio.md)
