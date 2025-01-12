Traduzione file [grover.ipynb](https://github.com/Qiskit/textbook/blob/main/notebooks/ch-algorithms/grover.ipynb)

# Introduzione
Probabilmente hai sentito dire che uno dei tanti vantaggi di un computer quantistico rispetto a un computer classico è la sua velocità superiore nella ricerca di database. L'algoritmo di Grover dimostra questa capacità. Questo algoritmo può accelerare quadraticamente un problema di ricerca non strutturato, ma i suoi usi si estendono oltre; può servire come trucco generale o subroutine per ottenere miglioramenti quadratici del tempo di esecuzione per una varietà di altri algoritmi. Questo è chiamato trucco dell'amplificazione dell'ampiezza.

Supponiamo che ti venga dato un lungo elenco di elementi $N$. Tra questi elementi c'è un elemento con una proprietà unica che desideriamo individuare; lo chiameremo vincitore $w.$ Pensa a ogni elemento nell'elenco come a una casella di un colore particolare. Diciamo che tutti gli elementi nell'elenco sono grigi tranne il vincitore $w$, che è viola.

![[database.png]]

Per trovare la casella viola, l'elemento contrassegnato, usando il calcolo classico, si dovrebbero spuntare in media $N/2$ di queste caselle e, nel caso peggiore, tutte $N$ di esse. Su un computer quantistico, tuttavia, possiamo trovare l'elemento contrassegnato in circa $\sqrt{N}$ passaggi con il trucco dell'amplificazione dell'ampiezza di Grover. Un'accelerazione quadratica è davvero un notevole risparmio di tempo per trovare elementi contrassegnati in lunghe liste. Inoltre, l'algoritmo non usa la struttura interna della lista, il che lo rende *generico*; ecco perché fornisce immediatamente un'accelerazione quantistica quadratica per molti problemi classici.

## Overview dell'algoritmo
L'algoritmo di Grover è costituito da tre fasi principali dell'algoritmo: preparazione dello stato, oracolo e operatore di diffusione. La preparazione dello stato è dove creiamo lo spazio di ricerca, che è costituito da tutti i possibili casi che la risposta potrebbe assumere. Nell'esempio di elenco che abbiamo menzionato sopra, lo spazio di ricerca sarebbe costituito da tutti gli elementi di quell'elenco. L'oracolo è ciò che contrassegna la risposta corretta, o le risposte che stiamo cercando, e l'operatore di diffusione ingrandisce queste risposte in modo che possano risaltare ed essere misurate alla fine dell'algoritmo.

![[grover-overview.png]]

Quindi, come funziona l'algoritmo? Prima di guardare l'elenco degli elementi, non abbiamo idea di dove si trovi l'elemento contrassegnato. Pertanto, qualsiasi ipotesi sulla sua posizione è valida quanto qualsiasi altra, il che può essere espresso in termini di una
sovrapposizione uniforme: $\ket{s} = \frac{1}{\sqrt{N}} \sum_{x = 0}^{N -1} \ket{x}.$

Se a questo punto dovessimo misurare nella base standard $\{ \ket{x} \}$, questa sovrapposizione collasserebbe, secondo la quinta legge quantistica, in uno qualsiasi degli stati di base con la stessa probabilità di $\frac{1}{N} = \frac{1}{2^n}$. Le nostre possibilità di indovinare il valore corretto $w$ sono quindi $1$ in $2^n$, come ci si potrebbe aspettare. Quindi, in media dovremmo provare circa $N/2 = 2^{n-1}$ volte per indovinare l'elemento corretto.

Entra in gioco la procedura chiamata amplificazione dell'ampiezza, che è il modo in cui un computer quantistico aumenta significativamente questa probabilità. Questa procedura allunga (amplifica) l'ampiezza dell'elemento contrassegnato, il che riduce l'ampiezza degli altri elementi, in modo che la misurazione dello stato finale restituisca l'elemento giusto con quasi certezza.

Questo algoritmo ha una bella interpretazione geometrica in termini di due riflessioni, che generano una rotazione in un piano bidimensionale. Gli unici due stati speciali che dobbiamo considerare sono il vincitore $\ket{w}$ e la sovrapposizione uniforme $\ket{s}$. Questi due vettori abbracciano un piano bidimensionale nello spazio vettoriale $\cssId{vspace}{\mathbb{C}^N}$. Non sono del tutto perpendicolari perché $\ket{w}$ si verifica nella sovrapposizione con ampiezza $N^{-1/2}$.
Possiamo, tuttavia, introdurre uno stato aggiuntivo $\ket{s'}$ che si trova nell'intervallo di questi due vettori, che è perpendicolare a $\ket{w}$ e si ottiene da $\ket{s}$ rimuovendo $\ket{w}$ e
ridimensionando.

**Passaggio 1**: La procedura di amplificazione dell'ampiezza inizia nella sovrapposizione uniforme $\ket{s}$, che è facilmente costruita da $\ket{s} = H^{\otimes n} \ket{0}^n$ oppure utilizzando altri stati simmetrici entangled.

![[step1.png]]

Il grafico a sinistra corrisponde al piano bidimensionale attraversato dai vettori perpendicolari $|w\rangle$ e $|s'\rangle$ che consente di esprimere lo stato iniziale come $|s\rangle = \sin \theta | w \rangle + \cos \theta | s' \rangle,$ dove $\theta = \arcsin \langle s | w \rangle = \arcsin \frac{1}{\sqrt{N}}$. Il grafico a destra è un grafico a barre delle ampiezze dello stato $| s \rangle$.

**Passaggio 2**: Applichiamo la riflessione dell'oracolo $U_f$ allo stato $|s\rangle$.

![[step2.png]]

Geometricamente questo corrisponde a una riflessione dello stato $|s\rangle$ su $|s'\rangle$. Questa trasformazione significa che l'ampiezza di fronte allo stato $|w\rangle$ diventa negativa, il che a sua volta significa che l'ampiezza media (indicata da una linea tratteggiata) è stata abbassata.

**Passaggio 3**: Ora applichiamo una riflessione aggiuntiva ($U_s$) sullo stato $|s\rangle$: $U_s = 2|s\rangle\langle s| - \mathbb{1}$. Questa trasformazione mappa lo stato su $U_s U_f| s \rangle$ e completa la trasformazione.

![[step3.png]]

Due riflessioni corrispondono sempre a una rotazione. La trasformazione $U_s U_f$ ruota lo stato iniziale $|s\rangle$ più vicino al vincitore $|w\rangle$. L'azione della riflessione $U_s$ nel diagramma a barre dell'ampiezza può essere intesa come una riflessione sull'ampiezza media. Poiché l'ampiezza media è stata abbassata dalla prima riflessione, questa trasformazione aumenta l'ampiezza negativa di $|w\rangle$ a circa tre volte il suo valore originale, mentre diminuisce le altre ampiezze. Passiamo quindi al **passo 2** per ripetere l'applicazione. Questa procedura verrà ripetuta più volte per azzerare il vincitore.

Dopo $t$ passi saremo nello stato $|\psi_t\rangle$ dove: $| \psi_t \rangle = (U_s U_f)^t | s \rangle.$

Quante volte dobbiamo applicare la rotazione? Si scopre che circa $\sqrt{N}$ rotazioni sono sufficienti. Ciò diventa chiaro quando si osservano le ampiezze dello stato $| \psi \rangle$. Possiamo vedere che l'ampiezza di $| w \rangle$ cresce linearmente con il numero di applicazioni $\sim t N^{-1/2}$. Tuttavia, poiché abbiamo a che fare con ampiezze e non con probabilità, la dimensione dello spazio vettoriale entra come una radice quadrata. Pertanto è l'ampiezza, e non solo la probabilità, ad essere amplificata in questa procedura.

Per calcolare il numero di rotazioni dobbiamo conoscere la dimensione dello spazio di ricerca e il numero di risposte che stiamo cercando. Per ottenere il numero ottimale di iterazioni $t$, possiamo seguire l'equazione:

$$
t = \lfloor\frac{\pi}{4}\sqrt{\frac{N}{m}}\rfloor
$$

Dove N è la dimensione dello spazio di ricerca e m è il numero di risposte che vogliamo.![[grover_sketch.png]]

# Grover step-by-step
Ora che abbiamo visto come funziona effettivamente l'algoritmo di Grover, approfondiamo un po' la sua costruzione e analizziamo i diversi casi per ciascuno dei suoi componenti.

## Prepariamo lo spazio di ricerca
Il primo passaggio dell'algoritmo di Grover è la preparazione dello stato iniziale. Come abbiamo appena detto, lo spazio di ricerca è costituito da tutti i possibili valori che dobbiamo cercare per trovare la risposta che vogliamo. Per gli esempi in questo libro di testo, il nostro "database" è composto da tutti i possibili stati di base computazionale in cui possono trovarsi i nostri qubit. Ad esempio, se abbiamo 3 qubit, il nostro elenco è costituito dagli stati $|000\rangle, |001\rangle, \dots |111\rangle$ (ovvero gli stati $|0\rangle \rightarrow |7\rangle$). Quindi, in questo caso, la dimensione del nostro spazio di ricerca sarà $N = 2^{3} = 8$.

In alcuni casi, se conosciamo l'intervallo all'interno dello spazio di ricerca in cui è garantita la presenza della risposta, possiamo eliminare la base ridondante dal nostro spazio di ricerca per velocizzare l'algoritmo e ridurre la dimensione del circuito. In generale, possiamo preparare il nostro stato utilizzando qualsiasi stato simmetrico, come gli stati GHZ, gli stati W o gli stati Dicke.

Ad esempio, se stiamo cercando di risolvere un problema con una risposta usando 4 qubit e prepariamo il nostro stato usando la porta di Hadamard (cioè formando lo spazio di Hilbert), N sarà 16. Ma, se sappiamo che la risposta è all'interno di stati in cui solo un qubit ha il valore di 1 in qualsiasi momento, possiamo quindi usare lo stato W invece dell'intero spazio di Hilbert per preparare i nostri stati. Facendo ciò, abbiamo ridotto la dimensione dello spazio di ricerca da 16 a 4 e il numero di iterazioni ottimali $t$ da 3 a 1.

# Creiamo l'oracolo
Il secondo e più importante passaggio dell'algoritmo di Grover è l'oracolo. Gli oracoli aggiungono una fase negativa agli stati della soluzione in modo che possano distinguersi dal resto ed essere misurati. Ad esempio, per qualsiasi stato $|x\rangle$ nella base computazionale:

$$

U_\omega|x\rangle = \bigg\{

\begin{aligned}

\phantom{-}|x\rangle \quad \text{if} \; x \neq \omega \\

-|x\rangle \quad \text{if} \; x = \omega \\

\end{aligned}

$$
Questo oracolo sarà una matrice diagonale, dove la voce che corrisponde all'elemento contrassegnato avrà una fase negativa. Ad esempio, se abbiamo tre qubit e $\omega = \text{101}$, il nostro oracolo avrà la matrice:
$$

U_\omega =

\begin{bmatrix}

1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\

0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 \\

0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 \\

0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 \\

0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 \\

0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 \\

0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 \\

0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 \\

\end{bmatrix}

\begin{aligned}

\\

\\

\\

\\

\\

\\

\leftarrow \omega = \text{101}\\

\\

\\

\\

\end{aligned}

$$
Ciò che rende l'algoritmo di Grover così potente è la facilità con cui è possibile convertire un problema in un oracolo di questa forma. Ci sono molti problemi computazionali in cui è difficile _trovare_ una soluzione, ma relativamente facile _verificare_ una soluzione. Ad esempio, possiamo facilmente verificare una soluzione a un sudoku controllando che tutte le regole siano soddisfatte. Per questi problemi, possiamo creare una funzione $f$ che accetta una soluzione proposta $x$ e restituisce $f(x) = 0$ se $x$ non è una soluzione ($x \neq \omega$) e $f(x) = 1$ per una soluzione valida ($x = \omega$). Il nostro oracolo può quindi essere descritto come:

$$

U_\omega|x\rangle = (-1)^{f(x)}|x\rangle

$$
e la matrice dell'oracolo sarà una matrice diagonale della forma:

$$

U_\omega =

\begin{bmatrix}

(-1)^{f(0)} & 0 & \cdots & 0 \\

0 & (-1)^{f(1)} & \cdots & 0 \\

\vdots & 0 & \ddots & \vdots \\

0 & 0 & \cdots & (-1)^{f(2^n-1)} \\

\end{bmatrix}

$$

## Dettagli
Costruzione del circuito di un oracolo di Grover.
Se abbiamo la nostra funzione classica $f(x)$, possiamo convertirla in un circuito reversibile della forma:

![[circuit1.png]]

Se inizializziamo il qubit di 'output' nello stato $|{-}\rangle$, l'effetto di contraccolpo di fase lo trasforma in un oracolo di Grover (simile al funzionamento dell'oracolo Deutsch-Jozsa):

![[circuit2.png]]

Ignoriamo quindi il qubit ausiliario ($|{-}\rangle$).

## L'operatore di diffusione
Infine, dopo che l'oracolo ha contrassegnato la risposta corretta rendendola negativa, l'ultimo passaggio dell'algoritmo di Grover, ovvero l'operatore di diffusione.

La costruzione dell'operatore di diffusione dipende da cosa decidiamo di usare per preparare i nostri stati iniziali. In genere, l'operatore di diffusione ha la seguente costruzione.

![[operatore_di_diffusione.png]]

Per la parte successiva di questo capitolo, creeremo oracoli di esempio in cui conosciamo $\omega$ in anticipo, e non ci preoccuperemo se questi oracoli siano utili o meno. Alla fine del capitolo, tratteremo un breve esempio in cui creiamo un oracolo per risolvere un problema (sudoku) e un famoso problema grafico, il problema di ricerca del triangolo.
