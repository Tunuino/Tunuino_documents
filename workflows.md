# Integração contínua da Tunuino

Implementação: [commit beda9d0](https://github.com/Tunuino/Tunuino/commit/beda9d056e637e61230652d6f6f9e33b5b9160b4).

A definição ativa passou dos quatro workflows independentes para uma única pipeline: [Tunuino CI](https://github.com/Tunuino/Tunuino/actions/workflows/ci.yml). As execuções históricas e os artefactos anteriores continuam no GitHub.

## Problemas observados

Na execução antiga de bancada 39792846260, o passo de instalação ARM demorou **1193 segundos (19 min 53 s)** e o job acabou cancelado. A compilação/ligação em si demorou 4 segundos nessa execução; na imagem de bancada aprovada anterior, demorou 3 segundos. O problema medido principal era a instalação da toolchain, não o custo de compilar o firmware.

A mesma instalação apt era repetida em três jobs: biblioteca Cortex-M4, biblioteca Cortex-M7 e imagem de bancada. Todos os workflows arrancavam a cada alteração, mesmo quando os respetivos componentes não tinham mudado. Os jobs de empacotamento também não dependiam do resultado dos testes.

Tempos observados antes da alteração, pelos passos dos jobs:

| Trabalho | Tempo observado |
|---|---:|
| Testes GCC na última execução concluída | 14 s |
| Testes Clang na última execução concluída | 15 s |
| Compilação/ligação de bancada aprovada | 3 s |
| Job Windows completo com pacote e verificação do runtime | 65 s |
| Instalação ARM que ficou presa | 1193 s |

Estes tempos não são uma promessa de duração futura. As filas de runners e os downloads podem variar. Os timestamps globais da execução não medem apenas compilação.

## Ordem da nova pipeline

1. **Select affected components**: determina os componentes alterados.
2. **Core checks (gcc/clang)**: mantém os dois compiladores e a suite completa C/Java/KiCad, incluindo integração até ao CSV e callbacks USB simulados.
3. Depois dos testes aprovados, **ARM** e **Windows Console** correm em paralelo, quando necessários.
4. **CI result** resume todos os resultados. Se ambos os componentes de bancada foram construídos e passaram, descarrega apenas os artefactos desta execução e valida o commit e os hashes antes de publicar o kit completo.

Uma falha ou cancelamento impede a publicação do kit sincronizado. Não há mistura automática de um firmware novo com a Console de uma execução antiga.

## Redução de trabalho repetido

- A toolchain ARM passa a ser uma distribuição fixa xPack GCC 13.2.1-1.1, descarregada com limites de tempo e verificada pelo checksum publicado na mesma release. Não se executa apt update/install nos builds ARM.
- O diretório da toolchain usa cache e uma chave que inclui a versão, sistema, arquitetura e hash do instalador. Num acerto de cache não há novo download.
- Bibliotecas M4/M7 e imagem de bancada partilham uma única instalação no mesmo job.
- Maven usa cache das dependências e executa compilação e cópia das dependências na mesma invocação.
- Alterações apenas em testes não voltam a gerar firmware nem a Console portátil.
- Alterações de código C não recompilam automaticamente a Console; alterações da Console não voltam a instalar a toolchain ARM.
- README e documentação não disparam builds. Alterações na configuração CI ou licença validam todos os componentes.
- Push é executado na master; pull requests para master são validados antes de integrar. Isto evita o par push/PR duplicado para a mesma branch de trabalho.
- Uma nova execução na mesma referência/PR cancela a execução antiga da pipeline completa.

## Kits e artefactos

Os nomes dos artefactos incluem o SHA completo do commit. Quando firmware e Console são construídos juntos, é publicado **Tunuino-Proteus-v07-USB-Bench-Kit-<SHA>**, contendo as duas pastas e um manifesto comum.

A montagem verifica que o source_commit do manifesto do firmware e do manifesto da Console coincide com GITHUB_SHA. Também verifica os hashes das imagens e do JAR principal da Console. Um commit misturado ou ficheiro corrompido é recusado.

Se só um componente mudou, é publicado apenas o componente afetado, depois dos testes. Para obter um kit completo de um commit, executar **Tunuino CI → Run workflow**: a execução manual seleciona todos os componentes.

Os pacotes de bancada continuam experimentais e sem validação física. As bibliotecas ARM conservam NOT-FLASHABLE no nome. Não foram acrescentados deploy, publicação de releases ou gravação automática numa ECU.

## Verificação desta alteração

A sintaxe Python, shell e YAML foi verificada localmente, assim como a seleção de alterações de documentação, testes, código C, Console e placa. A montagem do kit foi exercitada com um pacote válido e com três recusas: commits diferentes, firmware corrompido e JAR da Console corrompido.

A [primeira execução real](https://github.com/Tunuino/Tunuino/actions/runs/39798031342) concluiu com **success** e produziu o [kit sincronizado](https://github.com/Tunuino/Tunuino/actions/runs/39798031342/artifacts/12435280193).

| Medida nesta execução | Resultado |
|---|---:|
| Pipeline, desde criação até conclusão | **2 min 16 s** |
| Preparação da toolchain, sem cache anterior | **6 s** |
| Compilar bibliotecas M4 e M7 | **2 s** |
| Compilar/ligar e verificar imagem de bancada | **2 s** |
| GCC/Clang, Console e verificação do kit | Aprovados |

O primeiro download foi verificado pelo checksum publicado. A cache da toolchain e a cache Maven foram guardadas para execuções seguintes; ainda não se mediu uma segunda execução com acerto de cache.

O manifesto comum foi validado para o commit beda9d056e637e61230652d6f6f9e33b5b9160b4 antes do upload. O kit contém firmware e Console desse mesmo commit. A imagem produzida pela distribuição xPack tem 16 412 bytes, com limites e vetores verificados. A diferença de tamanho face à primeira imagem, produzida pela toolchain Ubuntu, não é uma validação funcional adicional.

Os 2 min 16 s são uma observação de uma execução, não um prazo garantido: as filas do GitHub e a rede podem aumentar o tempo total. O antigo atraso de 1193 s foi medido no passo de instalação apt, que deixou de existir nesta pipeline.

O README principal mantém a apresentação curta do projeto.
