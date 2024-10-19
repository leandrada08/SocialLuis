# Italiano

## Codice

Il modulo `CPU_pipeline` implementa una CPU basata sull'ISA RISC-V con pipeline. Questo design è composto da cinque fasi: IF (Instruction Fetch), ID (Instruction Decode), EX (Execute), MEM (Memory Access), e WB (Write Back), già definite e descritte in precedenza. Vengono utilizzati registri del pipeline per memorizzare i risultati intermedi tra le fasi e mantenere un flusso continuo di istruzioni.

#### Interfaccia del Modulo
```verilog
module CPU (
    input           clk,
    input           reset,
    output [31:0]   pc,          // Program Counter
    output [31:0]   alu_result   // Risultato dell'ALU
);
```
- `clk`: Segnale di clock per sincronizzare le operazioni della CPU.
- `reset`: Segnale di reset per inizializzare la CPU.
- `pc`: Uscita del program counter che indica l'indirizzo dell'istruzione corrente.
- `alu_result`: Uscita del risultato dell'ALU che mostra l'ultimo risultato dell'operazione aritmetica/logica.

#### Segnali Interni e Registri del Pipeline
- **IF/ID Pipeline Registers**: Memorizzano l'istruzione e l'indirizzo attuale dopo la fase di Instruction Fetch (IF).
- **ID/EX Pipeline Registers**: Memorizzano i valori letti dai registri, la costante immediata, e i segnali di controllo dopo la fase di Instruction Decode (ID).
- **EX/MEM Pipeline Registers**: Memorizzano il risultato dell'ALU, il secondo operando, e i segnali di controllo dopo la fase di Execute (EX).
- **MEM/WB Pipeline Registers**: Memorizzano i dati letti dalla memoria e i segnali di controllo dopo la fase di Memory Access (MEM).

#### Fase IF (Instruction Fetch)
```verilog
// Fase IF
IF if_stage (
    .o_address(if_address),
    .o_instruccion(if_instruccion),
    .i_branch_address(exmem_PCBranch),
    .i_reset(reset),
    .i_select(mem_PCSrc),
    .i_clock(clk)
);
```
- `o_address`: Indirizzo corrente del PC.
- `o_instruccion`: Istruzione letta dalla memoria delle istruzioni.
- `i_branch_address`: Indirizzo di destinazione in caso di un'istruzione di salto.
- `i_reset`: Segnale di reset.
- `i_select`: Segnale per selezionare tra l'indirizzo successivo e l'indirizzo di salto.
- `i_clock`: Segnale di clock.

#### Registri del Pipeline IF/ID
```verilog
// Pipeline IF/ID
always @(posedge clk or posedge reset) begin
    if (reset) begin
        ifid_instruccion <= 32'b0;
        ifid_address <= 31'b0;
    end else begin
        ifid_instruccion <= if_instruccion;
        ifid_address <= if_address;
    end
end
```
- `ifid_instruccion`: Istruzione memorizzata per la fase di ID.
- `ifid_address`: Indirizzo corrente del PC memorizzato per la fase di ID.

#### Fase ID (Instruction Decode)
```verilog
// Fase ID
ID id_stage (
    .i_clk(clk),
    .i_reset(reset),
    .i_instruccion(ifid_instruccion),
    .i_WriteData(wb_WriteData),
    .i_RegWrite(memwb_RegWrite),
    .i_WriteReg(memwb_WriteReg),
    .o_register1(id_register1),
    .o_register2(id_register2),
    .o_constante(id_constante),
    .o_WriteReg(id_WriteReg),
    .o_RegWrite(id_RegWrite),
    .o_ALUSrc(id_ALUSrc),
    .o_MemWrite(id_MemWrite),
    .o_MemRead(id_MemRead),
    .o_Branch(id_Branch),
    .o_MemToReg(id_MemToReg),
    .o_SLTc(id_SLTc),
    .o_ALUControl(id_ALUControl),
    .o_BranchOp(id_BranchOp)
);
```
- `i_clk`: Segnale di clock.
- `i_reset`: Segnale di reset.
- `i_instruccion`: Istruzione da decodificare.
- `i_WriteData`: Dati da scrivere nel registro di destinazione.
- `i_RegWrite`: Segnale di controllo per abilitare la scrittura nei registri.
- `i_WriteReg`: Registro di destinazione per la scrittura.
- `o_register1`: Valore letto dal primo registro sorgente.
- `o_register2`: Valore letto dal secondo registro sorgente.
- `o_constante`: Costante immediata decodificata.
- `o_WriteReg`: Registro di destinazione decodificato.
- `o_RegWrite`: Segnale di controllo per abilitare la scrittura nei registri.
- `o_ALUSrc`: Segnale di controllo per selezionare il secondo operando dell'ALU.
- `o_MemWrite`: Segnale di controllo per scrivere nella memoria dati.
- `o_MemRead`: Segnale di controllo per leggere dalla memoria dati.
- `o_Branch`: Segnale di controllo per eseguire un salto condizionato.
- `o_MemToReg`: Segnale di controllo per selezionare il dato che viene scritto nel registro di destinazione.
- `o_SLTc`: Segnale di controllo per l'istruzione SLT.
- `o_ALUControl`: Segnale di controllo per determinare l'operazione dell'ALU.
- `o_BranchOp`: Segnale di controllo per determinare il tipo di salto.

#### Registri del Pipeline ID/EX
```verilog
// Pipeline ID/EX
always @(posedge clk or posedge reset) begin
    if (reset) begin
        idex_PC <= 32'b0;
        idex_register1 <= 32'b0;
        idex_register2 <= 32'b0;
        idex_constante <= 32'b0;
        idex_ALUSrc <= 1'b0;
        idex_ALUControl <= 3'b0;
        idex_MemWrite <= 1'b0;
        idex_MemRead <= 1'b0;
        idex_Branch <= 1'b0;
        idex_MemToReg <= 1'b0;
        idex_RegWrite <= 1'b0;
        idex_BranchOp <= 2'b0;
        idex_SLTc <= 1'b0;
        idex_WriteReg <= 5'b0;
    end else begin
        idex_PC <= {ifid_address, 1'b0};
        idex_register1 <= id_register1;
        idex_register2 <= id_register2;
        idex_constante <= id_constante;
        idex_ALUSrc <= id_ALUSrc;
        idex_ALUControl <= id_ALUControl;
        idex_MemWrite <= id_MemWrite;
        idex_MemRead <= id_MemRead;
        idex_Branch <= id_Branch;
        idex_MemToReg <= id_MemToReg;
        idex_RegWrite <= id_RegWrite;
        idex_BranchOp <= id_BranchOp;
        idex_SLTc <= id_SLTc;
        idex_WriteReg <= id_WriteReg;
    end
end
```
- `idex_PC`: Indirizzo corrente del PC memorizzato per la fase di EX.
- `idex_register1`: Valore del primo registro sorgente memorizzato per la fase di EX.
- `idex_register2`: Valore del secondo registro sorgente memorizzato per la fase di EX.
- `idex_constante`: Costante immediata memorizzata per la fase di EX.
- `idex_ALUSrc`: Segnale di controllo memorizzato per la fase di EX.
- `idex_ALUControl`: Segnale di controllo per l'ALU memorizzato per la fase di EX.
- `idex_MemWrite`: Segnale di controllo memorizzato per la fase di EX.
- `idex_MemRead`: Segnale di controllo memorizzato per la fase di EX.
- `idex_Branch`: Segnale di controllo memorizzato per la fase di EX.
- `idex_MemToReg`: Segnale di controllo memorizzato per la fase di EX.
- `idex_RegWrite`: Segnale di controllo memorizzato per la fase di EX.
- `idex_BranchOp`: Segnale di controllo memorizzato per la fase di EX.
- `idex_SLTc`: Segnale di controllo memorizzato per la fase di EX.
- `idex_WriteReg`: Registro di destinazione memorizzato per la fase di EX.

#### Fase EX (Execute)
```verilog
// Fase EX
EX ex_stage (
    .i_PC(idex_PC),
    .i_register1(idex_register1),
    .i_register2(idex_register2),
    .i_constante(idex_constante),
    .i_ALUSrc(idex_ALUSrc),
    .i_ALUControl(idex_ALUControl),
    .o_ALUResult(ex_ALUResult),
    .o_PCBranch(ex_PCBranch),
    .o_zero(ex_zero),
    .o_negative(ex_negative)
);
```
- `i_PC`: Indirizzo del PC in ingresso.
- `i_register1`: Primo operando dell'ALU.
- `i_register2`: Secondo operando dell'ALU.
- `i_constante`: Costante immediata.
- `i_ALUSrc`: Segnale di controllo per selezionare il secondo operando dell'ALU.
- `i_ALUControl`: Segnale di controllo per determinare l'operazione dell'ALU.
- `o_ALUResult`: Risultato dell'ALU.
- `o_PCBranch`: Indirizzo di salto calcolato.
- `o_zero`: Segnale che indica se il risultato dell'ALU è zero.
- `o_negative`: Segnale che indica se il risultato dell'ALU è negativo.

#### Registri del Pipeline EX/MEM
```verilog
// Pipeline EX/MEM
always @(posedge clk or posedge reset) begin
    if (reset) begin
        exmem_ALUResult <= 32'b0;
        exmem_register2 <= 32'b0;
        exmem_PCBranch <= 32'b0;
        exmem_zero <= 1'b0;
        exmem_negative <= 1'b0;
        exmem_MemWrite <= 1'b0;
        exmem_MemRead <= 1'b0;
        exmem_Branch <= 1'b0;
        exmem_MemToReg <= 1'b0;
        exmem_RegWrite <= 1'b0;
        exmem_BranchOp <= 2'b0;
        exmem_SLTc <= 1'b0;
        exmem_WriteReg <= 5'b0;
    end else begin
        exmem_ALUResult <= ex_ALUResult;
        exmem_register2 <= idex_register2;
        exmem_PCBranch <= ex_PCBranch;
        exmem_zero <= ex_zero;
        exmem_negative <= ex_negative;
        exmem_MemWrite <= idex_MemWrite;
        exmem_MemRead <= idex_MemRead;
        exmem_Branch <= idex_Branch;
        exmem_MemToReg <= idex_MemToReg;
        exmem_RegWrite <= idex_RegWrite;
        exmem_BranchOp <= idex_BranchOp;
        exmem_SLTc <= idex_SLTc;
        exmem_WriteReg <= idex_WriteReg;
    end
end
```
- `exmem_ALUResult`: Risultato dell'ALU memorizzato per la fase di MEM.
- `exmem_register2`: Valore del secondo operando memorizzato per la fase di MEM.
- `exmem_PCBranch`: Indirizzo di salto calcolato memorizzato per la fase di MEM.
- `exmem_zero`: Segnale di risultato zero memorizzato per la fase di MEM.
- `exmem_negative`: Segnale di risultato negativo memorizzato per la fase di MEM.
- `exmem_MemWrite`: Segnale di controllo memorizzato per la fase di MEM.
- `exmem_MemRead`: Segnale di controllo memorizzato per la fase di MEM.
- `exmem_Branch`: Segnale di controllo memorizzato per la fase di MEM.
- `exmem_MemToReg`: Segnale di controllo memorizzato per la fase di MEM.
- `exmem_RegWrite`: Segnale di controllo memorizzato per la fase di MEM.
- `exmem_BranchOp`: Segnale di controllo memorizzato per la fase di MEM.
- `exmem_SLTc`: Segnale di controllo memorizzato per la fase di MEM.
- `exmem_WriteReg`: Registro di destinazione memorizzato per la fase di MEM.

#### Fase MEM (Memory Access)
```verilog
// Fase MEM
MEM mem_stage (
    .i_clk(clk),
    .i_reset(reset),
    .i_Address(exmem_ALUResult),
    .i_WriteData(exmem_register2),
    .i_BranchOp(exmem_BranchOp),
    .i_negative(exmem_negative),
    .i_zero(exmem_zero),
    .i_branch(exmem_Branch),
    .i_MemWrite(exmem_MemWrite),
    .i_MemRead(exmem_MemRead),
    .i_SLTc(exmem_SLTc),
    .o_PCSrc(mem_PCSrc),
    .o_ReadData(mem_ReadData),
    .o_Mux(mem_Mux)
);
```
- `i_clk`: Segnale di clock.
- `i_reset`: Segnale di reset.
- `i_Address`: Indirizzo di memoria da accedere.
- `i_WriteData`: Dati da scrivere nella memoria.
- `i_BranchOp`: Segnale di controllo per determinare il tipo di salto.
- `i_negative`: Segnale di risultato negativo.
- `i_zero`: Segnale di risultato zero.
- `i_branch`: Segnale di controllo per eseguire un salto condizionato.
- `i_MemWrite`: Segnale di controllo per scrivere nella memoria dati.
- `i_MemRead`: Segnale di controllo per leggere dalla memoria dati.
- `i_SLTc`: Segnale di controllo per l'istruzione SLT.
- `o_PCSrc`: Segnale di controllo per selezionare l'indirizzo successivo del PC.
- `o_ReadData`: Dati letti dalla memoria dati.
- `o_Mux`: Uscita del multiplexer.

#### Registri del Pipeline MEM/WB
```verilog
// Pipeline MEM/WB
always @(posedge clk or posedge reset) begin
    if (reset) begin
        memwb_ALUResult <= 32'b0;
        memwb_ReadData <= 32'b0;
        memwb_MemToReg <= 1'b0;
        memwb_RegWrite <= 1'b0;
        memwb_WriteReg <= 5'b0;
    end else begin
        memwb_ALUResult <= exmem_ALUResult;
        memwb_ReadData <= mem_ReadData;
        memwb_MemToReg <= exmem_MemToReg;
        memwb_RegWrite <= exmem_RegWrite;
        memwb_WriteReg <= exmem_WriteReg;
    end
end
```
- `memwb_ALUResult`: Risultato dell'ALU memorizzato per la fase di WB.
- `memwb_ReadData`: Dati letti dalla memoria memorizzati per la fase di WB.
- `memwb_MemToReg`: Segnale di controllo memorizzato per la fase di WB.
- `memwb_RegWrite`: Segnale di controllo memorizzato per la fase di WB.
- `memwb_WriteReg`: Registro di destinazione memorizzato per la fase di WB.

#### Fase WB (Write Back)
```verilog
// Fase WB
MUX32 wb_mux (
    .A(memwb_ALUResult),
    .B(memwb_ReadData),
    .select(memwb_MemToReg),
    .C(wb_WriteData)
);
```
- `A`: Risultato dell'ALU.
- `B`: Dati letti dalla memoria.
- `select`: Segnale di controllo per selezionare i dati che vengono scritti nel registro di destinazione.
- `C`: Dati da scrivere nel registro di destinazione.


## Analisi

![[Pasted image 20240716172751.png]]

### Codice di simulazione

#### Codice in Assembly RISC-V

```assembly
.section .text
.globl _start

_start:
    # Inizializzare i registri usando ADDI
    addi x1, x0, 0    # x1 = 0
    addi x2, x0, 4    # x2 = 4
    addi x3, x0, 0    # x3 = 0
    addi x4, x0, 16   # x4 = 16
    addi x5, x0, 256  # x5 = 256
    addi x6, x0, 512  # x6 = 512
	addi x7, x0, 1792  # x7 = 1792
    

    # Simulazione del ritardo del pipeline
    nop               # Nessuna operazione (simulare ritardo)
    nop               # Nessuna operazione (simulare ritardo)
    nop               # Nessuna operazione (simulare ritardo)
    nop               # Nessuna operazione (simulare ritardo)

loop:
    add x8, x7, x2    # Aggiungi offset al valore caricato, risultato in x8

    # Simulazione del ritardo del pipeline per operazione ALU
    nop               # Nessuna operazione (simulare ritardo ALU)
    nop               # Nessuna operazione (simulare ritardo ALU)

    sw x8, 0(x7)      # Memorizza il risultato in output_data

    # Simulazione del ritardo del pipeline per scrittura in memoria
    nop               # Nessuna operazione (simulare ritardo scrittura memoria)
    nop               # Nessuna operazione (simulare ritardo scrittura memoria)
    nop               # Nessuna operazione (simulare ritardo scrittura memoria)

    addi x5, x5, 4    # Passa al prossimo elemento input_data
    addi x6, x6, 4    # Passa al prossimo elemento output_data
    addi x3, x3, 4    # Aggiorna l'indirizzo di base per input_data
    
	nop              
    nop
    nop
    
    blt x3, x4, loop  # Loop se non tutti gli elementi sono stati processati

    # Fine del programma (ciclo infinito)
end:
    nop               # Ciclo infinito
    nop
    nop
```
#### File `mem.mem`

```
00000093
00400113
00000193
01000213
10000293
20000313
70100393
00000013
00000013
00000013
00000013
00238433
00000013
00000013
0073A023
00000013
00000013
00000013
00428293
00430313
00418193
00000013
00000013
00000013
FE41CAE3
00000013
00000013
00000013
00000013
```


#### Tabella di Esecuzione

| PC (Hex)   | Istruzione       | Registro/Spazio di Memoria Modificato  | Risultato dell'ALU                                     |
| ---------- | ---------------- | -------------------------------------- | ------------------------------------------------------ |
| 0x00000000 | addi x1, x0, 0    | x1                                     | 0                                                      |
| 0x00000001 | addi x2, x0, 4    | x2                                     | 4                                                      |
| 0x00000002 | addi x3, x0, 0    | x3                                     | 0                                                      |
| 0x00000003 | addi x4, x0, 16   | x4                                     | 16                                                     |
| 0x00000004 | addi x5, x0, 256  | x5                                     | 256                                                    |
| 0x00000005 | addi x6, x0, 512  | x6                                     | 512                                                    |
| 0x00000006 | addi x7, x0, 256  | x7                                     | 1792                                                   |
| 0x00000007 | nop               | -                                      | -                                                      |
| 0x00000008 | nop               | -                                      | -                                                      |
| 0x00000009 | nop               | -                                      | -                                                      |
| 0x0000000A | nop               | -                                      | -                                                      |
| 0x0000000B | add x8, x7, x2    | x8                                     | 14 (10 + 4)                                            |
| 0x0000000C | nop               | -                                      | -                                                      |
| 0x0000000D | nop               | -                                      | -                                                      |
| 0x0000000E | sw x8, 0(x7)      | Memoria in 0x1792                      | 14                                                     |
| 0x0000000F | nop               | -                                      | -                                                      |
| 0x00000010 | nop               | -                                      | -                                                      |
| 0x00000011 | nop               | -                                      | -                                                      |
| 0x00000012 | addi x5, x5, 4    | x5                                     | 260 (0x104)                                            |
| 0x00000013 | addi x6, x6, 4    | x6                                     | 516 (0x204)                                            |
| 0x00000014 | addi x3, x3, 4    | x3                                     | 4                                                      |
| 0x00000015 | blt x3, x4, loop  | PC                                     | Salta a 0x0000000B (x3 < x4)                           |
| 0x0000000B | add x8, x7, x2    | x8                                     | 24 (20 + 4) (nota: si deve eseguire lw x7, 0(x5) prima)|
| 0x0000000C | nop               | -                                      | -                                                      |
| 0x0000000D | nop               | -                                      | -                                                      |
| 0x0000000E | sw x8, 0(x7)      | Memoria in 0x204                       | 3073                                                   |
| 0x0000000F | nop               | -                                      | -                                                      |
| 0x00000010 | nop               | -                                      | -                                                      |
| 0x00000011 | nop               | -                                      | -                                                      |
| 0x00000012 | addi x5, x5, 4    | x5                                     | 264 (0x108)                                            |
| 0x00000013 | addi x6, x6, 4    | x6                                     | 520 (0x208)                                            |
| 0x00000014 | addi x3, x3, 4    | x3                                     | 8                                                      |
| 0x00000015 | blt x3, x4, loop  | PC                                     | Salta a 0x0000000B (x3 < x4)                           |
| 0x0000000B | add x8, x7, x2    | x8                                     | 34 (30 + 4) (nota: si deve eseguire lw x7, 0(x5) prima)|
| 0x0000000C | nop               | -                                      | -                                                      |
| 0x0000000D | nop               | -                                      | -                                                      |
| 0x0000000E | sw x8, 0(x6)      | Memoria in 0x208                       | 34                                                     |
| 0x0000000F | nop               | -                                      | -                                                      |
| 0x00000010 | nop               | -                                      | -                                                      |
| 0x00000011 | nop               | -                                      | -                                                      |
| 0x00000012 | addi x5, x5, 4    | x5                                     | 268 (0x10C)                                            |
| 0x00000013 | addi x6, x6, 4    | x6                                     | 524 (0x20C)                                            |
| 0x00000014 | addi x3, x3, 4    | x3                                     | 12                                                     |
| 0x00000015 | blt x3, x4, loop  | PC                                     | Salta a 0x0000000B (x3 < x4)                           |
| 0x0000000B | add x8, x7, x2    | x8                                     | 44 (40 + 4) (nota: si deve eseguire lw x7, 0(x5) prima)|
| 0x0000000C | nop               | -                                      | -                                                      |
| 0x0000000D | nop               | -                                      | -                                                      |
| 0x0000000E | sw x8, 0(x6)      | Memoria in 0x20C                       | 44                                                     |
| 0x0000000F | nop               | -                                      | -                                                      |
| 0x00000010 | nop               | -                                      | -                                                      |
| 0x00000011 | nop               | -                                      | -                                                      |
| 0x00000012 | addi x5, x5, 4    | x5                                     | 272 (0x110)                                            |
| 0x00000013 | addi x6, x6, 4    | x6                                     | 528 (0x210)                                            |
| 0x00000014 | addi x3, x3, 4    | x3                                     | 16                                                     |
| 0x00000015 | blt x3, x4, loop  | PC                                     | Non salta (x3 non < x4)                                |
| 0x00000016 | nop               | -                                      | -                                                      |
| 0x00000017 | nop               | -                                      | -                                                      |
| 0x00000018 | nop               | -                                      | -                                                      |

### Analisi con scheda Artix A7 35t
#### Pre sintesi
![[Pasted image 20240717212852.png]]
#### Post sintesi
![[Pasted image 20240717160714.png]]
![[Pasted image 20240716174921.png]]
![[Pasted image 20240716174923.png]]

##### Funzionale
![[Pasted image 20240717212911.png]]
##### Timing
![[Pasted image 20240717212933.png]]
#### Post implementazione
##### Timing
![[Pasted image 20240717161523.png]]
![[Pasted image 20240717161651.png]]

##### Utilizzo
![[Pasted image 20240717161558.png]]
![[Pasted image 20240717161602.png]]
##### Funzionale
![[Pasted image 20240717213024.png]]
##### Timing
![[Pasted image 20240717213107.png]]
![[Pasted image 20240716175006.png]]
### Analisi con scheda CMOD 35t
#### Simulazione pre sintesi
![[Pasted image 20240719115339.png]]
#### Utilizzo post sintesi
![[Pasted image 20240719130524.png]]
![[Pasted image 20240719130526.png]]
#### Utilizzo post implementazione
![[Pasted image 20240719130552.png]]
![[Pasted image 20240719130554.png]]
#### Analisi del timing post implementazione
![[Pasted image 20240719130356.png]]
![[Pasted image 20240719130908.png]]

### Analisi con VIO e ILA
#### Utilizzo post sintesi
![[Pasted image 20240719201545.png]]
![[Pasted image 20240719201548.png]]
#### Utilizzo post implementazione
![[Pasted image 20240719201556.png]]
![[Pasted image 20240719201558.png]]

#### Timing post implementazione
![[Pasted image 20240719201441.png]]


## Osservazioni
### Perché il modulo CPU ha porte e perché sono `pc` e `alu_result`?
1. Se il modulo non avesse porte, quando viene sintetizzato, tutta la logica verrebbe eliminata.
2. Nella realtà, l'interfaccia con il microprocessore è molto più complessa e spesso avviene tramite la condivisione della memoria.
3. Per realizzare questo tipo di interfacce, è necessario imparare standard molto complessi e, in genere, costruire sistemi di memoria con cache e una virtualizzazione della memoria.
4. Pertanto, si scelgono solo 2 uscite: `pc` e `alu_result`.
5. Questa scelta si basa sulla necessità di garantire che la logica interna della CPU non venga ottimizzata fuori durante la sintesi in Vivado. Ecco la giustificazione dettagliata per la scelta di questi segnali specifici:

#### Uscita `pc` (Program Counter)
1. **Visibilità del flusso di istruzioni**:
   - Il program counter (`pc`) è un segnale fondamentale in qualsiasi microprocessore. Rappresenta l'indirizzo dell'istruzione attualmente in esecuzione.
   - Avere `pc` come uscita fornisce visibilità diretta del flusso di istruzioni durante la simulazione e il debug, garantendo che il microprocessore stia avanzando correttamente attraverso il suo spazio di istruzioni.

2. **Assicurare la ritenzione della logica di controllo del flusso**:
   - Il `pc` è collegato a molti componenti nel design, come la memoria delle istruzioni e le unità di controllo del salto (branch). Esporre `pc` garantisce che la logica relativa al controllo del flusso di istruzioni non venga eliminata durante la sintesi.

#### Uscita `alu_result` (Risultato dell'ALU)
1. **Visibilità delle operazioni di calcolo**:
   - L'ALU (Unità Aritmetica e Logica) è un altro componente critico in qualsiasi CPU. Esegue operazioni matematiche e logiche sugli operandi.
   - Esporre `alu_result` consente di verificare che le operazioni di calcolo siano eseguite correttamente.

2. **Assicurare la ritenzione della logica di elaborazione**:
   - L'ALU interagisce con molte parti del design, come i registri e la memoria dati. Avere il risultato dell'ALU come uscita garantisce che tutta la logica relativa alle operazioni di calcolo non venga eliminata.

#### Considerazioni aggiuntive
- **Minimizzazione dell'interfaccia**: Abbiamo scelto il minimo numero di uscite necessarie per mantenere la semplicità del design, garantendo che Vivado non ottimizzi fuori la logica essenziale. Aggiungere altre uscite inutili potrebbe complicare l'interfaccia e rendere più difficile l'analisi e il debug.
- **Evitare segnali intermedi**: Non abbiamo scelto segnali intermedi o specifici delle fasi interne perché non garantiscono la ritenzione completa di tutta la logica. `pc` e `alu_result` sono segnali di alto livello che attraversano più componenti e fasi, aiutando a garantire che l'intera logica del microprocessore venga mantenuta.

### Constrain
Non è necessario aggiungere constraint di tempo esterni poiché la maggior parte del collegamento di questo elemento verrà eseguita internamente in un blocco di silicio.

## Utilizzo delle risorse

### C.1 Analisi dell'Utilizzo delle Risorse

Di seguito viene presentata un'analisi dettagliata dell'utilizzo delle risorse chiave nel design della CPU RISC-V. Le risorse valutate includono **LUT (Look-Up Tables)**, **FF (Flip-Flop)**, **BRAM (Block RAM)**, **I/O (Ingressi/Uscite)** e **BUFG (Global Buffers)**.

#### C.1.1 Look-Up Tables (LUT)
- **Utilizzo**: 4473 su 20800 (21,50%)

Le LUT rappresentano la logica combinatoria del design e sono fondamentali per l'implementazione delle operazioni aritmetiche e logiche all'interno della CPU. In questo caso, l'utilizzo delle LUT è solo del 21,5%, il che indica che il design ha spazio significativo per aggiungere più logica combinatoria se necessario.

**Benefici in un ASIC**: In un'implementazione ASIC, la logica combinatoria si traduce in transistor e porte logiche fisiche. Il fatto che il design utilizzi solo il 21,5% delle LUT disponibili nella FPGA suggerisce che l'area di silicio necessaria per implementare la logica combinatoria nell'ASIC sarà moderata, consentendo un design compatto. Inoltre, un basso utilizzo delle LUT può indicare un design efficiente dal punto di vista del consumo energetico, un aspetto chiave nella fabbricazione di ASIC.

#### C.1.2 Flip-Flop (FF)
- **Utilizzo**: 11660 su 41600 (28,03%)

I flip-flop rappresentano la logica sequenziale, ovvero i registri che memorizzano lo stato della CPU, come i registri di uso generale, i registri di controllo e gli stati intermedi delle operazioni. L'utilizzo del 28,03% dei FF suggerisce che il design ha un numero moderato di registri.

**Benefici in un ASIC**: In un'implementazione ASIC, i flip-flop si traducono in celle di memoria a basso consumo e alta velocità. L'utilizzo del 28% è adeguato per garantire che il pipeline e la logica sequenziale della CPU funzionino in modo efficiente, senza consumare eccessivamente le risorse disponibili nell'ASIC. Un uso appropriato dei flip-flop è cruciale per assicurare che il design possa gestire più istruzioni in parallelo e ottimizzare le prestazioni del pipeline.

#### C.1.3 Block RAM (BRAM)
- **Utilizzo**: 1 su 50 (2%)

L'uso della BRAM, che è una memoria embedded ad alta velocità, in questo design è molto basso, con solo il 2% di utilizzo. Ciò indica che il design attuale non dipende da grandi blocchi di memoria interna.

**Benefici in un ASIC**: In un ASIC, la memoria viene generalmente implementata tramite SRAM, che è più efficiente in termini di densità e consumo energetico rispetto alla BRAM in una FPGA. Poiché il design attuale non dipende fortemente dalla BRAM, è probabile che la memoria interna dell'ASIC sia sufficiente per soddisfare le esigenze del design senza richiedere un aumento significativo dell'area o del consumo di energia. Inoltre, questa bassa dipendenza dalla memoria offre più flessibilità per ottimizzare il design o implementare più funzionalità di memoria in una versione ASIC.

#### C.1.4 Ingressi/Uscite (I/O)
- **Utilizzo**: 35 su 210 (16,67%)

L'uso moderato dei pin di ingresso/uscita indica che il design attuale non richiede un numero eccessivo di interconnessioni esterne, il che è positivo se si vuole mantenere l'efficienza del design.

**Benefici in un ASIC**: I pin di I/O sono una risorsa preziosa in qualsiasi design ASIC, poiché i pad di ingresso e uscita tendono a essere più costosi in termini di area e consumo energetico rispetto ad altri componenti. Utilizzando solo il 16,67% dei pin disponibili sulla FPGA, questo design dimostra che le esigenze di I/O sono gestibili, il che permetterà un'implementazione ASIC con un numero ridotto di pad, riducendo l'area complessiva e il consumo energetico.

#### C.1.5 Global Buffers (BUFG)
- **Utilizzo**: 2 su 32 (6,25%)

I buffer globali sono essenziali per una distribuzione efficiente del clock in un design FPGA, poiché consentono una segnalazione uniforme del clock in tutta la struttura del circuito. Il basso utilizzo dei BUFG (6,25%) indica che il design attuale non richiede un clock particolarmente complesso o ad alta frequenza.

**Benefici in un ASIC**: In un ASIC, la distribuzione del clock è un fattore critico, poiché influisce direttamente sulla velocità di operazione e sul consumo energetico. Il basso utilizzo dei BUFG suggerisce che questo design non dipende eccessivamente dalla rete di distribuzione del clock, il che potrebbe tradursi in una rete di clock più semplice e a basso consumo in un ASIC.

### C.2 Risorse Non Utilizzate e il loro Impatto nel Design ASIC

#### C.2.1 LUTRAM (RAM basata su LUT)
- **Utilizzo**: 124 su 9600 (1,29%)

La LUTRAM è una memoria ad accesso rapido implementata sulle LUT. Il suo utilizzo è molto basso in questo design (solo 1,29%), il che suggerisce che non viene utilizzata per implementare grandi quantità di memoria di uso generale.

**Benefici in un ASIC**: In un'implementazione ASIC, la LUTRAM si tradurrebbe in matrici di memoria o SRAM. Il basso utilizzo di questa risorsa indica che il design non dipende dalla LUTRAM per lo storage temporaneo, il che è positivo, poiché l'ASIC può utilizzare memorie più efficienti e compatte. Inoltre, poiché non dipende dalla LUTRAM, il design ha maggiore flessibilità su come distribuire la logica in un ASIC, permettendo una maggiore ottimizzazione dell'area del silicio.

#### C.2.2 SP (Single-Port RAM) e altre risorse di memoria
In questa implementazione non si osserva l'uso esplicito di memorie di tipo **SP (Single-Port RAM)** o di memorie più complesse con più porte. L'assenza di queste memorie suggerisce che il design è relativamente semplice in termini di gestione dei dati o che delega la gestione della memoria ad altri moduli esterni al processore.

**Benefici in un ASIC**: La mancanza di utilizzo di memorie complesse a più porte implica che il design non richiede l'accesso simultaneo a memorie di grandi dimensioni, il che semplifica considerevolmente l'implementazione in un ASIC. Piuttosto che dover progettare strutture di memoria avanzate, l'attenzione può concentrarsi sull'implementazione di SRAM o DRAM più semplici ed efficienti, riducendo l'area e il consumo di potenza.

#### C.2.3 DSP
Possiamo osservare un nullo utilizzo delle componenti DSP della FPGA, il che implica che le operazioni aritmetiche siano implementate interamente attraverso le LUT. Sebbene non sia ideale per mantenere un'implementazione in FPGA, dove le DSP sono progettate per ottimizzare risorse e velocità, nel nostro caso, trattandosi di un design destinato a un'eventuale implementazione in ASIC, va bene così.

### C.3 Conclusioni Generali

Questa analisi dimostra che il design della CPU RISC-V è ben bilanciato in termini di utilizzo delle risorse, con un uso moderato di LUT, flip-flop e pin I/O. I benefici principali di questo profilo di utilizzo per un design ASIC includono:

- **Scalabilità**: C'è abbastanza spazio nell'uso della logica combinatoria (LUT) e sequenziale (FF) per aggiungere più funzionalità o migliorare il pipeline.
- **Efficienza energetica**: Il basso utilizzo di BRAM e LUTRAM indica che il design è efficiente in termini di consumo energetico, il che è cruciale in un'implementazione ASIC.
- **Semplificazione del design ASIC**: La bassa dipendenza dalla rete di distribuzione del clock e da risorse complesse come LUTRAM e BRAM significa che il design può essere implementato in un ASIC in modo efficiente, con un uso controllato dell'area di silicio e del consumo energetico.

# Spagnolo
## Código

El módulo `CPU_pipeline` implementa una CPU basada en el ISA RISC-V con pipeline. Este diseño consta de cinco etapas: IF (Instruction Fetch), ID (Instruction Decode), EX (Execute), MEM (Memory Access), y WB (Write Back) ya definidas y conocidas anteriormente. Se utilizan registros de pipeline para almacenar los resultados intermedios entre las etapas y mantener el flujo continuo de instrucciones.

#### Interfaz del Módulo
```verilog
module CPU (
    input           clk,
    input           reset,
    output [31:0]   pc,          // Contador de programa
    output [31:0]   alu_result   // Resultado de la ALU
);
```
- `clk`: Señal de reloj para sincronizar las operaciones de la CPU.
- `reset`: Señal de reset para inicializar la CPU.
- `pc`: Salida del contador de programa que indica la dirección de la instrucción actual.
- `alu_result`: Salida del resultado de la ALU que muestra el resultado de la operación aritmética/lógica más reciente.

#### Señales Internas y Registros de Pipeline
- **IF/ID Pipeline Registers**: Almacenan la instrucción y la dirección actual después de la etapa de Instruction Fetch (IF).
- **ID/EX Pipeline Registers**: Almacenan los valores leídos de los registros, la constante inmediata, y señales de control después de la etapa de Instruction Decode (ID).
- **EX/MEM Pipeline Registers**: Almacenan el resultado de la ALU, el segundo operando, y señales de control después de la etapa de Execute (EX).
- **MEM/WB Pipeline Registers**: Almacenan los datos leídos de la memoria y señales de control después de la etapa de Memory Access (MEM).

#### Etapa IF (Instruction Fetch)
```verilog
// Etapa IF
IF if_stage (
    .o_address(if_address),
    .o_instruccion(if_instruccion),
    .i_branch_address(exmem_PCBranch),
    .i_reset(reset),
    .i_select(mem_PCSrc),
    .i_clock(clk)
);
```
- `o_address`: Dirección actual del PC.
- `o_instruccion`: Instrucción leída de la memoria de instrucciones.
- `i_branch_address`: Dirección de destino en caso de una instrucción de salto.
- `i_reset`: Señal de reset.
- `i_select`: Señal para seleccionar entre la dirección siguiente y la dirección de salto.
- `i_clock`: Señal de reloj.

#### Registros de Pipeline IF/ID
```verilog
// Pipeline IF/ID
always @(posedge clk or posedge reset) begin
    if (reset) begin
        ifid_instruccion <= 32'b0;
        ifid_address <= 31'b0;
    end else begin
        ifid_instruccion <= if_instruccion;
        ifid_address <= if_address;
    end
end
```
- `ifid_instruccion`: Instrucción almacenada para la etapa de ID.
- `ifid_address`: Dirección actual del PC almacenada para la etapa de ID.

#### Etapa ID (Instruction Decode)
```verilog
// Etapa ID
ID id_stage (
    .i_clk(clk),
    .i_reset(reset),
    .i_instruccion(ifid_instruccion),
    .i_WriteData(wb_WriteData),
    .i_RegWrite(memwb_RegWrite),
    .i_WriteReg(memwb_WriteReg),
    .o_register1(id_register1),
    .o_register2(id_register2),
    .o_constante(id_constante),
    .o_WriteReg(id_WriteReg),
    .o_RegWrite(id_RegWrite),
    .o_ALUSrc(id_ALUSrc),
    .o_MemWrite(id_MemWrite),
    .o_MemRead(id_MemRead),
    .o_Branch(id_Branch),
    .o_MemToReg(id_MemToReg),
    .o_SLTc(id_SLTc),
    .o_ALUControl(id_ALUControl),
    .o_BranchOp(id_BranchOp)
);
```
- `i_clk`: Señal de reloj.
- `i_reset`: Señal de reset.
- `i_instruccion`: Instrucción a decodificar.
- `i_WriteData`: Datos a escribir en el registro destino.
- `i_RegWrite`: Señal de control para habilitar escritura en registros.
- `i_WriteReg`: Registro destino para la escritura.
- `o_register1`: Valor leído del primer registro fuente.
- `o_register2`: Valor leído del segundo registro fuente.
- `o_constante`: Constante inmediata decodificada.
- `o_WriteReg`: Registro destino decodificado.
- `o_RegWrite`: Señal de control para habilitar escritura en registros.
- `o_ALUSrc`: Señal de control para seleccionar el segundo operando de la ALU.
- `o_MemWrite`: Señal de control para escribir en la memoria de datos.
- `o_MemRead`: Señal de control para leer de la memoria de datos.
- `o_Branch`: Señal de control para realizar un salto condicional.
- `o_MemToReg`: Señal de control para seleccionar el dato que se escribe en el registro destino.
- `o_SLTc`: Señal de control para la instrucción SLT.
- `o_ALUControl`: Señal de control para determinar la operación de la ALU.
- `o_BranchOp`: Señal de control para determinar el tipo de salto.

#### Registros de Pipeline ID/EX
```verilog
// Pipeline ID/EX
always @(posedge clk or posedge reset) begin
    if (reset) begin
        idex_PC <= 32'b0;
        idex_register1 <= 32'b0;
        idex_register2 <= 32'b0;
        idex_constante <= 32'b0;
        idex_ALUSrc <= 1'b0;
        idex_ALUControl <= 3'b0;
        idex_MemWrite <= 1'b0;
        idex_MemRead <= 1'b0;
        idex_Branch <= 1'b0;
        idex_MemToReg <= 1'b0;
        idex_RegWrite <= 1'b0;
        idex_BranchOp <= 2'b0;
        idex_SLTc <= 1'b0;
        idex_WriteReg <= 5'b0;
    end else begin
        idex_PC <= {ifid_address, 1'b0};
        idex_register1 <= id_register1;
        idex_register2 <= id_register2;
        idex_constante <= id_constante;
        idex_ALUSrc <= id_ALUSrc;
        idex_ALUControl <= id_ALUControl;
        idex_MemWrite <= id_MemWrite;
        idex_MemRead <= id_MemRead;
        idex_Branch <= id_Branch;
        idex_MemToReg <= id_MemToReg;
        idex_RegWrite <= id_RegWrite;
        idex_BranchOp <= id_BranchOp;
        idex_SLTc <= id_SLTc;
        idex_WriteReg <= id_WriteReg;
    end
end
```
- `idex_PC`: Dirección actual del PC almacenada para la etapa de EX.
- `idex_register1`: Valor del primer registro fuente almacenado para la etapa de EX.
- `idex_register2`: Valor del segundo registro fuente almacenado para la etapa de EX.
- `idex_constante`: Constante inmediata almacenada para la etapa de EX.
- `idex_ALUSrc`: Señal de control almacenada para la etapa de EX.
- `idex_ALUControl`: Señal de control para la ALU almacenada para la etapa de EX.
- `idex_MemWrite`: Señal de control almacenada para la etapa de EX.
- `idex_MemRead`: Señal de control almacenada para la etapa de EX.
- `idex_Branch`: Señal de control almacenada para la etapa de EX.
- `idex_MemToReg`: Señal de control almacenada para la etapa de EX.
- `idex_RegWrite`: Señal de control almacenada para la etapa de EX.
- `idex_BranchOp`: Señal de control almacenada para la etapa de EX.
- `idex_SLTc`: Señal de control almacenada para la etapa de EX.
- `idex_WriteReg`: Registro destino almacenado para la etapa de EX.

#### Etapa EX (Execute)
```verilog
// Etapa EX
EX ex_stage (
    .i_PC(idex_PC),
    .i_register1(idex_register1),
    .i_register2(idex_register2),
    .i_constante(idex_constante),
    .i_ALUSrc(idex_ALUSrc),
    .i_ALUControl(idex_ALUControl),
    .o_ALUResult(ex_ALUResult),
    .o_PCBranch(ex_PCBranch),
    .o_zero(ex_zero),
    .o_negative(ex_negative)
);
```
- `i_PC`: Dirección del PC de entrada.
- `i_register1`: Primer operando de la ALU.
- `i_register2`: Segundo operando de la ALU.
- `i_constante`: Constante inmediata.
- `i_ALUSrc`: Señal de control

 para seleccionar el segundo operando de la ALU.
- `i_ALUControl`: Señal de control para determinar la operación de la ALU.
- `o_ALUResult`: Resultado de la ALU.
- `o_PCBranch`: Dirección de salto calculada.
- `o_zero`: Señal que indica si el resultado de la ALU es cero.
- `o_negative`: Señal que indica si el resultado de la ALU es negativo.

#### Registros de Pipeline EX/MEM
```verilog
// Pipeline EX/MEM
always @(posedge clk or posedge reset) begin
    if (reset) begin
        exmem_ALUResult <= 32'b0;
        exmem_register2 <= 32'b0;
        exmem_PCBranch <= 32'b0;
        exmem_zero <= 1'b0;
        exmem_negative <= 1'b0;
        exmem_MemWrite <= 1'b0;
        exmem_MemRead <= 1'b0;
        exmem_Branch <= 1'b0;
        exmem_MemToReg <= 1'b0;
        exmem_RegWrite <= 1'b0;
        exmem_BranchOp <= 2'b0;
        exmem_SLTc <= 1'b0;
        exmem_WriteReg <= 5'b0;
    end else begin
        exmem_ALUResult <= ex_ALUResult;
        exmem_register2 <= idex_register2;
        exmem_PCBranch <= ex_PCBranch;
        exmem_zero <= ex_zero;
        exmem_negative <= ex_negative;
        exmem_MemWrite <= idex_MemWrite;
        exmem_MemRead <= idex_MemRead;
        exmem_Branch <= idex_Branch;
        exmem_MemToReg <= idex_MemToReg;
        exmem_RegWrite <= idex_RegWrite;
        exmem_BranchOp <= idex_BranchOp;
        exmem_SLTc <= idex_SLTc;
        exmem_WriteReg <= idex_WriteReg;
    end
end
```
- `exmem_ALUResult`: Resultado de la ALU almacenado para la etapa de MEM.
- `exmem_register2`: Valor del segundo operando almacenado para la etapa de MEM.
- `exmem_PCBranch`: Dirección de salto calculada almacenada para la etapa de MEM.
- `exmem_zero`: Señal de resultado cero almacenada para la etapa de MEM.
- `exmem_negative`: Señal de resultado negativo almacenada para la etapa de MEM.
- `exmem_MemWrite`: Señal de control almacenada para la etapa de MEM.
- `exmem_MemRead`: Señal de control almacenada para la etapa de MEM.
- `exmem_Branch`: Señal de control almacenada para la etapa de MEM.
- `exmem_MemToReg`: Señal de control almacenada para la etapa de MEM.
- `exmem_RegWrite`: Señal de control almacenada para la etapa de MEM.
- `exmem_BranchOp`: Señal de control almacenada para la etapa de MEM.
- `exmem_SLTc`: Señal de control almacenada para la etapa de MEM.
- `exmem_WriteReg`: Registro destino almacenado para la etapa de MEM.

#### Etapa MEM (Memory Access)
```verilog
// Etapa MEM
MEM mem_stage (
    .i_clk(clk),
    .i_reset(reset),
    .i_Address(exmem_ALUResult),
    .i_WriteData(exmem_register2),
    .i_BranchOp(exmem_BranchOp),
    .i_negative(exmem_negative),
    .i_zero(exmem_zero),
    .i_branch(exmem_Branch),
    .i_MemWrite(exmem_MemWrite),
    .i_MemRead(exmem_MemRead),
    .i_SLTc(exmem_SLTc),
    .o_PCSrc(mem_PCSrc),
    .o_ReadData(mem_ReadData),
    .o_Mux(mem_Mux)
);
```
- `i_clk`: Señal de reloj.
- `i_reset`: Señal de reset.
- `i_Address`: Dirección de memoria a acceder.
- `i_WriteData`: Datos a escribir en la memoria.
- `i_BranchOp`: Señal de control para determinar el tipo de salto.
- `i_negative`: Señal de resultado negativo.
- `i_zero`: Señal de resultado cero.
- `i_branch`: Señal de control para realizar un salto condicional.
- `i_MemWrite`: Señal de control para escribir en la memoria de datos.
- `i_MemRead`: Señal de control para leer de la memoria de datos.
- `i_SLTc`: Señal de control para la instrucción SLT.
- `o_PCSrc`: Señal de control para seleccionar la dirección siguiente del PC.
- `o_ReadData`: Datos leídos de la memoria de datos.
- `o_Mux`: Salida del multiplexor.

#### Registros de Pipeline MEM/WB
```verilog
// Pipeline MEM/WB
always @(posedge clk or posedge reset) begin
    if (reset) begin
        memwb_ALUResult <= 32'b0;
        memwb_ReadData <= 32'b0;
        memwb_MemToReg <= 1'b0;
        memwb_RegWrite <= 1'b0;
        memwb_WriteReg <= 5'b0;
    end else begin
        memwb_ALUResult <= exmem_ALUResult;
        memwb_ReadData <= mem_ReadData;
        memwb_MemToReg <= exmem_MemToReg;
        memwb_RegWrite <= exmem_RegWrite;
        memwb_WriteReg <= exmem_WriteReg;
    end
end
```
- `memwb_ALUResult`: Resultado de la ALU almacenado para la etapa de WB.
- `memwb_ReadData`: Datos leídos de la memoria almacenados para la etapa de WB.
- `memwb_MemToReg`: Señal de control almacenada para la etapa de WB.
- `memwb_RegWrite`: Señal de control almacenada para la etapa de WB.
- `memwb_WriteReg`: Registro destino almacenado para la etapa de WB.

#### Etapa WB (Write Back)
```verilog
// Etapa WB
MUX32 wb_mux (
    .A(memwb_ALUResult),
    .B(memwb_ReadData),
    .select(memwb_MemToReg),
    .C(wb_WriteData)
);
```
- `A`: Resultado de la ALU.
- `B`: Datos leídos de la memoria.
- `select`: Señal de control para seleccionar el dato que se escribe en el registro destino.
- `C`: Datos a escribir en el registro destino.


## Análisis

![[Pasted image 20240716172751.png]]

### Codigo de simulacion

#### Código en Ensamblador RISC-V

```assembly
.section .text
.globl _start

_start:
    # Inicializar los registros usando ADDI
    addi x1, x0, 0    # x1 = 0
    addi x2, x0, 4    # x2 = 4
    addi x3, x0, 0    # x3 = 0
    addi x4, x0, 16   # x4 = 16
    addi x5, x0, 256  # x5 = 256
    addi x6, x0, 512  # x6 = 512
	addi x7, x0, 1792  # x7 = 1792
    

    # Pipeline delay simulation
    nop               # No operation (simulate delay)
    nop               # No operation (simulate delay)
    nop               # No operation (simulate delay)
    nop               # No operation (simulate delay)

loop:
    add x8, x7, x2    # Add offset to the loaded value, result in x8

    # Pipeline delay simulation for ALU operation
    nop               # No operation (simulate ALU delay)
    nop               # No operation (simulate ALU delay)

    sw x8, 0(x7)      # Store result to output_data

    # Pipeline delay simulation for memory write
    nop               # No operation (simulate memory write delay)
    nop               # No operation (simulate memory write delay)
    nop               # No operation (simulate memory write delay)

    addi x5, x5, 4    # Move to the next input_data element
    addi x6, x6, 4    # Move to the next output_data element
    addi x3, x3, 4    # Update base address for input_data
    
	nop              
    nop
    nop
    
    blt x3, x4, loop  # Loop if not all elements are processed

    # End of program (infinite loop)
end:
    nop               # Infinite loop
    nop
    nop
```
#### Archivo `mem.mem`

```
00000093
00400113
00000193
01000213
10000293
20000313
70100393
00000013
00000013
00000013
00000013
00238433
00000013
00000013
0073A023
00000013
00000013
00000013
00428293
00430313
00418193
00000013
00000013
00000013
FE41CAE3
00000013
00000013
00000013
00000013
```


#### Tabla de Ejecución

| PC (Hex)   | Instrucción      | Registro/Espacio de Memoria Modificado | Resultado de la ALU                                     |
| ---------- | ---------------- | -------------------------------------- | ------------------------------------------------------- |
| 0x00000000 | addi x1, x0, 0   | x1                                     | 0                                                       |
| 0x00000001 | addi x2, x0, 4   | x2                                     | 4                                                       |
| 0x00000002 | addi x3, x0, 0   | x3                                     | 0                                                       |
| 0x00000003 | addi x4, x0, 16  | x4                                     | 16                                                      |
| 0x00000004 | addi x5, x0, 256 | x5                                     | 256                                                     |
| 0x00000005 | addi x6, x0, 512 | x6                                     | 512                                                     |
| 0x00000006 | addi x7, x0, 256 | x7                                     | 1792                                                    |
| 0x00000007 | nop              | -                                      | -                                                       |
| 0x00000008 | nop              | -                                      | -                                                       |
| 0x00000009 | nop              | -                                      | -                                                       |
| 0x0000000A | nop              | -                                      | -                                                       |
| 0x0000000B | add x8, x7, x2   | x8                                     | 14 (10 + 4)                                             |
| 0x0000000C | nop              | -                                      | -                                                       |
| 0x0000000D | nop              | -                                      | -                                                       |
| 0x0000000E | sw x8, 0(x7)     | Memoria en 0x1792                      | 14                                                      |
| 0x0000000F | nop              | -                                      | -                                                       |
| 0x00000010 | nop              | -                                      | -                                                       |
| 0x00000011 | nop              | -                                      | -                                                       |
| 0x00000012 | addi x5, x5, 4   | x5                                     | 260 (0x104)                                             |
| 0x00000013 | addi x6, x6, 4   | x6                                     | 516 (0x204)                                             |
| 0x00000014 | addi x3, x3, 4   | x3                                     | 4                                                       |
| 0x00000015 | blt x3, x4, loop | PC                                     | Salta a 0x0000000B (x3 < x4)                            |
| 0x0000000B | add x8, x7, x2   | x8                                     | 24 (20 + 4) (nota: se debe ejecutar lw x7, 0(x5) antes) |
| 0x0000000C | nop              | -                                      | -                                                       |
| 0x0000000D | nop              | -                                      | -                                                       |
| 0x0000000E | sw x8, 0(x7)     | Memoria en 0x204                       | 3073                                                    |
| 0x0000000F | nop              | -                                      | -                                                       |
| 0x00000010 | nop              | -                                      | -                                                       |
| 0x00000011 | nop              | -                                      | -                                                       |
| 0x00000012 | addi x5, x5, 4   | x5                                     | 264 (0x108)                                             |
| 0x00000013 | addi x6, x6, 4   | x6                                     | 520 (0x208)                                             |
| 0x00000014 | addi x3, x3, 4   | x3                                     | 8                                                       |
| 0x00000015 | blt x3, x4, loop | PC                                     | Salta a 0x0000000B (x3 < x4)                            |
| 0x0000000B | add x8, x7, x2   | x8                                     | 34 (30 + 4) (nota: se debe ejecutar lw x7, 0(x5) antes) |
| 0x0000000C | nop              | -                                      | -                                                       |
| 0x0000000D | nop              | -                                      | -                                                       |
| 0x0000000E | sw x8, 0(x6)     | Memoria en 0x208                       | 34                                                      |
| 0x0000000F | nop              | -                                      | -                                                       |
| 0x00000010 | nop              | -                                      | -                                                       |
| 0x00000011 | nop              | -                                      | -                                                       |
| 0x00000012 | addi x5, x5, 4   | x5                                     | 268 (0x10C)                                             |
| 0x00000013 | addi x6, x6, 4   | x6                                     | 524 (0x20C)                                             |
| 0x00000014 | addi x3, x3, 4   | x3                                     | 12                                                      |
| 0x00000015 | blt x3, x4, loop | PC                                     | Salta a 0x0000000B (x3 < x4)                            |
| 0x0000000B | add x8, x7, x2   | x8                                     | 44 (40 + 4) (nota: se debe ejecutar lw x7, 0(x5) antes) |
| 0x0000000C | nop              | -                                      | -                                                       |
| 0x0000000D | nop              | -                                      | -                                                       |
| 0x0000000E | sw x8, 0(x6)     | Memoria en 0x20C                       | 44                                                      |
| 0x0000000F | nop              | -                                      | -                                                       |
| 0x00000010 | nop              | -                                      | -                                                       |
| 0x00000011 | nop              | -                                      | -                                                       |
| 0x00000012 | addi x5, x5, 4   | x5                                     | 272 (0x110)                                             |
| 0x00000013 | addi x6, x6, 4   | x6                                     | 528 (0x210)                                             |
| 0x00000014 | addi x3, x3, 4   | x3                                     | 16                                                      |
| 0x00000015 | blt x3, x4, loop | PC                                     | No salta (x3 no < x4)                                   |
| 0x00000016 | nop              | -                                      | -                                                       |
| 0x00000017 | nop              | -                                      | -                                                       |
| 0x00000018 | nop              | -                                      | -                                                       |

### Analisis con placa Artix A7 35t
#### Pre sintesis
![[Pasted image 20240717212852.png]]
#### Post sintesis
![[Pasted image 20240717160714.png]]
![[Pasted image 20240716174921.png]]
![[Pasted image 20240716174923.png]]

##### Funcional
![[Pasted image 20240717212911.png]]
##### Timing
![[Pasted image 20240717212933.png]]
#### Post implementacion
##### Timing
![[Pasted image 20240717161523.png]]
![[Pasted image 20240717161651.png]]

##### Utilizacion
![[Pasted image 20240717161558.png]]
![[Pasted image 20240717161602.png]]
##### Funcional
![[Pasted image 20240717213024.png]]
##### Timing
![[Pasted image 20240717213107.png]]
![[Pasted image 20240716175006.png]]
### Analisis con placa CMOD 35t
#### Simulacion pre sintesis
![[Pasted image 20240719115339.png]]
#### Utilizacion post sintesis
![[Pasted image 20240719130524.png]]
![[Pasted image 20240719130526.png]]
#### Utilizacion post implementacion
![[Pasted image 20240719130552.png]]
![[Pasted image 20240719130554.png]]
#### Analisis de timing post implementacion
![[Pasted image 20240719130356.png]]
![[Pasted image 20240719130908.png]]

### Analisis con VIO e ILA
#### Utilizacion post sintesis
![[Pasted image 20240719201545.png]]
![[Pasted image 20240719201548.png]]
#### Utilizacion post implementacion
![[Pasted image 20240719201556.png]]
![[Pasted image 20240719201558.png]]

#### Timing post implementacion
![[Pasted image 20240719201441.png]]

## Observaciones
### Porque tiene puerto el modulo CPU y porque son PC y ALU result?
1. Si el modulo no tendria puertos, cuando se sintetize se eliminaria toda la logica
2. En la realidad la interfaz con el microprocesador es mucho mas complicada y se suele dar mediante el compartimiento de memoria
3. Para realizar este tipo de interfaces, se necesita aprender estandares muy complicado y por lo general realizar sistemas de memoria con memorias caches y una virtualizacion de la memoria
4. Por lo tanto se eligen solo 2 salidas: `pc` y `alu_result` 
5. Esta eleccion se basa en la necesidad de asegurar que la lógica interna de la CPU no sea optimizada fuera durante la síntesis en Vivado. Aquí está la justificación detallada para la elección de estas señales específicas:

#### Salida `pc` (Program Counter)
1. **Visibilidad del Flujo de Instrucciones**:
   - El contador de programa (`pc`) es una señal fundamental en cualquier microprocesador. Representa la dirección de la instrucción actual que se está ejecutando.
   - Tener `pc` como una salida proporciona visibilidad directa del flujo de instrucciones durante la simulación y la depuración, asegurando que el microprocesador está avanzando correctamente a través de su espacio de instrucciones.

2. **Asegurar la Retención de la Lógica de Control del Flujo**:
   - El `pc` está conectado a múltiples componentes dentro del diseño, como la memoria de instrucciones y las unidades de control de salto (branch). Al exponer `pc`, garantizamos que la lógica relacionada con el control del flujo de instrucciones no sea eliminada durante la síntesis.

#### Salida `alu_result` (Resultado de la ALU)
1. **Visibilidad de las Operaciones de Cálculo**:
   - La Unidad Aritmética y Lógica (ALU) es otro componente crítico en cualquier CPU. Realiza operaciones matemáticas y lógicas sobre los operandos.
   - Exponer `alu_result` permite verificar que las operaciones de cálculo se están realizando correctamente.

2. **Asegurar la Retención de la Lógica de Procesamiento**:
   - La ALU interactúa con múltiples partes del diseño, como los registros y la memoria de datos. Tener el resultado de la ALU como una salida asegura que toda la lógica relacionada con las operaciones de cálculo no sea eliminada.


#### Consideraciones Adicionales
- **Minimización de la Interfaz**: Elegimos las salidas mínimas necesarias para mantener la simplicidad del diseño mientras garantizamos que Vivado no optimice fuera la lógica esencial. Añadir más salidas innecesarias podría complicar la interfaz y hacer más difícil el análisis y la depuración.
- **Evitar Señales Intermedias**: No elegimos señales intermedias o específicas de etapas internas porque no garantizan la retención completa de toda la lógica. `pc` y `alu_result` son señales de alto nivel que abarcan múltiples componentes y etapas, lo que ayuda a asegurar que la lógica completa del microprocesador se retenga.



### Constrain
No hace falta agregarle constrain de tiempo externos ya que el conexionado de este elemento en su gran mayoria se hara internamente en un bloque de silicio

## Utilizacion de recursos

### C.1 Análisis de la Utilización de Recursos

A continuación, se presenta un análisis detallado de la utilización de los recursos clave en el diseño de la CPU RISC-V. Los recursos evaluados incluyen las **LUT (Look-Up Tables)**, **FF (Flip-Flops)**, **BRAM (Block RAM)**, **I/O (Entradas/Salidas)**, y **BUFG (Global Buffers)**.

#### C.1.1 Look-Up Tables (LUT)
- **Utilización**: 4473 de 20800 (21.50%)
  
Las LUTs representan la lógica combinacional del diseño y son fundamentales para la implementación de las operaciones aritméticas y lógicas dentro de la CPU. En este caso, la utilización de LUT es de solo el 21.5%, lo que indica que el diseño tiene espacio significativo para añadir más lógica combinacional si fuera necesario.

**Beneficios en un ASIC**: En una implementación ASIC, la lógica combinacional se traduce en transistores y puertas lógicas físicas. El hecho de que el diseño utilice solo un 21.5% de las LUT disponibles en la FPGA sugiere que el área de silicio requerida para implementar la lógica combinacional en el ASIC será moderada, permitiendo un diseño compacto. Además, una utilización baja de LUT puede indicar un diseño eficiente en cuanto a consumo de energía, un aspecto clave en la fabricación de ASIC.

#### C.1.2 Flip-Flops (FF)
- **Utilización**: 11660 de 41600 (28.03%)

Los flip-flops representan la lógica secuencial, es decir, los registros que almacenan el estado de la CPU, como los registros de propósito general, registros de control y estados intermedios de las operaciones. La utilización del 28.03% de los FF sugiere que el diseño cuenta con un número moderado de registros.

**Beneficios en un ASIC**: En una implementación ASIC, los flip-flops se traducen en celdas de almacenamiento de bajo consumo y alta velocidad. La utilización de un 28% es adecuada para asegurar que el pipeline y la lógica secuencial de la CPU funcionen de manera eficiente, sin consumir excesivamente los recursos disponibles en el ASIC. Un uso adecuado de flip-flops es crucial para asegurar que el diseño pueda manejar múltiples instrucciones en paralelo, así como para optimizar el rendimiento del pipeline.

#### C.1.3 Block RAM (BRAM)
- **Utilización**: 1 de 50 (2%)

El uso de BRAM, que es una memoria embebida de alta velocidad, en este diseño es muy bajo, con solo un 2% de utilización. Esto indica que el diseño actual no depende de grandes bloques de memoria interna.

**Beneficios en un ASIC**: En un ASIC, la memoria se implementa generalmente mediante SRAM, que es más eficiente en cuanto a densidad y consumo de energía en comparación con BRAM en una FPGA. Dado que el diseño actual no depende fuertemente de BRAM, es probable que la memoria interna del ASIC sea suficiente para cubrir las necesidades del diseño sin requerir un aumento significativo en el área o el consumo de energía. Además, esta baja utilización de memoria puede permitir más flexibilidad para optimizar el diseño o implementar más funcionalidad de memoria en una versión ASIC.

#### C.1.4 Entradas/Salidas (I/O)
- **Utilización**: 35 de 210 (16.67%)

El uso moderado de los pines de entrada/salida indica que el diseño actual no requiere una gran cantidad de interconexiones externas, lo cual es positivo si el objetivo es mantener el diseño eficiente.

**Beneficios en un ASIC**: Los pines de I/O son un recurso valioso en cualquier diseño ASIC, ya que los pads de entrada y salida tienden a ser más costosos en términos de área y consumo energético que otros componentes. Al utilizar solo el 16.67% de los pines disponibles en la FPGA, este diseño muestra que las necesidades de I/O son manejables, lo que permitirá una implementación ASIC con un número reducido de pads, reduciendo el área total y el consumo energético.

#### C.1.5 Global Buffers (BUFG)
- **Utilización**: 2 de 32 (6.25%)

Los búferes globales son esenciales para la distribución eficiente del reloj en un diseño FPGA, ya que permiten una señal de reloj uniforme en toda la estructura del circuito. El uso bajo de BUFG (6.25%) indica que el diseño actual no requiere un reloj particularmente complejo o de alta frecuencia.

**Beneficios en un ASIC**: En un ASIC, la distribución del reloj es un factor crítico, ya que impacta directamente en la velocidad de operación y el consumo energético. El uso bajo de BUFG sugiere que este diseño no tiene dependencias excesivas en la red de distribución del reloj, lo que puede traducirse en una red de reloj más sencilla y de menor consumo en el ASIC.

### C.2 Recursos No Utilizados y su Impacto en el Diseño ASIC

#### C.2.1 LUTRAM (RAM basada en LUT)
- **Utilización**: 124 de 9600 (1.29%)

La LUTRAM es una memoria de acceso rápido implementada sobre las LUTs. Su uso es muy bajo en este diseño (solo un 1.29%), lo cual sugiere que no se está empleando para implementar grandes cantidades de memoria de propósito general.

**Beneficios en un ASIC**: En una implementación ASIC, la LUTRAM se traduciría en matrices de memoria o SRAM. El bajo uso de este recurso indica que el diseño no depende de la LUTRAM para almacenamiento temporal, lo cual es positivo, ya que el ASIC puede hacer uso de memorias más eficientes y compactas. Además, al no depender de LUTRAM, el diseño tiene más flexibilidad en términos de cómo se distribuye la lógica en un ASIC, permitiendo una mayor optimización del área de silicio.

#### C.2.2 SP (Single-Port RAM) y otros recursos de memoria
En esta implementación no se observa el uso explícito de memorias de tipo **SP (Single-Port RAM)** o memorias más complejas de varios puertos. La ausencia de estas memorias sugiere que el diseño es relativamente sencillo en términos de manejo de datos o que delega la administración de memoria en otros módulos fuera del procesador.

**Beneficios en un ASIC**: La falta de utilización de memorias complejas de múltiples puertos implica que el diseño no requiere acceso concurrente a memorias de alta capacidad, lo cual simplifica considerablemente la implementación en un ASIC. En lugar de tener que diseñar estructuras de memoria avanzadas, el enfoque puede centrarse en implementar SRAM o DRAM más sencillas y eficientes, lo que contribuye a reducir el área y el consumo de potencia.

#### C.2.3 DSP
Podemos ver una nula utilización de las componentes DSP de la FPGA, lo cual implica que las operaciones aritmeticas esten implementadas en su totalidad por LUTs, lo cual no es idoneo en caso de querer mantener una implementacion en FPGA, ya que la DSP estan diseñadas para optimizar recursos y velocidad, pero en nuestro caso, que es solo un diseño que en un futuro se podria aplicar sobre un ASIC, esta bastante bien nuestro diseño.
### C.3 Conclusiones Generales

Este análisis muestra que el diseño de la CPU RISC-V está bien balanceado en términos de utilización de recursos, con un uso moderado de LUTs, flip-flops y pines de I/O. Los beneficios clave de este perfil de utilización para un diseño ASIC incluyen:

- **Escalabilidad**: Hay suficiente espacio en el uso de lógica combinacional (LUT) y secuencial (FF) para agregar más funcionalidades o mejorar el pipeline.
- **Eficiencia energética**: El bajo uso de BRAM y LUTRAM indica que el diseño es eficiente en cuanto a consumo de energía, lo que es crucial en una implementación ASIC.
- **Simplificación del diseño ASIC**: La baja dependencia en la red de distribución de reloj y en recursos complejos como LUTRAM y BRAM significa que el diseño puede implementarse en un ASIC de manera eficiente, con un uso controlado del área de silicio y el consumo energético.

