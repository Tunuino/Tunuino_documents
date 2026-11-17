# Organização da Tunuino Console

A navegação reúne as configurações e leituras em sete áreas.

| Área | Conteúdo |
|---|---|
| Inputs and outputs | Atribuições por nome de sensor ou atuador, contacto da Proteus, acesso à calibração e Write to ECU |
| Engine setup | Motor e cilindros, roda dentada, avanço de referência, sincronismo de cames e compensação de bateria |
| Fueling | Fuel Table, AFR Target, enriquecimentos, correções, corte e Autotune |
| Ignition | Ignition Table, dwell, correções e limitadores |
| VVT | VTEC, controlo e PID da CAM 1 e duas tabelas de alvo |
| Diagnostics | Scope, mostradores, estado do motor e leituras de sensores, cambota, cames, pressões, velocidade, VVT e VTEC |
| Logs | Graph Log, Tune Display, Text Log e Scatter Plot |

As atribuições dos injetores e bobinas são feitas em Inputs and outputs. Os sensores e respetivas calibrações são abertos pelo botão Configure da função correspondente. Os assistentes disponíveis ficam na toolbar do respetivo editor de calibração. Os atalhos Show assigned inputs e Show assigned outputs abrem o mesmo painel central.

A antiga árvore Sensors and inputs e o grupo Advanced foram retirados da navegação. As leituras em direto estão concentradas em Diagnostics. Os controlos de VVT usam a designação Output and activation conditions. O menu Navigation sections permite escolher as áreas visíveis.

## Área de trabalho

Uma instalação nova apresenta seis separadores — Main, Fueling, Ignition, VVT, Diagnostics e Logs. Ao abrir uma disposição antiga, os separadores vazios com os nomes cranking, triggers, idle, boost ou Extra são retirados. Os separadores com janelas são conservados, incluindo os personalizados. A posição e dimensão das janelas continuam guardadas.

## Edição e gravação

Os acessos abrem os editores existentes e utilizam as mesmas atribuições, calibrações e transações de gravação. Cada editor mantém Write to ECU. O vermelho indica diferenças locais em relação à última configuração confirmada.

A ligação série e a recolha em direto continuam a usar o mecanismo existente. Gravações exigem a configuração lida da ECU e utilizam validação e releitura.

## Verificação

A organização segue o pedido do utilizador e foi comparada com as secções de sensores e configuração de entradas do guia fornecido. O pinout continua fundamentado no projeto KiCad Proteus v0.7. O guia não define o protocolo da Tunuino.

Passaram a compilação local e os 17 testes da Console, incluindo o percurso da navegação, compatibilidade dos projetos e migração das disposições antigas. Os ensaios são simulados. A validação das funções na placa física continua pendente.
