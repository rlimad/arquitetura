# **Pipeline com Previsão de Desvios**  
### Relatório – Implementação da ISA REDUX V 

---

## **Integrantes**

| Nome Completo | GRR | 
|----------------|:----:|
| João Pedro dos Santos Felipe| GRR20243340 | 
| Mateus Henrique Cheirubim Santos | GRR20245274 |
| Natan Felipe Ziojlo | GRR20232355  |
| Pedro Lucas | GRR20232366 | 
| Rafael Lima Dias | GRR20244378 | 


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

## 7. Evolução para Pipeline com Previsão de Desvios

Depois de validar o processador sequencial da REDUX V, a segunda etapa do projeto consistiu em transformar essa arquitetura em um pipeline de 5 estágios com suporte a:

- Hazards de dados (especialmente *load–use*);
- Hazards de controle (desvios condicionais);
- Uma forma simples de previsão de desvio (*branch prediction*) baseada em uma FSM de 2 bits;
- Registradores de pipeline entre todos os estágios (`IF/ID`, `ID/EX`, `EX/MEM`, `MEM/WB`).

O objetivo foi manter exatamente a mesma ISA da primeira etapa, porém agora permitindo que várias instruções estejam em execução ao mesmo tempo em estágios diferentes do pipeline.

---

## 8. Organização Geral do Pipeline

### 8.1 Estágios

O processador passou a seguir o pipeline clássico de 5 estágios:

- **IF – Instruction Fetch**  
  Busca da instrução na memória e cálculo do próximo PC.

- **ID – Instruction Decode / Register Fetch**  
  Decodificação da instrução, leitura do banco de registradores e geração dos sinais de controle.

- **EX – Execute / Address Calculation**  
  Execução das operações da ULA ou cálculo de endereços de memória.

- **MEM – Memory Access**  
  Acesso à memória de dados para instruções `ld` e `st`.

- **WB – Write Back**  
  Escrita do resultado da ULA ou da memória no banco de registradores.

### 8.2 Caminho de Dados Global

Na `main_pipeline` (circuito principal):

- O **PC** alimenta a **ROM de instruções**.
- A saída da ROM passa pelo registrador **IF/ID**, que guarda:
  - Instrução atual (`INSTRU_OUT`);
  - PC da instrução (`PC_OUT`).

Em seguida, no estágio **ID**, a instrução é decodificada:

- A unidade de controle (ROM de 10 bits) gera os sinais:

  - `PC_SEL`
  - `A_OU_0`
  - `W_BREG`
  - `MUX_ULA`
  - `DADO_BREG`
  - `WM`
  - `OP_ULA`

- O banco de registradores (`BANCO_REGISTRADORES`) lê `RS1` e `RS2` e fornece operandos:

  - `REG_A`
  - `REG_B`

- O campo imediato é extraído e estendido para 8 bits (`IMM`).

Esses valores entram no registrador **ID/EX**.

No estágio **EX**:

- A ULA recebe `A`, `B` (ou imediato) e executa a operação selecionada por `OP_ULA`.
- Para `ld/st`, calcula-se o endereço de memória.
- Para `brzr`, ocorre o teste de condição (comparação com zero).

Os resultados passam para o registrador **EX/MEM**.

No estágio **MEM**:

- A RAM de dados é acessada com `ALU_OUT` como endereço.
- `STORE_OUT` define o dado a ser escrito em `st`.
- Controle de escrita é feito via `WM`.
- A saída da memória e o resultado da ULA são armazenados em **MEM/WB**.

No estágio **WB**:

- Um multiplexador escolhe entre:
  - Resultado da ULA;
  - Dado lido da memória;
- A escolha depende de `DADO_BREG`.

O resultado é escrito no banco de registradores em `RD` quando `W_BREG = 1`.

Paralelamente existe a **HAZARD_UNIT**, que observa:

- Registradores de origem/destino em ID e EX;
- Sinal de load no EX;
- Resultado da condição de desvio em EX;

E, a partir disso, gera:

- `PC_WRITE`;
- `IF_ID_WRITE`;
- `FLUSH_ID_EX`, `FLUSH_EX_MEM`, `FLUSH_MEM_WB`;
- `BRANCH_TAKEN_EX`;
- `PC_SEL_EX` (seleção de PC que corrige a predição ou o comportamento padrão).

---

## 9. Estágio IF e Registrador IF/ID

### 9.1 Estágio IF (*Instruction Fetch*)

No `main_pipeline` o estágio IF é formado por:

- Registrador **PC** (um `Register` de 8 bits);
- Dois somadores:
  - `PC + 1` – incremento sequencial para próxima instrução;
  - `PC + IMM` – para saltos imediatos (`ji`) e branches com deslocamento.
- Multiplexadores que escolhem a próxima origem do PC:
  - `PC + 1`;
  - `PC + IMM`;
  - Valor em um registrador (para `brzr` com destino via registrador).
- Sinal de controle `PC_SEL` (2 bits) vindo da unidade de controle/hazard.

Além disso, a unidade de hazards fornece:

- `PC_WRITE` – se vale 0, o PC não é atualizado (stall do fetch).

A instrução é buscada em uma **ROM de 8 bits** cujo endereço é o PC. A saída da ROM é `INSTRU`.

### 9.2 Registrador IF/ID

O circuito `IF_ID` é exatamente o registrador de pipeline entre IF e ID. Ele tem:

**Entradas:**

- `PC_IN[7:0]` – PC da instrução buscada;
- `INSTRU_IN[7:0]` – instrução lida da ROM;
- `CLOCK` – clock global;
- `ESCREVE_OU_NAO` – sinal de *write enable* (vem de `IF_ID_WRITE` da hazard unit).

**Saídas:**

- `PC_OUT[7:0]` – PC repassado ao estágio ID;
- `INSTRU_OUT[7:0]` – instrução repassada ao estágio ID.

**Internamente:**

- Dois registradores:
  - `PC_REG` – guarda o PC;
  - `IR_REG` – guarda a instrução.
- Ambos compartilham:
  - O clock;
  - O mesmo sinal de *write enable* (`ESCREVE_OU_NAO`).

**Comportamento:**

- Quando `ESCREVE_OU_NAO = 1` e há borda de subida do clock:
  - PC e instrução são latched normalmente.
- Quando `ESCREVE_OU_NAO = 0`:
  - Os registradores mantêm o valor anterior, travando o front-end (*stall*).

Esse mecanismo é usado para resolver o hazard de *load–use*.

---

## 10. Estágio ID e Registrador ID/EX

### 10.1 Banco de Registradores no Pipeline

O módulo `BANCO_REGISTRADORES` é o mesmo da etapa sequencial, agora alimentado por:

- `RD` vindo do estágio WB (via `MEM_WB`);
- `DADO` sendo o valor de *write-back* (resultado da ULA ou da memória);
- `YES_NO = W_BREG` (sinal de escrita em registrador);
- `RS1`, `RS2` – lidos da instrução em ID.

Ele fornece:

- `A` – conteúdo de `R[RS1]`;
- `B` – conteúdo de `R[RS2]`.

Esses sinais são nomeados como `REG_A` e `REG_B` no circuito principal.

### 10.2 Geração de Imediato

Da instrução (`INSTRU_OUT`):

- Bits `[3:0]` representam o imediato para instruções tipo I.

Esse campo passa por um **Bit Extender**:

- Entrada de 4 bits;
- Saída de 8 bits;
- Com extensão de sinal em complemento de 2, permitindo saltos negativos.

Esse valor é rotulado como `IMM`.

### 10.3 Unidade de Controle no Pipeline

A unidade de controle continua sendo uma ROM com dados de 10 bits.  
O endereço é o **opcode** (bits `[7:4]` da instrução), e cada palavra de controle contém:

| Bits  | Sinal       | Função                                                                 |
|-------|-------------|-------------------------------------------------------------------------|
| [2:0] | `OP_ULA`    | Seleciona operação da ULA                                              |
| [3]   | `WM`        | *Write Memory* – 1 em `st`, 0 caso contrário                           |
| [4]   | `DADO_BREG` | Fonte do *write-back*: 0 = ULA, 1 = Memória                            |
| [5]   | `MUX_ULA`   | Fonte do operando B: 0 = `REG_B`, 1 = `IMM`                            |
| [6]   | `W_BREG`    | Habilita escrita no banco de registradores                             |
| [7]   | `A_OU_0`    | Seleciona se a ULA recebe `A` ou constante 0                           |
| [9:8] | `PC_SEL`    | Seleção da próxima origem do PC (sequencial/salto/branch via reg/imm) |

Essa palavra de controle é chamada de `CONTROL_ID` no pipeline (pois é gerada em ID).

### 10.4 Registrador ID/EX

O circuito `ID_EX` guarda tudo o que o estágio EX precisa.

**Entradas:**

- `RS1_IN[1:0]`, `RS2_IN[1:0]` – índices de registradores fonte (para forwarding/hazard);
- `A_IN[7:0]`, `B_IN[7:0]` – operandos lidos do banco;
- `IMM_IN[7:0]` – imediato estendido;
- `CONTROL_IN[9:0]` – palavra de controle;
- `RD_IN[1:0]` – registrador destino;
- `CLK` – clock;
- `ESCREVE_OU_NAO` – *write enable* do registrador (para stalls);
- `FLUSH` – se ativo, zera `CONTROL_OUT`, transformando a instrução em `nop`.

**Saídas:**

- `RS1_OUT[1:0]`, `RS2_OUT[1:0]` – repassados à hazard/forwarding;
- `A_OUT[7:0]`, `B_OUT[7:0]` – operandos para a ULA;
- `IMM_OUT[7:0]` – imediato para EX;
- `CONTROL_OUT[9:0]` – sinais de controle já alinhados com EX;
- `RD_OUT[1:0]` – registrador destino da instrução em EX.

**Internamente, há:**

- Registradores dedicados:
  - `RS1_REG`, `RS2_REG`, `A_REG`, `B_REG`, `IMM_REG`, `CTRL_REG`, `RD_REG`;
- Um MUX de 10 bits antes do `CTRL_REG`:
  - Entrada 0: `CONTROL_IN` normal;
  - Entrada 1: constante `0x000` – usada quando `FLUSH = 1` para transformar a instrução em `nop`.

---

## 11. Estágio EX e Registrador EX/MEM

### 11.1 ULA no Pipeline

A ULA é o mesmo módulo da primeira etapa, com:

- **Entradas:** `A[7:0]`, `B[7:0]`, `OPULA[2:0]`;
- **Saídas:**
  - `C[7:0]` – resultado;
  - `ZERO` – indica se `C == 0`.

No pipeline:

- Em instruções tipo R:
  - `A_OUT` e `B_OUT` vêm diretamente do banco de registradores (via `ID_EX`).
- Em instruções tipo I (`addi`, `ji`):
  - O sinal `MUX_ULA` escolhe entre `B_OUT` ou `IMM_OUT` como segundo operando.
- O sinal `A_OU_0` define se o primeiro operando da ULA é `A_OUT` ou constante 0:
  - Usado em `ji` (jump imediato) para fazer `PC = PC + Imm`, onde a ULA soma `0 + Imm`.

### 11.2 Cálculo de Branch e Sintaxe Lógica

A instrução de branch da ISA é:

```asm
brzr RA, RB    ; if (R[RA] == 0) PC = R[RB]
```

No pipeline:

- `A_OUT` recebe `R[RA]`.
- A ULA calcula `ZERO = (A_OUT == 0)`.
- O bit correspondente na palavra de controle (`BRZ_EX`, derivado de `PC_SEL` da UC) indica que a instrução é um branch condicional.
- A *hazard unit* recebe:
  - `ZERO_EX` – saída da ULA;
  - `BRZ_EX` – bit da UC indicando “esta instrução é `brzr`”.

Se `ZERO_EX && BRZ_EX` → `BRANCH_TAKEN_EX = 1`.

Para o **jump imediato** (`ji`):

```asm
ji Imm    ; PC = PC + Imm
```

- O `PC + Imm` é calculado no IF (somador `PC + IMM`);
- A UC configura `PC_SEL` adequadamente para selecionar esse caminho.

### 11.3 Load e Store na EX

Relembrando a semântica:

**LOAD**

```asm
LOAD RA, [RB]
; Vá ao endereço apontado por RB, pegue o valor e salve em RA
; R[RA] = M[R[RB]]
```

**STORE**

```asm
STORE RA, [RB]
; Grave o valor de RA no endereço apontado por RB
; M[R[RB]] = R[RA]
```

No estágio EX:

- Para ambas (`ld` e `st`), a ULA calcula o endereço:

  - `Endereço = conteúdo de R[RB]` (via `B_OUT` ou `IMM` dependendo da codificação);  
  - O projeto foi estruturado de forma que o operando “que interessa” para `load/store` é sempre o que entra como segundo operando da ULA.

- O resultado da ULA (`ALU_OUT`) é o endereço de memória que irá para a RAM no estágio MEM.
- No caso de `st`, o dado a ser escrito em memória vem de `A_OUT` (que corresponde a `R[RA]`).

### 11.4 Registrador EX/MEM

O circuito `EX_MEM` é o registrador de pipeline entre EX e MEM.

**Entradas:**

- `ALU_IN[7:0]` – resultado da ULA (endereço ou resultado aritmético);
- `STORE_IN[7:0]` – dado a ser armazenado em memória (para `st`);
- `CONTROL_IN[9:0]` – sinais de controle da instrução;
- `RD_IN[1:0]` – registrador destino;
- `CLK`, `ESCREVE_OU_NAO`, `FLUSH`.

**Saídas:**

- `ALU_OUT[7:0]` – endereço/resultado para MEM;
- `STORE_OUT[7:0]` – dado a ser escrito em memória;
- `CONTROL_OUT[9:0]` – sinais de controle alinhados com MEM;
- `RD_OUT[1:0]` – destino que chegará a WB.

Tal como em `ID_EX`, há um MUX antes de `CTRL_REG`:

- Entrada 0: `CONTROL_IN`;
- Entrada 1: `0x000` (`NOP`).

Quando `FLUSH_EX_MEM = 1` (vindo da hazard unit), a saída de controle é zerada e a instrução vira uma bolha.

---

## 12. Estágio MEM, Registrador MEM/WB e WB

### 12.1 Acesso à Memória de Dados (MEM)

O estágio MEM utiliza:

- RAM de 8 bits (RAM em `main_pipeline`).

**Entradas:**

- Endereço: `ALU_OUT` proveniente de `EX_MEM`;
- Dado de escrita: `STORE_OUT`;
- `WM` – bit de controle que habilita a escrita em memória;
- `CLK`.

**Comportamentos:**

- Se `WM = 1` → instrução `st`  
  `M[ALU_OUT] ← STORE_OUT`
- Se `WM = 0` → instrução `ld` ou outra  
  `MEM_OUT ← M[ALU_OUT]` (dado lido para WB)

### 12.2 Registrador MEM/WB

O circuito `MEM_WB` recebe:

**Entradas:**

- `MEM_IN[7:0]` – dado lido da memória;
- `ALU_IN[7:0]` – resultado da ULA;
- `CONTROL_IN[9:0]` – sinais de controle;
- `RD_IN[1:0]` – registrador destino;
- `CLK`, `ESCREVE_OU_NAO`, `FLUSH`.

**Saídas:**

- `MEM_OUT[7:0]` – dado de memória alinhado com WB;
- `ALU_OUT[7:0]` – resultado da ULA alinhado com WB;
- `CONTROL_OUT[9:0]` – palavra de controle na fase WB;
- `RD_OUT[1:0]` – registrador destino a ser escrito.

Assim como nos outros registradores:

- Um MUX de 10 bits permite zerar `CONTROL_OUT` quando `FLUSH_MEM_WB = 1`, inserindo uma bolha após um desvio.

### 12.3 Estágio WB (*Write Back*)

No estágio WB (dentro do próprio `main_pipeline`), a partir de `MEM_WB` chegam:

- `ALU_OUT` (resultado aritmético/lógico ou endereço calculado);
- `MEM_OUT` (dado retornado por um `ld`);
- `CONTROL_OUT` com os bits `W_BREG` e `DADO_BREG`;
- `RD_OUT` (registrador destino).

Com isso:

- Um multiplexador seleciona o dado de *write-back*:
  - Se `DADO_BREG = 0` → escreve `ALU_OUT`;
  - Se `DADO_BREG = 1` → escreve `MEM_OUT`.
- Se `W_BREG = 1`, o valor selecionado é escrito em `R[RD_OUT]` no banco de registradores na próxima borda de clock.

---

## 13. Unidade de Hazards

A `HAZARD_UNIT` monitora perigos de dados e de controle.

### 13.1 Entradas

- `RD_EX[1:0]` – registrador destino da instrução atualmente em EX;
- `RS1_ID[1:0]`, `RS2_ID[1:0]` – registradores fonte da instrução em ID;
- `ZERO_EX` – flag de zero para a instrução em EX;
- `BRZ_EX` – bit de controle indicando que a instrução em EX é `brzr`;
- `LOAD_EX` – bit indicando que a instrução em EX é `ld`;
- `PC_SEL_UC[1:0]` – seleção de PC proposta pela Unidade de Controle (predição ou default).

### 13.2 Saídas

- `HAZ_LOAD` – indica detecção de hazard de load;
- `PC_WRITE` – 0 para travar atualização do PC;
- `IF_ID_WRITE` – 0 para travar registrador IF/ID;
- `FLUSH_ID_EX` – limpa o registrador ID/EX (insere bolha);
- `FLUSH_EX_MEM` – limpa EX/MEM;
- `FLUSH_MEM_WB` – limpa MEM/WB;
- `BRANCH_TAKEN_EX` – indica que um branch foi efetivamente tomado;
- `PC_SEL_EX[1:0]` – nova seleção de PC (corrigindo predição, se necessário).

### 13.3 Hazard de Load–Use

Caso clássico:

```asm
ld   R0, [R1]
add  R2, R0
```

No ciclo em que `add` está em ID e `ld` está em EX:

- `RD_EX = R0`;
- `RS1_ID` ou `RS2_ID = R0`;
- `LOAD_EX = 1` (porque a instrução em EX é `ld`).

A `HAZARD_UNIT`:

- Compara `RD_EX` com `RS1_ID` e com `RS2_ID` através de dois comparadores de 2 bits;
- O resultado desses comparadores é ORado;
- O resultado é ANDado com `LOAD_EX`;
- Se o resultado é 1 → `HAZ_LOAD = 1`.

Quando `HAZ_LOAD = 1`:

- `PC_WRITE = 0` – PC não avança (repete o fetch da instrução dependente);
- `IF_ID_WRITE = 0` – IF/ID não é atualizado (ID continua com a mesma instrução);
- `FLUSH_ID_EX = 1` – o registrador ID/EX é zerado (a instrução em EX vira `nop`).

Isso implementa o clássico stall de 1 ciclo para dependência de load.

### 13.4 Hazard de Controle (Branches)

Quando uma instrução `brzr` está em EX:

- `BRZ_EX = 1` pela UC;
- `ZERO_EX` vem da ULA.

A `HAZARD_UNIT` gera:

```text
BRANCH_TAKEN_EX = ZERO_EX AND BRZ_EX
```

Em caso de **branch tomado**:

- `PC_SEL_EX` é ajustado para a origem correta do PC (endereço de destino – vindo de registrador ou imediato).
- Sinais de flush:
  - `FLUSH_ID_EX = 1` – instrução em ID/EX é descartada;
  - `FLUSH_EX_MEM = 1` – descarta instrução seguinte;
  - `FLUSH_MEM_WB = 1` – garante que qualquer instrução errada à frente seja convertida em `nop`.

Dessa forma, o pipeline corrige *mispredictions* ou comportamento padrão “*not taken*” descartando instruções que já estavam à frente.

---

## 14. Predição de Desvios

### 14.1 Motivação

Sem predição, todo branch condicional causa:

- Fetch especulativo da próxima instrução;
- Quando a condição é avaliada em EX, descobre-se se o fetch estava correto;
- Se não estava, o PC precisa ser corrigido e parte do pipeline deve ser *flushada*.

Isso gera um custo de vários ciclos por branch, diminuindo o IPC.  
Para reduzir essa penalidade, foi projetada uma unidade de *predição de desvio*, que tenta adivinhar se o branch será tomado ou não.

### 14.2 Preditor de 2 Bits (FSM)

O segundo arquivo (`project source="2.7.1"`) descreve um circuito independente que implementa uma FSM com dois flip-flops (`D1`, `D0`), produzindo estados `Q1 Q0` e seus complementos `NQ1`, `NQ0`.

**Interpretação lógica:**

Os dois bits de estado (`Q1`, `Q0`) codificam quatro estados possíveis, como em um contador saturado de 2 bits:

- `00` – fortemente “not taken”;
- `01` – fracamente “not taken`;
- `10` – fracamente “taken`;
- `11` – fortemente “taken`.

Existem duas entradas principais:

- `E` – (*taken* observado: desvio foi tomado de fato);
- `NE` – (*not taken* observado: desvio não foi tomado).

A lógica de AND/OR combinada com `Q1`, `Q0`, `NQ1`, `NQ0` implementa as transições padrão:

- Se o desvio foi tomado (`E = 1`): o estado caminha em direção a “taken`;
- Se o desvio foi não tomado (`NE = 1`): o estado caminha em direção a “not taken`.

A saída conectada ao pino final (em coordenadas `980,150`) representa a predição do próximo branch:

- `1 = take`;
- `0 = not take`.

Essa saída é usada como palpite inicial sobre o valor de `PC_SEL` para branches, antes da execução no estágio EX.

### 14.3 Integração com o Pipeline

A integração conceitual funciona assim:

- No estágio ID (ou IF), quando é decodada uma instrução de branch (`brzr`):
  - A unidade de controle gera um valor padrão de `PC_SEL_UC` (por exemplo, assumindo “not taken”);
  - A saída do preditor pode sobrescrever esse valor, produzindo uma seleção de PC que já aponta para o destino do branch quando a predição é “taken”.

- No estágio EX:
  - A condição é realmente avaliada (`ZERO_EX` e `BRZ_EX`);
  - A `HAZARD_UNIT` gera `BRANCH_TAKEN_EX`;
  - Esse bit é comparado à predição:
    - Se a predição estava correta → pipeline deveria continuar normalmente;
    - Se estava incorreta → `FLUSH_ID_EX`, `FLUSH_EX_MEM`, `FLUSH_MEM_WB` são ativados e `PC_SEL_EX` corrige o valor do PC.

- Em paralelo:
  - O preditor recebe `E` (branch tomado) ou `NE` (branch não tomado) e atualiza seu estado (`Q1`, `Q0`) na borda de clock, tornando a próxima predição mais precisa.

**Observação:**  
No nosso projeto, a ligação física entre o preditor e `PC_SEL_UC` ficou parcialmente implementada e, por limitações de tempo/debug, o comportamento efetivo dos testes ficou muito próximo de um esquema “always not taken”. O circuito do preditor está funcional isoladamente, mas não foi possível validar sua integração plena devido a problemas na arquitetura pipeline que não conseguimos localizar a tempo.

---

## 15. Forwarding (Tentativa e Ideia de Implementação)

### 15.1 Problema de Hazards de Dados Sem Load

Mesmo com o stall de *load–use*, ainda restam hazards de dados do tipo:

```asm
add  R0, R1
sub  R2, R0     ; R0 ainda não chegou ao banco quando o sub usa
```

Aqui a instrução `sub` necessita do resultado de `add` antes de ele ser escrito de volta no banco de registradores (WB).  
A solução típica é o **forwarding** (*bypass*):

- O resultado produzido em EX ou MEM é “desviado” diretamente para a entrada da ULA da próxima instrução dependente.

### 15.2 Estrutura Pensada

Na `main_pipeline` já há sinais e blocos previstos para forwarding:

- Túneis `RS1_EX` e `RS2_EX` (índices de registradores fonte da instrução em EX);
- Saídas `RD` dos estágios `EX/MEM` e `MEM/WB`;
- Sinais de controle planejados: `FWD_A[1:0]` e `FWD_B[1:0]`;
- Multiplexadores antes das entradas A e B da ULA (em coordenadas `4320,540` e `4740,320`).

A ideia era:

- **Comparadores de registradores:**
  - Comparar `RS1_EX` com `RD_EX_MEM` (destino da instrução em MEM) e `RD_MEM_WB` (destino da instrução em WB);
  - Repetir o mesmo para `RS2_EX`.

- **Geração de códigos de forwarding (2 bits cada):**

  - `00` → usar valor normal vindo do `ID_EX` (sem forwarding);
  - `01` → encaminhar resultado do estágio `EX/MEM`;
  - `10` → encaminhar resultado do estágio `MEM/WB`;
  - `11` → reservado/não usado.

- **MUX nas entradas da ULA:**

  - Para o operando A:
    - Entrada 0: `A_OUT` de `ID_EX`;
    - Entrada 1: resultado da ULA em `EX_MEM` (`ALU_OUT`);
    - Entrada 2: dado de *write-back* em `MEM_WB`.

  - Para o operando B:
    - Idem, considerando ainda o uso do imediato (`MUX_ULA`).

**Exemplo:**

```asm
add  R0, R1      ; produz R0 em EX
sub  R2, R0      ; precisa do novo R0 já no ciclo seguinte
```

No ciclo em que `sub` está em EX:

- `RS1_EX = R0`, `RD_EX_MEM = R0` (resultado de `add` no `EX/MEM`);
- O comparador detecta igualdade → `FWD_A = 01`;
- O MUX da entrada A seleciona o dado vindo de `ALU_OUT` do `EX/MEM`;
- Assim, o valor atualizado de `R0` é usado sem esperar o WB.

### 15.3 Dificuldades e Status Final

Na prática:

- Conseguimos desenhar boa parte da lógica de comparação e dos multiplexadores;
- Porém, surgiram problemas de:
  - Prioridade entre fontes (`EX/MEM` vs `MEM/WB`);
  - Interação entre forwarding e o stall de *load–use* (era preciso desabilitar forwarding em alguns casos);
  - Depuração visual em Logisim com múltiplas instruções em voo.

Por questões de tempo, acabamos não finalizando a ligação correta dos sinais `FWD_A` e `FWD_B` com todos os multiplexadores e comparadores e, na versão que tentamos testar, a arquitetura pipeline apresentou comportamentos incorretos (travamentos e resultados inesperados) que apontam para algum erro estrutural na combinação de hazards/forwarding/predição.

No relatório, portanto, registramos que:

- Forwarding foi planejado e parcialmente implementado, mas não totalmente depurado nem validado.  
- A estrutura está desenhada para ser completada em trabalhos futuros, pois o caminho de dados e os sinais auxiliares (`RS1/RS2` em vários estágios, `RD` em `EX/MEM` e `MEM/WB`) já estão presentes, porém os testes finais falharam e não conseguimos identificar exatamente onde está o erro na arquitetura.

---

## 16. Testes e Validação

### 16.1 Programas de Teste

Foram carregados na ROM de instruções pequenos programas que exercitam:

- **Operações aritméticas e lógicas:**

  - `add`, `sub`, `and`, `or`, `xor`, `not`, `slr`, `srr`;

- **Instruções de memória:**

  - `ld`, `st` com diferentes endereços;

- **Controle de fluxo:**

  - `ji` para saltos incondicionais;
  - `brzr` com casos em que o branch é tomado e não tomado.

### 16.2 Observação Ciclo a Ciclo

Durante a simulação em Logisim, foram observados:

- O PC (`PC_OUT`);
- O conteúdo dos registradores de pipeline (`IF/ID`, `ID/EX`, `EX/MEM`, `MEM/WB`);
- Os sinais de hazard (`PC_WRITE`, `IF_ID_WRITE`, `FLUSH_*`);
- O comportamento da memória de dados.

No entanto, os testes não se comportaram como esperado:

- Em programas um pouco mais complexos (especialmente combinando `ld`, operações aritméticas e `brzr` em sequência), o pipeline apresentava:
  - travamentos ocasionais;
  - resultados incorretos em registradores ou memória;
  - necessidade de flushes inesperados.

Esses sintomas indicam a existência de um ou mais erros estruturais na arquitetura pipeline (provavelmente na interação entre hazard de *load–use*, lógica de flush de desvio e forwarding parcial).

Apesar de diversas tentativas de depuração ciclo a ciclo, não foi possível localizar com precisão o ponto exato da falha dentro do prazo do projeto. Como consequência, consideramos que:

- A arquitetura pipeline foi projetada e parcialmente implementada;
- Mas os testes finais falharam, isto é, não conseguimos demonstrar um funcionamento completamente correto para todos os programas de teste planejados.

---

## 17. Conclusões

Com esta etapa, o projeto passou de um processador sequencial para uma arquitetura pipeline de 5 estágios compatível com a ISA REDUX V, do ponto de vista conceitual e de desenho do caminho de dados, incluindo:

- Pipeline completo `IF–ID–EX–MEM–WB` com registradores dedicados;
- Unidade de controle adaptada para gerar sinais utilizados em estágios diferentes;
- Tratamento de hazards projetado para:
  - Stall automático em dependências de *load* (*load–use*);
  - Flush dos estágios quando um branch é tomado;
- Infraestrutura de predição de desvio baseada em preditor de 2 bits, pronta para integração total;
- Estrutura de forwarding parcialmente implementada, com caminho de dados projetado e sinais auxiliares disponíveis.

Entretanto, é importante registrar explicitamente as limitações e o estado real da implementação:

- A lógica combinada de hazards (dados + controle), predição e forwarding não se mostrou estável nos testes finais;
- Os programas de teste mais completos revelaram:
  - travamentos do pipeline;
  - valores incorretos em registradores/memória;
  - comportamento inconsistente de flush e stall;

Dentro do tempo disponível, não conseguimos identificar nem corrigir o erro (ou erros) presente(s) na arquitetura pipeline.

Como consequência:

- Os testes finais falharam e a implementação pipeline não pode ser considerada funcional/validada no mesmo nível do processador sequencial da primeira etapa;
- A arquitetura resultante deve ser vista como uma base de estudo e ponto de partida:
  - o caminho de dados principal e a divisão em estágios estão bem definidos;
  - os módulos de pipeline (`IF/ID`, `ID/EX`, `EX/MEM`, `MEM/WB`) estão implementados;
  - a hazard unit e o preditor de desvio foram desenhados e parcialmente integrados;
  - porém ainda há um bug estrutural (possivelmente na priorização de forwarding vs. stall/flush) que precisa ser encontrado e corrigido.

Mesmo com essas limitações, o trabalho foi importante para:

- Consolidar o entendimento de:
  - pipeline de 5 estágios;
  - hazards de dados e de controle;
  - *branch prediction* com contador saturado de 2 bits;
  - interação entre unidade de controle, memória e banco de registradores;
- E fornecer uma plataforma concreta de experimentação em Logisim, que pode ser refinada em iterações futuras até que a arquitetura pipeline funcione corretamente em todos os cenários de teste desejados.
