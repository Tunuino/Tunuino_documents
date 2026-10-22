# Workflows automáticos

Implementados em Tunuino/Tunuino no commit 4b98cba8d2f35194cbfbd5b796cfbd11cef0821a.

## Execução

Os dois workflows arrancam em push, pull request e manualmente no separador Actions.

- Core checks: executa tools/check_core.py em Ubuntu 24.04, com GCC e Clang e Java 21. Inclui verificações de ferramentas KiCad, testes C, testes Java e integração simulada C → Java → CSV.
- Windows console build: executa Maven verify no Windows Server 2022 com Java 21, incluindo a compilação da consola JavaFX e do código de comunicação série.

Permissões limitadas à leitura do repositório, ações fixadas por SHA e duração máxima de 15 minutos por job. Execuções antigas da mesma referência são canceladas quando chegam alterações novas.

## Validação e limites

A suite tools/check_core.py passou localmente com GCC e Java 17 (o núcleo portátil usa --release 17; a consola usa Java 21). O ambiente local não dispõe de Clang, Maven ou Java 21; as verificações correspondentes dependem dos runners do GitHub. As primeiras execuções no GitHub concluíram com sucesso: GCC e Clang com Java 21 no Ubuntu, e Maven verify com Java 21 no Windows.

Maven verify compila e empacota a consola; os testes de comportamento existentes são executados pelo workflow Core checks, porque usam programas de verificação próprios e não JUnit.

Estes workflows não validam a interface gráfica em execução, USB físico, aquisição STM32, firmware gravável nas ECUs ou funcionamento de um motor. Não publicam releases nem instaladores. A validação física em Proteus e uaEFI continua pendente.

Primeiras execuções confirmadas:
- Core checks (GCC e Clang): https://github.com/Tunuino/Tunuino/actions/runs/39771980258
- Windows console build: https://github.com/Tunuino/Tunuino/actions/runs/39771979821

Consultar as execuções: https://github.com/Tunuino/Tunuino/actions

O README principal mantém apenas a descrição curta do projeto.
