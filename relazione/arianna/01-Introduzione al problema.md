## Sudoku
### Regole
La traduzione di Sudoku è "numero unico", questo ha senso in quanto lo scopo del gioco è completare tutte le celle di una griglia 9x9 con numeri che non sono ripetuti. Spieghiamo meglio: per un insieme di numeri che vanno da 1 a 9, abbiamo una griglia composta da 9x9 celle, questo perché, appunto, dobbiamo utilizzare tutti i numeri ad una cifra che stanno nell'intervallo $[1-9]$ (estremi inclusi).  
All'interno di ogni cella potremo scegliere di posizionare un solo numero a una cifra che l'intervallo $[1-9]$ ci fornisce, ma questo numero che scegliamo deve rispettare alcune regole fondamentali per la riuscita del Sudoku.  
La prima regola che ci viene fornita è il fatto che il numero che scegliamo per una cella precisa non deve ripetersi per tutta la riga e per tutta la colonna di cui la cella fa parte. In aggiunta, oltre a ciò, dobbiamo, una volta conclusa l'intera riga o colonna, avere per tutte le celle che si susseguono sulla riga o colonna l'insieme di tutti i numeri dell'intervallo a una cifra rappresentabili. Ma le regole non finiscono qui, in quanto all'interno della tabella 9x9 vengono definite anche delle matrici 3x3, e in queste dovremo avere l'insieme di tutti i numeri presenti nell'intervallo $[1-9]$ senza essere mai ripetuti.
Solitamente la griglia iniziale viene popolata con alcune cifre, note come "indizio".

### Problematiche
Possiamo dire ad oggi che il problema di risoluzione del Sudoku è un problema del tipo NP-completo. Cosa significa? Che l'esistenza di un algoritmo che risolva questo gioco logico in tempo polinomiale è ignoto.  
Vediamo un piccolo esempio di calcolo della soluzione: se dovessi applicare un tipo di soluzione *Generate and Test*, il funzionamento sarebbe che, per ogni cella vuota, inserisco un numero e verifico che le regole del gioco vengano rispettate. Se non succede, riprovo con un'altra combinazione di numeri. Possiamo già intuire che la complessità di questa tipologia di soluzione sarà eccessiva. Infatti, se noi abbiamo una tabella 9x9, quindi un totale di 81 celle da riempire, supponiamo che 31 delle celle siano già complete con gli indizi che abbiamo citato precedentemente; ne rimangono da riempire esattamente 50. A questo punto, il numero di combinazioni possibili da testare è $9^{50}$, circa $5*10^{47}$. 
Ma quale sarebbe il tempo di risoluzione previsto? Supponendo che ci voglia 1 nano-secondo per formulare una possibile combinazione, il tempo previsto sarà:  
- In un anno abbiamo $3.154*10^{16}$ nano-secondi.  
Se, per trovare una soluzione valida, nel caso peggiore, dovessi testare tutte le possibili soluzioni (cioè $5*10^{47}$), e per ogni soluzione dovessi impiegare 1 nano-secondo, abbiamo che:  
- Per tutte le possibili soluzioni impiego un tempo di $5*10^{47}$ nano-secondi,  
- che in anni si traduce in $5*10^{47}/3.154*10^{16} = 10^{31}$ anni.  
Dunque, possiamo capire che ci vorrebbe veramente un tempo esagerato per trovare una possibile soluzione.

### Perché utilizzare i computer quantistici?
I computer quantistici potrebbero offrire vantaggi significativi nella risoluzione del Sudoku, o di problemi NP-completi in generale, rispetto ai computer classici. Ecco alcune motivazioni per cui dovremmo considerare l'utilizzo dei computer quantistici per trovare una soluzione valida:

I computer quantistici sfruttano la **sovrapposizione quantistica**, che consente a un singolo qubit di rappresentare simultaneamente più stati. In pratica, invece di testare ogni possibile combinazione di numeri in sequenza come un computer classico, un computer quantistico potrebbe esplorare più combinazioni in parallelo. Ciò ridurrebbe drasticamente il tempo necessario per trovare una soluzione rispetto ai metodi tradizionali.

 L'**entanglement quantistico** consente a qubit separati di essere correlati tra loro in modo che un'operazione su uno possa influenzare istantaneamente gli altri. Questo fenomeno potrebbe, in teoria, consentire una ricerca più rapida e una risoluzione più efficiente di problemi complessi, come il Sudoku, dove è necessario considerare molteplici vincoli e condizioni in parallelo.

In breve, per i problemi che richiedono enormi quantità di tempo per essere risolti dai computer classici (come nel caso delle combinazioni esponenziali nel Sudoku), un computer quantistico potrebbe essere in grado di ridurre drasticamente il tempo di esecuzione. In pratica, una soluzione che potrebbe richiedere migliaia di anni su un computer classico potrebbe, in teoria, essere trovata in tempi molto più brevi grazie ai benefici del calcolo quantistico.

## Bibliografia
Regole: [Wikipedia](https://it.wikipedia.org/wiki/Sudoku)
Problematiche: [esempio complessità](https://www.ce.unipr.it/~aferrari/corda/lezioni/corda-sudoku.pdf)

