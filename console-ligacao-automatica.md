# Ligação automática da Tunuino Console

A console procura e liga-se automaticamente à ECU, sem selecionar uma porta COM nem confirmar a ligação. Mostra ONLINE ou OFFLINE na barra de estado.

## Funcionamento

1. Ao abrir a console, a descoberta procura o dispositivo USB da Tunuino.
2. A ligação interna envia apenas o pedido de identidade, valida protocolo, hardware e UID e começa a consultar a telemetria.
3. A presença do USB não é suficiente para ficar online. Só depois de confirmar a alimentação externa da Proteus a console entra online e lê automaticamente configuração, Fuel e Ignition.
4. Se a alimentação externa desaparecer, a console passa a offline mesmo que continue alimentada por USB. Limpa as leituras atuais e a mira dos mapas, desativa leituras manuais e termina o registo ativo, preservando as amostras recolhidas.
5. Quando a alimentação regressa, volta a entrar online e sincroniza automaticamente. Se houver perda da ligação USB ou erro de comunicação, tenta detetar e ligar novamente sem intervenção. Uma mudança do número COM não exige configuração.

Abrir Firmware / USB DFU suspende a comunicação normal e a reconexão enquanto essa janela estiver visível. Fechá-la permite retomar a deteção normal. A pesquisa DFU não interfere com uma sessão serial aberta.

## Origem da medição

A implementação atual é específica para Proteus v0.7 com STM32F427, protocolo de identidade 1 e runtime de 100 bytes. A descoberta USB usa VID CAFE / PID 4001, conforme o descritor do firmware atual; não envia pedidos a outras portas nem ao bootloader DFU.

A alimentação é medida no circuito 12V_RAW da placa. A referência é o esquema e PCB do hardware [Proteus](https://github.com/Tunuino/proteus/tree/383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46): R1503 de 82 kΩ e R1504 de 10 kΩ, rede 12V_DIVIDED, U1501 contacto 43, ADC1_IN7. A sequência ADC do firmware coloca esta medição no índice 11; o runtime já transmite esse ADC no deslocamento 82. Não foi necessário alterar o firmware para disponibilizar a medição.

A conversão usa referência nominal de 3,3 V e divisor de 9,2. Para entrar online, exige duas amostras válidas acima de 6 V, separadas por pelo menos 250 ms. Abaixo de 4 V fica offline. A histerese evita alternâncias durante pequenas quedas de tensão no arranque. ADC inválido, saturado, identidade incompatível ou perda da comunicação nunca confirmam online.

Esta medição acompanha a alimentação externa da ECU. Não é uma entrada separada que leia diretamente a posição mecânica da chave. Numa instalação em que a chave comuta essa alimentação, acompanha ligar/desligar a ignição. USB sozinho permanece offline. O estado de alimentação não constitui uma proteção do motor nem arma saídas.

## Verificação

- Compilação local completa com Qt 6.4.2 em Linux: passou. Foi omitida apenas a geração do script de distribuição na cópia temporária do CMake usada para verificação local, conforme o teste anterior.
- Arranque da console offscreen durante quatro segundos: passou.
- FrameCodecTest, LogDataTest e GraphLogWidgetTest: passaram.
- AutoConnectionTest: passou com portas e amostras sintéticas. Inclui descoberta seletiva, intervalo entre novas tentativas, mudança do número COM, reinserção USB, suspensão DFU, identidade, USB sem alimentação externa, alimentação válida, queda de tensão, perda e regresso da alimentação, ADC inválido, saturação e tempos de confirmação.
- O novo teste integra o CTest do workflow Windows existente.
- Ainda falta validação física na Proteus e confirmação dos limiares de alimentação em funcionamento real. Os testes simulados não demonstram o funcionamento físico da ignição.

Os redimensionamentos das tabelas não foram alterados.
