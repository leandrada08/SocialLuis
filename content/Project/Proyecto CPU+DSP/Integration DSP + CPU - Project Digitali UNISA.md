
# Italiano

## Integrazione del DSP nella CPU

### Introduzione
In questo progetto, l'ISA RISC-V è stato esteso con l'**estensione vettoriale (RVV)** per integrare un **Processore di Segnali Digitali (DSP)** nel sistema. Le istruzioni vettoriali selezionate consentono di eseguire operazioni di somma, sottrazione, moltiplicazione e filtraggio FIR, eseguite dal DSP sotto il controllo della CPU.

Le istruzioni vettoriali aggiunte sono:

- **`vadd` (Vector Add)**: Somma elemento per elemento di due vettori.
- **`vsub` (Vector Subtract)**: Sottrazione elemento per elemento di due vettori.
- **`vmul` (Vector Multiply)**: Moltiplicazione di vettori.
- **`vfmadd` (Vector Fused Multiply-Add)**: Moltiplica due vettori e somma il risultato a un terzo vettore. Utilizzata per implementare un filtro FIR.

Ogni istruzione è definita da una combinazione di `opcode`, `funct3` e `funct7`:

| Istruzione | Opcode    | Funct3 | Funct7    | Descrizione                |
|------------|-----------|--------|-----------|----------------------------|
| `vadd`     | `1010111` | `000`  | `0000000` | Somma di vettori            |
| `vsub`     | `1010111` | `000`  | `0000100` | Sottrazione di vettori      |
| `vmul`     | `1010111` | `001`  | `1000000` | Moltiplicazione di vettori  |
| `vfmadd`   | `1010111` | `011`  | `1000011` | Filtraggio FIR              |

### Blocchi Modificati per Supportare le Istruzioni Vettoriali

Per supportare le nuove istruzioni vettoriali, sono state apportate modifiche a tre blocchi chiave: **Controllo**, **ID** e **CPU**.

#### Controllo
Il blocco di controllo decodifica le istruzioni vettoriali e attiva i segnali per il DSP. È stata implementata una macchina a stati finiti (FSM) per gestire l'interazione con il DSP. A seconda dell'istruzione decodificata, viene inviata la segnalazione di avvio (`dsp_start`) e l'operazione da eseguire (`dsp_operation`).

- **Decodifica delle Istruzioni Vettoriali**:
  - `vadd`: **opcode** = `1010111`, **funct3** = `000`, **funct7** = `0000000`
  - `vsub`: **opcode** = `1010111`, **funct3** = `000`, **funct7** = `0000100`
  - `vmul`: **opcode** = `1010111`, **funct3** = `001`, **funct7** = `1000000`
  - `vfmadd`: **opcode** = `1010111`, **funct3** = `011`, **funct7** = `1000011`

- **Segnali generati dal controllo**:
  - **`dsp_start`**: Attiva l'avvio del DSP per eseguire un'operazione vettoriale.
  - **`dsp_operation`**: Indica al DSP quale operazione vettoriale deve eseguire.

#### Modifiche nella CPU e nella Fase ID

Il pipeline della CPU è stato modificato per includere il supporto delle istruzioni vettoriali e la comunicazione con il DSP. Nella fase di **ID**, la CPU decodifica le istruzioni vettoriali e invia gli operandi al DSP.

Il blocco CPU deve anche gestire i registri vettoriali che memorizzano gli operandi e i risultati delle operazioni del DSP.

## Blocco di Interfaccia `CPU-DSP`

L'integrazione tra la CPU e il DSP avviene tramite un blocco di interfaccia che gestisce i segnali di controllo e i dati tra i due. Questo blocco permette il trasferimento degli operandi e dei risultati attraverso registri condivisi.

- **Scambio di Dati**: La CPU invia gli operandi al DSP e riceve i risultati.
- **Memoria Condivisa**: La CPU e il DSP condividono una memoria che memorizza i dati vettoriali.

### Dettagli del Modulo `CPU_DSP`

#### Porte

| **Nome**     | **Tipo** | **Dimensione** | **Descrizione**                                   |
|--------------|----------|----------------|---------------------------------------------------|
| `clk`        | Ingresso | 1 bit          | Segnale di clock del sistema.                     |
| `reset`      | Ingresso | 1 bit          | Segnale di reset globale del sistema.             |
| `select_out` | Ingresso | 4 bit          | Segnale di selezione per l'uscita dei dati.       |
| `out_data`   | Uscita   | 32 bit         | Uscita dei dati processati dalla CPU o dal DSP.   |

#### Segnali Interni

- **`dsp_start`**: Segnale di avvio per il DSP dalla CPU.
- **`dsp_operation`**: Indica quale operazione eseguire (somma, sottrazione, moltiplicazione o FIR).
- **`dsp_done`**: Indica che il DSP ha completato l'operazione.
- **`dsp_A`, `dsp_B`**: Operandi di input per il DSP.
- **`dsp_result`**: Risultato dell'operazione eseguita dal DSP.

### Codice Verilog dell'Unità CPU-DSP

Il seguente blocco sincronizza i segnali tra la CPU e il DSP. Questo blocco è necessario poiché i due blocchi funzionano a frequenze diverse.

```verilog
always @(posedge clk or posedge reset) begin
    if (reset) begin
        dsp_op_sinc <= 2'b00;
        dsp_start_sinc <= 1'b0;
        addres_a_sinc  <= 2'b00;
        addres_b_sinc  <= 2'b00;
        addres_w_sinc  <= 2'b00;
    end else begin
        if (dsp_start) begin
            dsp_start_sinc <= 1'b1;       
            dsp_op_sinc <= dsp_operation;
            addres_a_sinc <= mem_addr_a;
            addres_b_sinc <= mem_addr_b;
            addres_w_sinc <= mem_addr_write;
        end else begin
            dsp_start_sinc <= 1'b0;
        end
    end
end
```

### Codice Assembly per Istruzioni Vettoriali

```asm
vadd x3, x1, x0    # x3 = x1 + x0
nop * 20           # Simulazione di ritardo

vsub x3, x1, x0    # x3 = x1 - x0
nop * 20           # Simulazione di ritardo

vmul x3, x1, x0    # x3 = x1 * x0
nop * 20           # Simulazione di ritardo

vfmadd x3, x1, x0  # x3 = conv(x1, -x0)
nop * 60           # Simulazione di ritardo
```

### Codice Hexadecimal delle Istruzioni

```
001001D7
00000013
00000013
...
00000013
081001D7
00000013
00000013
...
00000013
861031D7
00000013
00000013
...
00000013
```

### Selezione dell'Uscita

Il multiplexer seleziona l'uscita del sistema basata sul valore di `select_out`:

```verilog
assign out_data =
    (select_out == 4'b0000) ? pc_out :
    (select_out == 4'b0001) ? alu_result_out :
    (select_out == 4'b0010) ? dsp_result[0] :
    (select_out == 4'b0011) ? dsp_result[1] :
    (select_out == 4'b0100) ? dsp_result[2] :
    (select_out == 4'b0101) ? dsp_result[3] :
    (select_out == 4'b0110) ? dsp_result[4] :
    (select_out == 4'b0111) ? dsp_result[5] :
    (select_out == 4'b1000) ? dsp_result[6] :
    (select_out == 4'b1001) ? dsp_result[7] :
    pc_out;
```

### Sintesi e Implementazione
#### Simulazione funzionale
![[Captura de pantalla 2024-09-19 a las 11.10.25 a. m..png]]

#### Risorse (Sintesi)

![[Captura de pantalla 2024-09-19 a las 12.04.54 p. m..png]]
#### Timing (Sintesi)
![[Captura de pantalla 2024-09-19 a las 12.03.37 p. m..png]]
#### Simulazione Post-Sintesi
![[Captura de pantalla 2024-09-19 a las 12.02.57 p. m..png]]
#### Risorse (Implementazione)
![[Captura de pantalla 2024-09-19 a las 12.19.01 p. m..png]]

#### Timing (Implementazione)
![[Captura de pantalla 2024-09-19 a las 12.18.04 p. m..png]]
#### Simulazione Funzionale Post-Implementazione

![[Captura de pantalla 2024-09-19 a las 12.50.11 p. m. 1.png]]
# Spagnolo
## Incorporación del DSP al CPU

### Introducción
En este diseño, se ha extendido el ISA RISC-V con la **extensión vectorial (RVV)** para integrar un **Procesador de Señales Digitales (DSP)** en el sistema. Las instrucciones vectoriales seleccionadas permiten realizar operaciones de suma, resta, multiplicación y filtrado FIR, ejecutadas por el DSP desde el control de la CPU.

Las instrucciones vectoriales añadidas son:

- **`vadd` (Vector Add)**: Suma elemento a elemento de dos vectores.
- **`vsub` (Vector Subtract)**: Resta elemento a elemento de dos vectores.
- **`vmul` (Vector Multiply)**: Multiplicación de vectores.
- **`vfmadd` (Vector Fused Multiply-Add)**: Multiplica dos vectores y suma el resultado a un tercer vector. Usada para implementar un filtro FIR.

Cada instrucción está definida por una combinación de `opcode`, `funct3` y `funct7`:

| Instrucción | Opcode    | Funct3 | Funct7    | Descripción                |
| ----------- | --------- | ------ | --------- | -------------------------- |
| `vadd`      | `1010111` | `000`  | `0000000` | Suma de vectores           |
| `vsub`      | `1010111` | `000`  | `0000100` | Resta de vectores          |
| `vmul`      | `1010111` | `001`  | `1000000` | Multiplicación de vectores |
| `vfmadd`    | `1010111` | `011`  | `1000011` | Filtrado FIR               |

### Bloques Modificados para Soportar Instrucciones Vectoriales

Para soportar las nuevas instrucciones vectoriales, se han realizado modificaciones en tres bloques clave: **Control**, **ID** y **CPU**.

#### Control
El bloque de control decodifica las instrucciones vectoriales y activa las señales para el DSP. Se ha implementado una máquina de estados finitos (FSM) para gestionar la interacción con el DSP. Dependiendo de la instrucción decodificada, se envía la señal de inicio (`dsp_start`) y la operación a realizar (`dsp_operation`).

- **Decodificación de Instrucciones Vectoriales**:
  - `vadd`: **opcode** = `1010111`, **funct3** = `000`, **funct7** = `0000000`
  - `vsub`: **opcode** = `1010111`, **funct3** = `000`, **funct7** = `0000100`
  - `vmul`: **opcode** = `1010111`, **funct3** = `001`, **funct7** = `1000000`
  - `vfmadd`: **opcode** = `1010111`, **funct3** = `011`, **funct7** = `1000011`

- **Señales generadas por el control**:
  - **`dsp_start`**: Activa el inicio del DSP para ejecutar una operación vectorial.
  - **`dsp_operation`**: Indica al DSP la operación vectorial que debe ejecutar.

#### Modificaciones en la CPU y la Etapa ID

El pipeline de la CPU se ha modificado para incluir el soporte de las instrucciones vectoriales y la comunicación con el DSP. En la etapa de **ID**, la CPU decodifica las instrucciones vectoriales y envía los operandos al DSP. 

El bloque CPU también debe gestionar los registros vectoriales que almacenan los operandos y resultados de las operaciones del DSP.

## Bloque de Unión `CPU-DSP`

La integración entre la CPU y el DSP se realiza mediante un bloque de interfaz que gestiona las señales de control y datos entre ambos. Este bloque permite la transferencia de operandos y resultados a través de registros compartidos.

- **Intercambio de Datos**: La CPU envía los operandos al DSP y recibe los resultados.
- **Memoria Compartida**: La CPU y el DSP comparten una memoria, que almacena los datos vectoriales.

### Detalles del Módulo `CPU_DSP`

#### Puertos

| **Nombre**     | **Tipo** | **Tamaño** | **Descripción**                                           |
|----------------|----------|------------|-----------------------------------------------------------|
| `clk`          | Entrada  | 1 bit      | Señal de reloj del sistema.                               |
| `reset`        | Entrada  | 1 bit      | Señal de reinicio global del sistema.                     |
| `select_out`   | Entrada  | 4 bits     | Señal de selección para la salida de datos.               |
| `out_data`     | Salida   | 32 bits    | Salida de datos procesados por la CPU o DSP.              |

#### Señales Internas

- **`dsp_start`**: Señal de inicio para el DSP desde la CPU.
- **`dsp_operation`**: Indica qué operación realizar (suma, resta, multiplicación o FIR).
- **`dsp_done`**: Indica que el DSP ha completado la operación.
- **`dsp_A`, `dsp_B`**: Operandos de entrada al DSP.
- **`dsp_result`**: Resultado de la operación ejecutada por el DSP.

### Código Verilog de la Unidad CPU-DSP

El siguiente bloque sincroniza las señales entre la CPU y el DSP.
- Este bloque es necesario dado que ambos bloques funcionan a distintas 

```verilog
always @(posedge clk or posedge reset) begin
    if (reset) begin
        dsp_op_sinc <= 2'b00;
        dsp_start_sinc <= 1'b0;
        addres_a_sinc  <= 2'b00;
        addres_b_sinc  <= 2'b00;
        addres_w_sinc  <= 2'b00;
    end else begin
        if (dsp_start) begin
            dsp_start_sinc <= 1'b1;       
            dsp_op_sinc <= dsp_operation;
            addres_a_sinc <= mem_addr_a;
            addres_b_sinc <= mem_addr_b;
            addres_w_sinc <= mem_addr_write;
        end else begin
            dsp_start_sinc <= 1'b0;
        end
    end
end
```

### Código Ensamblador para Instrucciones Vectoriales

```asm
vadd x3, x1, x0    # x3 = x1 + x0
nop * 20           # Simulación de demora

vsub x3, x1, x0    # x3 = x1 - x0
nop * 20           # Simulación de demora

vmul x3, x1, x0    # x3 = x1 * x0
nop * 20           # Simulación de demora

vfmadd x3, x1, x0  # x3 = conv(x1,-x0)
nop * 60           # Simulación de demora
```

### Código Hexadecimal de las Instrucciones

```
001001D7
00000013
00000013
...
00000013
081001D7
00000013
00000013
...
00000013
861031D7
00000013
00000013
...
00000013
```

### Selección de Salida

El multiplexor selecciona la salida del sistema basada en el valor de `select_out`:

```verilog
assign out_data =
    (select_out == 4'b0000) ? pc_out :
    (select_out == 4'b0001) ? alu_result_out :
    (select_out == 4'b0010) ? dsp_result[0] :
    (select_out == 4'b0011) ? dsp_result[1] :
    (select_out == 4'b0100) ? dsp_result[2] :
    (select_out == 4'b0101) ? dsp_result[3] :
    (select_out == 4'b0110) ? dsp_result[4] :
    (select_out == 4'b0111) ? dsp_result[5] :
    (select_out == 4'b1000) ? dsp_result[6] :
    (select_out == 4'b1001) ? dsp_result[7] :
    pc_out;
```


### Síntesis e implementación

#### Simulacion funcional
![[Captura de pantalla 2024-09-19 a las 11.10.25 a. m..png]]

#### Recursos -> Síntesis

![[Captura de pantalla 2024-09-19 a las 12.04.54 p. m..png]]
#### Timing -> Síntesis
![[Captura de pantalla 2024-09-19 a las 12.03.37 p. m..png]]
#### Simulazione -> Síntesis
![[Captura de pantalla 2024-09-19 a las 12.02.57 p. m..png]]

#### Recursos -> Implementación
![[Captura de pantalla 2024-09-19 a las 12.19.01 p. m..png]]

#### Timing -> Implementación
![[Captura de pantalla 2024-09-19 a las 12.18.04 p. m..png]]

#### Simulación -> implementación
![[Captura de pantalla 2024-09-19 a las 12.50.11 p. m..png]]