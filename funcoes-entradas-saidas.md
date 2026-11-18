# Entradas, saídas e pinouts por ECU

**Inputs and outputs** passa a ser um grupo de navegação com duas ECUs:

- **ECU Proteus → Proteus Pinout**: painel completo de atribuições, configuração de pinos, gravação e fotografias dos três conectores.
- **ECU UAefi → UAefi Pinout**: consulta do pinout real da UAefi F3, com fotografias e funções dos 68 contactos externos.

A configuração física da Proteus fica no seu painel **Proteus Pinout**. Na coluna **ECU connection**, selecionar a ligação pretendida para o sensor ou atuador. **Configure...** abre a configuração da ligação física no próprio painel de I/O; **Write to ECU** grava através da transação existente.

## ECU Proteus — conectores com fotografias

O painel inclui as fotografias dos três conectores, com numeração sobreposta:

| Separador | Conector da placa | Contactos |
| --- | --- | --- |
| Black23 | J101 | 23 |
| Black35 | J102 | 35 |
| White35 | J103 | 35 |

A vista é a **face de encaixe da ECU, com o fecho em cima**, conforme as fotografias. Não é a vista traseira dos fios da ficha. A imagem mantém a proporção ao redimensionar; o divisor permite ajustar a área da lista e a do pinout.

- Selecionar uma função destaca os seus contactos e abre o conector correspondente. Motores ETB e sensores VR destacam os dois contactos.
- Clicar num pino mostra o seu sinal e a função atribuída. Quando existe atribuição, seleciona a respetiva linha na lista de funções.
- A tabela por baixo da fotografia apresenta **Pin**, **Signal**, **Function** e **Assigned / state**. **Function** explica para que serve o contacto; **Assigned / state** mostra a atribuição e o estado atuais do editor.
- O amarelo assinala a ligação selecionada; um ponto azul identifica contactos com função atribuída. A cor do contorno distingue o tipo elétrico.
- As alterações locais pendentes são identificadas como **pending**; a fotografia não é uma medição da cablagem instalada.
- As fotografias e o mapa de contactos estão incluídos no executável: não exigem navegador ou ligação à Internet. Os créditos e a licença das fotografias podem ser consultados no próprio painel.

O mapa foi confrontado com os **93 contactos do esquema e do PCB Proteus v0.7**, revisão `383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46`. Inclui AV, AT, entradas digitais/VR, pontes H, LS, ignição, HS, CAN, knock, massas e alimentações. Mostrar uma ligação física não acrescenta um controlador de firmware para esse sinal.

## ECU UAefi — pinout F3

A página **UAefi Pinout** usa as ligações da revisão **F3** do projeto [Tunuino/uaefi](https://github.com/Tunuino/uaefi/tree/fefb25a22f8b1968a828ce377363717c529c712f). Não copia os contactos nem as atribuições da Proteus.

| Conector | Referência PCB | Contactos |
| --- | --- | --- |
| A | J4 | 8 |
| B | J5 | 18 |
| C | J2 | 20 |
| D | J10 | 16 |
| E | J3 | 6 |

As cinco fotografias originais apresentam os contactos identificados como A1…E6. Selecionar um contacto destaca-o e mostra **Pin**, **PCB contact**, **Signal** e **Function**. A fotografia mantém a proporção ao redimensionar e a tabela acompanha o conector selecionado.

Os **68 contactos** foram comparados entre o esquema e o PCB da revisão F3. As posições sobre as fotografias correspondem ao mapa original dessas imagens. São usadas fotografias originais dos conectores, sem substituir a sua geometria por imagens geradas.

A página UAefi é uma **referência de hardware**. O firmware nativo Tunuino atualmente integrado na Console suporta Proteus; ainda não existe um alvo UAefi nesse firmware. Por isso, a página UAefi não apresenta seletores de atribuição ou comandos Write to ECU sem suporte. Abrir ou consultar esta página não muda a ECU ligada, a comunicação, os mapas ou as atribuições da Proteus. Os controladores e componentes efetivamente montados dependem da variante física da UAefi.

Referências congeladas: [esquema F3](https://github.com/Tunuino/uaefi/blob/fefb25a22f8b1968a828ce377363717c529c712f/uaefi.kicad_sch), [PCB F3](https://github.com/Tunuino/uaefi/blob/fefb25a22f8b1968a828ce377363717c529c712f/uaefi.kicad_pcb). Créditos e termos das fotografias estão incluídos no botão de licença da página e em assets/uaefi/PHOTO-LICENSE.txt no código da Console.

## Configure e estado dos pinos da Proteus

**Configure...** abre a janela **ECU pin** da função selecionada, com a ligação escolhida, os contactos e os seus sinais elétricos. Não abre Triggers nem os parâmetros de funcionamento da função.

- **Apply connection** aplica apenas a ligação no editor local.
- **Write to ECU** usa a gravação existente da função, com validação e releitura.
- **Close** sem aplicar descarta a seleção preparada nessa janela. Uma ligação já aplicada permanece no editor até ser gravada ou descartada pelos mecanismos existentes.

A seleção é a mesma da coluna ECU connection: não existe uma segunda configuração de cablagem nos parâmetros da função. Os tipos elétricos apresentados são informação do hardware; não foram acrescentados seletores fictícios de Hall/VR, pull-up ou limiares que o firmware não controla.

| Texto apresentado | Significado |
| --- | --- |
| No pin selected | A função ainda não tem uma ligação de ECU selecionada. |
| No function selected | O contacto configurável não tem uma função de software atribuída. Não indica se há um fio fisicamente ligado. |
| Crankshaft sensor — disabled | J101/5 (+) e J101/13 (−) continuam a ser a entrada dedicada de cambota; a captura está desativada no editor. |
| Função — disabled | Entrada digital dedicada de VSS ou CAM 2, com a função desativada. |
| Fixed hardware connection | Alimentação, massa ou outra ligação física fixa apresentada como informação do conector. |
| pending | Existe uma alteração local pendente de gravação. |

Os contactos dedicados continuam identificados, selecionáveis e destacados com a função desativada. Não são transformados em contactos livres nem ativados automaticamente para preencher o pinout.

## Function — finalidade do contacto

A coluna **Function** fica visível no pinout e descreve os 93 contactos, mesmo sem atribuição de software. Inclui a função elétrica, a polaridade dos pares VR e ETB, a identificação High/Low dos barramentos CAN, os sinais lógicos de ignição, saídas de potência, sensores, massas e alimentações.

Exemplos:

| Sinal | Function |
| --- | --- |
| VR1+ | Crankshaft VR sensor — positive |
| VR1− | Crankshaft VR sensor — negative |
| ETB1− | ETB bridge 1 motor — negative |
| IGN1 | Logic signal for smart ignition coil |
| LS1 | Low-side injector / solenoid / relay output |
| AV1 | Analog sensor input (TPS / MAP / pressure) |
| AT1 | Temperature sensor input (CLT / IAT) |

Os exemplos de sensores nas entradas AV/AT descrevem utilizações possíveis; não atribuem automaticamente TPS, MAP, CLT ou IAT a esses contactos. A finalidade apresentada não muda quando a função está desativada. A configuração efetiva continua indicada à parte, em **Assigned / state**.

O texto tem quebra de linha e as linhas ajustam a altura à largura da coluna. Continua disponível também nas dicas e na configuração do pino. Nenhum número de contacto, coordenada da fotografia, tipo elétrico, calibração ou valor de configuração foi alterado por esta apresentação.

Referências: fotografia fornecida pelo utilizador com **Typical Function**, mapa KiCad/PCB já verificado e secções de recursos do PDF, páginas 19–20. O PDF não descreve o pinout Proteus.

## Editar e gravar

As listas de ligações podem ser usadas **offline** para preparar o projeto. A disponibilidade de gravação na ECU não desativa a edição local. Ligações fixas são apresentadas sem uma escolha fictícia de terminal.

A gravação mantém as validações de compatibilidade e conflitos, a cópia anterior e a releitura. Não é feita qualquer gravação por selecionar um pino ou abrir uma fotografia. **Write to ECU** permanece dependente da disponibilidade e estado da ECU. O vermelho indica alterações locais pendentes.

As atribuições não se repetem nos parâmetros de Fueling, Ignition, VVT ou ETB. O TPS principal continua a ser a entrada única **TPS / ETB 1 main**. O pedal apresenta entradas principal e de verificação; cada motor ETB usa a ponte H escolhida no painel central. A nomenclatura dos sinais de verificação das borboletas identifica explicitamente **TPS**.

## Referências e verificação

Consultado o **EMU_PRO_Software_Guide.pdf**, páginas **10–11** (entradas/saídas atribuídas) e **19–20** (recursos por função), além das três fotografias fornecidas pelo utilizador. O PDF descreve organização e recursos; não contém o pinout Proteus. O pinout elétrico é fundamentado no KiCad/PCB da revisão indicada, e a posição dos números corresponde às fotografias originais dos conectores.

Verificação automatizada com a janela real da Console: carregamento das três fotografias incluídas, tabelas de 23/35/35 contactos, seleção por clique, destaque do injetor no contacto correto, edição offline com escrita indisponível e preservação do projeto ao consultar fotografias. A verificação dos 93 contactos compara as redes do esquema com as do PCB; as redes sem nome dos dois fusíveis são comparadas pela conectividade física.

Estes resultados são execução da interface em computador e inspeção dos ficheiros de hardware. Não constituem ensaio elétrico de uma cablagem física.


[Código da Console](https://github.com/Tunuino/Tunuino/commit/d937497813db161e03ca6ca8c807d1072ce5a6a8).

Validação local: 17/17 testes aprovados e verificação visual dos três conectores com o tema da aplicação. A compilação e os testes Windows são executados pelo workflow automático da Console.


Testes adicionais da correção: Configure da cambota abre ECU pin mantendo a mesma janela MDI; os dois contactos dedicados continuam identificados com captura desativada; escolher sem aplicar e fechar preserva o projeto; Apply altera a ligação no editor sem gravação automática; restauração preserva os restantes campos. Verificação visual da janela com o tema da aplicação.

[Compilação e testes Windows desta correção](https://github.com/Tunuino/Tunuino/actions/runs/44382300624).

## Configuração do pino na interface

![Pino dedicado da cambota identificado com a captura desativada](images/console/proteus-pinout-io.png)

![Configure abre a ligação do pino da ECU](images/console/ecu-pin-configuration.png)

Capturas locais de teste, sem ECU ligada. A primeira mostra a função dedicada desativada; a segunda mostra a escolha preparada antes de aplicar. Créditos e termos das fotografias integradas: [PHOTO-LICENSE.txt](https://github.com/Tunuino/Tunuino/blob/d937497813db161e03ca6ca8c807d1072ce5a6a8/qt_console/assets/proteus/PHOTO-LICENSE.txt).



## Compatibilidade e verificação da organização

A janela Proteus passa a chamar-se **Proteus Pinout** e conserva o identificador dos desktops anteriores. O atalho interno io.configuration continua a abrir o mesmo painel e os projetos existentes mantêm os seus dados.

Verificação local: 17/17 testes aprovados. A janela real verifica a hierarquia por ECU, as cinco fotografias UAefi, as contagens 8/18/20/16/6, a seleção dos contactos e a preservação integral do projeto Proteus ao consultar a UAefi. As fotografias e a navegação foram inspecionadas com o tema da aplicação. O esquema e o PCB concordam nos 68 contactos; os ficheiros locais conferem com os blobs da revisão de origem.

Estas verificações são execução em computador e análise dos ficheiros KiCad. Não foi gravada ou comandada uma ECU física. O suporte de configuração/atuação UAefi não é apresentado como validado.
