## Vantaggi dell'algoritmo quantistico: Grover
Come già accennato nel capitolo precedente, uno dei vantaggi di sviluppare un algoritmo con un computer quantistico rispetto a uno classico è la velocità con cui quello quantistico effettua una ricerca all'interno di una struttura dati.  
Se ci concentriamo sull'algoritmo di Grover, possiamo dire che esso può accelerare quadraticamente un problema di ricerca non strutturato.

Supponiamo di avere un elenco di $n$ elementi e, tra di questi, dobbiamo individuarne uno che abbia determinate caratteristiche che soddisfano la nostra esigenza. Solitamente, in un computer classico, per la ricerca all'interno di una struttura dati, tipo array, nel caso peggiore dobbiamo verificare tutti gli $n$ elementi prima di trovare quello di cui necessitiamo. In media, invece, dovremmo verificare $n/2$ elementi prima di trovare quello desiderato.

Parlando invece di computer quantistici, abbiamo che la caratteristica dell'algoritmo di Grover è quella di implicare un'accelerazione quadratica per trovare l'elemento con le caratteristiche di cui abbiamo bisogno. Questo significa che possiamo trovarlo in circa $\sqrt n.$  
Banalmente, possiamo dedurre che si tratta di un notevole risparmio di tempo, soprattutto se le grandezze delle strutture dati in cui dobbiamo effettuare la ricerca diventano sempre più grandi.

## Generalizzazione del problema con Grover
Sappiamo che l'algoritmo di Grover viene diviso in 3 parti fondamentali. **La preparazione dello stato iniziale** si occupa di creare lo spazio che viene utilizzato per la ricerca della soluzione, in breve, un range di stati in cui si può trovare una soluzione al problema. L'**oracolo** è una funzione che identifica le soluzioni che vengono considerate corrette per il problema dato. Infine, l'**operatore di diffusione** si occupa di "ingrandire" le risposte trovate, in modo che vengano risaltate e successivamente misurate al termine dell'algoritmo.

### Preparazione dello stato iniziale
Il primo passaggio per applicare l'algoritmo di Grover è la preparazione dello spazio in cui l'oracolo dovrà fare la ricerca per trovare la soluzione al nostro problema. Sostanzialmente, la nostra struttura dati deve contenere tutti i possibili stati/elementi in cui la nostra soluzione potrebbe trovarsi.

### Creazione dell'oracolo
Il secondo passaggio, quello più importante, è l'oracolo. Questo ha il compito di contraddistinguere la soluzione finale da tutti gli altri stati. E come? Semplicemente applica una fase negativa agli stati della soluzione. Vediamo un esempio per un qualsiasi stato $\ket{x}$:
$$
U_\omega\ket{x} = 
\begin{cases} 
\begin{aligned}[c]
\ket{x} \quad \text{if} \; x \neq w \\
-\ket{x} \quad \text{if} \; x = w 
\end{aligned}
\end{cases}
$$
dove $w$ è l'elemento che vogliamo come soluzione.  
L'oracolo viene, dunque, rappresentato come una matrice diagonale, dove la voce corrispondente all'elemento che soddisfa la soluzione avrà una fase negativa.  
La semplicità dell'oracolo sta nel fatto che è facilmente possibile convertire un problema in un oracolo. Infatti, ci sono molti problemi computazionali in cui è relativamente facile verificare una soluzione, ma in cui è veramente difficile trovarne una soluzione, come il Sudoku.

### Operatore di diffusione
L'ultimo passaggio, dopo che l'oracolo ha contrassegnato la risposta negandola, si occupa di enfatizzare le soluzioni corrette, aumentando la probabilità di trovarle al termine dell'algoritmo. L'operatore di diffusione è, dunque, una combinazione di due riflessioni (una specifica operazione matematica che modifica l'ampiezza di uno stato invertendola rispetto a un determinato riferimento):
- **Riflessione intorno alla media delle ampiezze**: ogni stato ha un'ampiezza associata (un numero che determina la sua probabilità). L'operatore calcola la media di tutte queste ampiezze e riflette ogni ampiezza rispetto a questa media. Possiamo dedurre, quindi, che ampiezze basse diventano ancora più basse e quelle alte ancora più alte;
- **Riflessione rispetto allo stato iniziale**: serve a reindirizzare l’amplificazione delle ampiezze verso la struttura dello stato iniziale.

## Grover per Sudoku 2x2
Come descritto precedentemente, abbiamo 3 fasi essenziali per sviluppare la soluzione. Nella parte della **preparazione dello stato iniziale**, dato che abbiamo un problema composto da $2x2$ celle, è ragionevole scegliere 4 bit (uno per ogni cella) che restituiranno un numero pari a $2^4$ possibili soluzioni.

Come prima cosa, a questo punto, serve capire quante iterazioni l'algoritmo ha bisogno di fare per arrivare a uno stato che comprenda le soluzioni esatte per il Sudoku. Abbiamo bisogno di utilizzare la seguente formula:$$t \approx \left\lfloor \frac{\pi}{4}\sqrt{\frac{N}{M}} \right\rfloor,$$
dove $N$ è il numero di tutte le possibili combinazioni di soluzione e $M$ è il numero di soluzioni valide per il Sudoku.

Come ultima cosa, si deve pensare a inizializzare i 4 stati che occuperanno le relative 4 celle del gioco logico, utilizzando banalmente una porta di Hadamard, che ci riserverà per ogni stato, con probabilità $\frac{1}{\sqrt 2}$, il qubit $\ket{0}$ o il qubit $\ket{1}$.

All'interno della parte dell'**oracolo**, abbiamo la necessità di trovare quali soluzioni sono corrette per il nostro sistema. Come descritto precedentemente, il Sudoku ha delle regole ben precise, che chiameremo clausole. Dobbiamo dunque controllare che esse vengano rispettate da tutti i qubit per arrivare a una prima soluzione.

Avendo un Sudoku della forma:
![[sudoku.jpeg]]

Dobbiamo applicare le regole del gioco. Quindi, avremo, ad esempio, un insieme di clausole dove $\{[a \neq b], [a \neq c], [b \neq d], [c \neq d]\}$. Queste sono le regole essenziali perché l'algoritmo ritorni una soluzione valida.

Abbiamo poi l'**operatore di diffusione**, che, successivamente all'oracolo, in cui vengono restituite, in questo caso, 2 possibili soluzioni valide, ha il compito di enfatizzarle per darle più probabilità di verificarsi.  
Proprio in questa fase entra in gioco l'algoritmo di Grover. Infatti, applichiamo in questa sezione le porte $$H^{\otimes n}Z_{OR}H^{\otimes n},$$che ci servono per la procedura di _amplificazione dell'ampiezza_, un meccanismo geometrico di due riflessioni, che generano una rotazione in un piano bidimensionale.  
Il primo passaggio si occupa della sovrapposizione uniforme dello stato iniziale, che si costruisce con le porte $H^{\otimes n} \ket{0}^n$, dove le ampiezze di tutte le soluzioni si trovano alla stessa altezza, come a destra nel seguente grafico, mentre a sinistra abbiamo un piano 2D attraversato dai vettori $\ket{w}$ (vettore della soluzione accettata) e $\ket{s'}$, dove $\ket{s} = \sin\theta\ket{w} + \cos\theta\ket{s'}$ e $\sin\theta = \braket{s|w} = \frac{1}{\sqrt{N}}.$

![[relazione/arianna/img-02/step1.png]]

IIl secondo passaggio applica la riflessione dell'oracolo allo stato $\ket{s}$. Questa trasformazione viene eseguita sullo stato $\ket{w}$ (stato che esprime la soluzione esatta), facendolo diventare negativo. Questo implica che la media delle altezze di tutte le possibili soluzioni si abbassa. Vediamo i grafici:

![[relazione/arianna/img-02/step2.png]]

Ora non ci resta che l'ultimo passaggio, in cui viene applicata la diffusione, che si occupa di riportare lo stato $\ket{w}$ nello stato originale (positivo). Tuttavia, dato che, sottraendo la media a tutte le soluzioni, incluso $\ket{w}$, quest'ultimo si trova nello stato negativo e gli viene sottratta la media, esso diventa ancora più negativo. Successivamente, riportandolo nello stato originale (positivo), esso avrà un'ampiezza più amplificata rispetto a tutti gli altri stati che non sono vere soluzioni.

![[relazione/arianna/img-02/step3.png]]

## Bibliografia
Algoritmo di Grover: [GitHub](https://github.com/Qiskit/textbook/blob/aebdd2bc86ddb7a79dd8441d52c839d312ffafbb/notebooks/ch-algorithms/grover.ipynb)
