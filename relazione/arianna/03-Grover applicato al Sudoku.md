## Vantaggi algoritmo di Grover
Come già accennato nel capitolo precedente, uno dei vantaggi di sviluppare un algoritmo con un computer quantistico rispetto ad uno classico è la velocità con cui quello quantistico effettua una ricerca all'interno di una struttura dati.
Se ci concentriamo sull'algoritmo di Grover, possiamo dire che esso può accelerare quadraticamente un problema di ricerca non strutturato.

Supponiamo di avere un elenco di $n$ elementi e tra di questi dobbiamo individuarne uno che abbia determinate caratteristiche che soddisfano la nostra esigenza. Solitamente in un computer classico la ricerca all'interno di una struttura dati, tipo array, nel caso peggiore, dobbiamo verificare tutti gli $n$ elementi prima di trovare quello di cui necessitiamo, in media, invece, abbiamo che dovremmo verificare $n/2$ elementi prima di trovare quello desiderato.
Parlando invece di computer quantistici, abbiamo che la caratteristica dell'algoritmo di Grover è che implica un'accelerazione quadratica per trovare l'elemento con le caratteristiche di cui abbiamo bisogno, questo significa che possiamo trovarlo in circa $\sqrt n.$ Banalmente possiamo dedurre che è un notevole risparmio di tempo, soprattutto se le grandezze delle strutture dati in cui dobbiamo effettuare la ricerca diventano sempre più grandi.

## Generalizzazione del problema con Grover
Sappiamo che l'algoritmo di Grover viene diviso in 3 parti fondamentali:
- la preparazione dello stato iniziale;
- l'oracolo (una funzione che identifica le soluzioni corrette al problema);
- l'operatore di diffusione;

### Preparazione dello stato iniziale
Il primo passaggio per applicare l'algoritmo di Grover è la preparazione dello spazio in cui l'oracolo dovrà fare la ricerca per trovare la soluzione al nostro problema. Sostanzialmente, la nostra struttura dati deve contenere tutti i possibili stati/elementi in cui la nostra soluzione potrebbe trovarsi.

### Creazione dell'oracolo
Il secondo passaggio, quello più importante, è l'oracolo. Questo ha il compito di contraddistinguere la soluzione finale, da tutti gli altri stati, e come? Semplicemente applica una fase negativa agli stati della soluzione, vediamo un esempio per un qualsiasi stato $\ket{x}$:
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
L'oracolo viene, dunque, rappresentato come una matrice diagonale, dove la voce corrisponde all'elemento che soddisfa la soluzione avrà una fase negativa.
La semplicità dell'oracolo sta nel fatto in cui è facilmente possibile convertire un problema in un oracolo, infatti ci sono molti problemi computazionali in cui è relativamente facile verificare una soluzione, ma in cui è veramente difficile trovarne una soluzione - come il Sudoku.

### Operatore di diffusione
