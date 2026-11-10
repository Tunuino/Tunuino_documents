# DBW: calibração e diagnóstico dos sinais

A Console tem duas janelas separadas em DBW: **APS / TPS calibration** e **Signal diagnostics**. Este bloco prepara os sensores; ainda não constitui controlo eletrónico da borboleta.

## Implementado

- Quatro entradas independentes: APS principal/verificação e TPS principal/verificação. A seleção habitual apresenta Desativada/Atual e Personalizar; os contactos AV são os da Proteus v0.7 já usados pela Console. Não existe uma ligação DBW predefinida ou recomendada inventada.
- Captura simultânea dos dois sinais de cada sensor nas posições 0% e 100%, exclusivamente a partir do ADC real recebido por USB. Captura exige amostra recente, identidade e geração atuais, configuração limpa, estado de saídas totalmente desarmado, RPM zero e ausência de erros de trigger reportados. Estes estados reportados não substituem a verificação física de motor parado.
- A captura de 100% exige uma captura anterior de 0% na mesma ECU/geração. Mudança de entrada, edição manual, abertura de projeto ou mudança de sessão invalida essa sequência. Uma nova captura de 0% deixa o par sem amplitude válida até capturar 100%.
- Sinais crescentes ou invertidos, verificação de saturação do ADC, amplitude mínima de 1 V por sinal e comparação linear normalizada principal/verificação. O limiar de divergência é diagnóstico da Console, sem função de proteção no firmware.
- Deteção de entradas repetidas e conflitos com as funções já configuradas na ECU; reutilização do TPS existente apenas no sinal TPS principal. Não há alteração automática de cablagem.
- Janelas de configuração e monitor distintas, redimensionáveis e reabríveis. Sem dados recentes, o monitor apaga valores e apresenta a indisponibilidade.
- Calibrações e limiar incluídos no projeto .tnproj e no Project Comparator. Projetos anteriores abrem com este bloco desativado. Valores inválidos são rejeitados antes de alterar editores. As páginas ECU guardadas continuam intactas.
- **Prepare primary TPS sensor** transfere explicitamente o TPS principal para o editor TPS existente. A gravação continua a usar Write to ECU, com cópia anterior, validação e releitura. Não há nova via de escrita nem ativação de saídas.

## Limitações reais

O firmware atual não tem driver nativo da ponte H para DBW, regulador PID de borboleta nem proteção redundante autónoma. Por isso, **Throttle motor control** permanece indisponível. Não existe varrimento motorizado, autoafinação PID, limite de corrente implementado, comando de abertura ou modo de emergência DBW. Não são apresentados como comandos funcionais.

APS e sinais de verificação ficam no projeto local: o protocolo atual só permite gravar o TPS principal através da calibração existente. Não se apresenta a gravação desse TPS como gravação de um controlador DBW completo. Curvas não lineares de verificação, dois corpos de borboleta e diagnóstico no firmware continuam pendentes.

A atualização posterior implementou **Switched cam profile - VTEC**, com comando nativo do solenoide, histerese e requisito opcional de pressão de óleo: ver [VTEC](console-vtec.md). O primeiro controlador contínuo de CAM 1 com alvo fixo está descrito em [VVT](console-vvt.md); tabelas e controlo de várias cames continuam pendentes. A comutação VTEC mantém-se separada do ajuste contínuo de fase; o controlo do motor DBW continua pendente.

## Referência consultada

PDF fornecido, **EMU_PRO_Software_Guide.pdf**, versão 261.2.2: Appendix C, páginas 97–100, para entradas principal/verificação, tensão, calibração TPS, amplitude e requisitos do controlador. Página 98 também inspecionada visualmente. Appendix D, páginas 113–116, para separar regulação contínua de fase e comutação de perfil; o próprio capítulo limita a descrição detalhada à regulação contínua.

A janela nova mantém a grelha, cores e toolbar da Console. Não é apresentada como reprodução integral do afinador automático mostrado no PDF: faltam a ponte H, gráfico de progressão não linear, autoafinação e relatório dessa afinação. O PDF não fornece o driver, pinout de comando da Proteus ou especificação de protocolo necessários para os implementar.

## Verificação

Compilação local da Console e **15/15 testes aprovados**, com harness do firmware real executado no computador nos testes já existentes. O novo DbwSignalsTest usa ADC simulado e cobre inversão, divergência, saturação, conflitos, estado armado, amostras antigas/de outra geração/identidade, sequência de capturas e validação de projetos. CompleteMapSmoke verifica navegação, separação das janelas, persistência, comparação, rejeição atómica e compatibilidade com projetos anteriores.

Verificação visual local das duas janelas. Estes resultados são simulação/execução em computador; não são validação física de pedal, borboleta ou ECU. A compilação e o instalador Windows são verificados pelo [workflow automático da Console](https://github.com/Tunuino/Tunuino/actions).

[Código publicado na master](https://github.com/Tunuino/Tunuino/commit/10bc30eb9e3f4d4a34dafe3ff0044b61d0e1ea76).
