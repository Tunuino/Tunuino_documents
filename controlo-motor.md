# Controlo experimental do motor — Proteus v0.7 F427

A implementação [8623c56](https://github.com/Tunuino/Tunuino/commit/8623c56254da0e74b854afa9200c4bd6869eef8d) acrescenta saídas físicas à Tunuino. **Compilação e testes simulados não equivalem a validação na ECU.** Começa com cargas de ensaio e osciloscópio; não existe calibração predefinida do teu motor.

[Descarregar o Bundle correspondente](https://github.com/Tunuino/Tunuino/releases/tag/nightly-2026-10-26-40342132371). Windows Console, Firmware, Unit tests, Desktop, TunerStudio, Bundle e Publish Release passaram para o mesmo commit `8623c56`.

## Sequência na Console

1. Instala firmware e Console do mesmo Bundle. Confirma a placa F427, as ligações e os drivers. IGN é sinal lógico para uma etapa externa; não liga diretamente uma bobina de potência.
2. Em Inputs / Outputs, configura a roda e calibra MAP e temperatura. Guarda e valida as entradas/mapas 16×16. Os mapas de fábrica a zero não permitem alimentar o motor.
3. Lê o estado das saídas. Firmware antigo sem esta função informa que é necessário atualizar e conserva a ligação USB.
4. No perfil do motor, define cilindros, ReqFuel de referência, dwell, tempo morto, pulso máximo, ângulos PMS e saída LS/IGN de cada cilindro. Define a posição do dente zero no ciclo 720° e a janela/volta de referência da fase conforme medições reais. Escolhe auxiliares LS opcionais; 0 mantém a função sem atribuição.
5. Envia o perfil desarmado. O firmware rejeita sobreposições de canais, ângulos repetidos e limites incoerentes. O perfil é volátil: perde-se ao reiniciar e não está incluído no backup de entradas `.tncfg`.
6. Para ensaio isolado, seleciona uma saída e um pulso em µs. Confirma a operação física. Só é aceite com o motor desarmado e sem rotação válida. LS tem limite 20 ms; IGN 5 ms.
7. Depois de medir PMS/polaridade/cablagem e verificar os limites das cargas de ensaio, confirma e arma explicitamente. Sem fase válida e sensores/mapas adequados, não surgem pulsos de injeção/ignição. Bomba em escorva e relé são exceções explícitas quando atribuídos no perfil.
8. Usa PARAR TODAS AS SAÍDAS para desarmar. Uma falha corta e não permite retoma automática. Alterações da calibração exigem novo perfil e nova armação.

A Console consulta as saídas a cada 750 ms quando armadas. A ausência de consulta por 2 segundos corta o controlo experimental. Não é ainda uma ECU autónoma sem Console. O monitor USB/CSV separado fica bloqueado durante a armação para preservar a disponibilidade do mesmo canal para estado e paragem.

## Implementado

- Injeção sequencial em quatro tempos, 1–12 cilindros; cálculo básico ReqFuel × VE × MAP + tempo morto fixo.
- Ignição sequencial com avanço do mapa, PMS por cilindro e dwell configurável.
- Bomba com escorva, relé principal e ventoinha com histerese, configuráveis em LS livres.
- Agendador com TIM2/GPT5 a 1 MHz, limitação de pulsos e supervisão por interrupção; corte por perda de sinal/fase, calibração, atraso e prazos expirados.
- Canais MCU conferidos contra o PCB fixado no repositório; **sem medição física de continuidade, níveis ou correntes**.

## Limitações que continuam abertas

Não estão concluídos: compensações IAT/tensão/aquecimento/aceleração, AFR-alvo e lambda em malha fechada/Autotune, knock, boost, marcha lenta, borboleta eletrónica, modos semi-sequencial/wasted-spark, controlo peak-and-hold e entradas invertidas de carga de bobina. Também faltam persistência do perfil, operação autónoma e configuração de atuação no TunerStudio. O tempo morto e o dwell são fixos, sem curvas por tensão.

O watchdog de 20 ms é servido pelo temporizador no mesmo MCU; não protege contra todos os bloqueios de CPU/interrupções. São necessários ensaios de precisão/jitter, aceleração, reset, energia, falhas e proteção de dwell independente antes de ligar a um motor. Os estados ativos mostrados pela Console são comandos GPIO, não medições da carga.

## Testes

Passaram os modelos C/Java do agendador e do adaptador F427 real compilado contra GPIO/GPT simulados. Foram testados 12 cilindros em seis velocidades entre 400 e 12.000 rpm, contador de 32 bits a dar a volta, limites de dwell/pulso, sobreposição, atraso, perda de fase, geração de configuração, comando de paragem e expiração de supervisão/comunicação. A interface JavaFX valida unidades PMS, bloqueia comandos offline e rejeita dwell fora dos limites. A imagem ARM foi compilada e o vetor GPT5 é inspecionado no executável.

[Estado geral e tarefas pendentes](firmware-f427-progresso.md) · [Contrato técnico](https://github.com/Tunuino/Tunuino/blob/master/docs/engine-outputs.md) · [Pinos LS/IGN](https://github.com/Tunuino/Tunuino/blob/master/hardware/review/proteus-v07-outputs.json) · [Releases](https://github.com/Tunuino/Tunuino/releases).
