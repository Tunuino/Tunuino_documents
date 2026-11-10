# VVT de CAM 1: primeiro controlador com alvo fixo

A Console tem janelas separadas em **VVT → Continuous cam timing - CAM 1**: Position, Solenoid and PID, Activation, Fixed target e VVT monitor. O firmware controla uma came por um solenoide low-side PWM. VTEC permanece uma função discreta separada.

## Implementado

- CAM 1 utiliza uma aresta selecionada por ciclo de 720°, já calculada pelo descodificador real de cambota/CAM. A posição é avanço relativo à referência: uma aresta anterior à referência corresponde a avanço positivo. Não é uma nova referência de TDC da ignição.
- Position configura referência, curso permitido e idade máxima do feedback. A toolbar pode copiar uma aresta atual recebida por USB, com identidade/geração, geometria e antiguidade verificadas. A cópia exige VVT desativado na configuração lida, preserva edições se a ECU/janela mudar e não escreve nem arma saídas. A leitura não prova que a came está no batente mecânico sem comando; esse ponto exige verificação física.
- A referência fica associada à roda fónica, dentes em falta, janela/fase de CAM e filtros. Alterar essa configuração exige rever e escrever Position antes de ativar VVT. O firmware rejeita ativação se o curso completo sair da janela de sincronização existente ou atravessar a fronteira da revolução. A Console não altera essa janela automaticamente.
- Solenoid and PID permite escolher uma saída LS livre pelo contacto J102, frequência 10–300 Hz, sentido de controlo, duty mínimo/máximo/estável e ganhos P/I/D. Os ganhos usam percentagem de duty por grau e unidades de tempo explícitas. Existe limitação da integral e proteção contra acumulação em saturação; D atua sobre a medição.
- O PID atualiza com uma nova medição de CAM. Repetir a mesma aresta não integra novamente nem recalcula a derivada. O duty ativo permanece dentro dos limites configurados; sem condições válidas, o comando é zero.
- Activation configura RPM e temperatura com histerese, tempo mínimo de funcionamento e pressão de óleo opcional, usando o sensor analógico já calibrado. Pressão insuficiente, perda de feedback enquanto ativo, curso excedido e erro de seguimento persistente desligam VVT com retenção até desarmar/rearmar. Pausas de processamento não são transformadas em saltos de integral.
- Fixed target contém **um único alvo constante** dentro do curso permitido. Não representa uma tabela RPM/MAP. A escrita mantém a política atual: motor parado e saídas desarmadas.
- O monitor lê alvo, posição, duty, motivo de inibição, tempo, idade do feedback, pressão e retenção. A posição relativa pode ser observada com o controlo desativado se houver referência configurada e feedback válido. O comando PWM não confirma movimento físico do solenoide. Respostas antigas, inválidas ou de outra ECU/geração são rejeitadas.

## Preservação e gravação

A página 6 passa à versão 2, mantendo os campos VTEC existentes e acrescentando a extensão VVT. As saídas VVT e VTEC são reservadas separadamente; conflitos entre elas ou com injetores, bomba, ventoinha, relé, idle e boost são rejeitados pelo firmware.

Cada janela escreve os seus campos. Edições independentes na mesma página são preservadas ao atualizar a base; alterações dos campos em edição bloqueiam a gravação até revisão/descarte. As seis páginas continuam incluídas na cópia anterior, validação, um único Burn e releitura. Não se arma o motor ao abrir janelas ou projetos.

Projetos anteriores recebem os novos editores desativados. A persistência usa o registo 7 e continua a ler os registos 2–6. A migração mantém o banco anterior durante gravações interrompidas. Firmware anterior não lê os novos registos: não presumir recuperação da configuração mais recente ao fazer downgrade; conservar uma cópia compatível.

## Limites atuais

Este é o primeiro bloco de VVT: **uma came, uma aresta por 720°, um solenoide low-side e alvo fixo**. Estão pendentes tabelas de alvo RPM/MAP, cames adicionais, rodas de CAM com várias arestas, CAM 2 com leitura de fase, duplo solenoide/H-bridge e afinador automático Relay/AMIGO. O controlo do motor DBW também continua pendente.

A monitorização visual é a cerca de 5 Hz, não o registo de 100 Hz indicado na referência para avaliar respostas rápidas. Não foi demonstrada a resposta de 200–300 ms num motor. As referências de frequência, duty, curso e PID são **inativas** e não equivalem à calibração de um motor/solenoide.

## Referência e verificação

Consultado o PDF fornecido **EMU_PRO_Software_Guide.pdf**, Appendix D, páginas 113–119: graus de avanço, fonte/aresta e janela, limites de duty, ativação, sentido, ganhos PID e distinção entre ajuste contínuo e comutação discreta. O PDF não fornece o protocolo/driver da Proteus nem uma calibração universal. A associação da referência à configuração e as retenções acima são políticas explícitas da Tunuino.

As janelas usam a grelha, cores e toolbar existentes; não são apresentadas como reprodução do afinador automático da página 119, que ainda não está implementado.

- **17/17 testes locais da Console aprovados**, incluindo projetos antigos, janelas separadas, conflitos, preservação do VTEC e escrita/releitura das seis páginas contra o dispatcher C real executado no computador.
- **14 testes nativos locais aprovados com ASan/UBSan**. A deteção de fugas foi desativada apenas localmente devido ao ptrace do ambiente. Testados PID/unidades, sentido inverso, medição repetida, limites, falhas/retenções, relógio, PWM/watchdog e mudança de fase pelo descodificador real de cambota/CAM.
- **Testes do firmware com GCC e Clang e compilação ARM STM32F427 aprovados** nos workflows.
- Inspeção visual da configuração realizada. **Validação física na Proteus/solenoide/motor pendente**; os testes no computador não a substituem.

Windows: [compilação, testes e instalador](https://github.com/Tunuino/Tunuino/actions/runs/42786956380).

Código: [Console](https://github.com/Tunuino/Tunuino/commit/b4e9d08d1965f3794b3c67bc9667b722b8a692c1) e [firmware](https://github.com/Tunuino/Tunuino_Firmware/commit/4e59143504f6e607ef1d7a12a1069d7cc0d19c19).
