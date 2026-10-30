# Correção da ligação USB ao guardar na ECU



## Falha identificada

Ao guardar um mapa, a Console podia apresentar «Ligação interrompida ou contagem de bytes inválida» e perder a identificação da ECU. A gravação em flash podia já ter terminado: a falha ocorria antes de a confirmação chegar ao computador.

O dispatcher executa o pedido de gravação e inicia o prazo de transmissão pelo relógio posterior à operação. O ciclo principal USB continuava a usar o instante anterior à gravação. Essa consulta fazia parecer que o relógio tinha recuado e ativava a recuperação USB, desligando a sessão.

## Alteração

O ciclo principal volta a ler o relógio depois do dispatcher, antes de consultar ou transmitir a resposta. Mantêm-se os limites de tempo e a rejeição de um recuo real do relógio. A confirmação só é produzida depois do resultado da persistência.

A Console passa a identificar se a falha ocorreu no envio dos blocos, na confirmação da flash ou na releitura do mapa. Se a gravação já foi pedida e o resultado é incerto, não descarta automaticamente os dados nem repete a gravação. Pede uma nova leitura para confirmar o estado.

O formato dos mapas, os registos persistentes, as ligações e o protocolo IO 0.2.0 permanecem compatíveis. A correção do relógio está no firmware da ECU.

## Aplicar a correção

1. Instala a Console deste Bundle e usa a imagem de firmware incluída no mesmo pacote.
2. Grava essa imagem na Proteus por DFU, no menu Firmware. O instalador da Console não executa essa gravação.
3. Repõe o arranque normal, reinicia a ECU, liga a COM e identifica-a.
4. Lê primeiro o mapa guardado. Se a gravação anterior tinha terminado antes da queda USB, o mapa pode já estar correto.
5. Para enviar um mapa, termina a monitorização, mantém o motor desarmado e usa Guardar e verificar na ECU. A operação inclui releitura para comparar o conteúdo.

Se a ligação se perder, conserva a cópia local .tnmap e volta a identificar a ECU antes de confirmar o mapa por leitura. Não assumir sucesso nem falha da persistência apenas pelo desaparecimento da COM.

## Verificação

O teste executa o main real da aplicação ChibiOS, compilado para o computador com USB e relógio modelados. Antes da alteração, uma persistência simulada de 250 ms reproduziu a recuperação USB indevida e impediu a confirmação.

Depois da alteração passaram os cenários de persistência de **2, 250, 1500 e 4500 ms**: confirmação recebida, estado persistido confirmado e nenhum reinício USB. Um erro de persistência devolve falha e mantém os dados pendentes, sem falsa confirmação de sucesso.

Também passaram descoberta, comunicação TunerStudio, transmissão sem confirmação, perda/reconfiguração USB e recuperação. Os testes C/Java de mapas/configuração verificam gravação e releitura, precisão, preservação de mapas e manutenção dos dados pendentes quando a confirmação da flash falha.



O [Bundle 40607141909](https://github.com/Tunuino/Tunuino/actions/runs/40607141909) verificou o MSI **1.0.106**, incluindo instalação, conteúdo instalado/runtime, atalhos, atualização, desinstalação e preservação de dados. Firmware e Console pertencem ao mesmo commit desta publicação.
