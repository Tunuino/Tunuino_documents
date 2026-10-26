# Firmware Tunuino STM32F427 — estado atual

Atualizado em **26 de outubro de 2026**. Alvo: **Proteus v0.7 / STM32F427**, protocolo IO **0.2.0**, assinatura `Tunuino Proteus v0.7 F427 IO v2`. A versão do protocolo mantém compatibilidade de sensores/mapas; identifica o pacote concreto pelo SHA do Bundle.

O commit [8623c56](https://github.com/Tunuino/Tunuino/commit/8623c56254da0e74b854afa9200c4bd6869eef8d) acrescenta **comando físico experimental de saídas**, com perfil de motor configurável na Console. Não se trata apenas de pré-visualização matemática. A validação física destas novas saídas ainda não foi realizada.

| Subsistema | Implementação e limites |
|---|---|
| Arranque F427, ChibiOS e USB CDC | Implementados; ligação COM e identificação de versões anteriores confirmadas pelo utilizador; nova imagem exige ensaio próprio |
| Sensores | 16 ADC, máscaras de validade e calibração de MAP/lambda/temperatura; TPS/IAT e outras medições calibradas ainda faltam |
| Cambota/cames | Captura EXTI, roda uniforme com dentes em falta, fase 720°; janela de cames e referência ao PMS configuráveis no perfil |
| Mapas | VE e ignição 16×16 nativos; leitura, edição, persistência e verificação completa; mapas de fábrica a zero |
| Configuração de entradas/mapas | Flash schema 2, 1.408 bytes de parâmetros, registos de 1.440 bytes com CRC; não grava enquanto saídas estão armadas |
| Injeção | Sequencial, 1–12 cilindros, ReqFuel/VE/MAP, tempo morto fixo e ângulo de fim por referência PMS; limite 20 ms e 80% do ciclo |
| Ignição | Sequencial com PMS e canal por cilindro, avanço pelo mapa e dwell configurável até 5 ms; sinal lógico IGN para etapa de potência externa |
| Saídas auxiliares | Bomba com escorva e funcionamento durante rotação, relé principal e ventoinha com histerese; canais LS escolhidos pelo utilizador |
| Agendador | Contador TIM2 e interrupção GPT5 a 1 MHz; pulso/lease independentes do ciclo de leitura USB, sem temporização por sleep |
| Cortes | Perda de sincronismo/sinal, limite RPM, calibração alterada/inválida, atraso de flanco e falta de serviço; falha exige armação explícita |
| Console | Perfil por cilindro em Inputs / Outputs, consulta real de estado, pulsos de bancada, armação confirmada e paragem de todas as saídas |
| Supervisão do anfitrião | Perfil volátil; Console consulta estado a cada 750 ms e ausência de consulta por 2 s corta saídas. Ainda não é modo autónomo sem Console |
| TunerStudio | INI e protocolo IO mantidos; o novo perfil de atuação é configurado pela Console. Painel de parâmetros de atuação no TunerStudio continua pendente |

As consultas `TUNUINO:E` e `TUNUINO:T` continuam só de leitura. O comando de atuação é separado, `TUNUINO:X`, com configuração, armação, paragem, estado e um único pulso de bancada. Ligar USB, identificar ou enviar um perfil **não arma as saídas**.

## O que ainda falta

- Validar na placa: reset/arranque, níveis e polaridade, cablagem, corrente, PMS, sincronismo, jitter e aceleração. Os modelos de registos não medem esses efeitos.
- Perfil persistente de motor, funcionamento autónomo, backup/restauro deste perfil e interface equivalente no TunerStudio.
- TPS/IAT/bateria calibrados, correções por temperatura/pressão/tensão, enriquecimento de arranque/aquecimento/aceleração e curva de tempo morto por tensão.
- Lambda em malha fechada/AFR-alvo/Autotune, knock, boost, marcha lenta e borboleta eletrónica com redundância e diagnóstico.
- Modos semi-sequencial/wasted-spark, entradas de carga invertidas e drivers peak-and-hold.
- Proteção independente contra bloqueios da CPU/interrupções, brown-out e falhas de drivers. O prazo de 20 ms atual é um watchdog de software servido por interrupção de temporizador, não um circuito independente de corte de dwell.
- Monitorização/datalogging em simultâneo com a armação: neste primeiro perfil experimental o monitor separado é bloqueado para manter disponíveis a consulta de estado e a paragem pelo mesmo proprietário da porta.

## Verificação desta alteração

Passaram localmente a compilação ARM F427, inspeção da imagem e vetores de interrupção, a bateria C/Java e a interface real JavaFX com dados sintéticos. Os testes novos incluem 12 cilindros entre 400 e 12.000 rpm, flancos LS/IGN, rejeição de perfis, sobreposição de pulsos, dwell máximo, contador a dar a volta, atrasos, perda de fase, alterações de configuração, paragem, bloqueio da rotina principal e expiração da comunicação. Os workflows do commit verificam separadamente o pacote Windows e a publicação do Bundle; consultar o estado atual antes de descarregar.

**Não houve ensaio físico destas saídas nem funcionamento confirmado num motor.** O firmware inicia desarmado e não inclui uma afinação para o motor do utilizador. Começar por cargas de ensaio e osciloscópio, com pinagem/polaridade verificadas e fontes/limites adequados.

[Configurar e testar saídas](controlo-motor.md) · [Contrato técnico e limites](https://github.com/Tunuino/Tunuino/blob/master/docs/engine-outputs.md) · [Mapa de pinos](https://github.com/Tunuino/Tunuino/blob/master/hardware/review/proteus-v07-outputs.json) · [Console](console-dashboard.md) · [Releases](https://github.com/Tunuino/Tunuino/releases).
