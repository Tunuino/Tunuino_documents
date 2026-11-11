# Ligação automática da Tunuino Console

A Console procura a ECU e liga-se automaticamente, sem selecionar a porta COM. O indicador ONLINE representa uma comunicação USB validada; a alimentação externa é apresentada separadamente.

## Funcionamento

1. A descoberta procura o dispositivo USB da Tunuino.
2. A Console pede a identidade, valida protocolo, hardware e UID e começa a consultar o runtime.
3. Uma resposta de runtime válida da mesma ECU estabelece ONLINE, mesmo sem os 12 V externos ou com ADC indisponível. A identidade isolada ou uma porta USB presente não bastam.
4. Com uma configuração limpa, a Console lê automaticamente as páginas suportadas de configuração, sensores e mapas. Na versão corrigida, leitura, edição e gravação explícita ficam disponíveis em bancada por USB, mantendo as verificações existentes de identidade, geração, motor parado, saídas desarmadas, cópia anterior e releitura.
5. A perda ou o regresso da alimentação externa, enquanto a ECU continuar a responder por USB, atualiza apenas a indicação de alimentação. Não reinicia a sincronização, não limpa a configuração nem desliga o registo.
6. Perda do transporte USB, erro de sessão, resposta inválida ou identidade diferente continuam a interromper a ligação. A descoberta retoma automaticamente e uma mudança do número COM não exige seleção manual.

Sem confirmação de alimentação externa, o estado mostra “Online | USB bench connection; external ECU power not confirmed”. Após identificar a ECU e antes do primeiro runtime, mostra “Connecting | ECU identified; waiting for runtime data”.

A janela Firmware / USB DFU continua a suspender a comunicação normal enquanto estiver visível. Fechá-la permite retomar a descoberta. A pesquisa DFU não interfere com uma sessão serial aberta.

## Alimentação e saídas

A medição é específica da Proteus v0.7 STM32F427. O circuito 12V_RAW usa R1503 de 82 kΩ e R1504 de 10 kΩ, rede 12V_DIVIDED, U1501 contacto 43, ADC1_IN7, conforme o [hardware suportado](https://github.com/Tunuino/proteus/tree/383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46). O firmware transmite esta entrada em raw[11], deslocamento 82 do runtime; na resposta com byte de estado está nos bytes 83–84.

A conversão mantém a referência nominal de 3,3 V e divisor de 9,2. Duas amostras válidas de pelo menos 6 V, separadas por 250 ms, confirmam alimentação externa; até 5,5 V deixa de estar confirmada. ADC inválido ou saturado também não confirma alimentação. Estes critérios descrevem a alimentação, sem determinar se existe comunicação USB.

Ligar a Console, ler um mapa ou reconhecer alimentação não envia comandos para armar saídas. Os testes de atuadores mantêm a sua verificação própria de tensão e de estado do motor. Esta alteração não modifica o firmware, os comandos dos atuadores ou os limiares de proteção.

## Correção da ausência da COM

O firmware [b7a5dc9](https://github.com/Tunuino/Tunuino_Firmware/commit/b7a5dc94e793c8183df7de522a219d431ff607c1) corrige separadamente uma ultrapassagem da pilha antes de iniciar o USB. O caminho de validação no arranque desceu de 8656 para 5008 bytes, para uma pilha de 8192 bytes. A compilação ARM e os testes nativos GCC/Clang passaram. O utilizador confirmou depois que a porta USB Serial COM10 apareceu na Proteus. Isso valida a enumeração naquele ensaio, não o controlo do motor.

## Referência e verificação

As páginas 7 e 15 do guia de software fornecido foram consultadas: descrevem deteção automática e o campo de estado da ligação. Essas secções não especificam a alimentação apenas por USB da Proteus. A separação entre comunicação e alimentação resulta do requisito de bancada e do protocolo Tunuino; não é uma capacidade atribuída ao PDF.

O teste de integração da janela real injeta respostas simuladas no sinal da sessão. Verifica identidade sem runtime, runtime sem ADC, runtime com alimentação a zero, confirmação de 12 V, perda dos 12 V mantendo ONLINE e perda de transporte voltando a OFFLINE. Não usa uma ECU física nem comprova o driver Windows. Os testes existentes de descoberta, alimentação, escrita e ferramentas permanecem ativos.

A validação física da nova Console em bancada ainda exige que a Proteus fique ONLINE, sincronize as páginas e permita uma leitura/gravação explícita por USB. O funcionamento do motor e das saídas não foi validado por estes testes.

Correção da Console: [dd955d3](https://github.com/Tunuino/Tunuino/commit/dd955d37c0569af3434d0637ddd169f8257c8564). A compilação local Qt 6.4.2 e os 17 testes CTest passaram, incluindo o novo cenário de bancada na janela real com respostas simuladas.

O [workflow Windows](https://github.com/Tunuino/Tunuino/actions/runs/42915844538) passou a compilação, testes, arranque offscreen, criação e validação do instalador e publicação do canal Beta 0.1.3.291. Para aplicar esta correção de estado, atualizar a Console pelo atualizador. A Proteus que já apresenta COM10 com o firmware b7a5dc9 não precisa de uma nova gravação por causa desta alteração da Console. O teste físico da comunicação e sincronização só por USB continua pendente.

## Complemento: gravação por USB na Beta 0.1.3.292

A validação física da Beta 0.1.3.291 confirmou runtime USB na Console, mas revelou uma segunda condição no fluxo de gravação. A janela “ECU map write stopped” indicava “ECU state, identity or readback changed; write stopped. No map write was requested.” A versão 291 corrigiu a ligação ONLINE, mas ainda exigia pelo menos 6 V externos no runtime antes de iniciar a escrita.

O commit [5cc9a2b](https://github.com/Tunuino/Tunuino/commit/5cc9a2b700045c867d96d115d08635bc1ef4fda6) retira apenas essa exigência de tensão da transação de calibração. Continuam obrigatórios runtime válido, ADC válido, mesma identidade e geração, motor parado, ausência de erros de trigger, saídas desarmadas, estado limpo/preparado coerente, cópia anterior e releitura de todas as páginas. Perda de USB ou incerteza de Burn continuam a parar a operação sem repetição automática. Os testes de atuadores conservam a sua exigência própria de alimentação externa.

Os 17 testes CTest locais passaram. A transação completa foi também executada com o dispatcher e validação reais do firmware compilados para o computador: Fuel/Ignition e AFR Target, em cenários de 12 V e de zero volts na entrada de alimentação externa. A persistência nesse ensaio é simulada em memória; não valida a escrita física da flash da Proteus. O teste sintético também verifica perda de tensão externa antes do Burn com USB mantido, e continua a rejeitar ADC inválido, mudança de identidade/geração, arranque do motor, saídas armadas e divergências de releitura.

A [compilação Windows da Beta 0.1.3.292](https://github.com/Tunuino/Tunuino/actions/runs/42918690786) passou a compilação, testes, arranque offscreen, criação/validação do instalador e publicação no canal do atualizador. O firmware b7a5dc9 mantém-se; esta correção altera a Console. Falta confirmar a gravação e persistência físicas na Proteus apenas por USB.

## Acesso às atribuições de cilindros

A alteração e404b9b acrescenta uma ação visível nas páginas Injector outputs e Ignition outputs para abrir o perfil partilhado em Firing order, já expandido nas atribuições. O expansor desse perfil também deixa de estar escondido no menu de contexto. Assim, uma configuração ainda sem perfil gravado oferece um caminho explícito para escolher injetores e bobinas antes da primeira gravação.

Não são inventados canais nem retiradas validações do perfil. Abrir a configuração não grava nem arma saídas. O seletor de predefinições continua a aplicar grupos localmente; esta alteração não confirma nem corrige por si só a falha de persistência do trigger relatada pelo utilizador.

Foram consultadas as páginas 9–10 do guia de software fornecido, que distinguem sincronização, gravação e inspeção/teste de saídas. Essas páginas não especificam o arranque de um perfil Tunuino sem atribuições; o acesso direto é uma correção do fluxo da Console, não uma reprodução comprovada desse detalhe da referência.

A compilação local passou e os 17 testes CTest passaram. O teste EngineProfileTest verifica que uma página sem perfil consegue abrir as atribuições sem emitir uma escrita, mantendo os testes de atribuições explícitas e conflitos. A compilação Windows e a validação física na Proteus continuam pendentes nesta publicação.
