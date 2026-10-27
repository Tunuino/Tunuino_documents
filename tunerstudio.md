> Atualização: [combustível, lambda, transientes, limites e funções pendentes](combustivel-protecoes.md). As novas opções ficam desativadas até configuração explícita.

> Atualização: [TPS, ralenti PWM e controlo de turbo](ralenti-turbo-tps.md), com migração da flash e cópia `.tnaux` separada.

> Atualização: [perfil persistente, funcionamento autónomo, mapas base e tempos em ms](configuracao-persistente-mapas-base.md). As referências anteriores a perfil volátil aplicam-se às versões antigas.

# Tunuino e TunerStudio — INI de bancada

**[Descarregar o Bundle com Console, firmware e INI](https://github.com/Tunuino/Tunuino/releases/download/nightly-2026-10-24-40123914479/tunuino_bundle_proteus_v07_f427_usb_bench.zip)** · [Release](https://github.com/Tunuino/Tunuino/releases/tag/nightly-2026-10-24-40123914479) · build **e50a296**.

A partir do firmware **0.0.3**, o Bundle Proteus v0.7 / STM32F427 inclui `firmware/tunuino_proteus_v07_f427_usb_bench.ini`, a Console Windows e o firmware do mesmo commit. O ficheiro já não depende de um download separado. Esta definição serve para identificação e leitura do estado da ligação USB em bancada.

A localização correta é [`firmware/tunerstudio/`](https://github.com/Tunuino/Tunuino/tree/master/firmware/tunerstudio). O INI fica em `generated/`, com esquema, template, fragmentos de gauges/menu e gerador. A geração também produz a assinatura C correspondente. Dentro do Bundle, o INI mantém-se em `firmware/`, junto da imagem. [Estrutura e geração](estrutura-e-funcoes.md).

## Ligar no TunerStudio

1. Extrair o Bundle completo para uma pasta nova e gravar o firmware que vem nesse mesmo ZIP, com o respetivo `.bin.properties`.
2. Repor BOOT0 em arranque normal e reiniciar a ECU. Confirmar a COM CDC e a versão **0.0.3** na Console.
3. Desligar a ligação COM da Tunuino Console. Uma única aplicação pode usar a COM de cada vez.
4. No TunerStudio, criar um projeto. Na seleção da definição ECU, escolher **Other/Browse** e abrir `firmware/tunuino_proteus_v07_f427_usb_bench.ini`.
5. Escolher a COM da ECU, **115200 baud** e protocolo **Firmware Default**.
6. Confirmar revisão **7**, versão **0.0.3** e capacidades de motor **0**. O menu **Tunuino → Proteus USB bench** apresenta informação de bancada e os três blocos decimais do UID STM32.

A assinatura esperada é `Tunuino Proteus v0.7 F427 USB bench v1`. Não ignorar uma incompatibilidade de assinatura nem escolher o INI de outro firmware. A imagem anterior **0.0.2** só responde ao protocolo da Console e precisa de ser atualizada para usar este INI.

## O que esta definição permite

- Identificação da Proteus/F4, revisão e versão do firmware.
- Leitura do UID do STM32 e confirmação periódica de resposta pela COM.
- Registo dos campos de identidade; o campo Time é o relógio do TunerStudio, não uma medição de tempo da ECU.

O perfil é **apenas de leitura**. Não apresenta RPM, MAP, lambda ou temperaturas, pois a imagem de bancada não adquire esses sinais. Não permite editar mapas, afinar combustível/ignição, fazer Autotune, testar saídas ou controlar um motor. As capacidades de motor permanecem a zero e pedidos diretos de escrita/gravação são recusados.

## Verificação

Os testes locais C/Java/KiCad passaram, incluindo a comunicação normal da Console. Os testes novos comparam as respostas C com a assinatura e os offsets do INI, verificam o CRC por uma implementação Python independente e ensaiam descoberta simples, pedidos fragmentados/juntos, conclusão física de TX simulada, timeout, CRC inválido e rejeição de escrita. O `main` ChibiOS real foi executado com transporte USB simulado e pedidos alternados dos dois protocolos.

A [pipeline 40123914479](https://github.com/Tunuino/Tunuino/actions/runs/40123914479) passou na geração/verificação do INI, testes GCC/Clang, imagem ARM F427, Console/runtime Windows e publicação sincronizada. A compilação ARM confirma que a assinatura do INI existe no binário ligado; a montagem recusa INI ausente, alterado ou incompatível.

**Estes ensaios não executaram a aplicação TunerStudio nem uma ECU física.** O funcionamento USB/identificação do firmware anterior 0.0.2 foi confirmado pelo utilizador em 22 de outubro. A ligação TunerStudio e a versão 0.0.3 precisam de ensaio físico; o resultado anterior não as valida automaticamente.

[Implementação técnica](https://github.com/Tunuino/Tunuino/blob/e50a296189eb57fee8cfa0d35b3cdb4d2dfcbb00/docs/tunerstudio-bench.md) · [Gravação USB/DFU](proteus-v07-bringup.md) · [Pipeline desta alteração](https://github.com/Tunuino/Tunuino/actions/runs/40123914479).
