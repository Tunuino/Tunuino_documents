# VVT e sinais CAM 1 / CAM 2 na Proteus v0.7

## Configuração

A navegação VVT apresenta VTEC, CAM 1 control, CAM 1 PID, CAM 1 target 1/2 e os quatro blocos equivalentes para CAM 2. Cada controlador tem referência, curso, direção, frequência e limites PWM, PID, deadband, limites integral/correção, ativação por RPM/CLT/tempo, pressão de óleo opcional, overrides e proteção de seguimento independentes.

As ligações físicas são escolhidas **apenas em Inputs and outputs**. CAM 1 utiliza J101/4 (+), 12 (−), entrada condicionada dedicada. CAM 2 utiliza J101/1, entrada digital Hall/óptica com pull-up fixo de 2,7 kΩ. Não dispõe de condicionamento VR interno. Os solenoides CAM 1, CAM 2 e VTEC usam saídas LS distintas, selecionadas explicitamente. Nenhum canal ocupado é retirado a outra função.

Nos controlos VVT não se repete o seletor de saída. Configure abre o bloco correspondente; a gravação continua explícita em Write to ECU, com vermelho reservado a alterações locais relativamente à base lida. A comunicação permanece aberta durante a edição.

Cada came tem dois mapas independentes de 16×16, com RPM em baixo, MAP kPa à esquerda e gráfico 3D ajustável. Seleciona-se explicitamente alvo fixo, tabela 1 ou tabela 2. Ler, abrir um projeto ou mostrar uma janela não ativa controlos nem arma saídas. Os valores iniciais são referências inativas, não calibrações de um motor.

## Aquisição e diagnóstico

Diagnostics separa CAM 1 signal, CAM 1 pulses, CAM 2 signal, CAM 2 pulses e os estados de cada VVT. Pulsos mostram frequência, período e contagem reais; a fase utiliza a posição capturada em relação ao ciclo sincronizado de 720°. Não se converte frequência em fase nem se gera uma forma de onda fictícia.

O sincronismo do motor continua dependente da CAM 1. A CAM 2 mede fase independentemente e qualifica uma aresta selecionada por ciclo de 720°, após duas ocorrências coerentes. Uma aresta adicional invalida a fase da CAM 2 sem alterar o sincronismo da CAM 1. O Scope nativo pode capturar cambota e as duas cames; os ficheiros/capturas antigos de dois canais continuam aceites.

O controlo exige feedback atual, dentro do curso e da janela configurados. Perda de feedback ou alvo, curso excedido, erro persistente de seguimento e perda de pressão de óleo enquanto ativo cortam o respetivo solenoide e retêm a falha. Overrides mantêm as proteções e não dispensam armamento explícito.

## Protocolo, gravação e projetos

| Página | Conteúdo |
|---|---|
| 6 | VTEC e controlo/PID CAM 1 |
| 7 / 8 | Alvos CAM 1, tabelas 1 / 2 |
| 9 | Controlo/PID CAM 2, janela e referência próprias |
| 10 / 11 | Alvos CAM 2, tabelas 1 / 2 |

A página 9 não contém um segundo VTEC. As novas capacidades anunciam VVT CAM 2 e pulsos CAM 1; a Console só solicita as novas páginas quando o firmware as anuncia. Firmware anterior conserva o conjunto de páginas suportado.

As consultas adicionais são TUNUINO:C para pulsos CAM 1, TUNUINO:G para fase CAM 2 e TUNUINO:X/11 para estado VVT CAM 2. Todas incluem proveniência de geração; respostas antigas/alteradas são recusadas.

A escrita verifica motor parado, saídas desarmadas, identidade/geração, base limpa, cópia anterior, preparação e releitura das **11 páginas**, um único Burn e releitura final. Não se repete automaticamente um Burn incerto. Os recursos de injetores, relés, auxiliares, VTEC e ambos os VVT são verificados contra conflitos.

O projeto .tnproj e o comparador conservam os quatro mapas e os dois conjuntos de parâmetros. Projetos anteriores recebem apenas editores CAM 2 locais desativados; não se alteram as páginas antigas ou a cablagem guardada. A persistência nativa passa ao formato 10, com migração dos formatos 2–9 e preservação do banco anterior em escritas interrompidas.

## Referências e limites

Contactos e condicionamento confirmados no KiCad/PCB Proteus v0.7, revisão 383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46. Referência de utilização: fotografias fornecidas e PDF EMU_PRO_Software_Guide.pdf, Appendix D, páginas 113–119, sobre posição relativa à cambota, janela, duty, ativação, PID e mapas interpolados.

A implementação suporta uma aresta selecionada por came por ciclo de 720° e um solenoide por controlador. Não se apresentam controlos de duplo solenoide, temperatura do óleo sem aquisição nativa, multi-aresta ou afinador automático como funções disponíveis.

## Verificação

- Compilação ARM STM32F427 e orçamento de stack de arranque aprovados.
- 15 testes nativos locais com GCC, ASan/UBSan aprovados; deteção de fugas desativada apenas pelo ptrace do ambiente.
- 17 testes locais da Console aprovados, incluindo gravação/releitura de 11 páginas contra o dispatcher C real, preservação VTEC/CAM 1, ambos os mapas CAM 2, projetos antigos e novos, configurações centralizadas e recusa por firmware anterior.
- Integração da Console por porta série simulada aprovada com ECU só USB, sem perfil/sincronismo e com falhas de leitura injetadas.
- Janelas CAM 2 Parameters, PID e tabela/gráfico verificadas por captura da interface.

**São testes simulados e compilações, não validação elétrica na ECU.** A resposta física dos sensores, solenoides e motor continua pendente. A atualização da Console não grava firmware na Proteus; CAM 2 VVT e as novas leituras exigem o firmware correspondente.

Código: [Console](https://github.com/Tunuino/Tunuino/commit/ac19f34b1023ac901ddc8d6c65468e898ff2fd3a) e [firmware](https://github.com/Tunuino/Tunuino_Firmware/commit/79b9eec35840751d0ac716606990187c26434b42). Compilação ARM [aprovada](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/44180696305) e testes nativos GCC/Clang [aprovados](https://github.com/Tunuino/Tunuino_Firmware/actions/runs/44180696218).
