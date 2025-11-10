# **Pipeline com Previsão de Desvios**  
### Relatório – Implementação da ISA REDUX V 

---

## **Integrantes**

## **Integrantes**

| Nome Completo | GRR | Responsável por |
|----------------|:----:|-----------------|
| João | GRR... | Implementação inicial da ISA REDUX V e integração dos módulos (ULA, Banco de Registradores, Memórias, Unidade de Controle) |
| Matheus | GRR... | Apoio na montagem da Unidade de Controle |
| Natan | GRR.. | Revisão da ISA e documentação do conjunto de instruções |
| Pedro | GRR... | Apoio na simulação e verificação dos sinais de controle |
| Rafael | GRR20244378 | Apoio nas análises e validação da lógica de controle |


---

## **1. Introdução**

Este relatório documenta a **primeira etapa** do projeto de um processador RISC de 8 bits baseado na arquitetura **REDUX V**, implementado no software **Logisim** (simulador de circuitos lógicos).  
Nesta fase, o processador ainda não é pipeline — o objetivo foi validar o funcionamento da **ISA (Instruction Set Architecture)** completa e garantir a execução correta de todas as instruções básicas, incluindo operações aritméticas, lógicas, de salto e de memória.

---

## **2. ISA REDUX V**

A **ISA (Instruction Set Architecture)** define o conjunto de instruções suportadas pelo processador, seus formatos e a semântica de execução.  
A REDUX V possui **instruções de 8 bits**, divididas em dois tipos principais:

| Tipo | Bits | Formato | Campos |
|------|------|----------|--------|
| **I (Imediato)** | 8 bits | `[7–4]: Opcode`  | `[3–0]: Imm (valor imediato)` |
| **R (Registrador)** | 8 bits | `[7–4]: Opcode`  | `[3–2]: Ra`  | `[1–0]: Rb` |

As instruções **Tipo R** operam sobre registradores do banco interno, enquanto as **Tipo I** utilizam valores imediatos embutidos na própria instrução.

---

## **3. Conjunto de Instruções (ISA)**

| Opcode | Tipo | Mnemonic | Nome | Operação |  
|:------:|:----:|:---------|:------|:-----------|  
| `0000` | R | **brzr** | Branch On Zero Register | if (R[ra] == 0) → PC = R[rb] |
| `0001` | I | **ji** | Jump Immediate | PC = PC + Imm |
| `0010` | R | **ld** | Load | R[ra] = M[R[rb]] |
| `0011` | R | **st** | Store | M[R[rb]] = R[ra] |
| `0100` | I | **addi** | Add Immediate | R[0] = R[0] + Imm |
| `1000` | R | **not** | Not | R[ra] = ¬R[rb] |
| `1001` | R | **and** | And | R[ra] = R[ra] & R[rb] |
| `1010` | R | **or** | Or | R[ra] = R[ra] ∨ R[rb] |
| `1011` | R | **xor** | Xor | R[ra] = R[ra] ⊕ R[rb] |
| `1100` | R | **add** | Add | R[ra] = R[ra] + R[rb] |
| `1101` | R | **sub** | Sub | R[ra] = R[ra] − R[rb] |
| `1110` | R | **slr** | Shift Left | R[ra] = R[ra] << R[rb] |
| `1111` | R | **srr** | Shift Right | R[ra] = R[ra] >> R[rb] |

---

## **4. Unidade de Controle**

A unidade de controle foi projetada para gerar os sinais que coordenam o fluxo de dados entre os blocos do processador:  
- **Banco de Registradores**  
- **ULA (Unidade Lógica e Aritmética)**  
- **Memória de Dados e de Instruções**  
- **Atualização do PC**

Cada instrução aciona uma combinação específica de sinais, conforme a tabela de controle abaixo.

---

### **Tabela de Sinais de Controle**

| Instrução | `pc_sel` | `reg_a` | `wbreg` | `mux_ula` | `dado_breg` | `wm` | `op_ula` |
|------------|:--------:|:-------:|:--------:|:-----------:|:-------------:|:-----:|:----------:|
| **brzr** (0000) | `10` | `0` | `0` | `0` | `0` | `0` | `000` |
| **ji** (0001) | `01` | `0` | `0` | `1` | `0` | `0` | `000` |
| **ld** (0010) | `00` | `1` | `1` | `0` | `1` | `0` | `000` |
| **st** (0011) | `00` | `0` | `0` | `0` | `0` | `1` | `000` |
| **addi** (0100) | `00` | `0` | `1` | `1` | `0` | `0` | `100` |
| **not** (1000) | `00` | `0` | `1` | `0` | `0` | `0` | `000` |
| **and** (1001) | `00` | `0` | `1` | `0` | `0` | `0` | `001` |
| **or** (1010) | `00` | `0` | `1` | `0` | `0` | `0` | `010` |
| **xor** (1011) | `00` | `0` | `1` | `0` | `0` | `0` | `011` |
| **add** (1100) | `00` | `0` | `1` | `0` | `0` | `0` | `100` |
| **sub** (1101) | `00` | `0` | `1` | `0` | `0` | `0` | `101` |
| **slr** (1110) | `00` | `0` | `1` | `0` | `0` | `0` | `110` |
| **srr** (1111) | `00` | `0` | `1` | `0` | `0` | `0` | `111` |

---

## **5. Circuitos Implementados**

### **5.1. Banco de Registradores**
O **Banco de Registradores** é composto por quatro registradores de 8 bits (R0–R3).  

**Entradas:** `DADO`, `CLK`, `WE`, `RS1`, `RS2`  
**Saídas:** `A` e `B`  
**Função:** Armazena valores intermediários e fornece operandos à ULA.  
**Controle:** O registrador a ser escrito é selecionado via decodificador; a leitura é feita por multiplexadores para as saídas `A` e `B`.

> 🔹 Cada registrador tem entrada de dado comum e controle de escrita individual.  
> 🔹 O sinal `YES_NO` define se escreve ou não.

---

### **5.2. ULA (Unidade Lógica e Aritmética)**
A **ULA** realiza as operações lógicas e aritméticas definidas pelo campo `op_ula` da unidade de controle.

| Operação | Sinal `op_ula` | Função |
|:-----------|:---------------:|:--------|
| `000` | NOT | Inverte os bits de `A` |
| `001` | AND | `A & B` |
| `010` | OR | `A ∨ B` |
| `011` | XOR | `A ⊕ B` |
| `100` | ADD | Soma (`A + B`) |
| `101` | SUB | Subtração (`A − B`) |
| `110` | SLR | Deslocamento lógico à esquerda |
| `111` | SRR | Deslocamento lógico à direita |

A ULA também gera o sinal **ZERO**, indicando se o resultado foi igual a zero — útil para instruções condicionais como `brzr`.

---

### **5.3. Unidade de Controle**
Implementada com base no **opcode** das instruções, a unidade gera todos os sinais de controle do processador.  
Seu papel é coordenar:
- **Seleção do PC** (`pc_sel`)
- **Fonte de dados da ULA** (`mux_ula`)
- **Escrita em registradores** (`wbreg`)
- **Leitura/Escrita de memória** (`wm`)
- **Operação da ULA** (`op_ula`)

---

### **5.4. Memória e Atualização de PC**
- O **PC (Program Counter)** é atualizado conforme o tipo de instrução (sequencial, salto ou branch).  
- O circuito possui um **MUX** para seleção da próxima instrução (`PC+1`, `PC+Imm` ou `R[rb]`).  
- A **memória de instruções** e a **memória de dados** são separadas, permitindo execução Harvard simples.

---

## **6. Considerações Finais (Parciais)**

Esta etapa conclui a implementação **completa da ISA REDUX V** no modelo sequencial.  
Todos os módulos funcionam corretamente em conjunto, e o processador é capaz de executar instruções básicas de:
- **Controle de fluxo (brzr, ji)**  
- **Acesso à memória (ld, st)**  
- **Operações aritméticas e lógicas (add, sub, and, or, xor, etc.)**

A próxima etapa consistirá em:
- Introduzir **pipeline de 5 estágios** (IF, ID, EX, MEM, WB).  
- Implementar **previsão de desvio (branch prediction)**.  
- Otimizar o controle de **hazards** e **forwarding**.

---


