# Instalação da Tunuino Console no Windows

O instalador Windows x64 é a descarga principal. O ficheiro `tunuino_setup_windows_x64.msi` instala a Console com Java incluído e os ficheiros de firmware Proteus v0.7 / STM32F427 e INI TunerStudio do mesmo commit.

1. Fechar uma Console já aberta e executar o MSI.
2. Escolher a pasta no assistente e concluir a instalação para o utilizador atual.
3. Abrir TunuinoConsole pelo menu Iniciar ou pelo atalho no ambiente de trabalho.

O firmware, os manifestos de gravação, o INI correspondente, o código-fonte e as licenças encontram-se na pasta `firmware` junto da aplicação. O seletor de firmware da Console começa nessa pasta mesmo quando a aplicação é aberta por um atalho com outra pasta de trabalho. A instalação não grava a ECU nem instala drivers ou ferramentas de terceiros automaticamente. As ferramentas de programação continuam configuráveis na Console.

## Dados e atualizações

Os diálogos da Console propõem estas pastas do utilizador:

| Conteúdo | Pasta predefinida |
|---|---|
| Mapas `.tnmap` | `%LOCALAPPDATA%\TunuinoConsole\maps` |
| Configurações `.tncfg` | `%LOCALAPPDATA%\TunuinoConsole\configurations` |
| Datalogs `.csv` | `%LOCALAPPDATA%\TunuinoConsole\logs` |
| Caminhos das ferramentas de programação | `%LOCALAPPDATA%\TunuinoConsole\programmers.properties` |

Continuam a poder ser escolhidas outras pastas. Os ficheiros antigos não são movidos nem eliminados. Os caminhos de programadores antes guardados em `%USERPROFILE%\.tunuino\programmers.properties` continuam legíveis; uma nova seleção passa a ser guardada na pasta atual.

Para atualizar, fechar a Console, descarregar e instalar o novo MSI. A identidade de atualização do Windows é fixa; a versão cresce com o número da execução do workflow Bundle. A atualização substitui a aplicação e os seus componentes. Os dados do utilizador ficam fora da pasta instalada e são preservados. Não existe atualização automática nesta implementação.

Para desinstalar, usar **Definições → Aplicações → TunuinoConsole → Desinstalar**. Os mapas, configurações e datalogs são preservados. O instalador ainda não tem assinatura digital de editor.

## ZIP durante os testes

O ZIP `tunuino_bundle_proteus_v07_f427_usb_bench.zip` continua disponível como alternativa portátil apenas durante a fase de testes. Só será retirado dos novos lançamentos quando o utilizador confirmar que os testes passaram. Não são eliminados os ZIP já publicados.

Ambos os formatos levam a Console, o firmware e o INI produzidos a partir do mesmo commit. O instalador contém também os avisos de licença e o código-fonte correspondente ao firmware.

## Verificação

A publicação exige compilações e testes dos componentes no mesmo commit. O workflow Bundle executa no Windows e só entrega o MSI depois de verificar:

- instalação de uma versão com número inferior e atualização para o MSI a publicar;
- instalação limpa do MSI a publicar;
- ficheiros instalados por SHA-256, atalhos e uma única entrada da versão atual no Windows;
- JavaFX, portas série e consulta DFU com o Java instalado, a partir de outra pasta de trabalho;
- descoberta e validação do firmware incluído;
- desinstalação e preservação de ficheiros de mapas, configurações, datalogs e definições de ferramentas.

O teste de atualização usa o mesmo conteúdo com dois números de versão para exercitar o mecanismo MSI. Os ensaios automáticos de empacotamento não verificam uma ECU física, a precisão elétrica das saídas nem o funcionamento num motor. As limitações do firmware experimental mantêm-se descritas em [arranque de bancada](proteus-v07-bringup.md).
