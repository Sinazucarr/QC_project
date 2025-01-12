Dopo aver analizzato l'implementazione dell'algoritmo di Grover applicato al caso di un Sudoku 2x2, è opportuno esaminare le possibili ottimizzazioni offerte dall'approccio quantistico. Sebbene Grover rappresenti una soluzione generale per problemi di ricerca non strutturati, esso presenta sfide significative in termini di risorse computazionali, progettazione dell'oracolo ed efficienza rispetto al numero di soluzioni.
### Ottimizzazione del numero di Iterazioni: approccio adattivo
L'approccio adattivo si basa sull'idea di regolare dinamicamente il numero di iterazioni R durante l'esecuzione, anziché determinarlo a priori. 
Questo metodo può essere particolarmente utile in scenari in cui il numero di soluzioni $M$ è sconosciuto o in cui si cerca di massimizzare l'efficienza complessiva.

L'algoritmo di Grover ruota lo stato iniziale $∣ψ⟩$ verso lo stato delle soluzioni $|\beta\rangle$
in uno spazio N-dimensionale, utilizzando l'angolazione definita da:
$$
\theta = 2 arcsin \sqrt{\frac{M}{N}}
$$
Il numero ottimale di iterazioni R è dato da:
$$

R= \left\lfloor \frac{\pi}{4\theta} \right\rfloor ≈\left\lfloor \frac{\pi}{4}  \sqrt{\frac{N}{M}}\right\rfloor 

$$
Tuttavia, se il valore di $R$ è troppo grande, lo stato qunatistico viene "ruotato troppo", passando oltre $|\beta\rangle$ e riducendo la probabilità di successo. Se invece $R$ è troppo piccolo, la probabilità di trovare la soluzione risulta subottimale. Per ovviare a queste problematiche, l'approccio adattivo prevede una regolazione dinamica del numero di iterazioni.
### Procedura Adattiva 
 1. **Avvio con un Numero Minimo di Iterazioni**
	Si inizia con un numero ridotto di iterazioni, ad esempio ad esempio $R_0=1$. 
	Dopo questa singola iterazione, la probabilità di trovare una soluzione è approssimativamente:
	$$
	P_1=sin⁡2(θ)
	$$
	dove $\theta = 2 \arcsin \sqrt{\frac{M}{N}}$​​ 
	
	Se la soluzione viene trovata durante la misurazione, il processo termina; altrimenti, si procede al passo successivo.

2. **Incremento del Numero di Iterazioni**

	Se la soluzione non è stata trovata, si incrementa il numero di iterazioni. 
	Si può fare in modo graduale, utilizzando un incremento lineare ($R_{k+1} = R_k + \Delta R$) o esponenziale ($R_{k+1} = R_k \times 2$).
	
	Ad esempio, dopo $R_1 = 2$ iterazioni, la probabilità di successo diventa:
	$$
	P_2 = \sin^2(2\theta)
	$$
	Se la soluzione viene trovata, il processo si conclude. Altrimenti, si ripete il passo, aumentando ulteriormente il numero di iterazioni.

3.  **Limitazione delle Iterazioni**

	Continua ad aumentare RRR fino a un massimo teorico determinato da:
	$$
	R_{\text{max}} = \left\lfloor \frac{\pi}{4\theta} \right\rfloor
	$$
	Questo massimo assicura che lo stato quantistico non venga "ruotato oltre" lo stato delle soluzioni $|\beta\rangle$, evitando un calo nella probabilità di successo.

4.  **Misurazioni Intermedie**

	Dopo ogni serie di iterazioni, misura lo stato del sistema. Se la soluzione viene trovata (ossia se la misura corrisponde a uno degli stati $|\beta\rangle)$, l'algoritmo termina. 
	In caso contrario, il sistema viene reimpostato e si procede con il passo successivo.

Questo approccio è particolarmente utile in contesti in cui MMM è piccolo o incerto, come nei problemi di ottimizzazione o ricerca con pochi risultati validi nello spazio di ricerca.

[Grover Adaptive Search for Constrained Polynomial Binary Optimization](https://arxiv.org/pdf/1912.04088)
#### Considerazioni sull'efficienza rispetto al numero di soluzioni
[Chapter_6.1.4](https://profmcruz.wordpress.com/wp-content/uploads/2017/08/quantum-computation-and-quantum-information-nielsen-chuang.pdf)
### Ottimizzazione con M noto
Se $M$ è noto in anticipo e  $M≥ \frac{N}{2}$, si può scegliere casualmente un elemento dello spazione di ricerca e verificarne la validità tramite l'oracolo. Questo approccio ha una probabilità di successo di almeno il 50% e richiede una consultazione dell'oracolo. Tuttavia presenta lo svantaggio che il numero di soluzioni M potrebbe non essere noto in anticipo.

### Ottimizzazione con M noto
Se $M≥ \frac{N}{2}$ , l'algoritmo di Grover diventa meno efficiente, poiché il numero di iterazioni richiesto cresce con $M$. In questi casi, un'alternativa consiste nel raddoppiare lo spazio di ricerca aggiungendo $N$ configurazioni fittizie (non soluzioni) e applicando un nuovo oracolo. 
Questo approccio garantisce che meno della metà delle configurazioni nel nuovo spazio siano soluzioni, mantenendo l'efficienza dell'algoritmo.
