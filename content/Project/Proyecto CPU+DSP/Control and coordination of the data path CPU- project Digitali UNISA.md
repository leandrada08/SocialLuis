

# Italiano
![[Pasted image 20240410174711.png]]
## Controllo generale

Il modulo **CONTROL** decodifica l'opcode dell'istruzione per generare i segnali di controllo necessari per le diverse fasi del pipeline della CPU.

- **Ingressi**:
  - `opcode`: Codice operativo dell'istruzione.

- **Uscite**:
  - `RegWrite`: Segnale per scrivere nel banco registri.
  - `ALUSrc`: Segnale per selezionare il secondo operando dell'ALU.
  - `MemWrite`: Segnale per scrivere nella memoria dati.
  - `MemRead`: Segnale per leggere dalla memoria dati.
  - `Branch`: Segnale per eseguire un salto condizionato.
  - `MemToReg`: Segnale per selezionare il dato che viene scritto nel registro di destinazione.
  - `ALUop`: Codice operativo per l'ALU.

- **Funzionamento**:
  - Decodifica l'`opcode` per assegnare i segnali di controllo corrispondenti.
  - Genera segnali specifici per istruzioni di tipo R, I, S, B e loads.

### Tabella di funzionamento

| Istruzione      | RegWrite | ALUSrc | MemWrite | MemRead | Branch | MemToReg | ALUop |
| --------------- | -------- | ------ | -------- | ------- | ------ | -------- | ----- |
| Tipo R          | 1        | 0      | 0        | 0       | 0      | 0        | 10    |
| Tipo I senza loads | 1      | 1      | 0        | 0       | 0      | 0        | 11    |
| Tipo I Loads    | 1        | 1      | 0        | 1       | 0      | 1        | 00    |
| Tipo S          | 0        | 1      | 1        | 0       | 0      | 0        | 00    |
| Tipo B          | 0        | 0      | 0        | 0       | 1      | 0        | 01    |

### RTL
![[Pasted image 20240712131438.png]]

### Simulazioni
![[Pasted image 20240422134017.png]]

### Utilizzo
![[Pasted image 20240712132910.png]]

## Controllo ALU
Il modulo **ALU_CONTROL** determina i segnali di controllo per l'ALU basandosi sui campi `funct3`, `funct7` dell'istruzione e sul codice operativo `ALUop`.

- **Ingressi**:
  - `ALUop`: Codice operativo dell'ALU.
  - `funct3`: Campo funct3 dell'istruzione.
  - `funct7`: Campo funct7 dell'istruzione.
  
- **Uscite**:
  - `ALUControl`: Segnale di controllo per l'ALU.
  - `BranchOp`: Segnale di operazione di branch.
  - `SLTc`: Segnale di controllo per SLT.

- **Funzionamento**:
  - `ALUControl` viene assegnato in base alle combinazioni di `funct3`, `funct7` e `ALUop`.
  - `BranchOp` determina l'operazione di branch in base a `ALUop` e `funct3`.
  - `SLTc` si attiva per le operazioni SLT e SLTU.

### Tabella di funzionamento

| Istruzione | ALUop | funct3      | funct7  | ALU_Control | BranchOp | SLTc |
| ---------- | ----- | ----------- | ------- | ----------- | -------- | ---- |
| ADD        | 10    | 000         | 0000000 | 000         | 00       | 0    |
| SUB        | 10    | 000         | 0100000 | 001         | 00       | 0    |
| SLL        | 10    | 001         | 0000000 | 010         | 00       | 0    |
| SLT        | 10    | 010         | 0000000 | 001         | 10       | 1    |
| SLTU       | 10    | 011         | 0000000 | 011         | 10       | 1    |
| XOR        | 10    | 100         | 0000000 | 101         | 00       | 0    |
| SRL        | 10    | 101         | 0000000 | 100         | 00       | 0    |
| SRA        | 10    | 101         | 0100000 | 100         | 00       | 0    |
| OR         | 10    | 110         | 0000000 | 110         | 00       | 0    |
| AND        | 10    | 111         | 0000000 | 111         | 00       | 0    |
| SRAI       | 11    | 101         | -       | 100         | 00       | 0    |
| ADDI       | 11    | 000         | -       | 000         | 00       | 0    |
| SRLI       | 11    | 101         | -       | 100         | 00       | 0    |
| SLLI       | 11    | 001         | -       | 010         | 00       | 0    |
| SLTI       | 11    | 010         | -       | 001         | 10       | 1    |
| SLTIU      | 11    | 011         | -       | 011         | 10       | 1    |
| XORI       | 11    | 100         | -       | 101         | 00       | 0    |
| ORI        | 11    | 110         | -       | 110         | 00       | 0    |
| ANDI       | 11    | 111         | -       | 111         | 00       | 0    |
| LW         | 00    | 010         | -       | 000         | 00       | 0    |
| SW         | 00    | 010         | -       | 000         | 00       | 0    |
| BEQ        | 01    | 000         | -       | 001         | 00       | 0    |
| BNE        | 01    | 001         | -       | 001         | 01       | 0    |
| BLT        | 01    | 100         | -       | 001         | 10       | 0    |
| BGE        | 01    | 101         | -       | 001         | 11       | 0    |
| BLTU       | 01    | 110         | -       | 011         | 10       | 0    |
| BGEU       | 01    | 111         | -       | 011         | 11       | 0    |

### RTL
![[Pasted image 20240712200423.png]]

### Simulazioni
![[Pasted image 20240422135200.png]]

### Utilizzo
![[Pasted image 20240712202242.png|400]]

## Controllo nel pipeline
Incorporeremo il nostro controllo nella fase ID, poiché è la prima fase in cui l'istruzione è già acquisita. Se posizionassimo il nostro controllo nella fase IF, questa potrebbe generare un collo di bottiglia nelle altre fasi.
- Bisogna anche tenere in considerazione come trasferiamo questi segnali lungo il pipeline per evitare problemi di congruenza.
   - Quindi, bisogna farlo come mostrato di seguito:
![[Pasted image 20230417093625.png]]

- Pertanto, il nostro percorso dati completo con i segnali di controllo sarà come mostrato di seguito:
![[Pasted image 20240712121150.png]]

---


# Spagnolo
![[Pasted image 20240410174711.png]]
## Control general

El módulo **CONTROL** decodifica el opcode de la instrucción para generar las señales de control necesarias para las diferentes etapas del pipeline del CPU.

- **Entradas**:
  - `opcode`: Código de operación de la instrucción.

- **Salidas**:
  - `RegWrite`: Señal para escribir en el banco de registros.
  - `ALUSrc`: Señal para seleccionar el segundo operando de la ALU.
  - `MemWrite`: Señal para escribir en la memoria de datos.
  - `MemRead`: Señal para leer de la memoria de datos.
  - `Branch`: Señal para realizar un salto condicional.
  - `MemToReg`: Señal para seleccionar el dato que se escribe en el registro destino.
  - `ALUop`: Código de operación para la ALU.

- **Funcionamiento**:
  - Decodifica `opcode` para asignar las señales de control correspondientes.
  - Genera señales específicas para instrucciones tipo R, I, S, B, y loads.

### Tabla de funcionamiento

| Instrucción      | RegWrite | ALUSrc | MemWrite | MemRead | Branch | MemToReg | ALUop |
| ---------------- | -------- | ------ | -------- | ------- | ------ | -------- | ----- |
| Tipo R           | 1        | 0      | 0        | 0       | 0      | 0        | 10    |
| Tipo I sin loads | 1        | 1      | 0        | 0       | 0      | 0        | 11    |
| Tipo I Loads     | 1        | 1      | 0        | 1       | 0      | 1        | 00    |
| Tipo S           | 0        | 1      | 1        | 0       | 0      | 0        | 00    |
| Tipo B           | 0        | 0      | 0        | 0       | 1      | 0        | 01    |

### RTL
![[Pasted image 20240712131438.png]]
### Simulaciones
![[Pasted image 20240422134017.png]]
### Utilizacion
![[Pasted image 20240712132910.png]]

## Control ALU
El módulo **ALU_CONTROL** determina las señales de control para la ALU basándose en los campos `funct3`, `funct7` de la instrucción y el código de operación ALUop.

- **Entradas**:
  - `ALUop`: Código de operación de la ALU.
  - `funct3`: Campo funct3 de la instrucción.
  - `funct7`: Campo funct7 de la instrucción.
  
- **Salidas**:
  - `ALUControl`: Señal de control para la ALU.
  - `BranchOp`: Señal de operación de branch.
  - `SLTc`: Señal de control para SLT.

- **Funcionamiento**:
  - `ALUControl` se asigna en base a las combinaciones de `funct3`, `funct7` y `ALUop`.
  - `BranchOp` determina la operación de branch en base a `ALUop` y `funct3`.
  - `SLTc` se activa para las operaciones SLT y SLTU.

### Tabla de funcionamiento

| Instrucción | ALUop | funct3      | funct7  | ALU_Control | BranchOp | SLTc |
| ----------- | ----- | ----------- | ------- | ----------- | -------- | ---- |
| ADD         | 10    | 000         | 0000000 | 000         | 00       | 0    |
| SUB         | 10    | 000         | 0100000 | 001         | 00       | 0    |
| SLL         | 10    | 001         | 0000000 | 010         | 00       | 0    |
| SLT         | 10    | 010         | 0000000 | 001         | 10       | 1    |
| SLTU        | 10    | 011         | 0000000 | 011         | 10       | 1    |
| XOR         | 10    | 100         | 0000000 | 101         | 00       | 0    |
| SRL         | 10    | 101         | 0000000 | 100         | 00       | 0    |
| SRA         | 10    | 101         | 0100000 | 100         | 00       | 0    |
| OR          | 10    | 110         | 0000000 | 110         | 00       | 0    |
| AND         | 10    | 111         | 0000000 | 111         | 00       | 0    |
| SRAI        | 11    | 101         | -       | 100         | 00       | 0    |
| ADDI        | 11    | 000         | -       | 000         | 00       | 0    |
| SRLI        | 11    | 101         | -       | 100         | 00       | 0    |
| SLLI        | 11    | 001         | -       | 010         | 00       | 0    |
| SLTI        | 11    | 010         | -       | 001         | 10       | 1    |
| SLTIU       | 11    | 011         | -       | 011         | 10       | 1    |
| XORI        | 11    | 100         | -       | 101         | 00       | 0    |
| ORI         | 11    | 110         | -       | 110         | 00       | 0    |
| ANDI        | 11    | 111         | -       | 111         | 00       | 0    |
| LW          | 00    | 010         | -       | 000         | 00       | 0    |
| SW          | 00    | 010         | -       | 000         | 00       | 0    |
| BEQ         | 01    | 000         | -       | 001         | 00       | 0    |
| BNE         | 01    | 001         | -       | 001         | 01       | 0    |
| BLT         | 01    | 100         | -       | 001         | 10       | 0    |
| BGE         | 01    | 101<br><br> | -       | 001         | 11       | 0    |
| BLTU        | 01    | 110         | -       | 011         | 10       | 0    |
| BGEU        | 01    | 111         | -       | 011         | 11       | 0    |
### RTL
![[Pasted image 20240712200423.png]]
### Simulaciones
![[Pasted image 20240422135200.png]]

### Utilizacion
![[Pasted image 20240712202242.png|400]]
## Control en pipeline
Incorporaremos nuestros control dentro de la etapa ID, ya que es la primera etapa donde ya tenemos adquirida la instrucción. Si pondriamos nuestro control en la etapa IF, esta misma podria generar un cuello de botella en la otras etapas.
- Tambien hay que tener en cuenta como transferimos estas señales a lo largo del pipeline para evitar problemas de congruencia
	- Por lo tanto hay que hacerlo como se ve a continuacion
![[Pasted image 20230417093625.png]]

- Por lo tanto nuestro camino de dato completo con las señales de control quedaria como se ve a continuación
![[Pasted image 20240712121150.png]]



