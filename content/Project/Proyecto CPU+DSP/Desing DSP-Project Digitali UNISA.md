
# Italiano


## Descrizione Generale

Il modulo `DSP` implementa un'**Unità di Elaborazione del Segnale (DSP)** progettata per eseguire operazioni di **somma**, **sottrazione**, **moltiplicazione** e **filtraggio FIR** su vettori di dati. Questo design utilizza una codifica a **punto fisso Q(16,16)**, garantendo un equilibrio tra intervallo e precisione nelle operazioni. Il modulo è organizzato tramite una **Macchina a Stati Finiti (FSM)** che gestisce il flusso di controllo e le operazioni in base al segnale di avvio (`start`) e alla selezione dell'operazione (`operation`).

### Porte

| **Nome**    | **Tipo**  | **Dimensione** | **Descrizione**                                                                             |
|-------------|-----------|----------------|---------------------------------------------------------------------------------------------|
| `clk`       | Ingresso  | 1 bit          | Segnale di clock.                                                                           |
| `rst`       | Ingresso  | 1 bit          | Segnale di reset globale.                                                                   |
| `start`     | Ingresso  | 1 bit          | Segnale di avvio dell'operazione.                                                           |
| `operation` | Ingresso  | 2 bit          | Codice dell'operazione (00: Somma, 01: Moltiplicazione, 10: Filtraggio FIR, 11: Sottrazione).|
| `A`         | Ingresso  | 32 bit x 8     | Vettore di coefficienti del filtro FIR o operandi per altre operazioni.                     |
| `B`         | Ingresso  | 32 bit x 8     | Vettore di segnali o operandi per altre operazioni.                                         |
| `result`    | Uscita    | 32 bit x 8     | Vettore di 8 valori intermedi che memorizza il risultato dell'operazione.                   |
| `done`      | Uscita    | 1 bit          | Segnale che indica il completamento dell'operazione.                                        |

### Codifica a Punto Fisso

Il DSP è progettato per lavorare con **punto fisso Q(16,16)**, il che significa che i dati sono rappresentati con 16 bit per la parte intera e 16 bit per la parte frazionaria. Questo approccio offre un buon bilanciamento tra **precisione** e **intervallo**, permettendo operazioni aritmetiche precise per applicazioni di filtraggio digitale.

### Diagramma a Blocchi
![[Pasted image 20240712132502.png]]

### Macchina a Stati Finiti (FSM)

La FSM del DSP gestisce il flusso delle operazioni con quattro stati:

- **IDLE**: Stato di inattività. Il DSP attende il segnale di avvio (`start`).
- **LOAD**: Carica i coefficienti e i dati di ingresso nei registri estesi per il filtraggio FIR.
- **EXECUTE**: Esegue l'operazione selezionata (somma, sottrazione, moltiplicazione o filtraggio FIR).
- **DONE**: Segnala il completamento dell'operazione e ritorna a `IDLE`.

> Il vantaggio di utilizzare un design basato su una macchina a stati finiti è il riutilizzo delle risorse. Questo è molto importante nel nostro caso, poiché abbiamo una FPGA molto piccola e una CPU ben progettata in termini di area, quindi non avrebbe senso che il DSP fosse 10 volte più grande della CPU.

### Operazioni Supportate

1. **Somma (`operation = 00`)**:
   - Somma gli elementi dei vettori `A` e `B`.
   - I risultati vengono memorizzati nel vettore `result`.

2. **Moltiplicazione (`operation = 01`)**:
   - Moltiplica gli elementi corrispondenti dei vettori `A` e `B`.
   - I risultati vengono memorizzati nel vettore `result`.

3. **Sottrazione (`operation = 11`)**:
   - Sottrae gli elementi del vettore `B` da quelli del vettore `A`.
   - I risultati vengono memorizzati in `result`.

4. **Filtraggio FIR (`operation = 10`)**:
   - Applica il **filtro FIR** ai vettori `A` (coefficienti) e `B` (valori di segnale). La convoluzione viene eseguita accumulando i prodotti dei coefficienti e dei valori di segnale, memorizzando i risultati intermedi nel vettore `result`.

## Struttura del Codice

### Dichiarazione del Modulo e Ingressi/Uscite

Il modulo ha diversi ingressi e uscite che includono il clock (`clk`), il reset (`rst`), il segnale di avvio (`start`), il codice dell'operazione (`operation`), i vettori di coefficienti (`A`) e di segnali (`B`), il vettore di uscita (`result`) e il segnale di completamento (`done`).

### Registri e Segnali Interni

- **`state`, `next_state`**: Stati attuali e successivi della FSM.
- **`index`, `k`**: Contatori per iterare sui vettori `A` e `B`.
- **`acc`**: Accumulatore utilizzato nel filtraggio FIR.
- **`a_ext[11:0]`, `x_ext[11:0]`**: Coefficienti e segnali estesi per il filtraggio FIR.
- **`sum`, `product`, `diff`**: Risultati intermedi delle operazioni.
- **`a_bus`, `b_bus`**: Bus che gestiscono gli ingressi dei moduli di operazione.

### Moduli Indipendenti

Il DSP utilizza moduli indipendenti per eseguire le operazioni di base:

- **SUM**: Esegue la somma di due operandi.
- **MULT**: Esegue la moltiplicazione di due operandi.
- **SUB**: Esegue la sottrazione di due operandi.

Questi moduli operano con valori a **punto fisso** e applicano tecniche di saturazione per evitare **overflow** e di arrotondamento per minimizzare la perdita di precisione.

### Definizione degli Stati della FSM

- **IDLE**: Stato di inattività. Si reimpostano i contatori e si attende il segnale `start`.
- **LOAD**: Carica i coefficienti e i segnali nei registri estesi (`a_ext`, `x_ext`).
- **EXECUTE**: Esegue l'operazione selezionata:
  - Nella somma, moltiplicazione o sottrazione, itera sugli elementi dei vettori `A` e `B`.
  - Nel filtraggio FIR, accumula i prodotti dei coefficienti e dei segnali.
- **DONE**: Indica il completamento dell'operazione e ritorna a `IDLE`.

### Logica Operativa

1. **Transizione degli Stati**: La FSM cambia tra gli stati `IDLE`, `LOAD`, `EXECUTE` e `DONE` in base ai segnali di controllo (`start`, `done`) e ai contatori (`index`, `k`).
   
2. **Assegnazione dei Bus**: I bus `a_bus` e `b_bus` gestiscono gli operandi di ingresso nei moduli di somma, moltiplicazione e sottrazione. Nel caso del filtraggio FIR, `a_bus` e `b_bus` selezionano i coefficienti e i segnali estesi.

3. **Controllo dell'Esecuzione**: Nello stato `EXECUTE`, viene eseguita l'operazione specificata da `operation`. I risultati intermedi vengono memorizzati in `result`. Per il filtraggio FIR, i prodotti accumulati vengono memorizzati in `acc` e poi trasferiti nel vettore di risultati.

4. **Completamento dell'Operazione**: Nello stato `DONE`, si attiva il segnale `done` e la FSM ritorna a `IDLE`, indicando che l'operazione è terminata.

### Esempio di Funzionamento del Filtraggio FIR

1. Il vettore di coefficienti `A` e il vettore di segnale `B` vengono caricati nei registri estesi.
2. La FSM entra nello stato `EXECUTE`, dove inizia la convoluzione.
3. Per ogni indice del vettore `A`, il DSP calcola il prodotto corrispondente con il vettore `B`, accumulando i risultati in `acc`.
4. Il risultato intermedio viene memorizzato nel vettore `result`.
5. La FSM passa allo stato `DONE`, segnalando il completamento dell'operazione.

## Simulazioni e Test
### Simulazione
![[Pasted image 20240711215918.png]]
- Nella figura possiamo vedere che:
	- Le operazioni di ADD, SUB e MULT richiedono circa 16 cicli per essere completate.
	- L'operazione FIR ha un ritardo superiore a 500ns, equivalente a più di 50 cicli.
- È importante notare che queste operazioni non bloccano la CPU, quindi il suo funzionamento continuerà naturalmente mentre queste operazioni vengono eseguite.

![[Pasted image 20240711215832.png]]
### RTL
![[Pasted image 20240711220016.png]]

### Utilizzo delle Risorse

![[Captura de pantalla 2024-09-18 a las 3.10.40 p. m..png]]
### Report di Timing Post Sintesi

- Inizialmente, il timing non era soddisfacente e doveva essere più che raddoppiato. Per risolvere questo problema, ho provato le seguenti alternative:
	- Ritimizzazione automatica tramite lo strumento Vivado.
		- Non ha funzionato.
	- Ritimizazione e pipelining manuali.
		- Non ha funzionato.
	- Ritimizazione lenta (s-slow).
		- Ha causato problemi di coerenza dei dati.
	- Analizzare i percorsi critici e risolvere altre caratteristiche.
		- Ho trovato un fan-out molto elevato di 256, che sono riuscito a ridurre a 97 tramite lo strumento automatico di Vivado, ma non è stato possibile ridurlo ulteriormente.
- Tutto ciò mi ha portato a concludere che la soluzione migliore era modificare il clock, quindi ho optato per uno da 50MHz. Con questo clock, sono riuscito a far funzionare il design con valori molto accettabili. Tuttavia, vale la pena considerare le caratteristiche precedenti, poiché mi hanno permesso di dimezzare la frequenza del DSP, facilitando molto il compito di sincronizzazione.

![[Captura de pantalla 2024-09-18 a las 3.02.08 p. m..png]]


# Spagnolo

## Descripción General

El módulo `DSP` implementa una **Unidad de Procesamiento Digital (DSP)** diseñada para realizar operaciones de **suma**, **resta**, **multiplicación** y **filtrado FIR** sobre vectores de datos. Este diseño se basa en una codificación de **punto fijo Q(16,16)**, lo que garantiza un equilibrio entre rango y precisión en las operaciones. El módulo se organiza mediante una **Máquina de Estados Finitos (FSM)** que gestiona el flujo de control y las operaciones en función de la señal de inicio (`start`) y la selección de la operación (`operation`).

### Puertos

| **Nombre**    | **Tipo**  | **Tamaño**   | **Descripción**                                                                                  |
|---------------|-----------|--------------|--------------------------------------------------------------------------------------------------|
| `clk`         | Entrada   | 1 bit        | Señal de reloj.                                                                                  |
| `rst`         | Entrada   | 1 bit        | Señal de reinicio global.                                                                        |
| `start`       | Entrada   | 1 bit        | Señal de inicio de la operación.                                                                 |
| `operation`   | Entrada   | 2 bits       | Código de operación (00: Suma, 01: Multiplicación, 10: Filtrado FIR, 11: Resta).                 |
| `A`           | Entrada   | 32 bits x 8  | Vector de coeficientes del filtro FIR o operandos para las otras operaciones.                    |
| `B`           | Entrada   | 32 bits x 8  | Vector de valores de señal o operandos para las otras operaciones.                               |
| `result`      | Salida    | 32 bits x 8  | Vector de 8 valores intermedios que almacenan el resultado de la operación.                      |
| `done`        | Salida    | 1 bit        | Señal que indica la finalización de la operación.                                                |

### Codificación de Punto Fijo

El DSP está diseñado para trabajar con **punto fijo Q(16,16)**, lo que significa que los datos se representan con 16 bits para la parte entera y 16 bits para la parte fraccionaria. Este enfoque ofrece un balance entre **precisión** y **rango**, permitiendo realizar operaciones aritméticas con alta precisión en aplicaciones de filtrado digital.

### Diagrama de Bloques
![[Pasted image 20240712132502.png]]

### Máquina de Estados Finitos (FSM)

La FSM del DSP gestiona el flujo de la operación con cuatro estados:

- **IDLE**: Estado de reposo. El DSP espera la señal de inicio (`start`).
- **LOAD**: Carga los coeficientes y los datos de entrada en los registros extendidos para el filtrado FIR.
- **EXECUTE**: Ejecuta la operación seleccionada (suma, resta, multiplicación o filtrado FIR).
- **DONE**: Señaliza la finalización de la operación y retorna a `IDLE`.

> Lo beneficioso de usar un diseño basado en maquina de estado finito es el hecho de reutilizacion de recursos, esto es muy importante en nuestro caso ya que tenemos una fpga muy pequeña y tambien porque nuestra CPU esta muy bien diseñada en area, por lo cual no tendria sentido que nuestra dsp sea de 10 veces el tamaño de la cpu

### Operaciones Soportadas

1. **Suma (`operation = 00`)**:
   - Suma los elementos de los vectores `A` y `B`.
   - Los resultados se almacenan en el vector `result`.

2. **Multiplicación (`operation = 01`)**:
   - Multiplica los elementos correspondientes de los vectores `A` y `B`.
   - Los resultados se almacenan en `result`.

3. **Resta (`operation = 11`)**:
   - Resta los elementos del vector `B` a los del vector `A`.
   - Los resultados se almacenan en `result`.

4. **Filtrado FIR (`operation = 10`)**:
   - Aplica el **filtro FIR** a los vectores `A` (coeficientes) y `B` (valores de señal). La convolución se realiza acumulando los productos de los coeficientes y los valores de la señal, almacenando los resultados intermedios en el vector `result`.

## Estructura del Código

### Declaración del Módulo y Entradas/Salidas

El módulo tiene varias entradas y salidas que incluyen el reloj (`clk`), el reinicio (`rst`), la señal de inicio (`start`), el código de operación (`operation`), los vectores de coeficientes (`A`) y señales (`B`), y el vector de salida (`result`), junto con la señal de finalización (`done`).

### Registros y Señales Internas

- **`state`, `next_state`**: Estados actuales y siguientes de la FSM.
- **`index`, `k`**: Contadores de índices para iterar sobre los vectores `A` y `B`.
- **`acc`**: Acumulador utilizado en el filtrado FIR.
- **`a_ext[11:0]`, `x_ext[11:0]`**: Coeficientes y señales extendidos para el filtrado FIR.
- **`sum`, `product`, `diff`**: Resultados intermedios de las operaciones.
- **`a_bus`, `b_bus`**: Buses que manejan las entradas de los módulos de operaciones.

### Módulos Independientes

El DSP utiliza módulos independientes para realizar las operaciones básicas:

- **SUM**: Realiza la suma de dos operandos.
- **MULT**: Realiza la multiplicación de dos operandos.
- **SUB**: Realiza la resta de dos operandos.

Estos módulos operan con valores de **punto fijo** y aplican técnicas de saturación para evitar **overflow** y **redondeo** para minimizar la pérdida de precisión.

### Definición de Estados del FSM

- **IDLE**: Estado de reposo. Se reinician los contadores y se espera la señal `start`.
- **LOAD**: Carga los coeficientes y las señales en los registros extendidos (`a_ext`, `x_ext`).
- **EXECUTE**: Ejecuta la operación seleccionada:
  - En la suma, multiplicación o resta, itera sobre los elementos de los vectores `A` y `B`.
  - En el filtrado FIR, realiza la acumulación de los productos de los coeficientes y las señales.
- **DONE**: Indica la finalización de la operación y retorna a `IDLE`.

### Lógica de Operación

1. **Transición de Estados**: La FSM cambia entre los estados `IDLE`, `LOAD`, `EXECUTE` y `DONE` en función de las señales de control (`start`, `done`) y los contadores (`index`, `k`).
   
2. **Asignación de Buses**: Los buses `a_bus` y `b_bus` manejan los operandos de entrada a los módulos de suma, multiplicación y resta. En el caso del filtrado FIR, `a_bus` y `b_bus` seleccionan los coeficientes y las señales extendidas.

3. **Control de la Ejecución**: En el estado `EXECUTE`, se ejecuta la operación especificada por `operation`. Los resultados intermedios se almacenan en `result`. Para el filtrado FIR, los productos acumulados se almacenan en `acc` y luego se transfieren al vector de resultados.

4. **Finalización de la Operación**: En el estado `DONE`, la señal `done` se activa y la FSM retorna a `IDLE`, indicando que la operación ha finalizado.

### Ejemplo de Funcionamiento del Filtrado FIR

1. Se carga el vector de coeficientes `A` y el vector de señal `B` en los registros extendidos.
2. La FSM entra en el estado `EXECUTE`, donde comienza la convolución.
3. Para cada índice del vector `A`, el DSP calcula el producto correspondiente con el vector `B`, acumulando los resultados en `acc`.
4. El resultado intermedio se almacena en el vector `result`.
5. La FSM transiciona al estado `DONE`, señalizando la finalización de la operación.

## Simulacion y pruebas
### Simulación
![[Pasted image 20240711215918.png]]
- En la imagen podemos ver que:
	- Las operaciones de ADD,SUB y MULT tardan un aproximado de 16 ciclos en completarse
	- La operación de FIR tiene un retardo mayor de 500ns, lo que es equivalente a mas de 50 ciclos
- También hay que tener en cuenta que estas operaciones no bloquean la CPU, por lo cual es funcionamiento de esta, continuara de manera natural mientras se siguen aplicando estas operaciones

![[Pasted image 20240711215832.png]]
### RTL
![[Pasted image 20240711220016.png]]

### Utilización

![[Captura de pantalla 2024-09-18 a las 3.10.40 p. m..png]]
### Reporte de timing post síntesis

- En primeras instancias, el timing no cumplia y debia aumentar a mas del doble, en la busqueda de resolver este problema, la primera opcion mas simplista seria cambiar el clock, para evitar tener que realizar esto, probe las siguientes alternativas
	- Retiming automatico por la herramienta vivado
		- No funciono
	- Retiming e pipelining manual 
		- No funciono
	- s-slow retiming
		- Generaba problemas en la coherencia de los datos
	- Analizar caminos criticos y buscar resolver otras caracteristicas
		- Se encontro un fan out muy elevado de 256 que lo pude reducir a 97 con la herramienta automatica de vivado pero fue lo maximo posible a reducir
- Todo esto me hizo darme cuenta que la mejor opción es cambiar el clock, por lo cual opto por uno de 50MHz. Con el cual pude correr el diseño teniendo valores demasiado aceptables. Lo mismo vale la pena tomar las caracteristicas anteriores ya que estas me ayudaron a poder solo dividir en 2 la frecuencia de la dsp, lo cual facilita mucho la tarea a la hora de trabajar con la sincronizacion.

![[Captura de pantalla 2024-09-18 a las 3.02.08 p. m..png]]





