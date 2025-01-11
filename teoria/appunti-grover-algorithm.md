```table-of-contents
```
# Introduzione
L'algoritmo di Grover, è un algoritmo quantistico per i cosiddetti problemi di ricerca non strutturati che offre un miglioramento quadratico rispetto agli algoritmi classici. Ciò significa che l'algoritmo di Grover richiede un numero di operazioni nell'ordine della radice quadrata del numero di operazioni richieste per risolvere la ricerca non strutturata in modo classico.

Sebbene l'ampia applicabilità della tecnica di ricerca di Grover sia convincente, si dovrebbe riconoscere che il vantaggio quadratico che offre sembra improbabile che porti a un vantaggio pratico del calcolo quantistico rispetto a quello classico in tempi brevi. L'hardware di calcolo classico è attualmente molto più avanzato dell'hardware di calcolo quantistico che il vantaggio quadratico quantistico-su-classico offerto dall'algoritmo di Grover è destinato a essere spazzato via dalle sbalorditive velocità di clock dei moderni computer classici per qualsiasi problema di ricerca non strutturato che potrebbe essere eseguito sui computer quantistici di oggi.

# Ricerca non strutturata
Descriviamo il problema che l'algoritmo di Grover risolve, come al solito abbiamo $\Sigma = \{0, 1\}$ che indica l'alfabeto binario.
Supponiamo che $f: \Sigma^n \rightarrow \Sigma$ è una funzione da stringhe binarie di lunghezza $n$ a bit. Quello che l'algoritmo di Grover fa è cercare una stringa $x \in \Sigma^n$ per cui $f(x) = 1$, dunque ci riferiamo a stringhe come queste come soluzione al problema di ricerca, se ci dovessero essere più soluzioni allora una qualsiasi di esse è considerata un output corretto, e se non ci sono soluzioni, allora una risposta corretta richiede che non ci sono soluzioni.

Descriviamo questo compito come un problema di ricerca non strutturato perché non possiamo contare sul fatto che $f$ abbia una struttura particolare per semplificarlo. Non stiamo cercando in un elenco ordinato o all'interno di una struttura dati specificamente progettata per facilitare la ricerca, stiamo essenzialmente cercando un ago in un pagliaio.

Un modo per eseguire questa attività di ricerca classicamente è semplicemente iterare attraverso tutte le stringhe $x \in \Sigma^n$ valutando $f$ su ognuna per trovare quale è o meno una soluzione. Di seguito, scriviamo $$N = 2^n$$solo per comodità, così possiamo dire che ci sono$N$ stringhe in $\Sigma^n$. Per procedere attraverso tutti questi passaggi sono necessarie $N$ valutazioni di $f$.
Operando sotto l'ipotesi che siamo limitati a valutare $f$ su input scelti, questo è il meglio che possiamo fare con un algoritmo deterministico se vogliamo garantire il successo. Con un algoritmo probabilistico, potremmo sperare di risparmiare tempo scegliendo casualmente stringhe di input per $f$, ma avremo comunque bisogno di valutazioni $O(N)$ di $f$ se vogliamo che questo metodo abbia successo con alta probabilità.

L'algoritmo di Grover risolve il problema di ricerca non strutturata descritto sopra con elevata probabilità e ha richiesto solo $O(\sqrt{N})$ valutazioni di $f$. L'algoritmo di Grover adotta un approccio iterativo: valuta $f$ su sovrapposizioni di stringhe di input e intercala queste valutazioni con altre operazioni che hanno l'effetto di creare pattern di interferenza, portando a una soluzione con elevata probabilità (se ne esiste una) dopo $O(\sqrt{N})$ iterazioni.

# Dichiarazione formale del problema
Formalizzeremo il problema che l'algoritmo di Grover risolve utilizzando il modello di query di calcolo. Vale a dire, assumeremo di avere accesso alla funzione $f:\Sigma^n \rightarrow \Sigma$ tramite un gate di query definito nel solito modo, che è come $$U_f(\ket{a}\ket{x}) = \ket{a\, \oplus \, f(x)}\ket{x}$$
per ogni $x \in \Sigma^n$ e $a \in \Sigma$. Questa è l'azione di $U_f$ sugli stati di base standard e la sua azione in generale è determinata dalla linearità.
Se abbiamo un circuito booleano per la computazione $f$, possiamo trasformarlo in un circuito quantistico implementando $U_f$ (Utilizzando un certo numero di qubit dello spazio di lavoro che iniziano e terminano il calcolo nello stato $\ket{0}$). Quindi, mentre utilizziamo il modello di query per formalizzare il problema risolto dall'algoritmo di Grover, non ci limitiamo a questo modello: possiamo eseguire l'algoritmo di Grover su qualsiasi funzione $f$ per la quale abbiamo un circuito booleano.

Ecco una formulazione precisa del problema, che si chiama Ricerca perché stiamo cercando una soluzione, ovvero una stringa $x$ che faccia sì che $f$ valga 1.

>**Ricerca**
>Input: una funzione $f: \Sigma^n \rightarrow \Sigma$ 
>Output: una stringa $x \in \Sigma^n$ che soddisfi $f(x)=1$, oppure "no soluzione" se non esistono stringhe $x$.

Si noti che questo non è un problema di promessa: la funzione $f$ è arbitraria. Sarà tuttavia utile considerare la seguente variante di promessa del problema, in cui ci è garantito che ci sia esattamente una soluzione. 

>**Ricerca unica**
>Input: una funzione della forma $f: \Sigma^n \rightarrow \Sigma$ 
>Promessa: esiste esattamente una stringa $z \in \Sigma^n$ per cui $f(z)=1$, con $f(x) = 0$ per tutte le stringhe $x \neq z$.
>Output: la stringa $z$

# Algoritmo di Grover
## Phase query gates
L'algoritmo di Grover fa uso di operazioni note come *phase query gates*. A differenza di un gate di query ordinario $U_f$, definito per una data funzione $f$ nel solito modo descritto sopra, un phase query gates per la funzione $f$ è definito come $$Z_f\ket{x} = (-1)^{f(x)}\ket{x}$$
per ogni stringa $x \in \Sigma^n$.
L'operazione $Z_f$ può essere implementata usando un query gate $U_f$ come il seguente diagramma suggerisce

![[phase_query_gates.png]]

Questa implementazione sfrutta il fenomeno del *phase kickback* e richiede che un qubit dello spazio di lavoro, inizializzato a uno stato $\ket{-}$, sia reso disponibile. Questo qubit rimane nello stato $\ket{-}$ dopo il completamento dell'implementazione e può essere riutilizzato (per implementare le porte $Z_f$ successive, ad esempio) o semplicemente scartato.

In oltre all'operazione $Z_f$, utilizzeremo anche una "porta di interrogazione di fase" per la funzione OR a $n$-bit, definita come segue per ogni stringa $x \in \Sigma^n$.
$$OR(x) = \begin{cases}0 & x = 0^n \\ 1 & x \neq 0^n \end{cases}$$
In modo esplicito, la porta di interrogazione di fase per la funzione OR a $n$-bit funziona in questo modo:
$$Z_{OR}\ket{x} = \begin{cases} \ket{x} & x = 0^n \\ -\ket{x} & x \neq 0^n \end{cases}$$
Per essere chiari, questo è il modo in cui $Z_{OR}$ opera su stati di base standard; il suo comportamento su stati arbitrari è determinato da questa espressione per linearità.

L'operazione $Z_{OR}$ può essere implementata come un circuito quantistico iniziando con un circuito booleano per la funzione OR, quindi costruendo un'operazione $U_{OR}$ (vale a dire, una query gate standard per la funzione OR a $n$-bit) e infine un'operazione $Z_{OR}$ utilizzando il fenomeno del *phase kickback* come descritto sopra.

Si noti che l'operazione $Z_{OR}$ non ha alcuna dipendenza dalla funzione $f$ e può quindi essere implementata da un circuito quantistico senza query gate.

## Descrizione dell'algoritmo
Ora che abbiamo due operazioni $Z_f$ e $Z_{OR}$, possiamo descrivere l'algoritmo di Grover.

L'algoritmo fa riferimento a un numero $t$, che è il numero di iterazioni che esegue, così come il numero di query alla funzione $f$ che richiede. Questo numero $t$ non è specificato dall'algoritmo di Grover (come lo stiamo descrivendo), e discuteremo nella sezione successiva a questa come può essere scelto.

>**Algoritmo di Grover**
>1. Inizializzare un registro $Q$ di $n$ qubit a tutti gli stati zero $\ket{0^n}$ e applicare l'operazione di Hadamard ad ogni qubit di $Q$.
>2. Applicare $t$ volte l'operazione unitaria $G = H^{\otimes n}Z_{OR}H^{\otimes n}Z_f$ al registro $Q$
>3. Misurare i qubits di $Q$ rispetto alle misurazione di base standard e restituisci la stringa risultante.

L'operazione $G = H^{\otimes n}Z_{OR}H^{\otimes n}Z_f$ iterata nel passaggio 2 sarà chiamato operazione di Grover per tutto il resto di questa lezione. Ecco una rappresentazione del circuito quantistico dell'operazione di Grover:

![[grover_circuit.png]]

Qui l'operazione $Z_f$ è raffigurata come più grande di $Z_{OR}$ per suggerire che è probabile che sia l'operazione più costosa (ma questo è solo un indizio visivo e non qualcosa con un significato formale). In particolare, quando lavoriamo all'interno del modello di query $Z_f$ richiede una query mentre $Z_{OR}$ non richiede query, e se invece abbiamo un circuito booleano per la funzione $f$ e lo convertiamo in un circuito quantistico per $Z_f$, possiamo ragionevolmente aspettarci che il circuito quantistico risultante sarà più grande e più complicato di uno per $Z_{OR}$.

Ecco una descrizione di un circuito quantistico per l'intero algoritmo quando $t=2$. Per valori maggiori di $t$ possiamo semplicemente inserire istanze aggiuntive dell'operazione Grover immediatamente prima delle misurazioni.

![[grover_cicuit_t2.png]]

## Applicazione per la ricerca
L'algoritmo di Grover può essere applicato al problema di ricerca descritto nella sezione precedente come segue:
- Scegli il numero $t$ nel passaggio 2. La sezione successiva a questa discute come possiamo scegliere $t$.
- Esegui l'algoritmo di Grover sulla funzione $f$, usando qualsiasi scelta fatta per $t$, per ottenere una stringa $x \in \Sigma^n$.
- Interroga la funzione $f$ sulla stringa $x$ per vedere se è una soluzione valida:
	- Se $f(x)=1$, allora abbiamo trovato una soluzione, quindi possiamo fermarci e produrre $x$.
	- Altrimenti, se $f(x)=0$, allora possiamo eseguire di nuovo la procedura, possibilmente con una scelta diversa per $t$, oppure possiamo decidere di rinunciare e produrre "nessuna soluzione".

Un po' più avanti, una volta analizzato il funzionamento dell'algoritmo di Grover, vedremo che prendendo $t = O(\sqrt{N})$ otterremo una soluzione al nostro problema di ricerca (se ne esiste una) con alta probabilità.

## Analisi
Ora analizzeremo l'algoritmo di Grover per capire come funziona. Inizieremo con quella che potrebbe essere descritta come un'analisi simbolica, in cui calcoliamo come l'operazione di Grover $G$ agisce su determinati stati, e poi collegheremo questa analisi simbolica a un'immagine geometrica utile per visualizzare come funziona l'algoritmo.
### Soluzione e Non soluzione
Iniziamo definendo due insiemi di stringhe.
$$\begin{matrix} A_0 = \{x \in \Sigma^n: f(x) = 0\} \\ A_1 = \{x \in \Sigma^n: f(x) = 1\}\end{matrix}$$
L'insieme $A_1$ contiene tutte le soluzioni del nostro problema di ricerca, e $A_0$ contiene tutte le stringhe che on sono soluzioni. Questi due insiemi soddisfano $A_0 \cap A_1 = \varnothing$ e $A_0 \cup A_1 = \Sigma^n$, il che significa che si tratta di una bipartizione di $\Sigma^n$.

Successivamente definiremo due vettori di unità che rappresentano sovrapposizioni uniformi sui set di soluzioni e non soluzioni.

$$\begin{equation*}
\begin{aligned}[c]
\ket{A_0} = \frac{1}{\sqrt{|A_0|}} \sum_{x \in A_0} \ket{x}\\
\ket{A_1} = \frac{1}{\sqrt{|A_1|}} \sum_{x \in A_1} \ket{x}
\end{aligned}
\end{equation*}$$
Formalmente parlando, ciascuno di questi vettori è definito solo quando il suo insieme corrispondente è non vuoto, ma di seguito ci concentreremo sul caso che né $A_0$ né $A_1$ sono vuoti. I casi che $A_0 = \varnothing$ e $A_1 = \varnothing$ sono facilmente gestiti separatamente e lo faremo più tardi.

A parte questo, questa notazione è abbastanza comune: ogni volta che abbiamo un set non vuoto $S$, possiamo scrivere $\ket{S}$ per indicare il vettore di stato quantico che è uniforme sugli elementi di $S$.

Definiamo anche $\ket{u}$ come uno stato quantico uniforme su tutte le stringhe $n$-bit:
$$\ket{u} = \frac{1}{\sqrt{N}} \sum_{x \in \Sigma^n} \ket{x}.$$
Notiamo che
$$\ket{u} = \sqrt{\frac{|A_0|}{N}}\ket{A_0} + \sqrt{\frac{|A_1|}{N}}\ket{A_1}.$$
Abbiamo anche $\ket{u} = H^{\otimes n}\ket{0^n}$, quindi $\ket{u}$ rappresenta lo stato del registro $Q$ dopo l'inizializzazione nel passaggio 1 dell'algoritmo di Grover. Ciò implica che poco prima che le iterazioni di $G$ avvengano nel passaggio 2, lo stato di $Q$ è contenuto nello spazio vettoriale bidimensionale attraversato da $\ket{A_0}$ e $\ket{A_1}$, e inoltre i coefficienti di questi vettori sono numeri reali.

Come vedremo, lo stato di $Q$ avrà sempre queste proprietà - il che significa che lo stato è una vera combinazione lineare di $\ket{A_0}$ e $\ket{A_1}$ - dopo qualsiasi numero di iterazioni dell'operazione $G$ nel passaggio 2.

### Un'osservazione dell'operazione di Grover
Ora prestiamo attenzione all'operazione di Grover
$$G = H^{\otimes n}Z_{OR}H^{\otimes n}Z_f$$
iniziamo con un'osservazione interessante.
Immagina per un momento che abbiamo sostituito la funzione $f$ dalla composizione di $f$ con la funzione NOT - o in altre parole la funzione che otteniamo lanciando il bit di uscita di $f$. Chiameremo questa nuova funzione $g$ e possiamo esprimerla usando i simboli in alcuni modi alternativi.
$$g(x) = \neg f(x)=1 \oplus f(x) = 1 - f(x) = \begin{cases} 1 & f(x) = 0 \\ 0 & f(x) = 1\end{cases}$$
Ora, notiamo che $$Z_f = -Z_g$$
Richiamando che $Z_f\ket{x} = (-1)^{f(x)}\ket{x}$ per ogni stringa $x \in \Sigma^n$, possiamo verificare questo osservando questo $$(-1)^{g(x)} = (-1)^{1\oplus f(x)} = -(-1)^{f(x)}$$
per ogni stringa $x \in \Sigma^n$.
Quindi, l'algoritmo di Grover si comporta esattamente lo stesso per $g$ come per $f.$ Intuitivamente parlando, all'algoritmo non importa davvero quali stringhe siano soluzioni-deve solo essere in grado di distinguere soluzioni e non soluzioni da operare come fa.

### Azione dell'operazione di Grover
Adesso consideriamo l'azione $G$ sui vettori $\ket{A_0}$ e $\ket{A_1}$.
Per prima cosa, osserviamo che l'operazione $Z_f$ ha un'azione molto semplice sui vettori $\ket{A_0}$ e $\ket{A_1}$. $$
\begin{equation*}
\begin{aligned}[c]
Z_f \ket{A_0} & = \ket{A_0}\\
Z_f \ket{A_1} & = -\ket{A_1}
\end{aligned}
\end{equation*}$$
Come seconda abbiamo l'operazione $H^{\otimes n}Z_{OR}H^{\otimes n}$. L'operazione $Z_{OR}$ è definita come $$Z_{OR}\ket{x} = \begin{cases} \ket{x} & x = 0^n \\ -\ket{x} & x \neq 0^n\end{cases}$$
ancora per ogni stringa $x \in \Sigma^n$, e un alternativa conveniente per esprimere questa operazione è $$Z_{OR}= 2\ket{0^n}\bra{0^n} - \mathbb{I}$$(Un modo semplice per verificare che l'espressione è corretta per la definizione di $Z_{OR}$ è valutare la sua azione su stati di base standard). L'operazione $H^{\otimes n}Z_{OR}H^{\otimes n}$ può, dunque, essere riscritta come segue: $$H^{\otimes n}Z_{OR}H^{\otimes n} = 2H^{\otimes n}\ket{0^n}\bra{0^n}H^{\otimes n} - \mathbb{I}$$
Usando la stessa notazione $\ket{u}$ che abbiamo usato sopra per la sovrapposizione uniforme su tutte le stringhe $n$-bit, possiamo alternativamente esprimere $H^{\otimes n}Z_{OR}H^{\otimes n}$ come: $$H^{\otimes n}Z_{OR}H^{\otimes n} = 2\ket{u}\bra{u}- \mathbb{I}$$![[applicazione_grover.jpg]]

Come abbiamo già notato, lo stato di $Q$ appena prima del passaggio 2 è contenuto nello spazio bidimensionale attraversato da $\ket{A_0}$ e $\ket{A_1}$, e abbiamo appena stabilito che $G$ mappa qualsiasi vettore in questo spazio a un altro vettore nello stesso spazio. Ciò significa che, per il bene dell'analisi, possiamo focalizzare la nostra attenzione esclusivamente su questo sotto-spazio.

Per comprendere meglio cosa sta accadendo all'interno di questo spazio bidimensionale, esprimiamo l'azione di $G$ su questo spazio come una matrice,
$$M = \begin{pmatrix} \frac{|A_0|-|A_1|}{N} & -\frac{2\sqrt{|A_0|\cdot|A_1|}}{N} \\ \frac{2\sqrt{|A_0|\cdot|A_1|}}{N} & \frac{|A_0|-|A_1|}{N}\end{pmatrix}$$
La cui prima e seconda riga/colonna corrispondono a $\ket{A_0}$ e $\ket{A_1}$, rispettivamente. 
Finora in questa serie abbiamo sempre collegato le righe e le colonne delle matrici con gli stati classici di un sistema, ma le matrici possono anche essere utilizzate per descrivere le azioni di mappature lineari su basi diverse come abbiamo qui.)

Sebbene non sia affatto ovvio a prima vista, la matrice $M$ è ciò che otteniamo elevando al quadrato una matrice dall'aspetto più semplice.
$$\begin{pmatrix} \sqrt{\frac{|A_0|}{N}} & -\sqrt{\frac{|A_1|}{N}} \\ \sqrt{\frac{|A_1|}{N}} & \sqrt{\frac{|A_0|}{N}}\end{pmatrix}^2 
=
\begin{pmatrix} \frac{|A_0|-|A_1|}{N} & -\frac{2\sqrt{|A_0|\cdot|A_1|}}{N} \\ \frac{2\sqrt{|A_0|\cdot|A_1|}}{N} & \frac{|A_0|-|A_1|}{N}\end{pmatrix} = M$$
La matrice $$\begin{pmatrix} \sqrt{\frac{|A_0|}{N}} & -\sqrt{\frac{|A_1|}{N}} \\ \sqrt{\frac{|A_1|}{N}} & \sqrt{\frac{|A_0|}{N}}\end{pmatrix}$$
è la matrice di rotazione, dove può essere espressa definitivamente come $$\begin{pmatrix} \cos(\theta) & -\sin(\theta) \\ \sin(\theta) & \cos(\theta)\end{pmatrix}$$
per $$\theta = \sin^{-1}\left(\sqrt{\frac{|A_1|}{N}}\right)$$
In sintesi, lo stato del registro $Q$ all'inizio dello step 2 è
$$\ket{u} = \sqrt{\frac{|A_0|}{N}}\ket{A_0} + \sqrt{\frac{|A_1|}{N}}\ket{A_1} = \cos(\theta)\ket{A_0}+\sin(\theta)\ket{A_1}$$
e l'effetto di applicare $G$ a questo stato è come ruotarlo di un angolo $2\theta$ all'interno dello spazio attraversato da $\ket{A_0}$ e $\ket{A_1}$.

### Figure geometriche
L'idea è che l'operazione $G$ sia il prodotto di due riflessioni, $Z_f$ e $H^{\otimes n}Z_{OR}H^{\otimes n}$. L'effetto netto dell'esecuzione di due riflessioni è quello di eseguire una rotazione.

Iniziamo con $Z_f$. Come abbiamo già visto, abbiamo
$$\begin{equation*}
\begin{aligned}[c]
Z_f \ket{A_0} & = \ket{A_0}\\
Z_f \ket{A_1} & = -\ket{A_1}
\end{aligned}
\end{equation*}$$
All'interno dello spazio vettoriale bidimensionale generato da $\ket{A_0}$ e $\ket{A_1}$ questa è una riflessione sulla retta parallela a $\ket{A_0}$, che chiameremo $L_1$. Ecco una figura che illustra l'azione di questa riflessione su un ipotetico vettore unitario $\ket{\psi}$, che supponiamo sia una combinazione lineare reale di $\ket{A_0}$ e $\ket{A_1}$.

![[geometric_picture.png]]

Poi abbiamo l'operazione $H^{\otimes n}Z_{OR}H^{\otimes n}$, dove abbiamo già visto che può essere scritta come $$H^{\otimes n}Z_{OR}H^{\otimes n} = 2\ket{u}\bra{u}- \mathbb{I}$$
Anche questa è una riflessione, questa volta sulla linea $L_2$ parallela al vettore $\ket{u}$. Ecco una figura che raffigura l'azione di questa riflessione su un vettore unitario $\ket{\psi}$.

![[geometric_picture_2.png]]

Quando componiamo queste due riflessioni, otteniamo una rotazione pari al doppio dell'angolo tra le linee di riflessione, come illustrato in questa figura.

## Scegliere il numero di iterazioni
Come abbiamo stabilito nella sezione precedente, il vettore di stato del registro $Q$ nell'algoritmo di Grover rimane nel sottospazio bidimensionale attraversato da $\ket{A_0}$ e $\ket{A_1}$ una volta eseguita la fase di inizializzazione. L'obiettivo è trovare un elemento $x \in A_1$, e questo obiettivo sarà raggiunto se riusciremo a ottenere lo stato $\ket{A_1}$, perché se misuriamo questo stato, abbiamo la garanzia di ottenere un risultato della misurazione $x \in A_1$ (naturalmente supponendo che $A_1$ non sia vuoto).
Dato che lo stato di $Q$ dopo $t$ iterazioni nel passaggio 2 è
$$G^t\ket{u} = \cos((2t+1)\theta)\ket{A_0} + \sin((2t+1)\theta)\ket{A_1}$$
questo significa che dovremmo scegliere $t$ in modo che
$$\braket{A_1|G^t|u} = \sin((2t+1)\theta)$$
è il più vicino possibile a 1 in valore assoluto, per massimizzare la probabilità di ottenere $x\in A_1$ dalla misurazione.
Si noti che per qualsiasi angolo $\theta \in (0, 2\pi)$ il valore $\sin((2t+1)\theta)$ *oscilla* all'aumentare di $t$, sebbene non sia necessariamente periodico: non c'è alcuna garanzia che otterremo mai lo stesso valore due volte.

Possiamo tracciare i valori che otteniamo per valori variabili di $t$ come segue. Per prima cosa importeremo le librerie richieste, quindi tracceremo il valore $\sin((2t+1)\theta)$ per $t$ variabile e una scelta fissa di $\theta$ (che può essere modificata a piacere).

```python
import numpy as np
import math
import matplotlib.pyplot as plt

%matplotlib inline

theta = 1 / 9  # angle in radians
t = np.arange(0, 300, 1)  # (start, stop, step)
y = np.sin((2 * t + 1) * theta)

print("Scatter plot")
plt.plot(t, y, "o", color="black")
plt.show()

print("Linear interpolation")
plt.plot(t, y)
plt.show()
```

![[quiskit_output.png]]
![[quiskit_output2.png]]

Naturalmente, oltre a rendere grande la probabilità di ottenere un elemento $x \in A_1$ dalla misurazione, vorremmo anche scegliere che $t$ sia il più piccolo possibile, perché $t$ applicazioni dell'operazione $G$ richiedono $t$ query alla funzione $f$. Poiché puntiamo a rendere $\sin((2t+1)\theta)$ vicino a 1 in valore assoluto, un modo naturale per farlo è scegliere $t$ in modo che
$$(2t + 1)\theta \approx \frac{\pi}{2}.$$
Risolvendo per $t$ si ottiene
$$t \approx \frac{\pi}{4\theta}- \frac{1}{2}.$$
Naturalmente, $t$ deve essere un numero intero, quindi non saremo necessariamente in grado di raggiungere esattamente questo valore, ma ciò che possiamo fare è prendere il numero intero più vicino a questo valore, che è
$$t = \left\lfloor \frac{\pi}{4\theta} \right\rfloor .$$
Man mano che procediamo con l'analisi, vedremo che la vicinanza di questo intero al valore target influisce naturalmente sulle prestazioni dell'algoritmo.

(Per inciso, se il valore target $\pi/(4\theta) - 1/2$ capita di essere esattamente a metà strada tra due interi, questa espressione di $t$ è ciò che otteniamo arrotondando per eccesso. In alternativa, potremmo arrotondare per difetto, il che ha senso perché significa una query in meno, ma questo è secondario e irrilevante ai fini della lezione.)

Ricordando che il valore dell'angolo $\theta$ è dato dalla formula
$$\theta = \sin^{-1}\left(\sqrt{\frac{|A_1|}{N}} \right),$$
vediamo anche che la nostra stima $t \approx \pi/(4\theta) - 1/2$ dipende dal numero di stringhe in $A_1$. Ciò rappresenta una sfida se non sappiamo quante soluzioni abbiamo, come discuteremo più avanti.

## Ricerca unica
Per prima cosa, concentriamoci sulla situazione in cui c'è una singola stringa $x$ tale che $f(x)=1$. Un altro modo per dirlo è che stiamo considerando un'istanza del problema di ricerca univoca.

In questo caso abbiamo $$\theta = \sin^{-1}\left(\sqrt{\frac{1}{N}} \right),$$che può essere convenientemente approssimato come $$\theta = \sin^{-1}\left(\sqrt{\frac{1}{N}} \right) \approx \sqrt{\frac{1}{N}}$$quando $N$ diventa grande. Se sostituiamo $\theta = 1/\sqrt N$ nell'espressione $$t = \left\lfloor \frac{\pi}{4\theta} \right\rfloor$$ otteniamo $$t = \left\lfloor \frac{\pi}{4\theta} \sqrt N\right\rfloor .$$Ricordando che $t$ non è solo il numero di volte in cui viene eseguita l'operazione $G$, ma anche il numero di query alla funzione $f$ richieste dall'algoritmo, vediamo che siamo sulla buona strada per ottenere un algoritmo che richiede $O(\sqrt N)$ query.

Ora esamineremo quanto bene funziona questa scelta di $t$. La probabilità che la misurazione finale risulti nella soluzione unica può essere espressa esplicitamente come $$p(N, 1) = \sin^2((2t+1)\theta)).$$(Il primo argomento, $N$, si riferisce al numero di possibili soluzioni e il secondo argomento, che in questo caso è 1, si riferisce al numero effettivo di soluzioni. Più avanti useremo la stessa notazione in modo più generale, dove ci sono più soluzioni).

Ecco una cella di codice che calcola la probabilità di successo per valori crescenti di $n$.

```python
for n in range(1, 20):
    N = 2**n
    theta = np.arcsin(np.sqrt(1 / N))
    t = math.floor(np.pi / (4 * theta))
    p = np.sin((2 * t + 1) * theta) ** 2
    print("%d\t %12.10f" % (N, p))
```
```output
2	 0.5000000000
4	 1.0000000000
8	 0.9453125000
16	 0.9613189697
32	 0.9991823155
64	 0.9965856808
128	 0.9956198657
256	 0.9999470421
512	 0.9994480262
1024	 0.9994612447
2048	 0.9999968478
4096	 0.9999453461
8192	 0.9999157752
16384	 0.9999997811
32768	 0.9999868295
65536	 0.9999882596
131072	 0.9999992587
262144	 0.9999978382
524288	 0.9999997279
```
Si noti che queste probabilità non sono strettamente crescenti. In particolare, abbiamo un'interessante anomalia quando $N=4$, dove otteniamo una soluzione con certezza. Si può, tuttavia, dimostrare in generale che $$p(N,1) \geq 1 - \frac{1}{N}$$per tutti gli $N$, quindi la probabilità di successo va a 1 nel limite man mano che $N$ diventa grande, come suggeriscono i valori prodotti dalla cella di codice.

Questo è positivo! Ma si noti che anche un limite debole come $p(N,1)\geq 1/2$ stabilisce l'utilità dell'algoritmo di Grover. Per qualsiasi risultato di misurazione $x$ otteniamo eseguendo la procedura, possiamo sempre verificare se $f(x)=1$ utilizzando una singola query a $f$. E se non riusciamo a ottenere la stringa univoca $x$ per cui $f(x)=1$ con probabilità al massimo $1/2$ eseguendo la procedura una volta, allora dopo $m$ esecuzioni indipendenti della procedura non saremo riusciti a ottenere questa stringa univoca $x$ con probabilità al massimo $2^{−m}$. Vale a dire, utilizzando $O(m\sqrt N)$ query a $f$, otterremo la soluzione univoca $x$ con probabilità di almeno $1−2^{−m}$. Utilizzando il limite migliore $p(N,1)\geq 1−1/N$ si scopre che la probabilità di trovare $x\in A_1$ utilizzando questo metodo è in realtà almeno $1−N^{−m}$.

## Soluzioni multiple
Al variare del numero di elementi in $A_1$, varia anche l'angolo $\theta$, che può avere un effetto significativo sulla probabilità di successo dell'algoritmo. Per brevità, scriviamo $s =|A_1|$ per indicare il numero di soluzioni e, come prima, supporremo che $s \geq 1$.

Come esempio motivante, immaginiamo di avere $s=4$ soluzioni anziché una singola soluzione, come abbiamo considerato sopra. Ciò significa che
$$\theta = \sin^{-1}\left(\sqrt{\frac{4}{N}} \right),$$
che è approssimativamente il doppio dell'angolo che avevamo nel caso $|A1|=1$ quando $N$ è grande.

Supponiamo di non sapere di meglio e di aver selezionato lo stesso valore di $t$ dell'impostazione della soluzione unica: $$t = \left\lfloor \frac{\pi}{4\sin^{-1}\left(\frac{1}{\sqrt{N}}\right)} \right\rfloor.$$L'effetto sarà catastrofico, come dimostra la prossima cella di codice.
```python
# We'll start the loop with n=2 to stay within the domain of arcsin.
# (Note that we can't have 4 solutions when n=1.)

for n in range(2, 20):
    N = 2**n
    old_theta = np.arcsin(np.sqrt(1 / N))
    new_theta = np.arcsin(np.sqrt(4 / N))
    t = math.floor(np.pi / (4 * old_theta))
    p = np.sin((2 * t + 1) * new_theta) ** 2
    print("%d\t %12.10f" % (N, p))
```
```output
4	 1.0000000000
8	 0.5000000000
16	 0.2500000000
32	 0.0122070313
64	 0.0203807689
128	 0.0144530758
256	 0.0000705058
512	 0.0019310741
1024	 0.0023009083
2048	 0.0000077506
4096	 0.0002301502
8192	 0.0003439882
16384	 0.0000007053
32768	 0.0000533810
65536	 0.0000472907
131072	 0.0000030066
262144	 0.0000086824
524288	 0.0000010820
```

La probabilità di successo va a 0 quando $N$ va a infinito. Questo accade perché stiamo effettivamente ruotando due volte più velocemente di quanto facessimo quando c'era una soluzione unica, quindi finiamo per sfrecciare oltre il bersaglio $\ket{A_1}$ e atterrare vicino a $-\ket{A_0}$.

Tuttavia, se invece utilizziamo la scelta consigliata di $t$, che è $$t = \left\lfloor \frac{\pi}{4\theta} \right\rfloor$$per $$\theta = \sin^{-1}\left(\sqrt{\frac{s}{N}} \right),$$allora la performance sarà migliore. Per essere più precisi, usare questa scelta di $t$ porta al successo con alta probabilità, come suggerisce la seguente cella di codice. Qui stiamo usando la notazione suggerita in precedenza: $p(N,s)$ denota la probabilità che l'algoritmo di Grover eseguito per t iterazioni riveli una soluzione quando ci sono $s$ soluzioni in totale su $N$ possibilità.
```python
s = 7  # Number of solutions. This can be any positive integer.

# The loop starts with the smallest value of n that allows s solutions.
for n in range(math.ceil(math.log2(s)), math.ceil(math.log2(s)) + 20):
    N = 2**n
    theta = np.arcsin(np.sqrt(s / N))
    t = math.floor(np.pi / (4 * theta))
    p = np.sin((2 * t + 1) * theta) ** 2
    print("%d\t %12.10f" % (N, np.sin((2 * t + 1) * theta) ** 2))
```
```output
8	 0.8750000000
16	 0.6835937500
32	 0.9877929688
64	 0.9869401455
128	 0.9933758959
256	 0.9942813445
512	 0.9977678832
1024	 0.9999963373
2048	 0.9999257666
4096	 0.9983374778
8192	 0.9995465664
16384	 0.9995822234
32768	 0.9999531497
65536	 0.9998961946
131072	 0.9999998224
262144	 0.9999745784
524288	 0.9999894829
1048576	 0.9999939313
2097152	 0.9999979874
4194304	 0.9999986243
```

Generalizzando quanto affermato in precedenza, si può dimostrare che $$p(N,s) \geq 1 -\frac{s}{N}.$$Questo limite inferiore di $1−s/N$ sulla probabilità di successo è leggermente peculiare in quanto più soluzioni implicano un limite inferiore peggiore, ma nell'ipotesi che $s$ sia significativamente più piccolo di $N$, concludiamo comunque che la probabilità di successo è ragionevolmente alta. Come prima, il semplice fatto che $p(N,s)$ sia ragionevolmente grande implica l'utilità dell'algoritmo.

È anche il caso che $$p(N,s) \geq \frac{s}{N}.$$Questo limite inferiore descrive la probabilità che una stringa $x \in \Sigma^n$ selezionata in modo uniforme a caso sia una soluzione, quindi l'algoritmo di Grover funziona sempre almeno quanto un'ipotesi casuale. Infatti, l'algoritmo di Grover è un'ipotesi casuale quando $t=0$.

Ora diamo un'occhiata al numero di iterazioni (e quindi al numero di query) $$t = \left\lfloor \frac{\pi}{4\theta} \right\rfloor,$$per $$\theta = \sin^{-1}\left(\sqrt{\frac{s}{N}} \right).$$Per ogni $\alpha \in [0,1]$, è il caso in cui $\sin^{-1}(\alpha) \geq \alpha$, e quindi $$\theta = \sin^{-1}\left(\sqrt{\frac{s}{N}} \right) \geq \sqrt{\frac{s}{N}}.$$Questo implica che $$t \leq \frac{\pi}{4\theta} \leq \frac{\pi}{4}\sqrt{\frac{N}{s}}.$$Ciò si traduce in un risparmio nel numero di query man mano che $s$ cresce. In particolare, il numero di query richieste è $$O\left(\sqrt{\frac{N}{s}} \right).$$
## Numero sconosciuto di soluzioni
