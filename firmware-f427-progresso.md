# Firmware Tunuino STM32F427 — progresso e próximos blocos

Atualizado: **25 de outubro de 2026**. Alvo atual: **Proteus v0.7 / STM32F427**, firmware IO **0.1.0**. A versão 0.1.0 identifica a família de firmware, não cada revisão de código: para identificar um pacote concreto, verificar também o SHA do Release.

## Funcionalidades no código

| Subsistema | Situação |
|---|---|
| Arranque STM32F427, ChibiOS e USB CDC | Implementado; ligação COM e identificação pela Tunuino Console confirmadas no hardware pelo utilizador |
| Identidade, assinatura Tunuino e CRC32 | Implementados; respostas `F`, `S` e `A` verificadas diretamente pela porta COM |
| Sensores analógicos | ADC1 com DMA, 16 canais e máscaras de validade; precisão elétrica ainda por validar no veículo |
| Cambota e cames | Captura EXTI e descodificação da roda com dentes em falta, com sincronização e deteção de falhas; referência real ao PMS pendente |
| Curvas MAP, lambda e temperatura | Conversão e validação de calibrações; a configuração padrão mantém sensores desativados |
| Mapas VE e ignição | **16×16 nativos** no firmware IO 0.2.0; validação, interpolação, leitura e escrita pela Console; mapas padrão não são afinação de motor |
| Persistência | **Schema 2**, registos de 1.440 bytes com CRC e proteção contra gravações interrompidas; versão antiga não é recuperada automaticamente |
| Diagnóstico de comunicação | Reabertura de USB após erros terminais, controlo de timeouts, testes de modelos USB |
| Leitura de indicadores TunerStudio | Comando `A` de 100 bytes preservado; acrescentado `O` com offset e comprimento, mantendo `ochBlockSize = 100` |
| Pré-visualização de mapas | Acrescentado comando nativo `TUNUINO:E`: cálculos VE/avanço, indicador de validade, motivos de indisponibilidade, RPM/MAP e geração da configuração, **sem controlo de atuadores** |
| Modelo temporal de motor (bancada) | Módulo C portátil `engine_math` para microssegundos por rotação, ciclo 720° e dente, desfasamento do avanço relativo a uma referência **hipotética** e índice de carga relativo; sem massa de combustível, dwell, PMS ou atuação |
| Consulta de tempos na Console | Novo botão de cálculo teórico na área Mapas, leitura por `TUNUINO:T`, descodificação estrita e explicação dos bloqueios de validade |
| Preparação 16×16 (sem gravação física) | Conversores independentes C/Java para refinar mapas 8×8 em 16×16, conservando eixos/células originais e calculando células adicionais; testes cruzados com CRC |
| Editor de mapas 16×16 | Botão de conversão no editor, nova janela local, visualização e exportação `.tnmap`; carregamento para ECU 8×8 deliberadamente bloqueado |
| Console Windows — cálculo de bancada | Painel Mapas com botão de pré-visualização, valores VE/ignição quando válidos e identificação em português das condições impeditivas; pedido USB exclusivo em segundo plano |
| Console Windows — leitura por intervalos | Cliente do comando `O` com limites, contagem exata e rejeição de respostas malformadas |
| TunerStudio Windows | O INI já abre e o protocolo responde corretamente a testes diretos. A ligação total no TunerStudio continua por confirmar; anteriormente aparecia `Unsupported Controller Firmware` |

Código da etapa de pré-visualização e leitura por blocos:
[commit `f04c7a3`](https://github.com/Tunuino/Tunuino/commit/f04c7a3721c40a32471ca4009fc6bd67c3aa6e87).

Ligação à Console (cliente Java, UI e testes):
[commit `b8f2e24`](https://github.com/Tunuino/Tunuino/commit/b8f2e24c5c4e2dd123e9285b5918d71e1be3a8c7).
A integração mantém o botão desativado sem uma ECU F427 IO identificada ou durante outra operação USB.

O modelo matemático de bancada foi implementado no [commit `9fb7bbd`](https://github.com/Tunuino/Tunuino/commit/9fb7bbd613ab81243104ac139254e94ddef5b263), com leitura e interface na Console no [commit `f565f439`](https://github.com/Tunuino/Tunuino/commit/f565f439bb914b1852bacc50e366653fa78c0eef). O teste completo requer o Bundle correspondente: a imagem F427 IO anterior, embora identifique a mesma versão 0.1.0, não implementa `TUNUINO:T` e devolve `0x83`. A Console mostra a indisponibilidade sem fechar uma ligação USB funcional.
O botão não escreve calibrações e não atua sobre o motor. A validação física da nova UI permanece pendente.

## Conversão gradual dos mapas 16×16

A etapa de preparação implementa [conversão matemática C/Java](https://github.com/Tunuino/Tunuino/commit/538110e8a7f5e73c5bc5142b0c1a62457cb64a5f) e
[botão de conversão da Console](https://github.com/Tunuino/Tunuino/commit/452e4a12ca12053d6fd384ecf0c184b4305da12d).
A conversão preserva os 64 valores originais nas interseções dos eixos e usa interpolação bilinear nas novas células, arredondando a três casas.
Também valida valores negativos de avanço na implementação C e Java.

**Atualização de 26 de outubro de 2026:** o firmware IO 0.2.0 passou a utilizar **exclusivamente mapas VE e ignição 16×16 nativos**. As páginas 2 e 3 têm agora 576 bytes cada, a configuração completa tem 1.408 bytes, o jornal de flash utiliza registos de 1.440 bytes e o schema foi atualizado para 2. A assinatura mudou para `Tunuino Proteus v0.7 F427 IO v2` e o INI para a revisão 5. A Tunuino Console lê, edita, envia e verifica mapas 16×16. O formato 8×8 não é suportado na ECU nova; apenas a ferramenta de importação local permite converter um ficheiro antigo numa direção.

**Atenção antes da gravação:** a versão 0.2.0 não interpreta automaticamente os registos de flash gravados no formato anterior. Quando não encontra um registo do schema 2, arranca com sensores desativados e mapas 16×16 a zero. Guardar previamente mapas e calibrações da ECU atual e manter os atuadores desligados. A funcionalidade não constitui controlo de motor funcional.

Implementação atual na `master`: [novo formato de firmware](https://github.com/Tunuino/Tunuino/commit/93af3b05f1dd9be5cca9b6d312736d3f17d4ccb3), [Console e INI 16×16](https://github.com/Tunuino/Tunuino/commit/9a6a0d8dc439f627b2c66b3c7d6c2465649a04bf).

## Funcionalidades que faltam para gestão completa do motor

1. **Confirmação do hardware:** mapa elétrico de saídas (injetores, bobinas, relés, bomba, marcha lenta, ventoinha), polaridade, estados de reset e isolamento dos drivers.
2. **Posição do motor:** referência PMS, fase de cames, escolha de roda dentada e tolerância de ruído/variação de velocidade com sensores reais.
3. **Agendamento de injeção e ignição:** modelo de cilindros e ordem de ignição, temporizadores dedicados, dwell, tempos de abertura, modos sequencial e semi-sequencial; inicialmente em simulador e com saídas desativadas.
4. **Gestão do combustível:** cálculo de carga, correções de temperatura, tensão de bateria, aceleração, arranque e aquecimento; limites de AFR e compensações com verificação de calibração.
5. **Proteções de funcionamento:** watchdog, corte de RPM, perda de sincronização, leitura de sensores inválida, brown-out, falhas de atuadores e desligamento seguro, com testes negativos.
6. **Comunicações e ferramentas:** concluir reconhecimento de família no TunerStudio, painéis operacionais da Console, CAN/diagnósticos e datalogging completo.
7. **Expansões opcionais:** hardware F767, Lua e funções de maior memória somente em perfis próprios, sem as tornar dependências da imagem F427.

## Ensaios e limites de segurança

Os testes GitHub Actions verificam compilação, coerência do INI e vetores binários em modelos de bancada. **Não representam certificação de funcionamento com motor nem verificam uma ECU montada no veículo.**

A imagem atual **não comanda injetores, ignição ou bomba**. Para ensaios de desenvolvimento, manter motor e atuadores desligados e guardar uma cópia do firmware/parametrização que já funciona. Não utilizar dados VE/ignição sintéticos como mapa de afinação.

O novo comando `TUNUINO:E` só devolve valores interpolados se houver RPM e MAP válidos, captura sincronizada, ADC válido, parametrização e mapas no intervalo. Mesmo quando o resultado é válido, isso **não autoriza nem provoca atuação**. O ficheiro INI é revisto junto do firmware no Bundle do mesmo commit; o protocolo da Console continua independente.

[Documentação técnica do firmware](https://github.com/Tunuino/Tunuino/blob/master/docs/proteus-io.md) ·
[Releases Tunuino](https://github.com/Tunuino/Tunuino/releases) ·
[Workflows](https://github.com/Tunuino/Tunuino/actions).
