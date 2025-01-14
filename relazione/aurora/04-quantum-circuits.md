[Canale YouTube](https://www.youtube.com/watch?v=42OiBzfdE2o&list=PLOFEBzvs-VvqKKMXX4vbi4EB1uaErFMSO)

```table-of-contents
```
# Circuiti
In informatica, i circuiti sono modelli di calcolo in cui le informazioni vengono trasportate da fili attraverso una rete di *gates*, che rappresentano operazioni che trasformano le informazioni trasportate dai fili. I circuiti quantistici sono solo un esempio di modello di calcolo basato su questo concetto più generale.

Sebbene la parola "circuito" si riferisca spesso a un percorso circolare, i percorsi circolari non sono effettivamente consentiti nei modelli di circuito più comuni di calcolo. Cioè, di solito studiamo i circuiti aciclici quando pensiamo ai circuiti come modelli computazionali. I circuiti quantistici seguono questo schema; sebbene potremmo eseguire un circuito quantistico tutte le volte che vorremmo, un circuito quantistico stesso rappresenta una sequenza finita di operazioni che non possono contenere cicli di feedback.

## Circuiti quantistici
Nel modello di circuito quantistico, i fili rappresentano qubit e i gates rappresentano le operazioni che agiscono su questi qubit. Per ora ci concentreremo sulle operazioni che abbiamo incontrato finora, vale a dire le operazioni unitarie e le misurazioni di base standard.
Ecco un esempio di un semplice circuito quantistico:

![[quantum_circuit.png|center|400]]

In questo circuito abbiamo il singolo qubit X che rappresenta la linea orizzontale e una sequenza di gates che rappresentano operazioni unitarie su questo qubit. Il flusso di informazioni va da sinistra a destra, quindi la prima operazione eseguita è un Hadamard, la seconda è un'operazione S il terzo è un altro Hadamard, e l'operazione finale è un'operazione T.
Se applicassimo lo stato $\ket{0}$ otteniamo come stato finale ${1+i \over 2}\ket{0} + {1 \over \sqrt 2}\ket{1}$.
Solitamente i circuiti quantici hanno tutti i qubit inizializzati a $\ket{0}$, come in questo caso, ma possiamo avere anche casi in cui desideriamo porre l'input a stati di qubit differenti.
Ora vediamo come possiamo specificare questo circuito in Qiskit, iniziando con le importazioni necessarie per la sezione corrente e creando il circuito aggiungendo le porte in sequenza da sinistra verso destra.

```python
from qiskit import QuantumCircuit, QuantumRegister, ClassicalRegister
from qiskit.primitives import Sampler
from qiskit.visualization import plot_histogram

circuit = QuantumCircuit(1)

circuit.h(0)
circuit.s(0)
circuit.h(0)
circuit.t(0)

circuit.draw("mpl")
```
![[qiskit_circuit02.png | center | 400]]

Un altro esempio di circuito è il seguente:

![[quantum_circuit01.png | center |400]]

>**Convenzione di Qiskit**: il qubit più alto con indice 0 corrisponde al qubit più a destra in una tupla di qubits, mentre il secondo qubit più alto corrisponde al secondo qubit più a destra nella tulpa e così via.

La prima operazione è un'operazione di Hadamard sulla Y, quando applichiamo un'operazione su un singolo qubit non accade nulla agli altri qubit.
La seconda operazione è il NOT controllato, dove Y è il controllo mentre X è il "bersaglio", quest'ultima operazione su un sistema standard funziona nel seguente modo:

![[controlled-NOT-circuit.png | center |400]]

Eseguiamo le operazioni del circuito e discutiamo i risultati:

![[circuit-explanation.jpeg]]

Quindi il circuito ci da un modo per creare lo stato $\ket{\phi^+}$ se lo stato dei due qubits è inizializzato a $\ket{00}$.

In generale, i circuiti quantistici possono contenere qualsiasi numero di "fili" qubit. Possiamo anche includere "fili" di bit classici, che sono indicati da doppie linee.

![[circuit-w-two-classical-bit.png|center|400]]

Abbiamo due bit classici A e B, le porte di misura rappresentano misurazioni di base standard: i qubit vengono modificati nei loro stati post-misurazione, mentre i risultati della misurazione vengono sovrascritti sui bit classici indicati dalle frecce.

Quindi quale sono i possibili risultati di questo circuito? Essendo inizializzato di default come Y = X = $\ket{0}$, dobbiamo applicare H = $\begin{pmatrix} {1 \over \sqrt 2} & {1 \over \sqrt 2} \\ {1 \over \sqrt 2} & - {1 \over \sqrt 2}\end{pmatrix}$ ad Y, quindi faccio il prodotto tensoriale tra H e $\ket{0}$ $$\ket{+} = \begin{pmatrix} {1 \over \sqrt 2} & {1 \over \sqrt 2} \\ {1 \over \sqrt 2} & - {1 \over \sqrt 2} \end{pmatrix} \otimes \begin{pmatrix} 1 \\ 0 \end{pmatrix} = \begin{pmatrix} {1 \over \sqrt 2} \\ {1 \over \sqrt 2}\end{pmatrix} = {1 \over \sqrt 2}(\ket{0} + \ket{1})$$
questo significa che Y può trovarsi per il 50% circa nello stato 0 e per il 50% circa nello stato 1.
Ora applichiamo il C-NOT che:
- nel caso in cui Y si trovi nello stato $\ket{0}$ X rimane $\ket{0}$
- nel caso in cui Y si trovi nello stato $\ket{1}$ X viene "flippato" a $\ket{1}$
Quindi abbiamo una percentuale del 50% circa che il risultato sia 00 mentre l'altro 50% circa che sia 11.

# Prodotti interni, ortonormalità e proiezioni
## Prodotti Interni
Il valore di $\braket{\psi | \phi}$ è chiamato prodotto interno tra i vettori $\ket{\psi}$ e $\ket{\phi}$ (ritorna uno scalare).
Raccogliamo ora insieme alcuni fatti di base sui prodotti interni dei vettori:
1. **Relazione con la norma euclidea**: il prodotto interno di qualsiasi vettore
$$\ket{\psi} = \sum_{a \in \Sigma} \alpha_a \ket{a}$$
	con se stesso
$$\braket{\psi | \psi} = \sum_{a \in \Sigma} \overline{\alpha_a} \alpha_a = \sum_{a \in \Sigma} |\alpha_a|^2 = ||\ket{\psi}||^2$$
	Pertanto, la norma euclidea di un vettore può essere alternativamente espressa come
$$||\ket{\psi}|| = \sqrt{\braket{\psi | \psi}}$$
	dove
$$\braket{\psi | \psi} \geq 0$$
	con  $\braket{\psi | \psi} = 0$ se e solo se $\ket{\psi} = 0$.

2. **Simmetria coniugata**: per due vettori qualsiasi
$$\ket{\psi} = \sum_{a \in \Sigma} \alpha_a\ket{a} \quad \text{e} \quad \ket{\phi} = \sum_{b\in\Sigma}\beta_b\ket{b}$$
	abbiamo 
$$\braket{\psi|\phi} = \sum_{a\in \Sigma}\overline{\alpha_a}\beta_a \quad \text{e} \quad \braket{\phi|\psi} = \sum_{a\in \Sigma}\overline{\beta_a}\alpha_a$$

	quindi
$$\overline{\braket{\psi | \phi}} = \braket{\phi | \psi}$$
3. **Linearità nel secondo argomento (e linearità coniugata nel primo)**: supponiamo che $\ket{\psi}$, $\ket{\phi_1}$ e $\ket{\phi_2}$ sono vettori e $\alpha_1$ e $\alpha_2$ sono numeri complessi. Se definiamo un nuovo vettore
$$\ket{\phi} = \alpha_1\ket{\phi_1} + \alpha_2 \ket{\phi_2}$$
	allora
$$\braket{\psi | \phi} = \bra{\psi}(\alpha_1\ket{\phi_1} + \alpha_2 \ket{\phi_2}) = \alpha_1\braket{\psi|\phi_1} + \alpha_2\braket{\psi | \phi_2}$$
	Cioè, il prodotto interno è lineare nel secondo argomento. Questo può essere verificato attraverso le formule di cui sopra o semplicemente notando che la moltiplicazione della matrice è lineare in ogni argomento (e in particolare nel secondo argomento).
	La combinazione di questo fatto con la simmetria coniugata rivela che il prodotto interno è coniugato lineare nel primo argomento. Questo è, se $\ket{\psi_1}$, $\ket{\psi_2}$ e $\ket{\phi}$ sono vettori e $\alpha_1$ e $\alpha_2$ sono numeri complessi, quindi definiamo
$$\ket{\psi} = \alpha_1\ket{\psi_1} + \alpha_2 \ket{\psi_2}$$
	allora
$$\braket{\psi | \phi} = (\overline{\alpha_1}\bra{\psi_1} + \overline{\alpha_2} \bra{\psi_2})\ket{\phi} = \overline{\alpha_1}\braket{\psi_1|\phi} + \overline{\alpha_2}\braket{\psi_2 | \phi}$$

4. **La disuguaglianza di Cauchy-Schwarz**: per qualsiasi scelta dei vettori $\ket{\psi}$ e $\ket{\phi}$ avendo lo stesso numero di elementi, abbiamo
$$|\braket{\psi | \phi}| \leq ||\ket{\psi}||\;||\ket{\phi}||$$
	Questa è una disuguaglianza incredibilmente utile che viene utilizzata abbastanza ampiamente nell'informazione quantistica (e in molti altri campi di studio).

## Insiemi ortogonali e ortonormali
Due vettori $\ket{\psi}$ e $\ket{\phi}$ si dicono **ortogonali** se il solo prodotto interno è zero: $\braket{\psi | \phi} = 0$.

Un insieme di vettori $\{\ket{\psi_1}, \dots, \ket{\psi_m}\}$ è chiamato insieme **ortogonale** se ogni vettore dell'insieme è ortogonale a ogni altro vettore dell'insieme. Cioè, questo set è ortogonale se $\braket{\psi_j | \psi_k} = 0$, per tutte le scelte di $j, k$ $\in \{1, \dots, m\}$.

Un insieme di vettori $\{\ket{\psi_1}, \dots, \ket{\psi_m}\}$ è chiamato insieme **ortonormale** se è un insieme ortogonale e, inoltre, ogni vettore nell'insieme è un vettore unitario (con modulo uguale a 1). 
In alternativa, questo insieme è un insieme ortonormale se abbiamo:
$$\braket{\psi_j|\psi_k} = \begin{cases} 1 & j = k \\ 0 & j \neq k\end{cases} \tag{3}$$
per tutte le scelte di $j, k$ $\in \{1, \dots, m\}$.

Un insieme di vettori $\{\ket{\psi_1}, \dots, \ket{\psi_m}\}$ è una base ortonormale se, oltre ad essere un insieme ortonormale, forma una base. Questo è equivalente a $\{\ket{\psi_1}, \dots, \ket{\psi_m}\}$ essendo un insieme ortonormale e $m$ essendo uguale alla dimensione dello spazio da cui $\ket{\psi_1}, \dots, \ket{\psi_m}$ sono disegnati.

### Estendere gli insiemi ortonormali alle basi ortonormali
Supponiamo che  $\ket{\psi_1}, \dots, \ket{\psi_m}$ sono vettori che stanno in uno spazio di $n$-dimensioni, assumiamo anche che  $\{\ket{\psi_1}, \dots, \ket{\psi_m}\}$ è un insieme ortonormale. Gli insiemi ortonormali sono sempre insiemi lineari indipendenti (ciò implica che non possiamo esprimere nessuno dei vettori dell'insieme come combinazione lineare degli altri), quindi questi vettori attraversano necessariamente un sotto-spazio di dimensione $m$. Da qui possiamo concludere che $m \leq n$ perché la dimensione del sotto-spazio attraversato da questi vettori non può essere maggiore della dimensione dell'intero spazio da cui sono disegnati.
Se ci troviamo nel caso che $m < n$, significa che il sotto-spazio generato da $\{\ket{\psi_1}, \dots, \ket{\psi_m}\}$ non coincide con tutto lo spazio $n$-dimensionale. In questo caso, è sempre possibile estendere l'insieme ortonormale aggiungendo $n - m$ nuovi vettori ortonormali, $\{\ket{\psi_{m+1}}, \dots, \ket{\psi_n}\}$ fino ad ottenere una base ortonormale completa per l'intero spazio $n$-dimensionale. La procedura è conosciuta come *ortogonalizzazione di Gram-Schmidt* che può essere usata per costruire vettori.

### Insiemi ortonormali e matrici unitarie
Insiemi di vettori ortonormali sono strettamente connessi con le matrici unitarie. Un modo per esprimere questa connessione è dire che le seguenti 3 definizioni sono logicamente equivalenti (il che significa che sono tutte vere o tutte false) per qualsiasi scelta di una matrice quadrata $U$: 
1. La matrice $U$ è unitaria ($U^{\dagger}U = \mathbb{I} = UU^{\dagger}$);
2. Le righe di $U$ formano un insieme ortonormale;
3. Le colonne di $U$ formano un insieme ortonormale;
Questa equivalenza è in realtà abbastanza semplice quando pensiamo a come funzionano la moltiplicazione della matrice e la trasposizione coniugata. Supponiamo, per esempio, di avere una matrice $3\times3$ come questa:
$$U = \begin{pmatrix}
\alpha_{1, 1} & \alpha_{1, 2} & \alpha_{1, 3} \\
\alpha_{2, 1} & \alpha_{2, 2} & \alpha_{2, 3} \\
\alpha_{3, 1} & \alpha_{3, 2} & \alpha_{3, 3} \\
\end{pmatrix}$$
La trasposizione coniugata di $U$ ha questo aspetto:
$$
U^{\dagger} = \begin{pmatrix}
\overline{\alpha_{1, 1}} & \overline{\alpha_{2, 1}} & \overline{\alpha_{3, 1}} \\
\overline{\alpha_{1, 2}} & \overline{\alpha_{2, 2}} & \overline{\alpha_{3, 2}} \\
\overline{\alpha_{1, 3}} & \overline{\alpha_{2, 3}} & \overline{\alpha_{3, 3}} \\
\end{pmatrix}
$$
Moltiplicando le due matrici, con la trasposizione coniugata sul lato sinistro, ci dà questa matrice:
$$
\begin{pmatrix}
\overline{\alpha_{1, 1}} & \overline{\alpha_{2, 1}} & \overline{\alpha_{3, 1}} \\
\overline{\alpha_{1, 2}} & \overline{\alpha_{2, 2}} & \overline{\alpha_{3, 2}} \\
\overline{\alpha_{1, 3}} & \overline{\alpha_{2, 3}} & \overline{\alpha_{3, 3}} \\
\end{pmatrix}
\begin{pmatrix}
\alpha_{1, 1} & \alpha_{1, 2} & \alpha_{1, 3} \\
\alpha_{2, 1} & \alpha_{2, 2} & \alpha_{2, 3} \\
\alpha_{3, 1} & \alpha_{3, 2} & \alpha_{3, 3} \\
\end{pmatrix}
=
\begin{pmatrix}
\overline{\alpha_{1, 1}}\alpha_{1, 1} + \overline{\alpha_{2, 1}}\alpha_{2, 1} + \overline{\alpha_{3, 1}}\alpha_{3, 1} & \overline{\alpha_{1, 1}}\alpha_{1, 2} + \overline{\alpha_{2, 1}}\alpha_{2, 2} + \overline{\alpha_{3, 1}}\alpha_{3, 2} &  \overline{\alpha_{1, 1}}\alpha_{1, 3} + \overline{\alpha_{2, 1}}\alpha_{2, 3} + \overline{\alpha_{3, 1}}\alpha_{3, 3} \\
\overline{\alpha_{1, 2}}\alpha_{1, 1} + \overline{\alpha_{2, 2}}\alpha_{2, 1} + \overline{\alpha_{3, 2}}\alpha_{3, 1} & \overline{\alpha_{1, 2}}\alpha_{1, 2} + \overline{\alpha_{2, 2}}\alpha_{2, 2} + \overline{\alpha_{3, 2}}\alpha_{3, 2} & \overline{\alpha_{1, 2}}\alpha_{1, 3} + \overline{\alpha_{2, 2}}\alpha_{2, 3} + \overline{\alpha_{3, 2}}\alpha_{3, 3} \\ 
\overline{\alpha_{1, 3}}\alpha_{1, 1} + \overline{\alpha_{3, 2}}\alpha_{2, 1} + \overline{\alpha_{3, 3}}\alpha_{3, 1} & \overline{\alpha_{1, 3}}\alpha_{1, 2} + \overline{\alpha_{2, 3}}\alpha_{2, 2} + \overline{\alpha_{3, 3}}\alpha_{3, 2} & \overline{\alpha_{1, 3}}\alpha_{1, 3} + \overline{\alpha_{2, 3}}\alpha_{2, 3} + \overline{\alpha_{3, 3}}\alpha_{3, 3}
\end{pmatrix}
$$
Se formiamo tre vettori dalle colonne di $U$:
$$
\ket{\psi_1} = \begin{pmatrix} \alpha_{1,1} \\ \alpha_{2,1} \\ \alpha_{3,1}\end{pmatrix}, \quad \ket{\psi_2} = \begin{pmatrix} \alpha_{1,2} \\ \alpha_{2,2} \\ \alpha_{3,2}\end{pmatrix}, \quad \ket{\psi_3} = \begin{pmatrix} \alpha_{1,3} \\ \alpha_{2,3} \\ \alpha_{3,3}\end{pmatrix},
$$
allora possiamo esprimere in alternativa il prodotto sopra come segue:
$$
U^{\dagger}U = \begin{pmatrix} \braket{\psi_1|\psi_1} & \braket{\psi_1|\psi_2} & \braket{\psi_1|\psi_3} \\ 
\braket{\psi_2|\psi_1} & \braket{\psi_2|\psi_2} & \braket{\psi_2|\psi_3}  \\
\braket{\psi_3|\psi_1} & \braket{\psi_3|\psi_2} & \braket{\psi_3|\psi_3} 
\end{pmatrix}
$$
Facendo riferimento all'equazione (3), vediamo ora che la condizione che questa matrice sia uguale alla matrice identità è equivalente all'ortonormalità dell'insieme $\{\ket{\psi_1}, \ket{\psi_2}, \ket{\psi_3}\}$.

Questo argomento si generalizza alle matrici unitarie di qualsiasi dimensione. Il fatto che le righe di una matrice formino una base ortonormale se e solo se la matrice è unitaria quindi deriva dal fatto che una matrice è unitaria se e solo se la sua trasposizione è unitaria.

Data l'equivalenza sopra descritta, insieme al fatto che ogni insieme ortonormale può essere esteso per formare una base ortonormale, concludiamo il seguente fatto utile: Dato un qualsiasi insieme ortonormale di vettori $\{\ket{\psi_1}, \dots, \ket{\psi_m}\}$ tratto da uno spazio di $n$-dimensioni, esiste una matrice unitaria $U$ di cui le prime colonne $m$ sono vettori $\ket{\psi_1}, \dots, \ket{\psi_m}$. Pittoricamente, possiamo sempre trovare una matrice unitaria che abbia questa forma:
$$
U = \begin{pmatrix} \vert & \vert & & \vert & \vert & & \vert \\
\ket{\psi_1} & \ket{\psi_2} & \dots & \ket{\psi_m} & \ket{\psi_{m+1}} & \dots & \ket{\psi_n} \\
\vert & \vert & & \vert & \vert & & \vert
\end{pmatrix}.
$$
Qui, l'ultime colonne $n - m$ sono riempite con qualsiasi scelta dei vettori $\ket{\psi_{m+1}}, \dots, \ket{\psi_n}$ che creano $\{\ket{\psi_1}, \dots, \ket{\psi_m}\}$ una base ortonormale.
## Proiezioni e misurazioni proiettive
### Matrici di proiezione
Una matrice quadrata $\Pi$ è chiamata una *proiezione* se soddisfa due proprietà:
1. $\Pi = \Pi^{\dagger}$.
2. $\Pi^2 = \Pi$.
Le matrici che soddisfano la prima condizione (che sono uguali alla loro trasposizione congiunta) sono chiamate *Matrici Hermitian*, e le matrici che soddisfano la seconda condizione (che "quadrarli" li lascia invariati) sono chiamate matrici *idempotenti*.

>Il termine proiezione è talvolta usato per riferirsi a qualsiasi matrice che soddisfi solo la seconda condizione ma non necessariamente la prima, e quando ciò avviene, il termine proiezione ortogonale è tipicamente usato per riferirsi a matrici che soddisfano entrambe le proprietà. In questa serie, tuttavia, useremo i termini proiezione e matrice di proiezione per indicare matrici che soddisfano entrambe le condizioni.

Un esempio di proiezione è la matrice:
$$\Pi = \ket{\psi}\bra{\psi} \tag{4}$$
per qualsiasi vettore unitario $\ket{\psi}$. Possiamo vedere che questa matrice è *Hermitian* come segue:
$$
\Pi^{\dagger} = (\ket{\psi}\bra{\psi})^{\dagger} = (\bra{\psi})^{\dagger}(\ket{\psi})^{\dagger} = \ket{\psi}\bra{\psi} = \Pi
$$
Per vedere che la matrice $\Pi$ del punto (4) è idempotente, possiamo assumere che $\ket{\psi}$ è un vettore unitario, in modo che soddisfi $\braket{\psi|\psi} = 1$. Quindi abbiamo:
$$
\Pi^2 = (\ket{\psi}\bra{\psi})^2 = \ket{\psi}\braket{\psi|\psi}\bra{\psi} = \ket{\psi}\bra{\psi} = \Pi.
$$
Più in generale, se $\{\ket{\psi_1}, \dots, \ket{\psi_m}\}$ è un qualunque insieme ortonormale di vettori, allora la matrice
$$
\Pi = \sum_{k = 1}^m \ket{\psi_k}\bra{\psi_k} \tag{5}
$$
è una proiezione. Nello specifico, abbiamo
$$
\Pi^{\dagger} = \left( \sum_{k = 1}^m \ket{\psi_k}\bra{\psi_k} \right)^{\dagger} = \sum_{k = 1}^m (\ket{\psi_k}\bra{\psi_k})^{\dagger} = \sum_{k = 1}^m \ket{\psi_k}\bra{\psi_k} = \Pi,
$$
e
$$
\Pi^2 = \left( \sum_{j = 1}^m \ket{\psi_j}\bra{\psi_j} \right) \left( \sum_{k = 1}^m \ket{\psi_k}\bra{\psi_k} \right) = \sum_{j = 1}^m \sum_{k = 1}^m \ket{\psi_j}\braket{\psi_j|\psi_k}\bra{\psi_k} = \sum_{k = 1}^m \ket{\psi_k}\bra{\psi_k} = \Pi,
$$
dove l'ortonormalità di $\{\ket{\psi_1}, \dots, \ket{\psi_m}\}$ è usata solo per la penultima ugualianza.
Infatti, questo esaurisce tutte le possibilità: tutte le proiezioni $\Pi$ possono essere scritte nella forma (5) per qualche scelta di un insieme ortonormale $\{\ket{\psi_1}, \dots, \ket{\psi_m}\}$.
### Misurazioni proiettive
Come è già stato detto, la nozione di una misurazione di un sistema quantistico è più generale delle sole misurazioni di base standard. Le misurazioni proiettive sono misurazioni descritte da una raccolta di proiezioni la cui somma è uguale alla matrice di identità. In simboli, una collezione $\{\Pi_0, \dots, \Pi_{m-1}\}$ di matrici di proiezione descrivono una misurazione proiettiva se
$$
\Pi_0 + \dots + \Pi_{m-1} = \mathbb{I}.
$$
Quando tale misurazione viene eseguita su un sistema X quando è in un certo stato $\ket{\psi}$,  due cose accadono:
1. Per ogni $k \in \{0, \dots, m-1\}$, il risultato della misurazione è $k$  con probabilità pari $$\text{Pr(risultato è}\; k) = ||\Pi_k\ket{\psi}||^2.$$
2. Per qualunque risultato $k$ che la misurazione produce, lo stato di X diventa $$\frac{\Pi_k\ket{\psi}}{||\Pi_k\ket{\psi}||}.$$
Possiamo anche scegliere risultati diversi da $\{0, \dots, m-1\}$ per misurazioni proiettive se lo vogliamo. In generale, per qualsiasi insieme finito e non vuoto $\Sigma$, se abbiamo una collezione di matrici di proiezione $\{\Pi_a:a \in \Sigma\}$ che soddisfa la condizione $$\sum_{a\in\Sigma}\Pi = \mathbb{I},$$quindi questa raccolta descrive una misurazione proiettiva i cui possibili risultati coincidono con l'insieme $\Sigma$, dove le regole sono le stesse di prima (1. e 2.).
Ad esempio, le misurazioni di base standard sono equivalenti alle misurazioni proiettive, dove $\Sigma$ è l'insieme degli stati classici di qualunque sistema X di cui stiamo parlando e il nostro insieme di matrici di proiezioni è $\{\ket{a}\bra{a} : a \in \Sigma\}$.
Un altro esmepio di una misurazione proiettiva, questa volta con due qubits (X, Y), dati dall'insieme $\{\Pi_0, \Pi_1\}$, dove $$\Pi_0 = \ket{\phi^+}\bra{\phi^+} + \ket{\phi^-}\bra{\phi^-} + \ket{\psi^+}\bra{\psi^+} \quad \text{e} \quad \Pi_1 = \ket{\psi^-}\bra{\psi^-}.$$Se abbiamo più sistemi che sono congiuntamente in qualche stato quantistico e una misurazione proiettiva viene eseguita su uno solo dei sistemi, l'azione è simile a quella che avevamo per le misurazioni su base standard - e infatti ora possiamo descrivere questa azione in termini molto più semplici di quanto non potessimo prima.
Per essere precisi, supponiamo che abbiamo due sistemi (X, Y) in uno stato quantico $\ket{\psi}$, e una misurazione proiettiva descritta da una collezione $\{\Pi_a : a \in \Sigma\}$ viene eseguita sul sistema X, quando non viene fatto nulla a Y. Fare questo è l'equivalente di eseguire la misurazione proiettiva descritta dalla collezione $\{\Pi_a \otimes \mathbb{I}: a \in \Sigma\}$ sul sistema congiunto (X, Y). Ogni risultato della misurazione $a$ esce con probabilità $||(\Pi_a \otimes \mathbb{I})\ket{\psi}||^2$, e condizionato alla comparsa del risultato $a$, lo stato del sistema congiunto (X, Y) diventa $$\frac{(\Pi_a \otimes \mathbb{I})\ket{\psi}}{||(\Pi_a \otimes \mathbb{I}) \ket{\psi}||}.$$
# Limitazioni sulle informazioni quantistiche
Nonostante condividano una struttura matematica sottostante comune, le informazioni quantistiche e classiche hanno alcune differenze chiave. Mentre continuiamo in questa serie, vedremo molti esempi di compiti che l'informazione quantistica consente, ma l'informazione classica no.

Prima di farlo, tuttavia, dovremmo notare alcune importanti limitazioni alle informazioni quantistiche. Capire le cose che le informazioni quantistiche non possono fare ci aiuta a identificare le cose che possono fare.

## Irrilevanza delle fasi globali
La prima limitazione che tratteremo - che è in realtà più una leggera degenerazione nel modo in cui gli stati quantistici sono rappresentati da vettori di stato quantistico, al contrario di una limitazione effettiva - riguarda la nozione di una fase globale.

Quello che intendiamo per fase globale è questo. Supponiamo che $\ket{\psi}$ e $\ket{\phi}$ sono vettori unitari che rappresentano stati quantistici di qualche sistema, e assumiamo inoltre che esiste un numero complesso $\alpha$ sul cerchio unitario (cioè che $|\alpha| = 1$, o alternativamente $\alpha = e^{i\theta}$ per un qualche numero reale $\theta$) tale che $\ket{\phi} = \alpha\ket{\psi}$ (il **cerchio unitario** si riferisce al cerchio centrato nell'origine del piano cartesiano (0,0) con raggio pari a 1).
I vettori $\ket{\psi}$ e $\ket{\phi}$ sono quindi detti che **differiscono di una fase globale**. Ci possiamo anche riferire ad $\alpha$ come fase globale, anche se questo dipende dal contesto: qualsiasi numero sul cerchio unitario può essere pensato come una fase globale quando moltiplicato per un vettore unitario.



Ora considera cosa succede quando un sistema si trova in uno dei due stati quantistici che differiscono per una fase globale, $\ket{\psi}$ e $\ket{\phi}$, e il sistema subisce una misurazione di base standard. Nel primo caso, quando il sistema è nello stato $\ket{\psi}$, la probabilità di misurare un qualsiasi stato classico $a$ è $|\braket{a|\psi}|^2$.
Nel secondo caso, dove il sistema si trova nello stato $\ket{\phi}$, la probabilità di misurare un qualsiasi stato $a$ è $|\braket{a|\phi}|^2 = |\alpha\braket{a|\psi}|^2 = |\alpha|^2|\braket{a|\psi}^2| = |\braket{a|\psi}|^2$, perché $|a| = 1$. Cioè, la probabilità che appaia un risultato è la stessa per entrambi gli stati (questo perché la fase globale non ha effetti osservabili nei calcoli delle probabilità).
Consideriamo cosa accade quando applichiamo un arbitrario operatore unitario $U$ ad entrambi gli stati. Nel primo caso, dove lo stato iniziale è $\ket{\psi}$, lo stato diventa $U\ket{\psi}$, e nel secondo caso, dove lo stato iniziale è $\bra{\phi}$, diventa $U\ket{\phi} = \alpha U\ket{\psi}$.
Cioè, i due stati risultanti differiscono ancora per la stessa fase globale $\alpha$.

Di conseguenza, i due stati quantistici $\ket{\psi}$ e $\ket{\phi}$ che differiscono di una fase globale sono completamente indistinguibili: non importa quale operazione, o sequenza di operazioni, applichiamo ai due stati, differiranno sempre di una fase globale e l'esecuzione di una misurazione di base standard produrrà risultati con esattamente le stesse probabilità dell'altro. 
>Per questo motivo, due vettori di stato quantistico che differiscono per una fase globale sono considerati equivalenti e sono effettivamente visti come lo stesso stato.

Per esempio, gli stati quantistici
$$
\ket{-} = \frac{1}{\sqrt{2}}\ket{0} - \frac{1}{\sqrt{2}}\ket{1} \quad \text{e} \quad -\ket{-} = -\frac{1}{\sqrt{2}}\ket{0} + \frac{1}{\sqrt{2}}\ket{1}
$$
differenziano di una fase globale (che è -1 in questo esempio), e sono quindi considerati lo stesso stato.
Nell'altro caso, gli stati quantistici
$$
\ket{+} = \frac{1}{\sqrt{2}}\ket{0} + \frac{1}{\sqrt{2}}\ket{1} \quad \text{e} \quad \ket{-} = \frac{1}{\sqrt{2}}\ket{0} - \frac{1}{\sqrt{2}}\ket{1}
$$
non differiscono di una fase globale. Anche se l'unica differenza tra gli stati è che c'è un segno più che diventa un segno meno, questa non è una differenza di fase globale, è una differenza di fase relativa perché non influisce su ogni vettore di entrata, ma solo su un sottoinsieme adeguato delle entrate. Questo è coerente con ciò che abbiamo già osservato nella Lezione 1, dove gli stati $\ket{+}$ e $\ket{-}$ eseguono un'operazione di *Hadamard* e poi la misurazione produce i risultati con probabilità come segue:
$$
\begin{array}{cc}
|\braket{0|H|+}|^2 = 1 \quad & \quad |\braket{0|H|-}|^2 = 0 \\
|\braket{1|H|+}|^2 = 0 \quad & \quad |\braket{1|H|-}|^2 = 1
\end{array}
$$
Per inciso, qui troviamo un altro vantaggio della descrizione generale delle informazioni quantistiche basata sulle matrici di densità rispetto alla descrizione semplificata basata sui vettori di stato quantistici. Nella descrizione generale delle informazioni quantistiche, la degenerazione in cui due vettori di stato quantistici possono differire per una fase globale e quindi rappresentare effettivamente lo stesso stato quantistico, scompare. Vale a dire, due matrici di densità che differiscono rappresentano necessariamente due stati quantistici distinti che possono essere discriminati in senso statistico.
## Teorema di Non-Clonazione
Il teorema di *no-cloning* mostra che è impossibile creare una copia perfetta di uno stato quantistico sconosciuto.

>**Teorema di Non-clonazione**. Siano X e Y due sistemi quantistici. Supponiamo che il sistema X si trovi in un stato quantistico arbitrario $\ket{\psi}$, che vogliamo copiare nel sistema Y. Il teorema afferma che **non esiste un'operazione unitaria $U$** che soddisfi $U(\ket{\psi} \otimes \ket{\phi}) = \ket{\psi} \otimes \ket{\psi}$ per ogni possibile stato $\ket{\psi}$ di X e per uno stato arbitrario $\ket{\phi}$ di Y.

In altre parole, non possiamo costruire una macchina che prenda come input uno stato $\ket{\psi}$ e produca come output due copie identiche $\ket{\psi} \otimes \ket{\psi}$.

La dimostrazione di questo teorema è in realtà piuttosto semplice: si riduce all'osservazione che la mappatura $\ket{\psi} \otimes \ket{\phi} \mapsto \ket{\psi} \otimes \ket{\psi}$ non è lineare in $\ket{\psi}$.
In particolare, perché $\Sigma$ ha almeno due elementi, possiamo scegliere $a, b \in \Sigma$ con $a \neq b$. Se esistesse un stato quantistico $\ket{\phi}$ di Y e un'operazione unitaria $U$ sulla coppia (X, Y) dove $U(\ket{\psi} \otimes \ket{\phi}) = \ket{\psi} \otimes \ket{\psi}$ per ogni stato quantistico $\ket{\psi}$ di x, allora sarebbe il caso che 
$$
U(\ket{a} \otimes \ket{\phi}) = \ket{a} \otimes \bra{a} \quad \text{e} \quad U(\ket{b} \otimes \ket{\phi}) = \ket{b} \otimes \ket{b}.
$$
Per linearità, intendendo specificamente la linearità del prodotto tensoriale nel primo argomento e la linearità della moltiplicazione matrice-vettore nel secondo argomento (vettore), dobbiamo quindi avere
$$
U\left(\left(\frac{1}{\sqrt 2}\ket{a} + \frac{1}{\sqrt 2}\ket{b}\right) \otimes \ket{\phi}\right) = \frac{1}{\sqrt 2}\ket{a} \otimes \ket{a} \; + \frac{1}{\sqrt 2}\ket{b} \otimes \ket{b}.
$$
Ad ogni caso, la richiesta che $U(\ket{\psi} \otimes \ket{\phi}) = \ket{\psi} \otimes \ket{\psi}$ per ogni stato quantistico $\ket{\psi}$ demanda che
$$
\begin{align*}
U\left(\left(\frac{1}{\sqrt{2}}\ket{a} + \frac{1}{\sqrt{2}}\ket{b}\right) \otimes \ket{\phi}\right) &= \left( \frac{1}{\sqrt{2}}\ket{a} + \frac{1}{\sqrt{2}}\ket{b} \right) \otimes \left(\frac{1}{\sqrt{2}}\ket{a} + \frac{1}{\sqrt{2}}\ket{b}\right) \\ &= \frac{1}{2}\ket{a} \otimes \ket{a} + \frac{1}{2}\ket{a} \otimes \ket{b} + \frac{1}{2}\ket{b} \otimes \ket{a} + \frac{1}{2}\ket{b} \otimes \ket{b} \\ &\neq \frac{1}{\sqrt{2}}\ket{a} \otimes \ket{a} + \frac{1}{\sqrt{2}}\ket{b} \otimes \ket{b}. 
\end{align*}
$$
Quindi non può esistere uno stato $\ket{\phi}$ e una operazione unitaria $U$ dove $U(\ket{\psi} \otimes \ket{\phi}) = \ket{\psi} \otimes \ket{\psi}$ per ogni vettore di stato quantistico $\ket{\psi}$.

Sono necessarie alcune osservazioni sul teorema di non-clonazione. La prima è che l'affermazione del teorema di non-clonazione descritta sopra è assoluta, nel senso che afferma che la clonazione perfetta è impossibile, ma non dice nulla sulla possibile clonazione con accuratezza limitata, in cui potremmo riuscire a produrre un clone approssimativo (rispetto a un modo di misurare quanto simili potrebbero essere due stati quantici diversi). Ci sono, infatti, affermazioni del teorema di non-clonazione che pongono limitazioni alla clonazione approssimativa, così come metodi per ottenere la clonazione approssimativa (con accuratezza limitata), ma rimanderemo questa discussione a una lezione successiva quando saranno a posto gli elementi necessari per spiegare la clonazione approssimativa.

La seconda osservazione è che il teorema di non clonazione è un'affermazione sull'impossibilità di clonare uno stato arbitrario $\ket{\psi}$. Possiamo facilmente creare un clone di qualsiasi stato base standard. Ad esempio, possiamo clonare uno stato base standard di qubit utilizzando un'operazione NOT controllata:

![[clone.png]]

Sebbene non vi siano difficoltà nel creare un clone di uno stato base standard, ciò non contraddice il teorema di non clonazione: questo approccio che prevede l'uso di una porta NOT controllata non riuscirebbe a creare un clone dello stato $\ket{+}$, ad esempio.

Un'ultima osservazione sul teorema di non clonazione è che non è davvero esclusivo delle informazioni quantistiche, è anche impossibile clonare uno stato probabilistico arbitrario usando un processo classico (deterministico o probabilistico). Questo è abbastanza intuitivo. Immagina che qualcuno ti dia un sistema in uno stato probabilistico, ma non sei sicuro di quale sia quello stato probabilistico. Ad esempio, forse hanno generato casualmente un numero tra 1 e 10, ma non ti hanno detto come hanno generato quel numero. Non esiste certamente alcun processo fisico attraverso il quale puoi ottenere due copie indipendenti dello stesso stato probabilistico: tutto ciò che hai tra le mani è un numero tra 1 e 10, e non ci sono abbastanza informazioni presenti per ricostruire in qualche modo le probabilità che appaiano tutti gli altri risultati. Matematicamente parlando, una versione del teorema di non-clonazione per gli stati probabilistici può essere dimostrata esattamente nello stesso modo del normale teorema di non-clonazione (per gli stati quantistici). Vale a dire, la clonazione di uno stato probabilistico arbitrario è un processo non lineare, quindi non può essere rappresentato da una matrice stocastica.

## Gli stati non ortogonali non possono essere discriminati perfettamente
Per l'ultima limitazione trattata in questa lezione, mostreremo che se abbiamo due stati quantistici $\ket{\psi}$ e $\ket{\phi}$ che non sono ortogonali, il che significa che $\braket{\phi | \psi} = 0$, allora è impossibile discriminarli (o, in altre parole, distinguerli) perfettamente.

>Infatti, mostreremo qualcosa di logicamente equivalente: che è possibile distinguere perfettamente due stati quantistici se e solo se sono **ortogonali**.

Limiteremo la nostra attenzione ai circuiti quantistici che consistono in un numero qualsiasi di porte unitarie, seguite da una singola misurazione di base standard del qubit superiore. Ciò che richiediamo a un circuito quantistico, per dire che discrimina perfettamente gli stati $\ket{\psi}$ e $\ket{\phi}$, è che la misurazione fornisca sempre il valore 0 per uno dei due stati e fornisca sempre 1 per l'altro stato. Per essere precisi, assumeremo di avere un circuito quantistico che funziona come suggeriscono i seguenti diagrammi:

![[discriminazione.png]]

La box chiamata $U$ denota l'operazione unitaria che rappresenta l'azione combinata di tutte le porte unitarie nel nostro circuito, ma non include la misurazione finale. Non vi è alcuna perdita di generalità nell'assumere che le uscite di misurazione 0 per $\ket{\psi}$ e 1 per $\ket{\phi}$; l'analisi non cambierebbe sostanzialmente se questi valori di output fossero invertiti.

Si noti che oltre ai qubit che inizialmente memorizzano $\ket{\psi}$ e $\ket{\phi}$, il circuito è libero di utilizzare qualsiasi numero di qubit di workspace aggiuntivi. Questi qubit sono inizialmente impostati ciascuno sullo stato $\ket{0}$, quindi il loro stato combinato è indicato con $\ket{0 \dots 0}$ nelle figure, e questi qubit possono essere utilizzati dal circuito in qualsiasi modo possa essere utile. È molto comune utilizzare i qubit di workspace in circuiti quantistici come questo, come vedremo nella prossima unità.

Ora, considera cosa succede quando eseguiamo il nostro circuito sullo stato $\ket{\psi}$ (insieme ai qubit inizializzati dello spazio di lavoro). Lo stato risultante, **immediatamente prima che venga eseguita la misurazione**, può essere scritto come
$$
U(\ket{0\dots0}\ket{\psi}) = \ket{\gamma_0}\ket{0} + \ket{\gamma_1}\ket{1}
$$
per i due vettori $\ket{\gamma_0}$ e $\ket{\gamma_1}$ che corrispondono a tutti i qubit eccetto il qubit superiore. In generale, per tale stato le probabilità che una misurazione del qubit superiore produca i risultati 0 e 1 sono le seguenti:
$$
\text{Pr(risultato è 0)} = ||\ket{\gamma_0}||^2 \quad \text{e} \quad \text{Pr(risultato è 1)} = ||\ket{\gamma_1}||^2.
$$
Quando il circuito processa $\ket{\psi}$, il risultato della misurazione è sempre 0, questo si traduce nell'uguaglianza$$
U(\ket{0\dots0}\ket{\psi}) = \ket{\gamma_0}\ket{0},
$$dove $\ket{\gamma_0}$ è uno stato di qubit ausiliari.
Moltiplicando entrambi i lati di questa equazione per $U^{\dagger}$, per invertire l'operazione unitaria $U$ e tornare agli stati iniziali, si applica la sua trasposta coniugata, ottenendo questa equazione:
$$
\ket{0\dots0}\ket{\psi} = U^{\dagger}(\ket{\gamma_0}\ket{0}). \tag{7}
$$

Ragionando in modo simile per $\ket{\phi}$ al posto di $\ket{\psi}$, concludiamo che
$$
U(\ket{0\dots0}\ket{\phi}) = \ket{\delta_1}\ket{1}.
$$
per qualche vettore $\ket{\delta_1}$, e quindi
$$
\ket{0\dots0}\ket{\phi} = U^{\dagger}(\ket{\delta_1}\ket{1}). \tag{8}
$$

Ora ci interessa calcolare il prodotto interno dei vettori rappresentati dalle equazioni (7) e (8) ($\braket{0\dots0\psi|0\dots0\phi}$), ma dato che $\ket{0\dots0}\ket{\psi}$ non è nella forma di cui abbiamo bisogno ($\bra{0\dots0\psi}$) dobbiamo fare la trasposizione per trasformare in ket in un bra. A questo punto prendiamo la parte equivalente destra dell'equazione (7) e facciamo la trasposizione:
$$
(U^{\dagger}(\ket{\gamma_0}\ket{0}))^{\dagger} = (\bra{\gamma_0}\bra{0})U, \tag{9}
$$
ora possiamo formare il prodotto interno del vettore (9) (cioè il (7) trasformato) con il vettore (8) che troviamo
$$
(\bra{\gamma_0}\bra{0})UU^{\dagger}(\ket{\delta}\ket{1}) = (\bra{\gamma_0}\bra{0})\,(\ket{\delta_1}\ket{1}) = \braket{\gamma_0|\delta_1}\braket{0|1} = 0.
$$
Qui abbiamo usato il fatto che $UU^{\dagger} = \mathbb{I}$, così come il fatto che il prodotto interno dei prodotti tensoriali è il prodotto dei prodotti interni:
$$
\braket{u \otimes v| w \otimes x} = \braket{u|w}\braket{v|x} \tag{10}
$$
per qualsiasi scelta di questi vettori (supponendo che $\ket{u}$ e $\ket{w}$ abbiano lo stesso numero di voci e $\ket{v}$ e $\ket{x}$ abbiano lo stesso numero di voci, così che abbia senso formare i prodotti interni $\braket{u|w}$ e $\braket{v|x}$). **Nota che il valore del prodotto interno $\braket{\gamma_0|\delta_1}$ è irrilevante perché è moltiplicato per $\braket{0|1} = 0$.** Questo è un bene perché in realtà non sappiamo molto di questi due vettori.

Infine, prendendo il prodotto interno dei vettori (7) e (8) in termini del lato sinistro delle equazioni deve risultare nello stesso valore zero, e quindi
$$
0 = (\ket{0\dots0}\ket{\psi})^{\dagger}(\ket{0\dots0}\ket{\phi}) = \braket{0\dots0|0\dots0}\braket{\psi|\phi} = \braket{\psi|\phi},
$$dato che la parte destra abbiamo trovato sia 0, l'abbiamo associata alla parte sinistra che ancora dovevamo spiegare.
Dunque abbiamo che, come prima, dobbiamo fare la trasposizione della parte sinistra del vettore (7) per trasformare il ket in bra e successivamente otteniamo con la regola (10) i braket 0 e di $\psi$ e $\phi$, infine dato che $\braket{0\dots0|0\dots0} = 1$ concludiamo ciò che volevamo, ovvero che $\ket{\psi}$ e $\ket{\phi}$ sono ortogonali: $\braket{\psi|\phi} = 0$.

**SPIEGAZIONE DETTAGLIATA**
![[distinzione-ortogonali-1.png]]
![[distinzione-ortogonali-2.png]]
![[distinzione-ortogonali-3.png]]

È possibile, tra l'altro, discriminare perfettamente due stati qualsiasi che siano ortogonali. Supponiamo che i due stati da discriminare siano $\ket{\phi}$ e $\ket{\psi}$, dove $\braket{\phi|\psi} = 0$. Possiamo quindi discriminare perfettamente questi stati eseguendo la misurazione proiettiva descritta da queste matrici, ad esempio:
$$
\{\ket{\phi}\bra{\phi}, \mathbb{I} - \ket{\phi}\bra{\phi}\}
$$
Per lo stato $\ket{\phi}$, si ottiene sempre il primo risultato:
$$ \begin{array}{c}
||\ket{\phi}\bra{\phi}\ket{\phi}||^2 = ||\ket{\phi}\braket{\phi|\phi}||^2 = ||\ket{\phi}||^2 = 1, \\
||(\mathbb{I}- \ket{\phi}\bra{\phi})\ket{\phi}||^2 = ||\ket{\phi} - \ket{\phi}\braket{\phi|\phi}||^2 = ||\ket{\phi}-\ket{\phi}||^2 = 0.
\end{array}
$$
E, per lo stato $\ket{\psi}$, si ottiene sempre il secondo risultato:
$$
 \begin{array}{c}
||\ket{\phi}\bra{\phi}\ket{\psi}||^2 = ||\ket{\phi}\braket{\phi|\psi}||^2 = ||\ket{0}||^2 = 0, \\
||(\mathbb{I}- \ket{\phi}\bra{\phi})\ket{\psi}||^2 = ||\ket{\psi} - \ket{\phi}\braket{\phi|\psi}||^2 = ||\ket{\psi}||^2 = 1.
\end{array}
$$
