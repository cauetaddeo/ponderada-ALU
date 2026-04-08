# ponderada-ALU



# Unidade Logica e Aritmetica de 8 Bits (ALU)

## Descricao

Este projeto implementa uma Unidade Logica e Aritmetica (ALU) de 8 bits construida a partir de componentes digitais basicos. A ALU e capaz de executar oito operacoes distintas sobre operandos de 8 bits, selecionadas por sinais de controle externos. O projeto foi desenvolvido como atividade pratica de organizacao e arquitetura de computadores.

---

## Operacoes Suportadas

| Sinal de Controle | Operacao         | Descricao                                          |
|-------------------|------------------|----------------------------------------------------|
| SOMA              | A + B            | Adicao de dois operandos de 8 bits com carry in    |
| SUBTRACAO         | A - B            | Subtracao em complemento de dois                   |
| MULTIPLICACAO     | A x B            | Multiplicacao, resultado de 16 bits                |
| DIVISAO           | A / B            | Divisao com restore, saidas quociente e resto      |
| SHIFT CIMA        | A << 1           | Deslocamento logico para a esquerda                |
| SHIFT BAIXO       | A >> 1           | Deslocamento logico para a direita                 |
| NAND              | A NAND B         | NAND bit a bit de 8 bits                           |
| XOR               | A XOR B          | XOR bit a bit de 8 bits                            |

---

## Arquitetura

### Visao Geral

A ALU e organizada em torno de um barramento interno de 8 bits (BUS-8) que conecta todos os modulos de operacao. As entradas A e B sao carregadas no barramento via registradores de 8 bits com clock. O resultado de cada operacao e escrito de volta ao barramento e pode ser armazenado no acumulador (AC) ou encaminhado para a saida.

### Componentes Principais

#### Registrador Acumulador (AC)
Registrador de 8 bits com entrada pelo barramento e saida para os modulos de operacao. Armazena o operando A e acumula resultados intermediarios. Opera sincronamente com o sinal de CLOCK.

#### Registrador de Entrada B
Registrador de 8 bits que armazena o segundo operando B, alimentado diretamente pelo barramento. Fornece o valor B para todos os modulos de operacao simultaneamente.

#### MUX de Selecao de Operacao (8-BIT MUX 7-7 I8)
Multiplexador de 8 entradas de 8 bits que seleciona qual resultado de operacao sera encaminhado para a saida MQ e de volta ao barramento. O seletor e controlado pela combinacao de sinais de controle (SOMA, SUBTRACAO, MULTIPLICACAO, DIVISAO, SHIFT CIMA, SHIFT BAIXO, NAND, XOR).

#### MUX de Escrita no Barramento (8-BIT MUX I8)
Multiplexador que decide se o valor escrito no barramento vem do acumulador, de uma entrada externa ou de um resultado intermediario. O seletor e controlado por um AND entre o sinal de controle ativo e o clock.

---

### Modulos de Operacao

#### 8-BIT ADDER
Somador ripple-carry de 8 bits. Recebe A (do acumulador via barramento), B (do registrador B) e CIN (carry in externo). Produz 8 bits de soma e 1 bit de carry out (COUT). O resultado passa por um registrador de 8 bits e um registrador de 1 bit (para o COUT) antes de ser encaminhado ao MUX de saida.

#### 8-BIT ADD SUB
Modulo de adicao e subtracao de 8 bits com complemento de dois. O sinal SUBTRACT, gerado por uma porta NOT a partir do sinal de controle SUBTRACAO, controla os XOR gates internos que invertem o operando B quando a operacao e subtracao. O CIN e fixo em 1 durante a subtracao para completar o complemento de dois. Produz 8 bits de resultado e COUT.

#### 8-BIT MULTIPLIER
Multiplicador combinacional de 8 bits. Recebe A e B e produz um resultado de 16 bits (P15 a P0). A implementacao utiliza produtos parciais gerados por AND gates e somadores para acumular o resultado. As saidas de 16 bits sao distribuidas em dois registradores de 8 bits.

#### 8-BIT DIVISOR
Divisor com restauracao de 8 bits. Recebe o dividendo A e o divisor B e produz quociente (8 bits) e resto (8 bits). O modulo utiliza subtracao com restauracao: em cada etapa, o subtrator de 8 bits tenta subtrair o divisor do resto parcial; se o COUT for 1 (resultado positivo), o bit do quociente e 1 e o resto e atualizado; se o COUT for 0 (resultado negativo), o bit do quociente e 0 e o resto anterior e restaurado via MUX.

#### UP DOWN SHIFT
Modulo de deslocamento de 8 bits controlavel. O sinal SHIFT UP realiza deslocamento logico para a esquerda (equivalente a multiplicar por 2) e o sinal SHIFT DOWN realiza deslocamento logico para a direita (equivalente a divisao inteira por 2). Recebe 8 bits do barramento e produz 8 bits de saida.

#### NAND-8
Executa a operacao NAND bit a bit entre os 8 bits de A e os 8 bits de B. Produz 8 bits de resultado. A saida passa por um registrador de 8 bits antes de ir ao MUX de selecao.

#### XOR-8
Executa a operacao XOR bit a bit entre os 8 bits de A e os 8 bits de B. Produz 8 bits de resultado. A saida passa por um registrador de 8 bits antes de ir ao MUX de selecao.

---

### Sinais de Controle e Clock

Todos os sinais de controle (SOMA, SUBTRACAO, MULTIPLICACAO, DIVISAO, SHIFT CIMA, SHIFT BAIXO, NAND, XOR) sao entradas de 1 bit que ativam a operacao correspondente. Apenas um sinal deve estar ativo por vez. O sinal CIN e uma entrada de 1 bit de carry externo, utilizado pela operacao de soma.

O sinal CLOCK sincroniza a escrita nos registradores internos. A cada borda de clock com o sinal de controle correspondente ativo, o resultado da operacao selecionada e capturado no registrador de saida e disponibilizado no barramento.

---

### Saidas

| Saida | Descricao                                                           |
|-------|---------------------------------------------------------------------|
| MQ    | Resultado principal de 8 bits da operacao selecionada              |
| COUT  | Carry out de 1 bit das operacoes aritmeticas (soma e subtracao)    |

---

## Como Foi Feito

O projeto foi construido de forma incremental, partindo dos componentes mais simples:

1. O full adder de 1 bit foi o bloco base, conectado em cascata para formar o somador de 8 bits.
2. O subtrator foi construido sobre o somador, adicionando XOR gates no operando B e fixando CIN em 1 para o complemento de dois.
3. O multiplicador foi construido a partir de produtos parciais gerados por AND gates e somadores encadeados.
4. O divisor foi construido usando o subtrator de 8 bits com MUXes de restauracao.
5. Os modulos logicos (NAND, XOR) e de deslocamento foram implementados diretamente com as portas correspondentes.
6. Todos os modulos foram integrados ao barramento central com registradores de saida e um MUX de selecao de operacao controlado pelos sinais de controle.

---

# Vídeo explicando:

**Obs: Vídeo está começando após 5 segundos devido ao um erro de exportação**
[Vídeo Ponderada - ALU 8 bits](https://youtu.be/0URJs0TJyIU)



