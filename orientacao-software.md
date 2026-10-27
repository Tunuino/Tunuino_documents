# Orientação da Tunuino Console

Esta orientação regista o funcionamento pretendido pelo utilizador para as próximas funções. É uma direção de produto, não uma declaração de que todas as capacidades abaixo já estão implementadas.

## Uso normal

- Organizar por função: sensores, arranque, combustível, ignição, ralenti, turbo, saídas, proteções e diagnóstico.
- Dar acesso rápido aos valores de base e a predefinições editáveis por sensor ou componente.
- Para ligações, apresentar a recomendação habitual e o contacto físico. Alternativas completas apenas em Personalizar/Avançado.
- Apresentar primeiro os parâmetros usados no dia a dia; detalhes técnicos e funções opcionais não devem dominar cada janela.
- Explicar para que serve cada parâmetro, quais as unidades e as suas dependências junto do próprio controlo.
- Usar milissegundos para injeção, dwell e tempo morto.

## Configuração e projetos

O objetivo é um projeto completo, editável offline, com sensores, perfil do motor, tabelas e funções. Abrir um projeto, alterar parâmetros, guardar um ficheiro e aplicar à ECU devem ser ações distintas e identificáveis. Os valores alterados devem ser visíveis e comparáveis, com possibilidade de desfazer onde suportado.

Estado atual: existem cópias separadas de perfil/sensores, mapas e auxiliares. Falta reuni-las num ficheiro de projeto único, com histórico e comparação de todas as secções.

## Afinação e análise

Manter tabelas 16×16, valores e visualizações consistentes. A orientação inclui curvas 2D, tabelas/superfícies 3D, gravação/reprodução de logs e comparação de resultados. O estado da ligação, sincronismo, modos de motor e falhas deve permanecer legível durante o trabalho.

Estado atual: mapas VE/ignição 16×16, leitura/edição e CSV estão disponíveis; a curva alvo lambda ainda é por MAP com três pontos. Falta mapa alvo lambda/AFR 16×16 e afinação assistida de VE baseada em logs coerentes com a calibração, filtrando transientes e condições inválidas.

## Atribuições, testes e funções futuras

Atribuições de entradas e saídas servem para configurar e verificar a instalação. Testes de atuadores ficam numa área própria, identificando a função real e o contacto. As opções devem refletir o hardware e as funções suportadas pelo firmware.

VVT, detonação, borboleta eletrónica, estratégias adicionais de injeção/ignição e integrações continuam a precisar de implementação e validação próprias. Não acrescentar apenas menus que façam parecer esses controlos operacionais.

## Primeira aplicação desta orientação

As [ligações rápidas da Proteus](ligacoes-rapidas-proteus.md) aplicam a regra de poucas escolhas e recomendação clara por função. A [lista de funções implementadas e pendentes](combustivel-protecoes.md) mantém o estado técnico separado desta direção de produto.
