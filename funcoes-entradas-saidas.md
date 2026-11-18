# Entradas, saídas e pinouts da Proteus

A configuração física fica num único local: **Inputs and outputs**, na navegação principal. Na coluna **ECU connection**, selecionar a ligação pretendida para o sensor ou atuador. **Configure...** abre os parâmetros da função; **Write to ECU** grava através da transação existente.

## Conectores com fotografias

O painel inclui as fotografias dos três conectores, com numeração sobreposta:

| Separador | Conector da placa | Contactos |
| --- | --- | --- |
| Black23 | J101 | 23 |
| Black35 | J102 | 35 |
| White35 | J103 | 35 |

A vista é a **face de encaixe da ECU, com o fecho em cima**, conforme as fotografias. Não é a vista traseira dos fios da ficha. A imagem mantém a proporção ao redimensionar; o divisor permite ajustar a área da lista e a do pinout.

- Selecionar uma função destaca os seus contactos e abre o conector correspondente. Motores ETB e sensores VR destacam os dois contactos.
- Clicar num pino mostra o seu sinal e a função atribuída. Quando existe atribuição, seleciona a respetiva linha na lista de funções.
- A tabela por baixo da fotografia lista todos os contactos do conector, o sinal elétrico e as funções atribuídas nos editores.
- O amarelo assinala a ligação selecionada; um ponto azul identifica contactos com função atribuída. A cor do contorno distingue o tipo elétrico.
- As alterações locais pendentes são identificadas como **pending**; a fotografia não é uma medição da cablagem instalada.
- As fotografias e o mapa de contactos estão incluídos no executável: não exigem navegador ou ligação à Internet. Os créditos e a licença das fotografias podem ser consultados no próprio painel.

O mapa foi confrontado com os **93 contactos do esquema e do PCB Proteus v0.7**, revisão `383558bf7ca1c53ad8bbd8fdc9dd3a4187013c46`. Inclui AV, AT, entradas digitais/VR, pontes H, LS, ignição, HS, CAN, knock, massas e alimentações. Mostrar uma ligação física não acrescenta um controlador de firmware para esse sinal.

## Editar e gravar

As listas de ligações podem ser usadas **offline** para preparar o projeto. A disponibilidade de gravação na ECU não desativa a edição local. Ligações fixas são apresentadas sem uma escolha fictícia de terminal.

A gravação mantém as validações de compatibilidade e conflitos, a cópia anterior e a releitura. Não é feita qualquer gravação por selecionar um pino ou abrir uma fotografia. **Write to ECU** permanece dependente da disponibilidade e estado da ECU. O vermelho indica alterações locais pendentes.

As atribuições não se repetem nos parâmetros de Fueling, Ignition, VVT ou ETB. O TPS principal continua a ser a entrada única **TPS / ETB 1 main**. O pedal apresenta entradas principal e de verificação; cada motor ETB usa a ponte H escolhida no painel central. A nomenclatura dos sinais de verificação das borboletas identifica explicitamente **TPS**.

## Referências e verificação

Consultado o **EMU_PRO_Software_Guide.pdf**, páginas **10–11** (entradas/saídas atribuídas) e **19–20** (recursos por função), além das três fotografias fornecidas pelo utilizador. O PDF descreve organização e recursos; não contém o pinout Proteus. O pinout elétrico é fundamentado no KiCad/PCB da revisão indicada, e a posição dos números corresponde às fotografias originais dos conectores.

Verificação automatizada com a janela real da Console: carregamento das três fotografias incluídas, tabelas de 23/35/35 contactos, seleção por clique, destaque do injetor no contacto correto, edição offline com escrita indisponível e preservação do projeto ao consultar fotografias. A verificação dos 93 contactos compara as redes do esquema com as do PCB; as redes sem nome dos dois fusíveis são comparadas pela conectividade física.

Estes resultados são execução da interface em computador e inspeção dos ficheiros de hardware. Não constituem ensaio elétrico de uma cablagem física.


[Código da Console](https://github.com/Tunuino/Tunuino/commit/a8808f914fe23eab170ce8469ad954633a4b83a5).

Validação local: 17/17 testes aprovados e verificação visual dos três conectores com o tema da aplicação. A compilação e os testes Windows são executados pelo workflow automático da Console.
