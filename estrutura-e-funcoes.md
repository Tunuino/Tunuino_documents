# Estrutura e funções Tunuino

Comparação das responsabilidades das pastas da imagem com a implementação Tunuino. A estrutura organiza código executável, testes, geradores e ferramentas; nomes semelhantes não significam a mesma cobertura funcional.

| Pasta | Situação anterior | Conteúdo funcional atual |
|---|---|---|
| `.vscode/` | Sem configuração publicada | Tarefas de testes, geração de INI e build do simulador; perfil GDB para o executável nativo |
| `android/` | Aplicação USB já implementada | Mantida: identificação offline por USB OTG e reconhecimento de DFU; APK e lint em workflow próprio |
| `bin/` | Comandos dispersos em scripts | Entrada Linux/Windows para testes, build do simulador, geração, diagnóstico, ferramentas Java e Console |
| `docs/` | Contratos técnicos | Mantidos junto do código; instruções de utilização e resultados ficam neste repositório de documentação |
| `firmware/` | Núcleo C e imagem USB F427 | Mantidos os handlers e algoritmos existentes; definições TunerStudio integradas no firmware |
| `gradle/` | Wrapper apenas no projeto Android | Wrapper na raiz com checksum, build dos módulos Java, distribuição das ferramentas e tarefas de verificação |
| `hardware/` | Relatórios/importação KiCad | Mantidos com a sua proveniência; a reorganização não cria uma nova pinagem validada |
| `java_console/` | Chamava-se `console/` | Código real da Console movido; Maven/workflow Windows atualizados e build Gradle acrescentado; novo transporte local por processo |
| `java_tools/` | Ausente | Conversão binário→cabeçalho C, resumo de logs, importação/exportação de mapas CSV e cliente do simulador com geração de datalogs |
| `misc/` | Auxiliares em `tools/` | Diagnóstico de ambiente, empacotador offline de programadores usado pelo build Windows e verificação dos ficheiros gerados |
| `simulator/` | Apenas peer usado pelos testes | Executável próprio que usa o núcleo C: capturas 36-1, RPM calculadas, sensores sintéticos, mapas, protocolos e cenários de falha/recuperação |
| `unit_tests/` | Testes C em `firmware/tests/` | Testes, stubs e peer movidos; runner/Makefile, verificações do gerador e seleção CI, testes das ferramentas e do simulador |
| `firmware/tunerstudio/` | INI colocado incorretamente na raiz | Esquema, template, gauges, menu, gerador e `generated/`; INI e assinatura C gerados em conjunto |

Os testes Java continuam nos `src/test` dos módulos, como é normal no build Java. A pasta `unit_tests/` reúne a suite nativa e os auxiliares de integração; a mudança de nome não acrescenta testes de funcionalidades de motor inexistentes.

## Compilar e testar

Requisitos: Python 3, compilador C11 e Java 17 ou superior para a suite portátil. O build Gradle dos módulos desktop usa JDK 21; o Android mantém JDK 17 e SDK 35.

```sh
python3 unit_tests/run.py
python3 tools/build_simulator.py
./gradlew :java_console:assemble :java_tools:installDist :java_tools:check
python3 bin/tunuino.py tools --help
```

No Windows, usar `python`, `gradlew.bat` e o executável `tunuino-simulator.exe`. A primeira resolução Gradle requer rede; a aplicação Console e as ferramentas operam localmente depois de compiladas.

```sh
python3 bin/tunuino.py tools simulate build/simulator/tunuino-simulator simulated-demo.csv 30 proteus normal
python3 bin/tunuino.py tools log-summary simulated-demo.csv
```

O comando `simulate` exige nomes `simulated-*` e cria um manifesto `.provenance.json` com `synthetic=true`. Não se liga a uma ECU. São suportados `normal`, `stall`, `sensor-loss` e `wrap`; o relógio da simulação avança em passos virtuais de 100 ms por pedido de telemetria, sem prometer tempo real de parede.

As ferramentas também disponibilizam:

```text
bin2header input.bin output.h symbol
map-export input.tnmap output.csv
map-import input.csv output.tnmap
```

A conversão de mapas preserva eixos, valores e tipo VE/ignição, recusa casas decimais que exijam arredondamento e não substitui ficheiros existentes. `bin2header` limita o tamanho do binário e valida o símbolo C.

O simulador permite carregar mapas locais com `--ve ficheiro.tnmap` e `--ignition ficheiro.tnmap`; o protocolo de leitura devolve esses mapas. Os mapas que traz por defeito são exemplos sintéticos para desenvolvimento, sem recomendação de afinação. A opção `--bench` usa o perfil de identificação TunerStudio da Proteus/F427; os sensores simulados continuam a usar o protocolo nativo.

## Definições TunerStudio

```sh
python3 firmware/tunerstudio/generate.py
python3 firmware/tunerstudio/generate.py --check
```

`schema.json`, `tunerstudio.template.ini`, `gauge_declarations.ini`, `main_page_gauges.ini` e `top_level_menu.ini` geram o INI em `generated/` e o header de assinatura do firmware. A verificação recusa definições desatualizadas, canais sobrepostos/fora do bloco e gauges inválidos. O build confirma a assinatura no binário e a montagem do Bundle verifica hashes/layout.

## Verificação e limites

A suite local passou: testes C/Java/KiCad existentes, gerador, interoperabilidade com o executável C real, leitura de mapas completos, mapas carregados de ficheiro, datalogs, recuperação de paragem/perda de sensor, volta do contador e timeout/fecho do processo. A [execução 40123914479](https://github.com/Tunuino/Tunuino/actions/runs/40123914479) passou em Gradle, GCC/Clang, ARM, Console/runtime Windows e Bundle; o [Android 40123809702](https://github.com/Tunuino/Tunuino/actions/runs/40123809702) também passou.

O simulador e as ferramentas não acrescentam aquisição física de sensores, comandos de injeção/ignição, CAN, Lua, Autotune ou imagens uaEFI/F767 ao firmware da ECU. Esses componentes continuam por implementar/integrar e validar; esta revisão não equivale a um firmware de motor completo. A imagem F427 distribuída continua de identificação USB em bancada.
