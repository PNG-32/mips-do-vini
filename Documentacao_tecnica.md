### DOCUMENTAÇÃO TÉCNICA

### 1 - Processador MIPs (Ciclo Único)

Projeto:  mips

Ferramenta:  Logisim 2.7.1

Arquitetura: MIPS ciclo único (single-cycle) 

Largura dos dados: 8 bits 

Largura da instrução de memória: 18 bits 

Subcircuitos utilizados:  Banco de Registradores, ULA, Unidade de Controle, Controle ULA 

Tipo do circuito: Datapath completo com elementos sequenciais e combinacionais 

<br>     



### 1.2 - Visao Geral e Função

O circuito main implementa o datapath completo do processador MIPS ciclo único, integrando todos os subcircuitos documentados a seguir (Banco de Registradores, ULA, Unidade de Controle, Controle ULA).
Em um processador ciclo único, cada instrução é executada integralmente em um único ciclo de clock. Isto significa que todos os estágios do datapath (busca da instrução, decodificação, leitura de registradores, execução na ULA, acesso à memória e escrita de resultado) devem ser concluídos dentro do mesmo período de clock. 


\
### 2 - Banco de Registradores


### 2.1 - Identificacao do Componente

| Campo | Informação |
|-------|-------------|
| **Nome do subcircuito** | Banco de Registradores |
| **Projeto pai** | mips  |
| **Ferramenta de simulacao** | Logisim (logisim 2.7.1 |
| **Arquitetura** | MIPS ciclo unico (single-cycle) |
| **Numero de registradores** | 8 registradores de uso geral (R0 a R7) |
| **Largura dos dados** | 8 bits por registrador |
| **Numero de portas de leitura** | 2 (leitura simultanea) |
| **Numero de portas de escrita** | 1 (escrita seletiva via RegWrite) |
| **Enderecamento** | 3 bits por registrador (rs, rt, rd) |


### 2.2- Visão Geral e Função

O Banco de Registradores é o componente responsavel por armazenar e disponibilizar os valores dos registradores de uso geral do processador MIPS. Em um processador ciclo unico, este bloco e acessado diretamente pelo datapath a cada ciclo de clock, provendo os operandos para a ULA e recebendo o resultado das instrucoes do tipo R ou das operacoes de load.
O subcircuito implementa as seguintes funcoes principais:
- Leitura combinacional de dois registradores simultaneamente (Read data 1 e Read data 2), sem necessidade de sinal de controle habilitador. Isso significa que, assim que os endereços de leitura (rs e rt) são fornecidos, os valores dos registradores correspondentes aparecem imediatamente nas saídas, sem aguardar o clock.
- Escrita síncrona em um registrador selecionado, condicionada ao sinal de controle RegWrite e sincronizada com a borda de subida do clock (CLOCK). Apenas quando RegWrite está em nível lógico 1 e ocorre uma borda de subida do clock, o dado presente em Write data é armazenado no registrador apontado por Write register.
- Reset assíncrono de todos os registradores atraves do sinal CLEAR. Diferente da escrita, o reset não depende do clock: sempre que CLEAR for ativado (nível lógico 1), todos os registradores são imediatamente zerados.
- Decodificacao interna dos enderecos de leitura e escrita por meio de multiplexadores e demultiplexadores de 3 bits, permitindo a seleção eficiente dos 8 registradores disponíveis.

### 2.3 - Arquitetura Interna

 Registradores de armazenamento
O bloco e composto por 8 flip-flops do tipo D com enable (WE - Write Enable), cada um com largura de 8 bits. Os registradores sao identificados de Reg0 a Reg7. Cada registrador possui:
- Entrada D: recebe o dado de Write data (8 bits) quando habilitado.
- Saida Q: disponibiliza o valor armazenado permanentemente para os multiplexadores de leitura.
- Entrada de enable WEn (onde n = 0..7): habilita a escrita quando ativo. Apenas um WEn e ativado por vez, garantindo que apenas um registrador seja modificado a cada ciclo de escrita.
- Entradas CLOCK e CLEAR: compartilhadas entre todos os registradores, permitindo a sincronização global e o reset simultâneo.

Caminho de escrita
O caminho de escrita e composto por dois estagios de decodificacao em cascata:
Estagio 1 - Decodificador do endereço de escrita (DMX 1:8)
O sinal Write register (3 bits) alimenta um demultiplexador (DMX) de 1 para 8. Este DMX propaga um nivel logico '1' para a saida correspondente ao endereco binario codificado em Write register. As saidas sao denominadas WE0 a WE7 e representam a selecao do registrador de destino.

Estagio 2 - Mascaramento por RegWrite (porta AND)
O sinal RegWrite e combinado (via porta AND) com cada uma das saidas WE0 a WE7. Somente quando RegWrite = 1 a saida WEn correspondente ao registrador selecionado e propagada como nivel alto, efetivamente habilitando a escrita no flip-flop D daquele registrador. Quando RegWrite = 0, todas as saidas WEn permanecem em nivel baixo, impedindo qualquer escrita, independentemente do valor de Write register.

Essa arquitetura em cascata garante que apenas um registrador seja escrito por ciclo e somente quando explicitamente autorizado pelo sinal de controle RegWrite. A combinação do demultiplexador com as portas AND cria uma estrutura segura e eficiente para o controle de escrita.

### Caminho de leitura

Dois multiplexadores independentes de 8 entradas x 8 bits (MUX 8:1) realizam a leitura:
- MUX de leitura 1: seleciona a saida Q do registrador apontado por Read register 1 (3 bits) e disponibiliza em Read data 1 (8 bits).
- MUX de leitura 2: seleciona a saida Q do registrador apontado por Read register 2 (3 bits) e disponibiliza em Read data 2 (8 bits).

A leitura e puramente combinacional: qualquer alteracao nos sinais de selecao se reflete imediatamente nas saidas, sem aguardar o clock. Isso é fundamental para o desempenho do processador ciclo unico, pois permite que os operandos estejam disponíveis para a ULA no mesmo ciclo de clock em que a instrução é decodificada.

### 2.4 - Portas de Interface

| Porta | Largura (bits) | Direcao | Descricao |
|-------|----------------|---------|-----------|
| **Read register 1** | 3 | Entrada | Endereco do registrador a ser lido pela porta 1 (campo rs da instrucao) |
| **Read register 2** | 3 | Entrada | Endereco do registrador a ser lido pela porta 2 (campo rt da instrucao) |
| **Write register** | 3 | Entrada | Endereco do registrador de destino para escrita (campo rd ou rt, selecionado pelo MUX RegDst) |
| **Write data** | 8 | Entrada | Dado a ser escrito no registrador selecionado (resultado da ULA ou dado da memoria) |
| **RegWrite** | 1 | Entrada | Habilita a escrita quando em nivel alto (1 = escrita ativa). Gerado pela Unidade de Controle. |
| **CLOCK** | 1 | Entrada | Sinal de clock; escrita ocorre na borda de subida |
| **CLEAR** | 1 | Entrada | Reset assincrono; zera todos os registradores quando em nivel alto |
| **Read data 1** | 8 | Saida | Valor do registrador apontado por Read register 1 (conteudo de rs) |
| **Read data 2** | 8 | Saida | Valor do registrador apontado por Read register 2 (conteudo de rt, também chamado de contRT) |

### 2.5 -  Descricao do Comportamento

 ### Operacao de leitura

A leitura e assincrona e ocorre a qualquer momento, independentemente do clock:
- Read data 1 = Reg[Read register 1]
- Read data 2 = Reg[Read register 2]

Ambas as leituras podem ocorrer simultaneamente e os dois enderecos podem coincidir (leitura do mesmo registrador nas duas portas) ou diferir. Este comportamento é essencial para instruções como beq, que precisam comparar dois registradores no mesmo ciclo.
\
\
\
### Operacao de escrita

A escrita e sincrona e ocorre na borda de subida de CLOCK, condicionada a:
- RegWrite = 1: escrita habilitada.
- Write register: seleciona qual dos 8 registradores recebera o dado.
- Write data: dado de 8 bits que sera armazenado.

Condicao formal: se (RegWrite == 1), entao na borda de subida de CLOCK: Reg[Write register] <- Write data.

\
### Reset (CLEAR)


Quando CLEAR = 1, todos os 8 registradores sao zerados assincronamente, independentemente do estado do clock ou do sinal RegWrite. O reset e imediato e afeta simultaneamente Reg0 a Reg7. Este sinal é tipicamente usado no inicio da simulação para garantir que o processador comece em um estado conhecido.

\
### 2.6 - Sinais de Controle Relacionados

O Banco de Registradores e controlado pela Unidade de Controle do processador. O sinal RegWrite e o unico sinal de controle direto; os demais sinais de enderecamento provem do campo de instrucao (bits da instrucao decodificada):

| Sinal | Origem | Descrição |
|-------|--------|-----------|
| **RegWrite** | Unidade de Controle | Ativo (1) para instrucoes do tipo R (add, sub, etc.) e para load (lw, addi). Inativo (0) para store (sw), branch (beq) e jump (jmp). |
| **Read register 1** | Campo rs da instrucao (bits 14-12) | Endereço do primeiro registrador fonte |
| **Read register 2** | Campo rt da instrucao (bits 11-9) | Endereço do segundo registrador fonte |
| **Write register** | Campo rd (bits 8-6) ou rt (bits 11-9) | Selecionado pelo sinal RegDst da Unidade de Controle |
| **Write data** | Saida da ULA (instrucoes tipo R) ou da Memoria de Dados (lw) | Selecionado pelo sinal MemToReg |

### 2.7 - Integracao no Datapath

No contexto do processador MIPS ciclo unico implementado no Logisim, o Banco de Registradores se conecta aos seguintes blocos:
- **Entrada Write data**: recebe o resultado da ULA (instrucoes tipo R) ou o dado lido da Memoria de Dados (instrucao lw), via MUX controlado por MemToReg.
- **Saida Read data 1**: conectada diretamente a entrada A da ULA (operando rs).
- **Saida Read data 2**: conectada ao MUX ALUSrc (que a encaminha para a entrada B da ULA) e também diretamente a Memoria de Dados como dado de escrita (para instrucao sw).
- **Controle de escrita (RegWrite)**: gerado pela Unidade de Controle com base no opcode da instrucao.
- **CLOCK e CLEAR**: sinais globais do sistema, compartilhados com todos os elementos sequenciais (PC, Memoria de Dados).



---

### 3 - Unidade Logica e Aritmetica (ULA)


### 3.1 - Identificacao do Componente

| Campo | Informação |
|-------|-------------|
| **Nome do subcircuito** | ULA (Unidade Logica e Aritmetica) |
| **Projeto** | mips  |
| **Ferramenta** | Logisim 2.7.1 |
| **Arquitetura** | MIPS ciclo unico (single-cycle) |
| **Largura dos operandos** | 8 bits (entradas A e B) |
| **Largura da saida** | 8 bits (resultado R) |
| **Sinal de controle** | Funcao — 3 bits (seleciona 1 de 8 operacoes) |
| **Operacoes suportadas** | 8: ADD, SUB, MULT, DIV, NOT, SLT, SLL, SRL |
| **Saidas auxiliares** | Igual (flag de igualdade A == B) |

### 3.2 - Visao Geral e Funcao

A ULA e o componente combinacional central do datapath do processador MIPS, responsavel por executar todas as operacoes aritmeticas e logicas definidas pelo conjunto de instrucoes do processador. Este projeto implementa as 8 operações mapeadas diretamente pelas instruções suportadas: add, sub, mult, div, not, slt, sll e srl.

Em um processador ciclo unico, a ULA opera integralmente dentro de um unico periodo de clock, sem registradores internos de pipeline. Isso significa que o resultado deve estar estável e pronto antes da próxima borda do clock, pois será usado imediatamente para escrita no Banco de Registradores ou para acesso à Memória de Dados.

O subcircuito recebe dois operandos de 8 bits (A e B), um sinal de controle de 3 bits (Funcao) proveniente do bloco Controle ULA, e produz um resultado de 8 bits (R) alem do sinal de status Igual. A selecao da operacao e feita por um par de demultiplexadores (DMX) que distribuem os operandos para as unidades funcionais internas, cujas saidas sao consolidadas por um multiplexador final (MUX) que encaminha apenas o resultado relevante para a saida. Esta arquitetura permite que todas as operações sejam calculadas em paralelo a cada ciclo, mas apenas o resultado da operação selecionada é propagado para a saída.

### 3.3 - Arquitetura Interna

O subcircuito e organizado em tres estagios funcionais: distribuicao dos operandos, execucao paralela das operacoes e selecao do resultado.

### Estagio de distribuicao — DMX duplo

Dois demultiplexadores de 1:8 (um para o operando A e outro para o operando B) recebem o sinal Funcao (3 bits) como seletor. Cada DMX roteia o operando de 8 bits correspondente para a entrada da unidade funcional selecionada, mantendo as demais entradas em zero. Esta técnica de "distribuição por demultiplexação" garante que cada unidade funcional receba apenas os operandos que lhe dizem respeito, evitando que operações não selecionadas influenciem indevidamente o resultado final.

Os pares de saida do DMX sao nomeados de acordo com a operacao destino:
- ASoma / BSoma — alimentam o somador
- ASub / BSub — alimentam o subtrator
- AMult / BMult — alimentam o multiplicador
- ADiv / BDiv — alimentam o divisor
- ANeg — alimenta o negador (operando B nao e usado nesta operação)
- AComp / BComp — alimentam o comparador
- AShiftEsq — alimenta o deslocador para a esquerda
- AShiftDir — alimenta o deslocador para a direita

#### Estagio de execucao — unidades funcionais paralelas

Todas as unidades funcionais operam em paralelo a cada ciclo; apenas a saida selecionada pelo MUX final e utilizada. Isto significa que, embora o circuito consuma mais energia por calcular todas as operações simultaneamente, a latência é reduzida pois não há necessidade de ativar as unidades sequencialmente.

**Somador (ADD)**
- Entradas: ASoma, BSoma (8 bits cada)
- Saidas: resultado de 8 bits (A + B), carry out (vai para fora, não utilizado externamente)
- Implementado com somador ripple-carry de 8 bits do Logisim
- Utilizado pelas instruções: add, addi, lw, sw (cálculo de endereço)

**Subtrator (SUB)**
- Entradas: ASub, BSub (8 bits cada)
- Saidas: resultado de 8 bits (A - B), borrow out (não utilizado externamente)
- Utilizado pelas instruções: sub, beq (na comparação de igualdade indireta)

**Multiplicador (MULT)**
- Entradas: AMult, BMult (8 bits cada)
- Saidas: resultado de 8 bits (parte baixa do produto de 16 bits)
- Somente os 8 bits menos significativos do produto sao encaminhados ao MUX final. A parte alta (bits 15-8) é descartada internamente, o que significa que multiplicações cujo resultado ultrapassa 8 bits sofrem truncamento.
- Utilizado pela instrução: mult

**Divisor (DIV)**
- Entradas: ADiv, BDiv (8 bits cada)
- Saidas: quociente (8 bits), resto (não utilizado externamente)
- Divisão por zero (B = 0) gera resultado indefinido; o circuito não implementa tratamento de exceção.
- Utilizado pela instrução: div

**Negador / Complemento de dois (NOT)**
- Entrada: ANeg (8 bits)
- Saida: ~A (complemento bit a bit, não complemento de dois)
- Diferente da negação aritmética que seria -A (complemento de dois), esta operação implementa o complemento lógico bit a bit (inversão de bits).
- Utilizado pela instrução: not

**Comparador (SLT - Set Less Than)**
- Entradas: AComp, BComp (8 bits cada)
- Funcionamento: se A < B (comparação com sinal, ou sem sinal dependendo da implementação), a saída é 1; caso contrário, 0.
- O resultado (1 bit) é estendido para 8 bits (0x01 ou 0x00) antes de ser entregue ao MUX final.
- Utilizado pela instrução: slt

**Deslocador a esquerda (SLL - Shift Left Logical)**
- Entrada: AShiftEsq (8 bits)
- Saida: A deslocado 1 posicao a esquerda (equivale a multiplicar por 2)
- O bit menos significativo e preenchido com 0; o bit mais significativo é descartado.
- O campo shamt da instrução (3 bits) teoricamente permitiria deslocamentos de 0 a 7 posições, mas nesta implementação o deslocamento é fixo em 1 posição por simplicidade didática.
- Utilizado pela instrução: sll

**Deslocador a direita (SRL - Shift Right Logical)**
- Entrada: AShiftDir (8 bits)
- Saida: A deslocado 1 posicao a direita (equivale a divisao inteira por 2)
- O bit mais significativo e preenchido com 0 (shift logico, não aritmético); o bit menos significativo é descartado.
- Utilizado pela instrução: srl

#### Estagio de selecao — MUX final

Um multiplexador de 8 entradas x 8 bits recebe os resultados de todas as unidades funcionais. O sinal Funcao (3 bits) seleciona qual resultado e propagado para a saida R. A correspondencia entre Funcao e a entrada do MUX segue a mesma codificacao dos DMX de entrada, garantindo que o operando distribuido e o resultado selecionado sejam sempre da mesma operacao.

| Funcao (3 bits) | Entrada MUX selecionada |
|-----------------|------------------------|
| 000 | Somador |
| 001 | Subtrator |
| 010 | Multiplicador |
| 011 | Divisor |
| 100 | Negador |
| 101 | Comparador (SLT) |
| 110 | Deslocador esquerda |
| 111 | Deslocador direita |

#### 3.4 - Saida auxiliar — flag Igual

Alem do resultado R, a ULA produz um sinal adicional chamado Igual (1 bit) que indica se os operandos A e B são iguais. Este sinal é gerado por um circuito comparador de igualdade independente, que não depende do valor de Funcao e opera em paralelo às demais operações.

- Igual = 1 quando A == B
- Igual = 0 quando A != B

Este sinal e utilizado pelo bloco principal (main) para a instrucao de branch condicional (BEQ - branch if equal). Quando o sinal Branch da Unidade de Controle está ativo e Igual = 1, o desvio é tomado. A vantagem de ter um sinal de igualdade separado é que a ULA não precisa executar uma subtração para fazer a comparação, o que poderia alterar o resultado R em casos onde a operação selecionada não é SUB.

### 3.5 - Portas de Interface

| Porta | Largura (bits) | Direcao | Descricao |
|-------|----------------|---------|-----------|
| **A** | 8 | Entrada | Operando A — proveniente de Read data 1 do Banco de Registradores (conteúdo de rs) |
| **B** | 8 | Entrada | Operando B — proveniente de Read data 2 (rt) ou do imediato estendido, selecionado pelo sinal ALUSrc |
| **Funcao** | 3 | Entrada | Codigo de operacao gerado pelo Controle ULA; seleciona qual das 8 operações será executada |
| **R** | 8 | Saida | Resultado da operacao selecionada (8 bits) |
| **Igual** | 1 | Saida | Flag de igualdade: 1 se A == B; usado para instrucao BEQ |

### 3.6 - Tabela de Operacoes

| Funcao (3b) | Operacao | Instrucao | Descricao |
|-------------|----------|-----------|-----------|
| 000 | R = A + B | add, addi, lw, sw | Adicao inteira. Utilizada para operações aritméticas e cálculo de endereço de memória. |
| 001 | R = A - B | sub, beq | Subtracao inteira. Utilizada para comparação indireta (beq usa Igual, não o resultado). |
| 010 | R = A * B (low) | mult | Multiplicacao; R recebe os 8 bits menos significativos do produto de 16 bits. |
| 011 | R = A / B (quot) | div | Divisao inteira; R recebe o quociente. Divisão por zero gera resultado indefinido. |
| 100 | R = ~A | not | Negacao logica (complemento bit a bit). |
| 101 | R = 1 se A < B, senao 0 | slt | Comparacao com sinal; resultado estendido para 8 bits (0x00 ou 0x01). |
| 110 | R = A << 1 | sll | Deslocamento logico para a esquerda em 1 posicao (multiplicação por 2). |
| 111 | R = A >> 1 | srl | Deslocamento logico para a direita em 1 posicao (divisão por 2). |

**Observacao**: o sinal Igual e calculado independentemente do campo Funcao e reflete A == B para qualquer operacao ativa. Ele está sempre disponível, independentemente do que a ULA está calculando em R.



### 3.7 - Integracao no Datapath

No contexto do processador MIPS ciclo unico, a ULA se conecta aos seguintes blocos:

| Conexão | Origem/Destino | Finalidade |
|---------|----------------|------------|
| Entrada A | Read data 1 do Banco de Registradores (conteúdo de rs) | Primeiro operando da ULA |
| Entrada B | Saida do MUX ALUSrc (Read data 2 ou imediato estendido) | Segundo operando da ULA |
| Saida R | Write data do Banco de Registradores (via MUX MemToReg) | Resultado para instruções tipo R |
| Saida R | Endereço da Memoria de Dados | Cálculo de endereço para lw e sw |
| Saida Igual | Porta AND (com Branch) no main | Decisão de desvio condicional |
| Entrada Funcao | Saida do Controle ULA | Seleção da operação a ser executada |





### 4 - Unidade de Controle


### 4.1 - Identificacao do Componente

| Campo | Informação |
|-------|-------------|
| **Nome do subcircuito** | Unidade de Controle |
| **Projeto** | mips  |
| **Ferramenta** | Logisim 2.7.1 |
| **Arquitetura** | MIPS ciclo unico (single-cycle) |
| **Entrada principal** | opcode — 3 bits (17-15)|
| **Numero de saidas** | 9 sinais de controle |
| **Tipo do circuito** | Combinacional puro — logica de portas AND/OR/NOT |
| **Tecnica de decodificacao** | Logica de portas individuais por sinal de saida (PLA-like) |

### 4.2 - Visao Geral e Funcao

A Unidade de Controle e o bloco responsavel por interpretar o campo opcode de cada instrucao e gerar os sinais de controle que governam o comportamento de todos os outros subcircuitos do processador (Banco de Registradores, ULA, Memoria de Dados e multiplexadores do datapath). Em um processador MIPS ciclo unico, este bloco opera de forma puramente combinacional: dado o opcode, os sinais de controle sao estabelecidos imediatamente, sem esperar o clock. Isto significa que no mesmo ciclo de clock em que a instrução é buscada da memória, os sinais de controle já estão disponíveis para os demais componentes.

O principio de funcionamento observado no circuito e uma decodificacao do tipo PLA (Programmable Logic Array) simplificada: cada sinal de saida e produzido por uma porta logica (AND, OR ou NOT) que recebe combinacoes dos bits do opcode. Nao ha ROM de microcodigo nem decodificador de prioridade — cada saida tem sua propria equacao logica minima implementada diretamente em portas. Esta abordagem é eficiente para um conjunto pequeno de instruções (13 instruções no total, com 6 opcodes distintos utilizados).

### 4.3 -  Arquitetura Interna

### Estrutura geral
O circuito recebe o opcode de 3 bits (bits 17-15 da instrução) e distribui cada bit (e suas inversoes) para um conjunto de portas logicas. Cada sinal de controle de saida e gerado por uma unica porta AND (ou combinação de AND e OR) multi-entrada que combina os bits do opcode necessarios para identificar as instrucoes que ativam aquele sinal.

#### Descricao detalhada de cada sinal de controle

**RegDst (Registro Destino)**
- Ativo quando a instrucao e do tipo R (opcode = 000).
- Determina que o registrador de destino da escrita no Banco de Registradores e o campo rd (bits 8-6 da instrucao), e nao o campo rt (bits 11-9).
- RegDst = 1: destino e rd (instrucoes tipo R)
- RegDst = 0: destino e rt (instrucoes tipo I como lw e addi)

**RegWrite (Escrita no Banco de Registradores)**
- Ativo para instrucoes que escrevem no Banco de Registradores: tipo R (opcode=000), lw (opcode=001) e addi (opcode=100).
- Inativo para store (sw, opcode=010), branch (beq, opcode=011) e jump (jmp, opcode=111).
- RegWrite = 1: habilita escrita no Banco de Registradores
- RegWrite = 0: Banco de Registradores em modo somente leitura

**ALUSrc (Origem do segundo operando da ULA)**
- Seleciona a segunda entrada da ULA.
- Para instrucoes tipo R (opcode=000), B vem do Banco de Registradores (Read data 2, conteúdo de rt).
- Para instrucoes tipo I (lw, sw, addi), B vem do imediato estendido (campo address/imm da instrução).
- ALUSrc = 0: operando B e Read data 2 (tipo R, beq)
- ALUSrc = 1: operando B e imediato estendido (lw, sw, addi)

**MemWrite (Escrita na Memoria de Dados)**
- Ativo exclusivamente para a instrucao store (sw, opcode=010).
- Habilita a escrita na Memoria de Dados no endereco calculado pela ULA.
- MemWrite = 1: escrita na Memoria de Dados habilitada
- MemWrite = 0: Memoria de Dados em modo leitura

**MemRead (Leitura da Memoria de Dados)**
- Ativo exclusivamente para a instrucao load (lw, opcode=001).
- Habilita a leitura da Memoria de Dados. O dado lido será posteriormente escrito no Banco de Registradores.
- MemRead = 1: leitura da Memoria de Dados habilitada
- MemRead = 0: saida da Memoria de Dados ignorada pelo datapath

**MemToReg (Origem do dado a ser escrito no Banco)**
- Seleciona a origem do dado a ser escrito no Banco de Registradores.
- Ativo para lw (opcode=001): o dado vem da Memoria de Dados.
- Inativo para tipo R e addi: o dado vem da ULA.
- MemToReg = 1: Write data = saida da Memoria de Dados (lw)
- MemToReg = 0: Write data = resultado da ULA (tipo R, addi)

**Branch (Desvio Condicional)**
- Ativo para a instrucao BEQ (branch if equal, opcode=011).
- Quando Branch = 1 e o sinal Igual da ULA tambem e 1, o proximo PC recebe o endereco de desvio calculado (PC + imediato). Caso contrario (Branch=0 ou Igual=0), o fluxo sequencial e mantido (PC+1).
- Branch = 1: desvio condicional habilitado (a efetivacao depende de Igual)
- Branch = 0: desvio condicional desabilitado

**Jump (Desvio Incondicional)**
- Ativo para a instrucao JMP (jump incondicional, opcode=111).
- Quando ativo, o PC recebe o endereco absoluto codificado no campo address da instrucao (bits 7-0 do formato J), ignorando Branch, Igual e o calculo de desvio relativo.
- Jump = 1: salto incondicional — PC recebe endereco absoluto
- Jump = 0: fluxo normal ou branch condicional

**ALUOp (Classe da Operacao para o Controle ULA)**
- Sinal de 2 bits que informa ao Controle ULA a classe da instrucao. O Controle ULA usa este sinal em conjunto com o campo funct (para instruções tipo R) para determinar o sinal Funcao (3 bits) da ULA.
- ALUOp = 10 (binario): instrucao tipo R (opcode=000) — o campo funct define a operacao
- ALUOp = 00: load (lw), store (sw) ou addi (opcodes 001, 010, 100) — a ULA realiza soma para calculo de endereço ou operação addi
- ALUOp = 01: branch (beq, opcode=011) — a ULA utiliza o sinal Igual (a subtração é calculada internamente, mas o resultado R não é usado)
- ALUOp = XX: jump (jmp, opcode=111) — valor irrelevante (don't care), pois a ULA não é usada

### 4.4 - Portas de Interface

| Porta | Largura (bits) | Direcao | Descricao |
|-------|----------------|---------|-----------|
| **opcode** | 3 | Entrada | Campo de operacao da instrucao (bits 17-15). Identifica o tipo da instrucao (000=R, 001=lw, 010=sw, 011=beq, 100=addi, 111=jmp). |
| **RegDst** | 1 | Saida | Seleciona o registrador de destino: 1 = rd (tipo R), 0 = rt (tipo I). |
| **RegWrite** | 1 | Saida | Habilita escrita no Banco de Registradores. |
| **ALUSrc** | 1 | Saida | Seleciona segunda entrada da ULA: 0 = Read data 2 (rt), 1 = imediato estendido. |
| **MemWrite** | 1 | Saida | Habilita escrita na Memoria de Dados (sw). |
| **MemRead** | 1 | Saida | Habilita leitura da Memoria de Dados (lw). |
| **MemToReg** | 1 | Saida | Seleciona fonte do Write data: 0 = resultado ULA, 1 = dado da memoria. |
| **Branch** | 1 | Saida | Habilita logica de desvio condicional (beq). |
| **Jump** | 1 | Saida | Habilita desvio incondicional (jmp). |
| **ALUOp** | 2 | Saida | Classe da operacao para o Controle ULA: 10=tipo R, 00=load/store/addi, 01=branch. |

### 4.5 -  Tabela de Decodificacao (opcode de 3 bits)

| opcode | Instrucao | RegDst | RegWrite | ALUSrc | MemWrite | MemRead | MemToReg | Branch | Jump | ALUOp |
|--------|-----------|--------|----------|--------|----------|---------|----------|--------|------|-------|
| 000 | R-type | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 10 |
| 001 | lw | 0 | 1 | 1 | 0 | 1 | 1 | 0 | 0 | 00 |
| 010 | sw | X | 0 | 1 | 1 | 0 | X | 0 | 0 | 00 |
| 011 | beq | X | 0 | 0 | 0 | 0 | X | 1 | 0 | 01 |
| 100 | addi | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 00 |
| 111 | jmp | X | 0 | X | 0 | 0 | X | 0 | 1 | XX |

**Legenda:** X = don't care (valor irrelevante; o circuito pode produzir 0 ou 1 sem afetar a execução)


### 4.6 -  Relacao com os Demais Subcircuitos

| Sinal | Origem | Destino(s) | Funcao no destino |
|-------|--------|------------|-------------------|
| RegDst | Unid. Controle | MUX RegDst (main) | Seleciona rd ou rt como Write register |
| RegWrite | Unid. Controle | Banco de Registradores | Habilita a porta de escrita |
| ALUSrc | Unid. Controle | MUX ALUSrc (main) | Seleciona Read data 2 ou imediato como operando B da ULA |
| MemWrite | Unid. Controle | Memoria de Dados | Habilita a escrita na memoria (sw) |
| MemRead | Unid. Controle | Memoria de Dados | Habilita a leitura da memoria (lw) |
| MemToReg | Unid. Controle | MUX MemToReg (main) | Seleciona resultado ULA ou dado da memoria como Write data |
| Branch | Unid. Controle | Porta AND (main) | Combinado com Igual da ULA para decidir desvio |
| Jump | Unid. Controle | MUX PC (main) | Forca o PC a receber endereco absoluto |
| ALUOp | Unid. Controle | Controle ULA | Indica a classe da instrucao; o Controle ULA combina com funct |



#### 4.7 - Sinais don't care
Para instrucoes que nao usam determinados recursos (ex.: RegDst em store e branch, MemToReg em tipo R), os sinais correspondentes podem assumir qualquer valor sem afetar o resultado. No circuito Logisim, os valores não conectados produzem 0 por padrão, o que é funcionalmente correto para esta implementacao. O projetista deve garantir que os multiplexadores e componentes que recebem sinais don't care não dependam desses valores para funcionar corretamente.

---

# 5. Controle ULA
## Documentacao Tecnica do Subcircuito

### 1. Identificacao do Componente

| Campo | Informação |
|-------|-------------|
| **Nome do subcircuito** | Controle ULA |
| **Projeto pai** | mips (3) |
| **Ferramenta** | Logisim (logisim.app) |
| **Entradas** | ALUOp (2 bits), funct (3 bits) |
| **Saida** | Funcao (3 bits) para a ULA |
| **Tipo do circuito** | Combinacional puro |

### 2. Visao Geral e Funcao

O subcircuito **Controle ULA** e responsavel por gerar o sinal de selecao da operacao que a ULA deve executar. Ele recebe informacoes da **Unidade de Controle principal** (sinal ALUOp, 2 bits) e, quando necessario, o campo `funct` da instrucao (3 bits, bits 2-0 da instrução, presente apenas em instrucoes do tipo R), e produz um codigo de operacao (`Funcao`, 3 bits) que define exatamente o que a ULA deve fazer: somar, subtrair, multiplicar, dividir, negar, comparar, deslocar à esquerda ou à direita.

A lógica do Controle ULA é relativamente simples porque a Unidade de Controle principal já faz uma pré-classificação via ALUOp:
- Se ALUOp = 00 (instruções lw, sw, addi): a ULA deve sempre fazer ADD (cálculo de endereço ou soma com imediato).
- Se ALUOp = 01 (instrução beq): a ULA deve fazer SUB (para comparação de igualdade, embora o resultado R não seja usado, o sinal Igual é gerado independentemente).
- Se ALUOp = 10 (instruções tipo R): a ULA deve executar a operação especificada pelo campo funct.

Esta decomposição simplifica enormemente o circuito: apenas para o caso ALUOp = 10 é necessário examinar o funct; nos demais casos, a saída é constante.

### 3. Arquitetura Interna e Lógica de Operacao

O Controle ULA segue uma lógica de decisão baseada nos dois sinais de entrada:

```
SE (ALUOp = 00) ENTÃO
    Funcao = 000  (ADD)   // usado por lw, sw, addi
SE NÃO SE (ALUOp = 01) ENTÃO
    Funcao = 001  (SUB)   // usado por beq
SE NÃO SE (ALUOp = 10) ENTÃO
    // Instrução do tipo R – depende do funct
    ESCOLHA funct (3 bits):
        CASO 000 (add):  Funcao = 000 (ADD)
        CASO 001 (sub):  Funcao = 001 (SUB)
        CASO 010 (mult): Funcao = 010 (MULT)
        CASO 011 (div):  Funcao = 011 (DIV)
        CASO 100 (not):  Funcao = 100 (NOT)
        CASO 101 (slt):  Funcao = 101 (SLT)
        CASO 110 (sll):  Funcao = 110 (SLL)
        CASO 111 (srl):  Funcao = 111 (SRL)
```

### 4. Tabela Verdade Completa

| ALUOp1 | ALUOp0 | Instrucao | funct (binário, 3 bits) | Funcao (saída) | Operacao da ULA |
|--------|--------|-----------|-------------------------|----------------|-----------------|
| 0 | 0 | lw / sw / addi | XXX | 000 | ADD |
| 0 | 1 | beq | XXX | 001 | SUB |
| 1 | 0 | R-type | 000 (add) | 000 | ADD |
| 1 | 0 | R-type | 001 (sub) | 001 | SUB |
| 1 | 0 | R-type | 010 (mult) | 010 | MULT |
| 1 | 0 | R-type | 011 (div) | 011 | DIV |
| 1 | 0 | R-type | 100 (not) | 100 | NOT |
| 1 | 0 | R-type | 101 (slt) | 101 | SLT |
| 1 | 0 | R-type | 110 (sll) | 110 | SLL |
| 1 | 0 | R-type | 111 (srl) | 111 | SRL |
| 1 | 1 | (não utilizado) | XXX | XXX | Indefinido |

**Legenda:** XXX = qualquer valor (don't care)

### 5. Equacoes Logicas Inferidas

Para o circuito combinacional, as equações podem ser expressas como:

| Funcao bit | Equação lógica |
|------------|----------------|
| Funcao[2] (MSB) | (ALUOp[1] . ALUOp[0]' . funct[2]) + (ALUOp[1] . ALUOp[0]' . funct[1] . funct[0]) + ... |
| Funcao[1] | (ALUOp[1] . ALUOp[0]' . funct[1]) + (ALUOp[1] . ALUOp[0]' . funct[2]' . funct[0]) + ... |
| Funcao[0] (LSB) | (ALUOp[1] . ALUOp[0]' . funct[0]) + (ALUOp[0]) + ... |

Na prática, uma implementação mais simples no Logisim pode utilizar um pequeno decodificador para o campo funct, combinado com um multiplexador que seleciona entre os valores constantes (para ALUOp=00 e 01) e a saída do decodificador (para ALUOp=10). Esta abordagem é mais fácil de visualizar e depurar.

### 6. Exemplos de Operacao

**Exemplo 1: Instrucao `lw R1, 2(R2)`**
- ALUOp = 00 (Unidade de Controle identifica lw)
- funct (ignorado) = XXX
- Funcao = 000 (ADD)
- A ULA executa ADD (R2 + 2) para calcular o endereço de memória

**Exemplo 2: Instrucao `beq R1, R2, label`**
- ALUOp = 01 (Unidade de Controle identifica beq)
- funct (ignorado) = XXX
- Funcao = 001 (SUB)
- A ULA executa SUB (R1 - R2); o resultado R é descartado, mas o sinal Igual é gerado

**Exemplo 3: Instrucao `add R1, R2, R3`**
- ALUOp = 10 (tipo R)
- funct = 000 (add)
- Funcao = 000 (ADD)
- A ULA executa ADD (R2 + R3) e o resultado é escrito em R1

**Exemplo 4: Instrucao `mult R1, R2, R3`**
- ALUOp = 10 (tipo R)
- funct = 010 (mult)
- Funcao = 010 (MULT)
- A ULA executa MULT (R2 * R3) e os 8 bits baixos são escritos em R1

**Exemplo 5: Instrucao `sll R1, R2, N`**
- ALUOp = 10 (tipo R)
- funct = 110 (sll)
- Funcao = 110 (SLL)
- A ULA executa SLL (R2 << 1) e o resultado é escrito em R1 (o campo shamt é ignorado)

### 7. Integracao no Processador

```
Unidade de Controle ──(ALUOp [1:0])──► Controle ULA
                                              │
Instrucao[2:0] ───────(funct [2:0])──► Controle ULA
                                              │
                                              ▼ (Funcao [2:0])
                                              │
                                              ▼
                                            ULA
```

**Fluxo da informação:**
1. A **Unidade de Controle principal** lê o opcode da instrução (bits 17-15)
2. Ela gera `ALUOp` (2 bits) e envia ao Controle ULA
3. Se a instrução for do tipo R (ALUOp = 10), o campo `funct` (bits 2-0) também é enviado ao Controle ULA
4. O Controle ULA combina essas informações e gera `Funcao` (3 bits)
5. A ULA recebe `Funcao` e executa a operação correspondente

### 8. Observacoes Importantes

- O subcircuito e **puramente combinacional** (não possui elementos de memória/clock). Isto significa que a saída Funcao responde imediatamente a qualquer alteração nas entradas ALUOp ou funct.
- O sinal `ALUOp = 11` não é utilizado no conjunto de instruções implementado (opcode 101 e 110 não são usados, e jmp tem ALUOp = XX).
- Todos os 8 códigos de funct (000 a 111) são mapeados para as 8 operações da ULA, cobrindo exatamente as instruções R-type implementadas.
- O Controle ULA não interage diretamente com o sinal `contRT` (conteúdo de rt), pois `contRT` é um dado que flui do Banco de Registradores para a ULA e Memória, não um sinal de controle.

---

5. main (Datapath Completo)
Documentacao Tecnica do Subcircuito

1. Identificacao do Componente


O datapath e responsável por:
- **Buscar a instrucao** na Memoria de Instrucoes, utilizando o PC (Program Counter) como endereço
- **Decodificar a instrucao**, extraindo os campos opcode, rs, rt, rd, shamt, funct e address/imm
- **Ler os registradores fonte** (rs e rt) no Banco de Registradores
- **Executar a operacao** na ULA (ADD, SUB, MULT, DIV, NOT, SLT, SLL, SRL)
- **Acessar a Memoria de Dados** (para instruções lw e sw)
- **Escrever o resultado** no Banco de Registradores (para instruções que escrevem)
- **Atualizar o PC** (para sequencial, desvio condicional beq, ou salto incondicional jmp)

### 3. Formatos de Instrucao (18 bits)

A instrucao de 18 bits e decomposta em campos conforme o formato da instrucao. O processador suporta tres formatos: R (registrador), I (imediato) e J (jump).

#### 3.1 Formato R
Utilizado por instrucoes que manipulam apenas registradores: add, sub, mult, div, not, slt, sll, srl.

| Bits | Campo | Tamanho | Descricao |
|------|-------|---------|-----------|
| 17-15 | opcode | 3 bits | Codigo da operacao (sempre 000 para tipo R) |
| 14-12 | rs | 3 bits | Endereco do primeiro registrador fonte |
| 11-9 | rt | 3 bits | Endereco do segundo registrador fonte |
| 8-6 | rd | 3 bits | Endereco do registrador destino |
| 5-3 | shamt | 3 bits | Quantidade de deslocamento (shift amount) para instrucoes sll/srl |
| 2-0 | funct | 3 bits | Codigo da funcao (especifica a operacao: 000=add, 001=sub, 010=mult, 011=div, 100=not, 101=slt, 110=sll, 111=srl) |

#### 3.2 Formato I
Utilizado por instrucoes que envolvem um valor imediato ou acesso a memoria: lw, sw, beq, addi.

| Bits | Campo | Tamanho | Descricao |
|------|-------|---------|-----------|
| 17-15 | opcode | 3 bits | Codigo da operacao (001=lw, 010=sw, 011=beq, 100=addi) |
| 14-12 | rs | 3 bits | Endereco do registrador base (para lw/sw) ou primeiro operando (beq/addi) |
| 11-9 | rt | 3 bits | Endereco do registrador destino (lw/addi) ou segundo operando (beq) ou origem (sw) |
| 8 | x | 1 bit | Don't care (nao utilizado) |
| 7-0 | address/imm | 8 bits | Endereco (para lw/sw) ou constante (para addi) ou deslocamento (para beq) |

#### 3.3 Formato J
Utilizado pela instrucao de desvio incondicional: jmp.

| Bits | Campo | Tamanho | Descricao |
|------|-------|---------|-----------|
| 17-15 | opcode | 3 bits | Codigo da operacao (111 para jmp) |
| 14-8 | xxxxxxx | 7 bits | Don't cares (nao utilizados) |
| 7-0 | address | 8 bits | Endereco absoluto do salto |

### 4. Componentes do Datapath

O datapath completo e composto pelos seguintes componentes, interligados por barramentos de 8 bits (dados) e 3 bits (enderecos de registradores):

#### 4.1 Componentes sequenciais (sincronizados pelo clock)

| Componente | Largura | Funcao |
|------------|---------|--------|
| **PC (Program Counter)** | 8 bits | Registrador que armazena o endereco da instrucao atual. Atualizado na borda de subida do clock. Possui sinal CLEAR assincrono para zerar o processador no inicio da simulacao. |
| **Memoria de Instrucoes** | 18 bits (instrucao) | ROM que armazena o programa a ser executado. Recebe o endereco de 8 bits do PC e entrega a instrucao de 18 bits correspondente. |
| **Memoria de Dados** | 8 bits (dado) | RAM que armazena dados. Endereco (8 bits) fornecido pela ULA. Escrita habilitada por MemWrite, leitura habilitada por MemRead. Sincronizada pelo clock para escrita. |
| **Banco de Registradores** | 8 registradores x 8 bits | Armazena os dados dos registradores R0 a R7. Leitura assincrona, escrita sincrona na borda do clock quando RegWrite = 1. |

#### 4.2 Componentes combinacionais

| Componente | Largura | Funcao |
|------------|---------|--------|
| **Unidade de Controle** | - | Recebe opcode (3 bits) da instrucao; gera 9 sinais de controle (RegDst, RegWrite, ALUSrc, MemWrite, MemRead, MemToReg, Branch, Jump, ALUOp) |
| **Controle ULA** | - | Recebe ALUOp (2 bits) e funct (3 bits); gera Funcao (3 bits) para a ULA |
| **ULA** | 8 bits | Executa operacao aritmetica ou logica conforme Funcao; produz Resultado (8 bits) e Igual (1 bit) |
| **Extensor de sinal** | 8 bits saída | Recebe o campo address/imm (8 bits) da instrucao I e estende para 8 bits (sem sinal, pois já tem 8 bits) |
| **Somador PC+1** | 8 bits | Incrementa o PC em 1 (para a proxima instrucao sequencial) |
| **Somador de branch** | 8 bits | Soma o PC com o campo address/imm (deslocamento) para calcular o endereco alvo do desvio condicional |
| **Gerador de endereco jump** | 8 bits | Concatena os bits superiores do PC com o campo address da instrucao J (ou simplesmente usa o campo address, dependendo da implementacao) |

#### 4.3 Multiplexadores

| MUX | Entradas | Saida | Controle | Funcao |
|-----|----------|-------|----------|--------|
| **MUX RegDst** | 0: rt (bits 11-9), 1: rd (bits 8-6) | Write register (3 bits) | RegDst | Seleciona o registrador destino: rt para I-type, rd para R-type |
| **MUX ALUSrc** | 0: Read data 2 (contRT), 1: imediato estendido (8 bits) | Operando B da ULA | ALUSrc | Seleciona segunda entrada da ULA |
| **MUX MemToReg** | 0: Resultado da ULA, 1: Dado lido da Memoria de Dados | Write data do Banco de Registradores | MemToReg | Seleciona a origem do dado a ser escrito no Banco |
| **MUX PC (3 entradas)** | 0: PC+1, 1: Endereco do branch, 2: Endereco do jump | Proximo PC | Branch+Igual, Jump | Seleciona o proximo valor do PC: sequencial, branch ou jump |

### 5. Fluxo de Dados por Tipo de Instrucao

#### 5.1 Instrucao tipo R (ex: `add rd, rs, rt`)

1. **Busca:** PC → Memoria de Instrucoes → instrucao (opcode=000, rs, rt, rd, shamt, funct)
2. **Decodificacao:** Unidade de Controle recebe opcode=000 → gera RegDst=1, RegWrite=1, ALUSrc=0, ALUOp=10
3. **Leitura de registradores:** Banco de Registradores recebe Read register 1=rs, Read register 2=rt → Read data 1 = rs, Read data 2 = rt
4. **Selecao do operando B:** MUX ALUSrc (ALUSrc=0) → operando B = Read data 2 (rt)
   - O sinal `contRT` (conteúdo de rt) flui através deste MUX
5. **Controle ULA:** ALUOp=10, funct (bits 2-0) → Funcao (000 para add, 001 para sub, etc.)
6. **Execucao na ULA:** A=rs, B=rt, Funcao → Resultado = rs op rt, Igual = (rs == rt)
7. **Selecao do destino:** MUX RegDst (RegDst=1) → Write register = rd
8. **Selecao do dado de escrita:** MUX MemToReg (MemToReg=0) → Write data = Resultado da ULA
9. **Escrita:** Banco de Registradores escreve Write data em rd na borda do clock
10. **Atualizacao do PC:** MUX PC (Jump=0, Branch=0) → PC = PC + 1

#### 5.2 Instrucao lw (ex: `lw rt, N(rs)`)

1. **Busca:** PC → Memoria Instrucoes → instrucao (opcode=001, rs, rt, x, address/imm=N)
2. **Decodificacao:** Unidade de Controle → RegDst=0, RegWrite=1, ALUSrc=1, MemRead=1, MemToReg=1, ALUOp=00
3. **Leitura de registradores:** Banco de Registradores → Read data 1 = rs (endereco base), Read data 2 = rt (ignorado)
4. **Selecao do operando B:** MUX ALUSrc (ALUSrc=1) → operando B = imediato estendido (N)
5. **Controle ULA:** ALUOp=00 → Funcao = 000 (ADD) independente do funct
6. **Execucao na ULA:** A=rs, B=N → Resultado = rs + N (endereco de memoria)
7. **Acesso a memoria:** Memoria de Dados (MemRead=1) → Dado lido = Mem[rs + N]
8. **Selecao do destino:** MUX RegDst (RegDst=0) → Write register = rt
9. **Selecao do dado de escrita:** MUX MemToReg (MemToReg=1) → Write data = Dado da memoria
10. **Escrita:** Banco de Registradores escreve dado da memoria em rt na borda do clock
11. **Atualizacao do PC:** PC = PC + 1

#### 5.3 Instrucao sw (ex: `sw rt, N(rs)`)

1. **Busca:** PC → Memoria Instrucoes → instrucao (opcode=010, rs, rt, x, address/imm=N)
2. **Decodificacao:** Unidade de Controle → RegWrite=0, ALUSrc=1, MemWrite=1, ALUOp=00
3. **Leitura de registradores:** Banco de Registradores → Read data 1 = rs (endereco base), Read data 2 = rt (dado a ser escrito)
4. **Selecao do operando B:** MUX ALUSrc (ALUSrc=1) → operando B = imediato estendido (N)
5. **Controle ULA:** ALUOp=00 → Funcao = 000 (ADD)
6. **Execucao na ULA:** A=rs, B=N → Resultado = rs + N (endereco de memoria)
7. **Acesso a memoria:** Memoria de Dados (MemWrite=1) → Mem[rs + N] = Read data 2 (rt)
8. **Nao ha escrita no Banco de Registradores (RegWrite=0)**
9. **Atualizacao do PC:** PC = PC + 1

#### 5.4 Instrucao beq (ex: `beq rs, rt, label`)

1. **Busca:** PC → Memoria Instrucoes → instrucao (opcode=011, rs, rt, x, address/imm=deslocamento)
2. **Decodificacao:** Unidade de Controle → ALUSrc=0, Branch=1, ALUOp=01
3. **Leitura de registradores:** Banco de Registradores → Read data 1 = rs, Read data 2 = rt
4. **Selecao do operando B:** MUX ALUSrc (ALUSrc=0) → operando B = Read data 2 (rt)
5. **Controle ULA:** ALUOp=01 → Funcao = 001 (SUB)
6. **Execucao na ULA:** A=rs, B=rt → Igual = (rs == rt) ? 1 : 0 (Resultado descartado)
7. **Logica de desvio:** Branch AND Igual = tomado se Branch=1 e Igual=1
8. **Calculo do endereco alvo:** Somador branch → PC + deslocamento
9. **Atualizacao do PC:** MUX PC: se tomado → endereco branch; senao → PC+1
10. **Nao ha escrita no Banco de Registradores (RegWrite=0)**

#### 5.5 Instrucao jmp (ex: `jmp endereco`)

1. **Busca:** PC → Memoria Instrucoes → instrucao (opcode=111, xxxxxxx, address)
2. **Decodificacao:** Unidade de Controle → Jump=1
3. **Leitura de registradores:** Ignorada (RegWrite=0, ALUSrc=X)
4. **Controle ULA:** ALUOp=XX (ignorado, ULA nao utilizada)
5. **Geracao do endereco jump:** endereco = campo address (bits 7-0)
6. **Atualizacao do PC:** MUX PC (Jump=1) → PC = endereco
7. **Nao ha acesso a memoria, ULA ou Banco de Registradores**

### 6. Caminho Critico e Temporizacao

Em um processador ciclo unico, o caminho critico determina o periodo minimo de clock. A instrucao que geralmente define o pior caso é **lw** (load word), pois utiliza quase todos os componentes:

```
Caminho critico para lw:

1. Borda do clock anterior (PC ja esta estavel)
2. Acesso a Memoria de Instrucoes (ROM) → instrucao disponivel
3. Propagacao pela Unidade de Controle (logica combinacional)
4. Acesso ao Banco de Registradores (leitura combinacional de rs)
5. Propagacao pelo extensor de sinal (imediato)
6. Propagacao pelo MUX ALUSrc (seleciona imediato)
7. Propagacao pelo Controle ULA (gera Funcao=000)
8. Execucao na ULA (operacao ADD, propagacao pelo somador)
9. Acesso a Memoria de Dados (leitura RAM)
10. Propagacao pelo MUX MemToReg (seleciona dado da memoria)
11. Setup time do Banco de Registradores (escrita na borda seguinte)
```

O periodo de clock deve ser maior ou igual a soma dos atrasos de todos esses componentes. Operacoes mais lentas (multiplicacao, divisao) podem aumentar o caminho critico para instrucoes R-type.

### 7. Tabela de Instrucoes Suportadas

| Instrucao | opcode | funct | Formato | Descricao | Exemplo |
|-----------|--------|-------|---------|-----------|---------|
| add | 000 | 000 | R | rd = rs + rt | add R1, R2, R3 |
| sub | 000 | 001 | R | rd = rs - rt | sub R1, R2, R3 |
| mult | 000 | 010 | R | rd = rs * rt (8 bits baixos) | mult R1, R2, R3 |
| div | 000 | 011 | R | rd = rs / rt (quociente) | div R1, R2, R3 |
| not | 000 | 100 | R | rd = ~rs | not R1, R2 |
| slt | 000 | 101 | R | rd = 1 se rs < rt, senao 0 | slt R1, R2, R3 |
| sll | 000 | 110 | R | rd = rs << 1 (shamt ignorado) | sll R1, R2, N |
| srl | 000 | 111 | R | rd = rs >> 1 (shamt ignorado) | srl R1, R2, N |
| lw | 001 | XXX | I | rt = Mem[rs + imm] | lw R1, 2(R2) |
| sw | 010 | XXX | I | Mem[rs + imm] = rt | sw R1, 2(R2) |
| beq | 011 | XXX | I | if rs == rt then PC += imm | beq R1, R2, label |
| addi | 100 | XXX | I | rt = rs + imm | addi R1, R2, 5 |
| jmp | 111 | XXX | J | PC = address | jmp 0x20 |

### 8. Constantes e Configuracoes do Projeto

| Constante | Valor | Descricao |
|-----------|-------|-----------|
| **Largura do PC** | 8 bits | Contador de programa |
| **Tamanho da Memoria de Instrucoes** | 256 x 18 bits | Capacidade maxima de 256 instrucoes de 18 bits |
| **Tamanho da Memoria de Dados** | 256 x 8 bits | Capacidade maxima de 256 bytes |
| **Numero de registradores** | 8 (R0 a R7) | Enderecado por 3 bits (rs, rt, rd) |
| **Incremento do PC** | 1 (byte) | Diferente do MIPS padrao que incrementa de 4 em 4 (palavra) |
| **Reset (CLEAR)** | Ativo em alto | Zera PC e todos os registradores |
| **Instrucao** | 18 bits | Mais larga que os dados (8 bits) |

### 9. Observacoes de Implementacao

#### 9.1 Diferencas do MIPS padrao

Esta implementacao didatica apresenta as seguintes simplificacoes em relacao ao MIPS padrao (32 bits):

| Caracteristica | MIPS padrao | Este projeto |
|----------------|-------------|---------------|
| Largura do dado | 32 bits | 8 bits |
| Largura da instrucao | 32 bits | 18 bits |
| Numero de registradores | 32 | 8 |
| Bits de opcode | 6 | 3 |
| Bits de endereco de registrador | 5 | 3 |
| Instrucoes | +100 | 13 |
| Pipeline | normalmente sim | ciclo unico |
| Registrador $zero | hardwired para 0 | nao implementado (R0 pode ser escrito) |
| Deslocamento (shift) | variavel (shamt de 5 bits) | fixo em 1 bit (shamt ignorado) |

#### 9.2 Sinal contRT (conteudo de rt)

O sinal identificado no circuito como **contRT** e simplesmente o valor de Read data 2 do Banco de Registradores, ou seja, o conteudo do registrador apontado pelo campo rt da instrucao. Este sinal e roteado para:
- Entrada do MUX ALUSrc (que pode encaminha-lo para a ULA)
- Entrada de escrita da Memoria de Dados (para instrucao sw)

Nao se trata de um sinal de controle, mas sim de um dado que trafega pelo datapath.

#### 9.3 Campo shamt

Embora o campo shamt (3 bits) esteja presente no formato R, a implementacao atual da ULA ignora este valor, realizando sempre deslocamento de 1 posicao para as instrucoes sll e srl. Uma melhoria futura seria conectar o campo shamt ao bloco de deslocamento para permitir deslocamentos variaveis de 0 a 7 posicoes.

#### 9.4 Verificacao e Testes

Para testar o processador, recomenda-se:
1. Carregar um programa simples na Memoria de Instrucoes (ex: soma de dois numeros)
2. Inicializar a Memoria de Dados com valores de teste
3. Aplicar CLEAR = 1 por um pulso, depois CLEAR = 0
4. Habilitar o clock (passo a passo via "Tick" no Logisim)
5. Observar a execucao instrucao por instrucao, verificando os valores dos registradores e da memoria

#### 9.5 Possiveis Aprimoramentos

- Implementar o registrador `$zero` (R0 permanentemente zero)
- Conectar o campo shamt ao deslocador da ULA
- Adicionar suporte a mais instrucoes (bne, jal, jr)
- Implementar pipeline (estagios IF, ID, EX, MEM, WB)
- Adicionar sinal de overflow na ULA
- Tratar divisao por zero e outros erros
