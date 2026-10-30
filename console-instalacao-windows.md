# Instalação da Tunuino Console no Windows

A aplicação atual em desenvolvimento utiliza **Qt 6**. O workflow Windows produz um instalador executável e uma distribuição portátil. A migração do Bundle principal para Qt ainda está em curso; não confundir o instalador Qt com os pacotes MSI da consola anterior.

## Instalação

1. Obter o instalador Qt do workflow Windows validado.
2. Fechar qualquer instância da consola antes de executar o instalador.
3. Seguir o assistente de instalação e abrir a Tunuino Console.
4. Para utilização portátil, extrair integralmente a distribuição e iniciar `TunuinoConsole.exe` na pasta extraída.

A instalação da aplicação não grava automaticamente o firmware na ECU, nem instala necessariamente drivers USB ou programadores de terceiros.

## Firmware e USB

Verificar o modelo da placa e a imagem F427 antes de gravar. A gravação USB/DFU exige que a ECU esteja no modo correto e que as ferramentas e drivers necessários estejam disponíveis. Consultar [USB e DFU](console-firmware-usb.md) e [arranque de bancada](proteus-v07-bringup.md).

O Bundle Qt e firmware encontra-se em fase de integração e validação. O nome previsto é `Tunuino-F427-Bundle.zip`. A presença de um artefacto compilado não comprova funcionamento na ECU física.

## Atualização e dados

Antes de atualizar, guardar uma cópia dos mapas e configurações. Fechar a consola e instalar a versão nova; não substituir ficheiros enquanto a aplicação está em execução. Confirmar que os dados do utilizador continuam acessíveis após a atualização.

## Validação

A compilação e os testes automáticos verificam apenas as condições cobertas pelo workflow. Permanecem necessários ensaios físicos de USB, leitura de sensores, gravação de firmware, sincronismo e segurança das saídas antes de utilizar a ECU num motor.
