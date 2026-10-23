# Tunuino e TunerStudio — INI de bancada

**[Descarregar o Bundle com Console, firmware e INI](https://github.com/Tunuino/Tunuino/releases/download/nightly-2026-10-23-39927868860/tunuino_bundle_proteus_v07_f427_usb_bench.zip)** · [Release](https://github.com/Tunuino/Tunuino/releases/tag/nightly-2026-10-23-39927868860) · build **f105ed0**.

A partir do firmware **0.0.3**, o Bundle Proteus v0.7 / STM32F427 inclui `firmware/tunuino_proteus_v07_f427_usb_bench.ini`, a Console Windows e o firmware do mesmo commit. O ficheiro já não depende de um download separado. Esta definição serve para identificação e leitura do estado da ligação USB em bancada.

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

A [pipeline 39927868860](https://github.com/Tunuino/Tunuino/actions/runs/39927868860) passou nos sete jobs, incluindo GCC/Clang, imagem ARM F427, Console e runtime Windows, kit sincronizado e publicação. O empacotador foi também verificado com um kit de teste: um único ZIP, INI incluído e todos os hashes internos conferidos. A compilação ARM verifica que a assinatura do INI existe no binário ligado. A pipeline recusa publicar um Bundle sem INI, com checksum errado ou assinatura/layout diferente.

**Estes ensaios não executaram a aplicação TunerStudio nem uma ECU física.** O funcionamento USB/identificação do firmware anterior 0.0.2 foi confirmado pelo utilizador em 22 de outubro. A ligação TunerStudio e a versão 0.0.3 precisam de ensaio físico; o resultado anterior não as valida automaticamente.

[Implementação técnica](https://github.com/Tunuino/Tunuino/blob/f105ed0b7ff90b5dd37cceb49a32883202adcb84/docs/tunerstudio-bench.md) · [Gravação USB/DFU](proteus-v07-bringup.md) · [Pipeline desta alteração](https://github.com/Tunuino/Tunuino/actions/runs/39927868860).
