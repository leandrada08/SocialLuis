## IF

El bloque IF (Instruction Fetch) se encarga de obtener la instrucción desde la memoria de instrucciones en base a la dirección actual del programa. Este bloque maneja la lógica para seleccionar la siguiente dirección de instrucción, incrementarla, y acceder a la memoria para obtener la instrucción correspondiente.
1. **Entradas y Salidas**:
   - **Entradas**:
     - `i_branch_address`: Dirección de instrucción en caso de un salto (branch).
     - `i_select`: Señal de control para seleccionar entre la dirección de salto y la dirección incrementada.
     - `i_clock`: Señal de reloj para sincronizar las operaciones.
   - **Salidas**:
     - `o_address`: Dirección actual de la instrucción.
     - `o_instruccion`: Instrucción obtenida de la memoria.

2. **Conexión entre Componentes**:
   - El `MUX32` selecciona la próxima dirección de instrucción (`next_address`), que puede ser una dirección de salto o la dirección incrementada (`address_4`).
   - El `ADD4` incrementa la dirección actual (`address`) en 4.
   - El `PC` actualiza la dirección del programa a la siguiente dirección (`next_address`) en cada ciclo de reloj.
   - La `MEM_INST` obtiene la instrucción desde la memoria en la dirección actual (`address`).

3. **Funcionamiento**:
   - En cada ciclo de reloj, el bloque IF selecciona la siguiente dirección de instrucción (`next_address`) basada en si hay un salto o no (`i_select`).
   - La dirección actual (`address`) es incrementada en 4 por el `ADD4` y actualizada por el `PC`.
   - La instrucción en la dirección actual es obtenida desde la `MEM_INST` y enviada como salida (`o_instruccion`).





### Simulacion
#### Pre sintesis
![[Pasted image 20240709185305.png]]
![[Pasted image 20240709185335.png]]
- Este bloque funciona bien solo cuando se lo mantiene dentro de la memoria creada, cuando sale de esa memoria nos da instrucciones indefinidas
#### Post sintesis 
### RTL
![[Pasted image 20240713213122.png]]
### Analisis 
#### Reporte utilizacion post sintesis
![[Pasted image 20240715215520.png]]

#### Reporte utilizacion post implementacion
![[Pasted image 20240715215554.png]]


#### Reporte de tiempo post implementacion
![[Pasted image 20240715215613.png]]


![[Pasted image 20240715215708.png]]
## EX

El módulo **EX** (Execute) es responsable de realizar operaciones aritméticas y lógicas, así como de calcular la dirección de salto.

- **Entradas**:
  - `i_PC`: Dirección del contador de programa.
  - `i_register1`, `i_register2`: Datos leídos de los registros.
  - `i_constante`: Valor inmediato generado.
  - `i_ALUSrc`: Señal de control para seleccionar el segundo operando de la ALU.
  - `i_ALUControl`: Señal de control de la ALU.

- **Salidas**:
  - `o_ALUResult`: Resultado de la operación de la ALU.
  - `o_PCBranch`: Dirección de salto calculada.
  - `o_zero`: Señal que indica si el resultado de la ALU es cero.
  - `o_negative`: Señal que indica si el resultado de la ALU es negativo.

- **Funcionamiento**:
  - La ALU realiza operaciones aritméticas y lógicas en base a las señales de control.
  - El módulo ADD calcula la dirección de salto sumando `i_PC` y `i_constante`.
  - El multiplexor selecciona entre `i_register2` e `i_constante` como segundo operando de la ALU.
### Simulacion
![[Pasted image 20240710155533.png]]
![[Pasted image 20240710155553.png]]
### RTL
![[Pasted image 20240710150137.png]]
### Utilización
![[Pasted image 20240710193859.png]]
Podemos ver que basicamente se utilizan LUTs y MUX, casi todos en la implementacion de la ALU. Esta podria utilizar 
## ID

El módulo **ID** (Instruction Decode) decodifica la instrucción y genera señales de control necesarias para las siguientes etapas del pipeline.

- **Entradas**:
  - `i_clk`: Señal de reloj.
  - `i_instruccion`: Instrucción a decodificar.
  - `i_WriteData`: Datos a escribir en el registro.
  - `i_RegWrite`: Señal de control para escribir en el registro.

- **Salidas**:
  - `o_register1`, `o_register2`: Datos leídos de los registros.
  - `o_constante`: Valor inmediato generado.
  - `o_WriteReg`: Dirección del registro a escribir.
  - Señales de control: `o_RegWrite`, `o_ALUSrc`, `o_MemWrite`, `o_MemRead`, `o_Branch`, `o_MemToReg`, `o_SLTc`, `o_ALUControl`, `o_BranchOp`.

- **Funcionamiento**:
  - Extrae los campos de la instrucción para obtener las direcciones de los registros.
  - Genera la constante inmediata usando el módulo GENERADOR_CONSTANTE.
  - Lee los datos de los registros usando el módulo REGISTERS.
  - Decodifica el opcode para generar señales de control usando el módulo CONTROL.
  - Determina las señales de control para la ALU usando el módulo ALU_CONTROL.
### Simulacion
![[Pasted image 20240710121251.png]]
![[Pasted image 20240710121255.png]]

### RTL
![[Pasted image 20240710104530.png]]

### Utilizacion
![[Pasted image 20240710194224.png]]

## MEM

El módulo **MEM** (Memory Access) gestiona las operaciones de lectura y escritura en la memoria de datos, así como la evaluación de las condiciones de salto.

- **Entradas**:
  - `i_Address`: Dirección de la memoria.
  - `i_WriteData`: Datos a escribir en la memoria.
  - `i_BranchOp`: Código de operación de salto.
  - `i_negative`, `i_zero`: Señales de la ALU.
  - `i_branch`, `i_MemWrite`, `i_MemRead`, `i_SLTc`: Señales de control.

- **Salidas**:
  - `o_PCSrc`: Señal para seleccionar la fuente del PC.
  - `o_ReadData`: Datos leídos de la memoria.
  - `o_Mux`: Salida del multiplexor.

- **Funcionamiento**:
  - Evalúa las condiciones de salto usando el módulo BRANCH.
  - Realiza operaciones de lectura y escritura en la memoria de datos usando el módulo MEM_DATA.
  - Selecciona entre `a_slt_data` y `i_Address` para la salida del multiplexor usando el módulo MUX32.

### Simulacion
#### Pres sintesis
![[Pasted image 20240710204532.png]]
![[Pasted image 20240710204535.png]]
#### Post sintesis
![[Pasted image 20240716115654.png]]
![[Pasted image 20240716115649.png]]
### RTL
![[Pasted image 20240716105757.png]]

### Utilizacion
![[Pasted image 20240716112115.png]]

