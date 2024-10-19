
## REGISTERS

El módulo **REGISTERS** se encarga de manejar los registros del CPU. Tiene las siguientes características:

- **Entradas**:
  - `clk`: Señal de reloj.
  - `RA`, `RB`, `RW`: Direcciones de los registros de lectura A, lectura B y escritura, respectivamente.
  - `RegWrite`: Señal de control para permitir la escritura en un registro.
  - `busW`: Datos a escribir en el registro.
  
- **Salidas**:
  - `busA`, `busB`: Datos leídos de los registros A y B.
  
- **Funcionamiento**:
  - Los registros se actualizan en el flanco positivo del reloj (`clk`).
  - Si `RegWrite` está activo, el dato de `busW` se escribe en el registro especificado por `RW`.
  - `busA` y `busB` siempre reflejan el contenido de los registros especificados por `RA` y `RB`.

### RTL
![[Pasted image 20240712125709.png]]

### Utilizacion
![[Pasted image 20240712131231.png]]

### Conclusion
- Para una implementacion en caso de necesitar recursos, se debe reducir el tamañó
	- Esto se debe a que la FPGA no cuenta con componente de memoria de doble lectura
## PC

El módulo **PC** (Program Counter) es responsable de mantener la dirección actual de la instrucción que se está ejecutando.

- **Entradas**:
  - `clock`: Señal de reloj.
  - `next_address`: La dirección de la siguiente instrucción a ejecutar.

- **Salida**:
  - `address`: La dirección actual de la instrucción.

- **Funcionamiento**:
  - En el flanco positivo del reloj, se actualiza `address` con `next_address`.
  - Si `next_address` contiene bits indeterminados, `address` se reinicia a `32'h1000`.

### RTL
![[Pasted image 20240714195632.png]]

### Utilizacion
![[Pasted image 20240712220248.png]]
## ADD4

El módulo **ADD4** incrementa la dirección de la instrucción en 4, que es el tamaño de una instrucción en RISC-V.

- **Entrada**:
  - `current_address`: La dirección actual de la instrucción.

- **Salida**:
  - `next_address`: La dirección incrementada en 4.

- **Funcionamiento**:
  - `next_address` se actualiza continuamente con `current_address + 4`.
### RTL
![[Pasted image 20240712204643.png]]

### Utilizacion
![[Pasted image 20240712205008.png]]


### Observaciones
- Teoricamente es PC+4 pero en la realidad esto no sucede
- Debemos hacer PC+1, ya que definimos la memoria con espacios de 64 bits y no de 8
## ADD

El módulo **ADD** realiza la suma de dos valores de entrada.

- **Entradas**:
  - `A`, `B`: Valores a sumar.

- **Salida**:
  - `C`: Resultado de la suma.

- **Funcionamiento**:
  - `C` es el resultado de `A + B`.

### Simulaciones
![[Pasted image 20240422105711.png]]

## MUX32

El módulo **MUX32** es un multiplexor que selecciona entre dos entradas de 32 bits basado en una señal de selección.

- **Entradas**:
  - `A`, `B`: Entradas de datos.
  - `select`: Señal de control para seleccionar entre `A` y `B`.

- **Salida**:
  - `C`: Salida de datos seleccionada.

- **Funcionamiento**:
  - Si `select` es `1`, `C` toma el valor de `B`.
  - Si `select` es `0`, `C` toma el valor de `A`.


### RTL 
![[Pasted image 20240712202547.png]]


### Simulaciones
![[Pasted image 20240422110229.png]]

### Utilizacion
![[Pasted image 20240712204439.png]]

## GENERADOR_CONSTANTE

El módulo **GENERADOR_CONSTANTE** genera la constante inmediata (inmediato) a partir de la instrucción, necesaria para operaciones aritméticas y de acceso a memoria.

- **Entradas**:
  - `instruccion`: La instrucción de la cual se extrae la constante inmediata.

- **Salida**:
  - `constante`: La constante inmediata generada.

- **Funcionamiento**:
  - Extrae y asigna los bits correspondientes para los diferentes tipos de instrucciones (I, S, B).
  - Sign-extiende las constantes inmediatas según sea necesario.

### Tabla de funcionamiento

| Tipo de Instrucción | Opcode    | Constante Generada          |
| ------------------- | --------- | --------------------------- |
| Tipo I (sin loads)  | 0010011   | Sign-extend de I_imm        |
| Tipo I (loads)      | 0000011   | Sign-extend de I_imm        |
| Tipo S              | 0100011   | Sign-extend de S_imm        |
| Tipo B              | 1100011   | Sign-extend de B_imm (formato especial) |
| Otro                | Otros     | 0                           |

- **I_imm**: Bits \[31:20] de la instrucción.
- **S_imm**: Bits \[31:25] y \[11:7] de la instrucción.
- **B_imm**: Bits \[31], \[7], \[30:25], \[11:8] de la instrucción.


### RTL
![[Pasted image 20240712125547.png]]
### Simulaciones

![[Pasted image 20240429111654.png]]

### Utilizacion
![[Pasted image 20240712125354.png]]
## MEM_INST

El módulo **MEM_INST** es responsable de almacenar y recuperar las instrucciones de la memoria.

- **Entradas**:
  - `i_pc`: Dirección del contador de programa (PC).
  
- **Salida**:
  - `o_instruccion`: Instrucción leída de la memoria.

- **Funcionamiento**:
  - La memoria de instrucciones (`inst`) se inicializa con el contenido del archivo especificado por `MEM_INIT_FILE`.
  - La instrucción correspondiente a la dirección `i_pc` se recupera de la memoria (`inst[i_pc/4]`).

### RTL
![[Pasted image 20240712221503.png]]

### Utilizacion
![[Pasted image 20240712221512.png]]
## MEM_DATA

El módulo **MEM_DATA** gestiona la memoria de datos, permitiendo leer y escribir datos en la memoria.

- **Entradas**:
  - `i_memwrite`: Señal de control para habilitar la escritura en la memoria.
  - `i_memread`: Señal de control para habilitar la lectura de la memoria.
  - `i_address`: Dirección de memoria.
  - `i_write_data`: Datos a escribir en la memoria.

- **Salida**:
  - `o_read_data`: Datos leídos de la memoria.

- **Funcionamiento**:
  - La memoria de datos (`data`) se inicializa a 0.
  - Si `i_memread` está activo, se leen los datos de la dirección especificada (`data[i_address>>2]`).
  - Si `i_memwrite` está activo, se escriben los datos en la dirección especificada (`data[i_address>>2]`).

### Utilizacion
![[Pasted image 20240713102709.png]]



### Conclusion
- Se deberia reducir el tamaño de la memoria ya que utiliza muchos recursos por la naturaleza de la misma
	- Esto se debe a que la FPGA no cuenta con la componente de memoria de 

## BRANCH

El módulo **BRANCH** determina si se debe realizar una operación de salto (branch) en base a las señales de control y los resultados de la ALU.

- **Entradas**:
  - `i_zero`: Señal de la ALU que indica si el resultado es cero.
  - `i_neg`: Señal de la ALU que indica si el resultado es negativo.
  - `i_branch`: Señal de control para habilitar la operación de branch.
  - `i_BranchOp`: Código de operación de branch.

- **Salidas**:
  - `o_PCSrc`: Señal de control para la fuente del PC.
  - `o_slt_data`: Resultado del set less than (slt).

- **Funcionamiento**:
  - Calcula si el resultado de la ALU es igual, menor o mayor que cero.
  - Determina la señal `PCSrc` en base a `i_BranchOp`.
  - Si `i_branch` está activo, `PCSrc` se asigna a `PCSrc_aux`.

### Tabla de operaciones

| BranchOp | Comparacion logica |
| -------- | ------------------ |
| 00       | Igual que          |
| 01       | Distinto que       |
| 10       | Menor que          |
| 11       | Mayor o igual que  |

### RTL
![[Pasted image 20240712223042.png]]
### Utilizacion
![[Pasted image 20240712223250.png]]
### Simulaciones
![[Pasted image 20240422120131.png]]

## ALU

El módulo **ALU** (Arithmetic Logic Unit) realiza operaciones aritméticas y lógicas en dos operandos.

- **Entradas**:
  - `A`, `B`: Operandos de entrada.
  - `ALU_Sel`: Código de operación de la ALU.

- **Salidas**:
  - `ALU_Out`: Resultado de la operación de la ALU.
  - `Zero`: Señal que indica si el resultado es cero.
  - `Negativo`: Señal que indica si el resultado es negativo.

- **Funcionamiento**:
  - La ALU realiza operaciones como suma, resta, desplazamiento lógico, AND, OR y XOR en base a `ALU_Sel`.
  - `Zero` es `1` si el resultado es cero.
  - `Negativo` es `1` si el bit más significativo del resultado es `1`.

> En la ALU usaremos la aritmetica entera estandar ya que la ALU suele manejar enteros sin signos y esto mejora el utilizo de recursos y la velocidad en la ALU.
### Tabla de operaciones

| ALU_Control | Operacion         |
| ----------- | ----------------- |
| 000         | ADD               |
| 001         | SUB               |
| 010         | Shift Left Logico |
| 011         | SUB unsigned      |
| 100         | Shift Righ        |
| 101         | XOR               |
| 110         | OR                |
| 111         | AND               |
|             |                   |
### RTL
![[Pasted image 20240712221728.png]]
### Utilizacion
![[Pasted image 20240712222801.png]]
### Simulaciones

![[Pasted image 20240422114327.png]]


