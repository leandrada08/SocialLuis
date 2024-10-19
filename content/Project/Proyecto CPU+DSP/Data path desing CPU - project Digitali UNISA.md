
# Italiano

## Analisi del nostro ISA
### Sottogruppo dell'ISA di RISC-V
Implementeremo la base RV32I di RISC-V che è la base di questo ISA, implementeremo anche le istruzioni di moltiplicazione e una di moltiplicazione e somma per poter realizzare il filtro FIR. Di seguito nominerò tutte le istruzioni che verranno implementate e poi le descriverò:
- **SHIFT**
	- SLL
	- SLLI
	- SRL
	- SRLI
	- SRA
	- SRAI
- **Aritmetica**
	- ADD
	- ADDI
	- SUB
	- LUI
	- AUIPC
- **Logica**
	- XOR
	- XORI
	- OR
	- ORI
	- AND
	- ANDI
- **Confronto**
	- SLT
	- SLTI
	- SLTU
	- SLTIU
- **Branches**
	- BEW
	- BNE
	- BLT
	- BGE
	- BLTU
	- BGEU
- **LOADS**
	- LW
- **Store**
	- SW

### Descrizione delle nostre istruzioni

#### SHIFT
- **SLL**: Spostamento logico a sinistra. Sposta i bit di un registro (rs1) a sinistra di un numero di posizioni specificato da un altro registro (rs2), e salva il risultato in un terzo registro (rd). I bit vuoti vengono riempiti con zeri. Ha un formato R. Ad esempio, `SLL x1, x2, x3` sposta i bit di x2 a sinistra di x3 posizioni e li salva in x1.
- **SLLI**: Spostamento logico a sinistra immediato. Sposta i bit di un registro (rs1) a sinistra di un numero di posizioni specificato da un valore immediato (imm), e salva il risultato in un altro registro (rd). I bit vuoti vengono riempiti con zeri. Ha un formato I. Ad esempio, `SLLI x1, x2, 4` sposta i bit di x2 a sinistra di 4 posizioni e li salva in x1.
- **SRL**: Spostamento logico a destra. Sposta i bit di un registro (rs1) a destra di un numero di posizioni specificato da un altro registro (rs2), e salva il risultato in un terzo registro (rd). I bit vuoti vengono riempiti con zeri. Ha un formato R. Ad esempio, `SRL x1, x2, x3` sposta i bit di x2 a destra di x3 posizioni e li salva in x1.
- **SRLI**: Spostamento logico a destra immediato. Sposta i bit di un registro (rs1) a destra di un numero di posizioni specificato da un valore immediato (imm), e salva il risultato in un altro registro (rd). I bit vuoti vengono riempiti con zeri. Ha un formato I. Ad esempio, `SRLI x1, x2, 4` sposta i bit di x2 a destra di 4 posizioni e li salva in x1.
- **SRA**: Spostamento aritmetico a destra. Sposta i bit di un registro (rs1) a destra di un numero di posizioni specificato da un altro registro (rs2), e salva il risultato in un terzo registro (rd). I bit vuoti vengono riempiti con il bit di segno del valore originale. Ha un formato R. Ad esempio, `SRA x1, x2, x3` sposta i bit di x2 a destra di x3 posizioni e li salva in x1, mantenendo il segno di x2.
- **SRAI**: Spostamento aritmetico a destra immediato. Sposta i bit di un registro (rs1) a destra di un numero di posizioni specificato da un valore immediato (imm), e salva il risultato in un altro registro (rd). I bit vuoti vengono riempiti con il bit di segno del valore originale. Ha un formato I. Ad esempio, `SRAI x1, x2, 4` sposta i bit di x2 a destra di 4 posizioni e li salva in x1, mantenendo il segno di x2.

#### Aritmetica
- **ADD**: Somma due registri e salva il risultato in un altro registro. Ha un formato R. Ad esempio, `ADD x1, x2, x3` somma i valori di x2 e x3 e li salva in x1.
- **ADDI**: Somma un registro e un valore immediato e salva il risultato in un altro registro. Ha un formato I. Ad esempio, `ADDI x1, x2, 4` somma il valore di x2 e 4 e lo salva in x1.
- **SUB**: Sottrae due registri e salva il risultato in un altro registro. Ha un formato R. Ad esempio, `SUB x1, x2, x3` sottrae il valore di x3 da x2 e lo salva in x1.

#### Logica
- **XOR**: Operazione logica esclusiva o. Restituisce 1 solo se i bit corrispondenti dei due operandi sono diversi, e 0 se sono uguali. Ha un formato R. Ad esempio, `XOR x1, x2, x3` esegue l'operazione x2 XOR x3 e salva il risultato in x1.
- **XORI**: Operazione logica esclusiva o immediata. Restituisce 1 solo se i bit corrispondenti del registro e del valore immediato sono diversi, e 0 se sono uguali. Ha un formato I. Ad esempio, `XORI x1, x2, 4` esegue l'operazione x2 XOR 4 e salva il risultato in x1.
- **OR**: Operazione logica o. Restituisce 1 se almeno uno dei bit corrispondenti dei due operandi è 1, e 0 se entrambi sono 0. Ha un formato R. Ad esempio, `OR x1, x2, x3` esegue l'operazione x2 OR x3 e salva il risultato in x1.
- **ORI**: Operazione logica o immediata. Restituisce 1 se almeno uno dei bit corrispondenti del registro e del valore immediato è 1, e 0 se entrambi sono 0. Ha un formato I. Ad esempio, `ORI x1, x2, 4` esegue l'operazione x2 OR 4 e salva il risultato in x1.
- **AND**: Operazione logica e. Restituisce 1 se i bit corrispondenti dei due operandi sono 1, e 0 se uno di essi è 0. Ha un formato R. Ad esempio, `AND x1, x2, x3` esegue l'operazione x2 AND x3 e salva il risultato in x1.
- **ANDI**: Operazione logica e immediata. Restituisce 1 se i bit corrispondenti del registro e del valore immediato sono 1, e 0 se uno di essi è 0. Ha un formato I. Ad esempio, `ANDI x1, x2, 4` esegue l'operazione x2 AND 4 e salva il risultato in x1.

#### Confronto
- **SLT**: Imposta meno di. Imposta un registro (rd) a 1 se un registro (rs1) è inferiore a un altro registro (rs2), considerando il segno, e a 0 se non lo è. Ha un formato R. Ad esempio, `SLT x1, x2, x3` imposta x1 a 1 se x2 è inferiore a x3, considerando il segno, e a 0 se non lo è.
- **SLTI**: Imposta meno di immediato. Imposta un registro (rd) a 1 se un registro (rs1) è inferiore a un valore immediato (imm), considerando il segno, e a 0 se non lo è. Ha un formato I. Ad esempio, `SLTI x1, x2, 4` imposta x1 a 1 se x2 è inferiore a 4, considerando il segno, e a 0 se non lo è.
- **SLTU**: Imposta meno di senza segno. Imposta un registro (rd) a 1 se un registro (rs1) è inferiore a un altro registro (rs2), senza considerare il segno, e a 0 se non lo è. Ha un formato R. Ad esempio, `SLTU x1, x2, x3` imposta x1 a 1 se x2 è inferiore a x3, senza considerare il segno, e a 0 se non lo è.
- **SLTIU**: Imposta meno di immediato senza segno. Imposta un registro (rd) a 1 se un registro (rs1) è inferiore a un valore immediato (imm), senza considerare il segno, e a 0 se non lo è. Ha un formato I. Ad esempio, `SLTIU x1, x2, 4` imposta x1 a 1 se x2 è inferiore a 4, senza considerare il segno, e a 0 se non lo è.

#### Branches
- **BEQ**: Salta se uguali. Salta a un'etichetta se due registri (rs1 e rs2) sono uguali, e continua l'esecuzione se non lo sono. Ha un formato SB. Ad esempio, `BEQ x1, x2, loop` salta all'etichetta loop se x1 e x2 sono uguali, e continua l'esecuzione se non lo sono.
- **BNE**: Salta se non uguali. Salta a un'etichetta se due registri (rs1 e rs2) non sono uguali, e continua l'esecuzione se lo sono. Ha un formato SB. Ad esempio, `BNE x1, x2, exit` salta all'etichetta exit se x1 e x2 non sono uguali, e continua l'esecuzione se lo sono.
- **BLT**: Salta se minore di. Salta a un'etichetta se un registro (rs1) è minore di un altro registro (rs2), considerando il segno, e continua l'esecuzione se non lo è. Ha un formato SB. Ad esempio, `BLT x1, x2, then` salta all'etichetta then se x1 è minore di x2, considerando il segno, e continua l'esecuzione se non lo è.
- **BGE**: Salta se maggiore o uguale. Salta a un'etichetta se un registro (rs1) è maggiore o uguale a un altro registro (rs2), considerando il segno, e continua l'esecuzione se non lo è. Ha un formato SB. Ad esempio, `BGE x1, x2, else` salta all'etichetta else se x1 è maggiore o uguale a x2, considerando il segno, e continua l'esecuzione se non lo è.
- **BLTU**: Salta se minore di senza segno. Salta a un'etichetta se un registro (rs1) è minore di un altro registro (rs2), senza considerare il segno, e continua l'esecuzione se non lo è. Ha un formato SB. Ad esempio, `BLTU x1, x2, then` salta all'etichetta then se x1 è minore di x2, senza considerare il segno, e continua l'esecuzione se non lo è.
- **BGEU**: Salta se maggiore o uguale senza segno. Salta a un'etichetta se un registro (rs1) è maggiore o uguale a un altro registro (rs2), senza considerare il segno, e continua l'esecuzione se non lo è. Ha un formato SB. Ad esempio, `BGEU x1, x2, else` salta all'etichetta else se x1 è maggiore o uguale a x2, senza considerare il segno, e continua l'esecuzione se non lo è.

#### Loads
- **LW**: Carica una parola (32 bit) dalla memoria e la memorizza nel registro di destinazione (32 bit). Ha un formato I. Ad esempio, `LW x1, 4(x2)` carica una parola dall'indirizzo di memoria x2 + 4 e la memorizza in x1.

#### Store
- **SW**: Memorizza una parola (32 bit) da un registro (rs2) nella memoria. Ha un formato S. Ad esempio, `SW x1, 4(x2)` memorizza una parola da x1 nell'indirizzo di memoria x2 + 4.



## Progettando le nostre istruzioni
### Comune a tutte le istruzioni

1. PC: comune a tutte le istruzioni sequenziali.
2. Una memoria per le istruzioni: l'uscita della memoria contiene l'istruzione da eseguire.
3. Un sommatore che somma 4.

#### Percorso dei dati per il fetch
Componenti necessari:
- PC
- Instruction Memory
- ADD
![[Pasted image 20231210133119.png]]

### Istruzioni di tipo R
- *Fetch*
1. È necessario leggere due registri dal banco registri.
2. È necessario eseguire operazioni aritmetiche/logiche in una ALU.
3. Scrivere nuovamente nel banco registri.

Per tutto questo avremo bisogno di un banco registri che possa essere scritto e letto, e di una ALU.

### Istruzione Load/Store
- *Fetch*
1. È necessario leggere anche dal banco registri.
2. Calcola l'indirizzo di memoria nell'ALU usando la costante immediata che accompagna l'istruzione.
- Questa memoria deve essere diversa da quella delle istruzioni poiché vogliamo che tutto venga eseguito in un ciclo.

#### Percorso dei dati Tipo R/Load/Store
Nuovi componenti necessari:
1. Estensore di segno per la costante immediata.
2. Una memoria dati
   - Deve avere un ingresso per l'indirizzo.
   - Ingresso per i dati.
   - Uscita per i dati.
   - Segnali di controllo per lettura e scrittura.

![[Pasted image 20231210140229.png|600]]

- Sorge la necessità di aggiungere 2 multiplexer:
   - Il secondo operando dell'ALU può provenire dal banco registri o dalla costante immediata.
   - I dati da scrivere nel banco registri possono provenire dall'ALU o dalla memoria.

### Istruzione Branch
1. Leggono 2 registri dal banco registri.
2. Utilizzano l'ALU per confrontarli.
3. Calcolano l'indirizzo di destinazione.
4. Generano la necessità di un nuovo sommatore.
   - E di un nuovo multiplexer.

#### Percorso dei dati branch
![[Pasted image 20231210172207.png]]

#### Osservazioni
- Per poter analizzare il tipo di confronto, verrà utilizzato un blocco che, con il segnale di controllo, deciderà in base ai segnali di negativo e zero dell'ALU se il salto debba essere eseguito o meno.
- Questo blocco verrà visto successivamente nel pipeline completo.
- Si potrebbe dire che fa parte dei blocchi di controllo.



## CPU
Per realizzare la nostra CPU, dobbiamo semplicemente raggruppare le diverse fasi del design delle nostre istruzioni. Questo design di base non tiene conto della fase di controllo, che verrà aggiunta in seguito. Possiamo notare che è stata data priorità alla velocità senza considerare il consumo di risorse logiche; avremmo potuto riutilizzare blocchi come l'ALU per evitare di definire blocchi ADD. L'inconveniente è che, per fare ciò, avremmo dovuto utilizzare un sistema di controllo e coordinazione basato su una macchina a stati finiti, il che sarebbe anch'esso costoso in termini di risorse, più lento e più complesso da progettare. Pertanto, abbiamo dato priorità a questo design.

![[Pasted image 20231210172803.png|900]]

# Spagnolo
## Análisis de nuestro ISA
### Subconjunto del ISA de RISC-V
Implementaremos la base RV32I de RISC-V que es la base de este ISA, tambien implementaremos las intrucciones de multiplicacion y una de multiplicacion y suma para poder realizar el filtro FIR. A continuacion nombrare todas las instrucciones que se implementaran y luego se las describira:
- **SHIFT**
	- SLL
	- SLLI
	- SRL
	- SRLI
	- SRA
	- SRAI
- **Aritmetica**
	- ADD
	- ADDI
	- SUB
	- LUI
	- AUIPC
- **Logica**
	- XOR
	- XORI
	- OR
	- ORI
	- AND
	- ANDI
- **Comparacion**
	- SLT
	- SLTI
	- SLTU
	- SLTIU
- **Branches**
	- BEW
	- BNE
	- BLT
	- BGE
	- BLTU
	- BGEU
- **LOADS**
	- LW
- **Store**
	- SW

### Descripcion de nuestras instrucciones

#### SHIFT
- **SLL**: Desplazamiento lógico a la izquierda. Mueve los bits de un registro (rs1) hacia la izquierda un número de posiciones especificado por otro registro (rs2), y guarda el resultado en un tercer registro (rd). Los bits vacíos se rellenan con ceros. Tiene un formato R. Por ejemplo, `SLL x1, x2, x3` desplaza los bits de x2 hacia la izquierda x3 posiciones y los guarda en x1.
- **SLLI**: Desplazamiento lógico a la izquierda inmediato. Mueve los bits de un registro (rs1) hacia la izquierda un número de posiciones especificado por un valor inmediato (imm), y guarda el resultado en otro registro (rd). Los bits vacíos se rellenan con ceros. Tiene un formato I. Por ejemplo, `SLLI x1, x2, 4` desplaza los bits de x2 hacia la izquierda 4 posiciones y los guarda en x1.
- **SRL**: Desplazamiento lógico a la derecha. Mueve los bits de un registro (rs1) hacia la derecha un número de posiciones especificado por otro registro (rs2), y guarda el resultado en un tercer registro (rd). Los bits vacíos se rellenan con ceros. Tiene un formato R. Por ejemplo, `SRL x1, x2, x3` desplaza los bits de x2 hacia la derecha x3 posiciones y los guarda en x1.
- **SRLI**: Desplazamiento lógico a la derecha inmediato. Mueve los bits de un registro (rs1) hacia la derecha un número de posiciones especificado por un valor inmediato (imm), y guarda el resultado en otro registro (rd). Los bits vacíos se rellenan con ceros. Tiene un formato I. Por ejemplo, `SRLI x1, x2, 4` desplaza los bits de x2 hacia la derecha 4 posiciones y los guarda en x1.
- **SRA**: Desplazamiento aritmético a la derecha. Mueve los bits de un registro (rs1) hacia la derecha un número de posiciones especificado por otro registro (rs2), y guarda el resultado en un tercer registro (rd). Los bits vacíos se rellenan con el bit de signo del valor original. Tiene un formato R. Por ejemplo, `SRA x1, x2, x3` desplaza los bits de x2 hacia la derecha x3 posiciones y los guarda en x1, conservando el signo de x2.
- **SRAI**: Desplazamiento aritmético a la derecha inmediato. Mueve los bits de un registro (rs1) hacia la derecha un número de posiciones especificado por un valor inmediato (imm), y guarda el resultado en otro registro (rd). Los bits vacíos se rellenan con el bit de signo del valor original. Tiene un formato I. Por ejemplo, `SRAI x1, x2, 4` desplaza los bits de x2 hacia la derecha 4 posiciones y los guarda en x1, conservando el signo de x2.

#### Aritmetica
- **ADD**: Suma dos registros y guarda el resultado en otro registro. Tiene un formato R. Por ejemplo, `ADD x1, x2, x3` suma los valores de x2 y x3 y los guarda en x1.
- **ADDI**: Suma un registro y un valor inmediato y guarda el resultado en otro registro. Tiene un formato I. Por ejemplo, `ADDI x1, x2, 4` suma el valor de x2 y 4 y lo guarda en x1.
- **SUB**: Resta dos registros y guarda el resultado en otro registro. Tiene un formato R. Por ejemplo, `SUB x1, x2, x3` resta el valor de x3 a x2 y lo guarda en x1.
####  Logica
- **XOR**: Operación lógica de exclusivo o. Solo devuelve 1 si los bits correspondientes de los dos operandos son diferentes, y 0 si son iguales. Tiene un formato R. Por ejemplo, `XOR x1, x2, x3` hace la operación x2 XOR x3 y guarda el resultado en x1.
- **XORI**: Operación lógica de exclusivo o inmediato. Solo devuelve 1 si los bits correspondientes del registro y del valor inmediato son diferentes, y 0 si son iguales. Tiene un formato I. Por ejemplo, `XORI x1, x2, 4` hace la operación x2 XOR 4 y guarda el resultado en x1.
- **OR**: Operación lógica de o. Devuelve 1 si al menos uno de los bits correspondientes de los dos operandos es 1, y 0 si ambos son 0. Tiene un formato R. Por ejemplo, `OR x1, x2, x3` hace la operación x2 OR x3 y guarda el resultado en x1.
- **ORI**: Operación lógica de o inmediato. Devuelve 1 si al menos uno de los bits correspondientes del registro y del valor inmediato es 1, y 0 si ambos son 0. Tiene un formato I. Por ejemplo, `ORI x1, x2, 4` hace la operación x2 OR 4 y guarda el resultado en x1.
- **AND**: Operación lógica de y. Devuelve 1 si los bits correspondientes de los dos operandos son 1, y 0 si alguno es 0. Tiene un formato R. Por ejemplo, `AND x1, x2, x3` hace la operación x2 AND x3 y guarda el resultado en x1.
- **ANDI**: Operación lógica de y inmediato. Devuelve 1 si los bits correspondientes del registro y del valor inmediato son 1, y 0 si alguno es 0. Tiene un formato I. Por ejemplo, `ANDI x1, x2, 4` hace la operación x2 AND 4 y guarda el resultado en x1.



#### Comparacion
- **SLT**: Establece menos que. Establece un registro (rd) a 1 si un registro (rs1) es menor que otro registro (rs2), considerando el signo, y a 0 si no. Tiene un formato R. Por ejemplo, `SLT x1, x2, x3` establece x1 a 1 si x2 es menor que x3, considerando el signo, y a 0 si no.
- **SLTI**: Establece menos que inmediato. Establece un registro (rd) a 1 si un registro (rs1) es menor que un valor inmediato (imm), considerando el signo, y a 0 si no. Tiene un formato I. Por ejemplo, `SLTI x1, x2, 4` establece x1 a 1 si x2 es menor que 4, considerando el signo, y a 0 si no.
- **SLTU**: Establece menos que sin signo. Establece un registro (rd) a 1 si un registro (rs1) es menor que otro registro (rs2), sin considerar el signo, y a 0 si no. Tiene un formato R. Por ejemplo, `SLTU x1, x2, x3` establece x1 a 1 si x2 es menor que x3, sin considerar el signo, y a 0 si no.
- **SLTIU**: Establece menos que inmediato sin signo. Establece un registro (rd) a 1 si un registro (rs1) es menor que un valor inmediato (imm), sin considerar el signo, y a 0 si no. Tiene un formato I. Por ejemplo, `SLTIU x1, x2, 4` establece x1 a 1 si x2 es menor que 4, sin considerar el signo, y a 0 si no.

#### Branches

- **BEQ**: Salta si son iguales. Salta a una etiqueta si dos registros (rs1 y rs2) son iguales, y continúa la ejecución si no. Tiene un formato SB. Por ejemplo, `BEQ x1, x2, loop` salta a la etiqueta loop si x1 y x2 son iguales, y continúa la ejecución si no.
- **BNE**: Salta si no son iguales. Salta a una etiqueta si dos registros (rs1 y rs2) no son iguales, y continúa la ejecución si sí. Tiene un formato SB. Por ejemplo, `BNE x1, x2, exit` salta a la etiqueta exit si x1 y x2 no son iguales, y continúa la ejecución si sí.
- **BLT**: Salta si es menor que. Salta a una etiqueta si un registro (rs1) es menor que otro registro (rs2), considerando el signo, y continúa la ejecución si no. Tiene un formato SB. Por ejemplo, `BLT x1, x2, then` salta a la etiqueta then si x1 es menor que x2, considerando el signo, y continúa la ejecución si no.
- **BGE**: Salta si es mayor o igual que. Salta a una etiqueta si un registro (rs1) es mayor o igual que otro registro (rs2), considerando el signo, y continúa la ejecución si no. Tiene un formato SB. Por ejemplo, `BGE x1, x2, else` salta a la etiqueta else si x1 es mayor o igual que x2, considerando el signo, y continúa la ejecución si no.
- **BLTU**: Salta si es menor que sin signo. Salta a una etiqueta si un registro (rs1) es menor que otro registro (rs2), sin considerar el signo, y continúa la ejecución si no. Tiene un formato SB. Por ejemplo, `BLTU x1, x2, then` salta a la etiqueta then si x1 es menor que x2, sin considerar el signo, y continúa la ejecución si no.
- **BGEU**: Salta si es mayor o igual que sin signo. Salta a una etiqueta si un registro (rs1) es mayor o igual que otro registro (rs2), sin considerar el signo, y continúa la ejecución si no. Tiene un formato SB. Por ejemplo, `BGEU x1, x2, else` salta a la etiqueta else si x1 es mayor o igual que x2, sin considerar el signo, y continúa la ejecución si no.

#### Loads
- **LW**: Carga una palabra (32 bits) desde la memoria y la almacena en el registro de destino (32 bits). Tiene un formato I. Por ejemplo, `LW x1, 4(x2)` carga una palabra desde la dirección de memoria x2 + 4 y la guarda en x1.

#### Store
- **SW**: Almacena una palabra (32 bits) desde un registro (rs2) en la memoria. Tiene un formato S. Por ejemplo, `SW x1, 4(x2)` almacena una palabra desde x1 en la dirección de memoria x2 + 4.




## Diseñando nuestras intrucciones
### Comun en todas las instrucciones

1. PC: comun a todas las intrucciones secuenciales
2. Una memoria para instrucciones: la salida de la memoria contiene la instruccion a ejecutar.
3. Un sumador que sume 4
#### Camino de datos para el fetch
Componentes necesario:
- PC
- Instruccion Memory
- ADD
![[Pasted image 20231210133119.png]]
### Instrucciones tipo R
- *Fech*
1. Necesito leer dos registros del banco del registro
2. Necesito realizar operaciones aritmeticos/logica en una ALU
3. Escribir nuevamente en el banco de registros
Para todo esto necesitremos a banco de registro que se pueda escribir y leer y una ALU
### Instruccion Load/Store
- *Fech*
1. Tambien se necesitan leer banco de registros.
2. Calcula la direccion de memoria en la ALU usando la constante inmediata que viene con la instruccion.
- Esta memoria tiene que ser distinta a las de instrucciones ya que queremos que todo se ejecute en un ciclo.
#### Camino de datos Tipo R/Load/Store
Componentes nuevos que necesitaremos
1. Extensor de signo para la constante inmediata
2. Una memoria de datos
	- Esta debe tener una entrada de direccion
	- Entrada de datos
	- Salida de datos
	- Seniales de contro de lectura y escritura

![[Pasted image 20231210140229.png|600]]
- Surge la necesitadad de agregar 2 multiplexores
	- El segundo operando de la alu puede venir del banco de registros o de la constante inmediata
	- El dato a escribir en el banco de registros puede venir de la ALU o de la memoria
### Instruccion Branch
1. Leen 2 registros de los bancos de registros
2. Usan la ALU para compararlos
3. Calcula la direccion destino
4. Genera la necesidad de un nuevo sumador
	- Y de un nuevo multiplexor


#### Camino de datos branch
![[Pasted image 20231210172207.png]]


#### Observaciones
- Para poder analizar el tipo de comparacion se usara un bloque que con la señal de control decidira en base a las señales de negativo y zero de la alu si se debe o no realizar el salto
- Este bloque se lo vera luego en el pipeline completo
- Se podria decir que forma parte de los bloques de control
## CPU
Para realizar nuestro CPU, solo debemos agrupar las distintas etapas del diseño de nuestras instrucciones, este diseño basico no tiene en cuenta la etapa de control, la cual sera agregada las adelante. Podemos ver que se priorizo la velocidad sin tener en cuenta el gasto de recursos logico, podriamos haber reutilizado bloque como la ALU para evitar tener que definir bloques ADD. El incoveniente es que para realizar esto deberiamos utilizar un sistema de control y coordinacion que sea una maquina de estado finito, lo cual es tambien costoso en recursos y es mas lento, al igual que es mas dificil en diseño, por lo cual priorizamos este diseño.
![[Pasted image 20231210172803.png|900]]



##  [[Description and analysis of each block CPU - project Digitali UNISA]]