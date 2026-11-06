# Tunuino Console — gravação de firmware por USB DFU

## Fluxo atual na Console Qt

1. Abrir **ECU → Firmware / USB DFU**. A comunicação normal e a reconexão automática ficam suspensas enquanto esta janela estiver visível.
2. Colocar a Proteus em modo DFU, usando BOOT0 e reset conforme a placa, e ligar o USB ao PC.
3. Carregar em **Detect USB DFU**. A deteção procura o bootloader STM32 `0483:DF11`; não exige uma porta COM nem que a console esteja ONLINE.
4. Em **Select firmware .bin...**, selecionar a imagem destinada à Proteus STM32F427.
5. Usar **Validate firmware .bin** para verificar o tamanho e os vetores de arranque. **Prepare flash (no write)** apresenta o plano de setores e blocos sem gravar.
6. Carregar em **Flash and verify**. A confirmação identifica o ficheiro, tamanho e SHA-256 e informa que os setores serão apagados. Não existe seleção, criação ou exigência de ficheiro de backup neste fluxo.
7. Aguardar **Flash and verify PASS**. Durante a gravação, manter alimentação e USB estáveis, com motor e atuadores desligados.
8. Repor BOOT0 em arranque normal e reiniciar a ECU. Fechar a janela de firmware para permitir a deteção automática da ligação normal.

## Implementação

A gravação utiliza WinUSB e o protocolo STM32 ROM DfuSe, no Windows. Requer exatamente um dispositivo DFU acessível através do driver WinUSB. A deteção do hardware no Windows, por si só, não garante que o driver permite abrir a interface.

O backend valida tamanho e vetores da imagem, identidade USB e interface DFU. Apaga apenas os setores calculados para a imagem, programa a partir de `0x08000000` e relê todos os bytes escritos. Só apresenta sucesso quando a releitura coincide integralmente com o firmware selecionado. O nome do ficheiro e os vetores não provam sozinhos a compatibilidade da imagem com a revisão da placa.

Foram removidos o pedido de backup prévio, a leitura e comparação desse ficheiro antes do apagamento, a opção de guardar um backup no diagnóstico e os textos ou bloqueios associados. A deteção pode continuar a ler e comparar o firmware instalado com a imagem selecionada para diagnóstico, sem guardar essa leitura num ficheiro. A verificação por releitura após a programação mantém-se.

Não existe mudança automática do firmware normal para o bootloader DFU nem saída automática do DFU neste fluxo. A entrada e a saída do modo de arranque continuam a exigir a ação correspondente na placa.

## Validação

A compilação e os testes Windows são executados pelo workflow **Qt Console Windows** na branch `master`. O runner não tem uma ECU física ligada. A verificação local em Linux não compila o backend WinUSB, protegido por `Q_OS_WIN`; a compilação Windows é necessária para validar esse código.

A identificação USB/COM numa Proteus F427 foi observada em testes anteriores. Isso não valida a gravação DFU desta revisão. Ainda falta testar fisicamente a sequência completa de gravação e releitura na Proteus.
