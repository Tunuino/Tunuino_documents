# Tunuino para Android

**[Descarregar o APK de teste e manifesto](https://github.com/Tunuino/Tunuino/actions/runs/40121722278/artifacts/12571491778)** · build **23310c6**.

Código em [`android/`](https://github.com/Tunuino/Tunuino/tree/master/android). Aplicação offline para Android 8 ou superior, com USB Host/OTG. Identifica a ECU através do mesmo protocolo de identificação Tunuino usado pela Console.

## O que está implementado

- Procura dispositivos USB CDC do firmware de bancada e permite selecionar a ECU quando existe mais de uma.
- Pede a autorização USB do Android e configura CDC a 115200 baud, 8N1, DTR/RTS.
- Apresenta família, série STM32, revisão, versão, capacidades e UID recebidos da ECU.
- Reconhece o STM32 em modo DFU e indica como regressar ao arranque normal.
- Faz comunicação fora da interface gráfica, valida tamanho/CRC, limita o pedido a dois segundos e encerra a ligação após erro, remoção do cabo ou saída da aplicação.
- Usa o logótipo Tunuino e mostra o commit do build.

A aplicação não pede permissões de internet ou armazenamento. Os IDs USB de desenvolvimento suportados são `CAFE:4001` para a aplicação e `0483:DF11` para o STM32 DFU. Um dispositivo só é apresentado como ECU identificada depois de responder com identidade válida.

## Instalar e ligar

1. Abrir a [execução Android](https://github.com/Tunuino/Tunuino/actions/workflows/build-android.yml), escolher uma execução concluída com sucesso e descarregar o artefacto `android-debug-<SHA>`.
2. Extrair e instalar `tunuino-android-debug.apk`. É um APK assinado com chave de teste, não uma publicação na Play Store.
3. Usar o firmware Tunuino Proteus v0.7 F427 de bancada e iniciar a ECU com BOOT0 normal.
4. Ligar o telefone à ECU com um cabo/adaptador USB OTG e manter a alimentação de bancada da ECU.
5. Abrir Tunuino, usar **Procurar dispositivos**, selecionar o USB CDC e tocar em **Ligar à ECU**. Autorizar o acesso USB quando o Android o pedir.
6. Confirmar a identificação recebida. **Ler identificação** repete a leitura; **Desligar** encerra a ligação.

Se aparecer **STM32 DFU**, a aplicação não consegue consultar a identificação da aplicação. Repor BOOT0 e reiniciar a ECU depois de gravar o firmware. Fechar outra aplicação que esteja a usar a mesma ligação USB.

A assinatura debug pode mudar entre runners, pelo que uma atualização poderá exigir desinstalar o APK anterior. A aplicação não guarda calibrações nem dados pessoais.

## Limites e validação

Esta versão é de identificação e leitura. Não grava firmware por DFU, não envia mapas, não controla injeção/ignição e não apresenta sensores que o firmware de bancada não fornece. Não equivale às funcionalidades completas da Console Windows.

Os testes locais comparam o cliente Android com a resposta do código C do firmware e ensaiam fragmentação, CRC incorreto, comprimentos inválidos, UID, família, capacidades, ausência de resposta e cancelamento. A [execução Android 40121722278](https://github.com/Tunuino/Tunuino/actions/runs/40121722278) compilou o APK e passou os testes unitários e Android lint. A ligação USB OTG, permissões/lifecycle num telefone real e a comunicação com a ECU ainda precisam de teste físico.

Compilação local: abrir `android/` no Android Studio ou usar JDK 17, SDK 35 e `./gradlew assembleDebug testDebugUnitTest lintDebug`. A primeira compilação resolve dependências pela rede; a aplicação instalada funciona offline.
