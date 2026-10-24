# Workflows Tunuino

A partir do [commit 4b61dd8](https://github.com/Tunuino/Tunuino/commit/4b61dd8a1556f884708809920049dc4dc561049b), o repositório tem sete ficheiros de workflow, separados por responsabilidade. O antigo `ci.yml` foi removido.

| Ficheiro | Função |
|---|---|
| [check-core.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/check-core.yml) | Testes C/Java/KiCad com GCC e Clang, incluindo protocolos Android e TunerStudio |
| [build-firmware.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/build-firmware.yml) | Bibliotecas M4/M7 e imagem USB de bancada Proteus v0.7 F427 |
| [build-console.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/build-console.yml) | Console Windows, Java incluído e verificação do executável distribuído |
| [validate-tunerstudio.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/validate-tunerstudio.yml) | Verifica o INI contra respostas reais do código C |
| [build-android.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/build-android.yml) | APK de teste, testes unitários, Android lint e manifesto com SHA-256 |
| [build-bundle.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/build-bundle.yml) | Seleciona alterações, chama os workflows reutilizáveis e monta o kit sincronizado |
| [publish-release.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/publish-release.yml) | Publica apenas o Bundle ZIP, depois das verificações |

## Execução

**Tunuino Bundle** corre nos pushes relevantes para `master`, nos pull requests, manualmente e diariamente às 02:20 UTC (04:20 em Berlim no horário de verão; 03:20 no horário de inverno). Os testes, firmware e Console podem correr em paralelo; uma falha impede a montagem/publicação. As alterações só em testes não recompilam os pacotes. Documentação não dispara builds.

O workflow do Bundle chama os workflows dos componentes, sem duplicar as suas definições. **Core checks**, **Firmware**, **Windows Console** e **TunerStudio** também podem ser executados individualmente por **Run workflow**. Estas execuções individuais produzem resultados/artefactos, sem publicar uma Release parcial.

**Android** tem execução independente, automática para alterações Android/protocolo de identificação e manual. Uma alteração apenas na aplicação Android não recompila a Console Windows nem o firmware. O build usa JDK 17, Gradle 8.11.1 e SDK 35; Gradle Wrapper e distribuição têm integridade verificada. A aplicação funciona offline depois de instalada.

Mantêm-se a cache da toolchain ARM xPack e das dependências Maven; o Android usa a cache Gradle. Uma nova execução da mesma referência cancela a anterior do mesmo workflow.

## Sincronização e downloads

Uma alteração que exija distribuir firmware ou Console constrói os dois a partir do mesmo commit. A montagem usa exclusivamente os artefactos da execução atual, compara os commits e verifica hashes, assinatura e layout do INI. Não reutiliza um componente de uma execução antiga.

A Release continua a ter um único anexo: `tunuino_bundle_proteus_v07_f427_usb_bench.zip`. Contém `console/`, `firmware/`, o INI, fontes/licenças, instruções e checksums. O INI no código-fonte passou para [`tunerstudio/`](https://github.com/Tunuino/Tunuino/tree/master/tunerstudio); dentro do Bundle mantém-se em `firmware/`, junto da imagem correspondente.

O APK Android experimental é um artefacto da execução **Android**, chamado `android-debug-<SHA>`, com `tunuino-android-debug.apk` e `package-manifest.json`. Não é acrescentado como outro ZIP à Release. A assinatura debug serve para testes; ainda não existe uma chave de distribuição Android estável.

## Verificação desta revisão

A suite local C/Java/KiCad passou, incluindo os novos testes do cliente Android contra uma resposta do firmware C, CRC, fragmentação, UID/capacidades inválidos, ausência de resposta e cancelamento. Também foram verificados YAML, chamadas reutilizáveis, dependências, scripts shell e seleção de componentes.

O [build completo do Bundle](https://github.com/Tunuino/Tunuino/actions/runs/40121598035) passou com GCC/Clang, firmware ARM, Console Windows, verificação do runtime e montagem sincronizada. A [Release](https://github.com/Tunuino/Tunuino/releases/tag/nightly-2026-10-24-40121598035) foi publicada com um único anexo Bundle ZIP. Estes builds não fazem ensaios numa ECU física.

O [build Android 23310c6](https://github.com/Tunuino/Tunuino/actions/runs/40121722278) compilou o APK e passou os testes unitários e Android lint. [Descarregar o APK de teste e manifesto](https://github.com/Tunuino/Tunuino/actions/runs/40121722278/artifacts/12571491778). O registo do recetor USB usa ContextCompat para compatibilidade entre as versões suportadas; esta alteração apenas Android não disparou a compilação de firmware/Console.

O arranque e a identificação USB do firmware anterior 0.0.2 na Proteus F427 foram confirmados pelo utilizador em 22 de outubro. A ligação Android, o TunerStudio e o firmware 0.0.3 continuam a exigir validação física. O firmware distribuído é de bancada, sem controlo de motor, sensores físicos ou mapas.
