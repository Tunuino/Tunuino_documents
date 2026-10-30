# Workflows Tunuino

Os workflows estão organizados por responsabilidade. A lista abaixo descreve os pipelines existentes na estrutura documentada; confirmar os ficheiros atuais no repositório antes de executar. A estrutura dos módulos e os comandos estão em [estrutura e funções](estrutura-e-funcoes.md).

| Ficheiro | Função |
|---|---|
| [build-unit-tests.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/build-unit-tests.yml) | Testes C/Java/KiCad com GCC e Clang, incluindo protocolos Android e TunerStudio |
| [build-firmware.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/build-firmware.yml) | Bibliotecas M4/M7 e imagem USB de bancada Proteus v0.7 F427 |
| [build-console.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/build-console.yml) | Console Windows, Java incluído e verificação do executável distribuído |
| [validate-tunerstudio.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/validate-tunerstudio.yml) | Verifica o INI contra respostas reais do código C |
| [build-desktop.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/build-desktop.yml) | Compila os módulos Gradle e verifica as ferramentas Java contra o simulador C real |
| [build-android.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/build-android.yml) | APK de teste, testes unitários, Android lint e manifesto com SHA-256 |
| [build-bundle.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/build-bundle.yml) | Seleciona alterações, chama os workflows reutilizáveis e monta o kit sincronizado |
| [publish-release.yml](https://github.com/Tunuino/Tunuino/blob/master/.github/workflows/publish-release.yml) | Publica apenas o Bundle ZIP, depois das verificações |

## Execução

**Tunuino Bundle** corre nos pushes relevantes para `master`, nos pull requests, manualmente e diariamente às 02:20 UTC (04:20 em Berlim no horário de verão; 03:20 no horário de inverno). Os testes, firmware e Console podem correr em paralelo; uma falha impede a montagem/publicação. As alterações só em testes não recompilam os pacotes. Documentação não dispara builds.

O workflow do Bundle chama os workflows dos componentes, sem duplicar as suas definições. **Unit tests**, **Firmware**, **Windows Console** e **TunerStudio** também podem ser executados individualmente por **Run workflow**. Estas execuções individuais produzem resultados/artefactos, sem publicar uma Release parcial.

**Android** tem execução independente, automática para alterações Android/protocolo de identificação e manual. Uma alteração apenas na aplicação Android não recompila a Console Windows nem o firmware. O build usa JDK 17, Gradle 8.11.1 e SDK 35; Gradle Wrapper e distribuição têm integridade verificada. A aplicação funciona offline depois de instalada.

Mantêm-se a cache da toolchain ARM xPack e das dependências Maven; Android e desktop usam a cache Gradle. As alterações ao empacotador de programadores produzem um novo par Console/firmware. Alterações só em testes, ferramentas Java ou simulador ficam limitadas aos testes/builds desktop. Uma nova execução da mesma referência cancela a anterior do mesmo workflow.

## Sincronização e downloads

Uma distribuição conjunta deve utilizar firmware e Console provenientes da mesma revisão de código. A montagem usa exclusivamente os artefactos da execução atual, compara as revisões e verifica hashes, assinatura e layout do INI. Não reutiliza um componente de uma execução antiga.

O nome previsto para o Bundle F427 é `Tunuino-F427-Bundle.zip`. A publicação de um único anexo deve ser confirmada no workflow de Release. Contém `console/`, `firmware/`, o INI, fontes/licenças, instruções e checksums. O INI no código-fonte está em [`firmware/tunerstudio/generated/`](https://github.com/Tunuino/Tunuino/tree/master/firmware/tunerstudio/generated); dentro do Bundle mantém-se em `firmware/`, junto da imagem correspondente.

O APK Android experimental é um artefacto da execução **Android**, chamado `android-debug-<SHA>`, com `tunuino-android-debug.apk` e `package-manifest.json`. Não é acrescentado como outro ZIP à Release. A assinatura debug serve para testes; ainda não existe uma chave de distribuição Android estável.

## Verificação e limitações

Confirmar o estado de cada execução em [GitHub Actions](https://github.com/Tunuino/Tunuino/actions) e os anexos em [Releases](https://github.com/Tunuino/Tunuino/releases). Uma execução antiga bem-sucedida não comprova o estado atual do código nem a criação do Bundle Qt.

Os testes automáticos podem utilizar dados sintéticos e transportes simulados. A identificação USB numa Proteus F427 não valida, por si só, o funcionamento do motor, os sensores físicos ou a compatibilidade com TunerStudio e Android.
