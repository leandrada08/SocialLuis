
# Italiano


## CPU con pipeline
Per realizzare la CPU con pipeline dobbiamo analizzare i componenti che abbiamo, poiché dobbiamo distribuirli lungo il pipeline per evitare un collo di bottiglia. Quando abbiamo progettato questi componenti e li abbiamo misurati, ci siamo resi conto che il banco registri, l'ALU e la memoria hanno tempi simili, quindi si potrebbe optare per una CPU con un pipeline di 2, 3 o 5 fasi. Per cercare di ridurre il numero di fasi, dovremmo considerare di dividere uno di questi componenti in 2.

- Per tutto ciò che è stato detto, abbiamo optato per un pipeline a 5 fasi, che sono:
   - IF: Fetch dell'istruzione
   - ID: Decodifica dell'istruzione
   - EX: Esecuzione dell'operazione
   - MEM: Accesso alla memoria
   - WRITE: Scrittura del risultato

### Analisi della soluzione proposta
Quando facciamo un'analisi del tempo di accelerazione, ci rendiamo conto che è 4, tranne in alcuni casi.
- In casi come, ad esempio, 3 LW consecutivi, l'accelerazione è molto inferiore, è tra 1,5 e 2.
- Riteniamo che questa sia la migliore opzione per il design che realizzeremo.

## RTL
![[Pasted image 20240402184814.png]]

---

Avísame si necesitas más traducciones o ajustes.




# Spagnolo
## CPU con pipeline 
Para realizar la CPU con pipeline debemos analizar las componentes que tenemos, ya que debemos distribuirla a lo largo del pipeline para evitar un cuello de botella, cuando diseñamos estas componentes y las medimos nos dimos cuenta que el banco de registro, la alu y las memoria tienen un tiempo similar, por lo cual se podria optar por un CPU con un pipeline de 2,3 o 5 etapas. Para buscar hacer una cantidad menor de etapas deberiamos analizar dividir cualquiera de estas componentes antes mencionada en 2.
- Por todo lo dicho anteriormente optamos por un pipeline de 5 etapas, las cuales son:
	- IF: Busqueda de la instruccion
	- ID: Decodificacion de la instruccion
	- EX: Ejecucion de la operacion
	- MEM: Acceso a memoria
	- WRITE: Escritura del resultado

### Analisis de solucion propuesta
Cuando hacemos un analisis en el tiempo de la acelaracion, nos damos cuenta que es 4, a excepcion de algunos casos
- En casos como por ejemplo 3 LW consecutivos, la aceleracion es mucho menos, es entre 1,5 y 2
- Creemos que esta es la mejor opcion para el diseño que haremos.

## RTL
![[Pasted image 20240402184814.png]]


