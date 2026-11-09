# VTEC: comutação discreta do perfil de cames

VTEC tem configuração própria em **VVT → Switched cam profile - VTEC** e um **VTEC monitor** separado. O firmware comanda uma saída low-side LS da Proteus v0.7. Este bloco não regula o ângulo das cames nem implementa VVT contínuo.

## Funcionamento implementado

- Ativação explicitamente selecionada e gravada; desativada nas configurações antigas e nas referências iniciais. Mostrar, abrir ou guardar um projeto não arma saídas.
- Escolha explícita do solenoide por contacto J102, com a lista completa em Personalizar. Não existe um contacto VTEC recomendado inventado. O firmware rejeita conflitos com injetores, bomba, ventoinha, relé, idle e boost.
- Comando permitido apenas com o motor em funcionamento no controlador existente e saídas armadas. São exigidos TPS, MAP e temperatura válidos, limites de temperatura, tempo mínimo desde o funcionamento do motor e janela de RPM.
- Limiares distintos para ligar/desligar por RPM, TPS e MAP. Existe um limite superior de RPM para desligar VTEC.
- Pressão de óleo opcional, usando a entrada analógica de pressão de óleo já calibrada, com limiares de ligação/manutenção. Não se confunde esta leitura com o interruptor de confirmação hidráulica do próprio VTEC.
- Falha de um sensor exigido ou perda de pressão exigida durante o comando desliga VTEC e retém essa função desligada enquanto o sistema permanecer armado. Recuperar a leitura não volta a ligar VTEC. Desarmar/rearmar explicitamente limpa essa retenção; uma paragem geral do motor também desarma as saídas.
- Paragem, perda de sincronismo, configuração inválida e watchdog continuam a desligar as saídas pelo controlador existente. Pausas excessivas na atualização não acumulam o atraso de ativação; se VTEC estava ligado, a função fica retida desligada.
- O monitor apresenta comando On/Off, motivo de inibição, tempo de funcionamento, pressão medida e retenção. Usa resposta nativa da mesma ECU/geração e apaga valores antigos ou inválidos. O comando mostrado não confirma movimento físico do solenoide.

## Gravação e compatibilidade

A página de controlo 6 tem 256 bytes, versão 1, e é anunciada por uma capacidade própria do firmware. A Console lê-a automaticamente após as páginas existentes. Firmware sem essa capacidade mantém a edição local disponível e a gravação VTEC indisponível.

A gravação usa a transação existente: verifica ECU/geração e saídas desarmadas, cria cópia anterior, prepara a alteração, relê todas as seis páginas, executa um único Burn e relê novamente. As outras páginas, incluindo AFR Target, são preservadas.

A persistência passa ao registo 6. Os registos 2–5 continuam a ser lidos e são migrados para o outro banco, preservando a configuração anterior durante gravações interrompidas. A extensão VTEC começa vazia/desativada. O formato 6 não é lido por versões antigas do firmware: uma eventual reversão exige uma cópia compatível, não deve presumir que o firmware antigo recupera a configuração mais recente.

O projeto .tnproj inclui a página de controlo quando a ECU a fornece e os valores do editor VTEC. Projetos antigos recebem o editor desativado. Abrir um projeto não transfere valores para a ECU. Alterações da página enquanto existe edição local bloqueiam a gravação até descartar/rever a edição.

## Referência e limites

Consultado o PDF fornecido **EMU_PRO_Software_Guide.pdf**, Appendix D, páginas 113–116. A página 113 distingue ajuste discreto de regulação contínua e identifica VTEC como comutação de perfil. O capítulo declara que a explicação detalhada cobre regulação contínua. Por isso, os limiares e a política de falhas acima são implementação explícita da Tunuino, não uma alegação de que o PDF especifica estes campos.

A grelha e a toolbar seguem as janelas existentes da Console. Não se apresenta este bloco como uma reprodução integral de uma janela de comutação que o PDF não detalha.

Continuam pendentes: interruptor de confirmação hidráulica do VTEC, comutação entre mapas de combustível/ignição, VVT contínuo e controlo de motor DBW/PID. Os valores iniciais são referências inativas, não uma calibração validada para um motor.

## Verificação

- **16/16 testes locais da Console aprovados**, incluindo navegação, projetos antigos, edição/conflitos e escrita/releitura das seis páginas contra o dispatcher C real executado no computador.
- **13 testes nativos locais aprovados** com AddressSanitizer/UndefinedBehaviorSanitizer. A deteção de fugas foi desativada apenas na execução local porque o ambiente usa ptrace; os sanitizadores de acesso e comportamento indefinido permaneceram ativos.
- **Workflows do firmware aprovados com GCC e Clang**, incluindo a nova lógica e migração com interrupção de gravação.
- **Compilação ARM STM32F427 e verificação da imagem aprovadas** no workflow de firmware.
- Configuração inspecionada visualmente. **Validação física na Proteus, com solenoide e motor, pendente**. Nenhum resultado simulado é apresentado como teste físico.

A compilação, testes e instalador Windows desta alteração são acompanhados no [workflow da Console](https://github.com/Tunuino/Tunuino/actions).

Código: [Console master](https://github.com/Tunuino/Tunuino/commit/62af0df27f0d8d134d41cd92333c8ee22bd71c79), [firmware master](https://github.com/Tunuino/Tunuino_Firmware/commit/bd3e3dc72a65453a087294cdedd153e9c7858cb7). Testes e compilação ARM: [workflows do firmware](https://github.com/Tunuino/Tunuino_Firmware/actions).
