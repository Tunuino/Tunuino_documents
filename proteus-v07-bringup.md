# Primeiro ensaio da Proteus v0.7

## Prioridade e alvo

O primeiro ensaio físico será na **Proteus v0.7**, escolhida pelo utilizador. A prioridade passa a ser uma imagem de bancada, antes de acrescentar outras funções ao núcleo.

A configuração base será preparada para F427; F767 fica como variante opcional com mais recursos, incluindo memória para funcionalidades como Lua, conforme esclarecido pelo utilizador. Não há seleção automática da F767 a partir da propriedade PN do esquema. F427 e F767 usam alvos diferentes de compilação e exigem confirmação de clocks, memória, periféricos e imagem gravável; a variante não é apenas uma alteração do tamanho de memória no linker.

Falta confirmar a referência completa do MCU montado na unidade de teste e o meio de gravação disponível (ST-Link/SWD ou apenas USB). A indicação v0.7, por si só, não confirma esses dados. O conflito entre símbolo F427 e propriedade PN F767 continua documentado em Tunuino/Tunuino/hardware até a identificação da variante real.

## Estado real do firmware

O repositório tem um núcleo C portátil e uma Console Java, com comunicação, calibração, captura e telemetria testadas por simulação. Até esta etapa não existia compilação ARM automática. Ainda não há uma imagem STM32 ligada, com startup e drivers de placa, pronta para gravar na Proteus.

Não é possível concluir que tudo funciona na ECU pelos testes em PC, nem pelo sucesso de compilação ARM.

## Compilação ARM acrescentada

[Commit 6a81078](https://github.com/Tunuino/Tunuino/commit/6a810781bd56360f842666a49f0888d619cd6354).

O workflow **ARM core compilation** compila todos os ficheiros C de firmware/src com o GCC bare-metal, separadamente para Cortex-M4 e Cortex-M7. Usa Thumb e ABI soft-float, verifica que não existem dependências de helpers atómicos externos e produz uma biblioteca estática, os símbolos ainda por resolver, atributos ARM e um manifesto com hashes e versão do compilador.

Os artefactos têm NOT-FLASHABLE no nome. São bibliotecas de preparação, **não são ficheiros de firmware e não devem ser gravados na ECU**. Não incluem startup, tabela de vetores, mapa de memória, inicialização da placa, USB físico ou drivers dos sensores. Bibliotecas C, matemática e runtime do compilador ainda precisam de resolução na ligação final.

Reprodução com GCC ARM instalado:

```sh
python3 tools/build_arm_core.py --cpu cortex-m4 --output build/arm/cortex-m4
python3 tools/build_arm_core.py --cpu cortex-m7 --output build/arm/cortex-m7
```

O diretório de saída deve ser novo. O script recusa sobrescrever uma saída existente.

## Sequência para chegar à bancada

1. Confirmar MCU e meio de gravação da Proteus v0.7 física; identificar também o método de recuperação e o layout do firmware/bootloader existente antes de decidir endereços de gravação.
2. Preparar startup, linker e configuração da placa para esse MCU, com um primeiro alvo de bancada sem comando de injeção ou ignição.
3. Implementar USB físico e identificação real (incluindo UID), disponibilizando uma imagem e instruções específicas.
4. Gravar com os atuadores desligados e confirmar arranque, identificação e estabilidade da ligação.
5. Depois integrar entradas reais de sensores/cambota e verificar datalog. Ensaio do controlo do motor fica para uma etapa posterior.

Este plano não declara uma imagem pronta nem promete que o primeiro ensaio comprova todas as funções. O objetivo imediato é obter um primeiro arranque e comunicação verificáveis na placa correta.
