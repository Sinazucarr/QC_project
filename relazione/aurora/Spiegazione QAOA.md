### **Modellazione del Sudoku 2 \times 2 con Valori Binari**

Ogni cella può contenere solo 0 o 1. Questo riduce la complessità della rappresentazione e permette di utilizzare direttamente i qubit per modellare le celle.

#### **1. Celle valide**

Le celle hanno valori limitati a 0 e 1, quindi non c’è bisogno di un controllo ulteriore sulla validità dei valori stessi. L’attenzione si sposta sul rispetto delle regole del Sudoku.

#### **2. Penalità per configurazioni non valide**

Le regole da rispettare rimangono:

- **Regole sulle righe e colonne:** I valori nelle righe e nelle colonne non devono essere duplicati.
- **Regole sul blocco:** Non ci devono essere duplicati nel blocco 2×2.

Con valori binari, possiamo tradurre queste regole in condizioni di ottimizzazione.

---

### **Funzione di Costo**

La funzione di costo assegna penalità alle configurazioni che violano le regole.

1. **Penalità per righe:** Per una riga, se la somma dei valori è maggiore di 1, c’è una violazione. Ad esempio, se una riga è [1,1], la somma è 2, e quindi la regola è violata.
    
    Penalità:
    $$
       C_{\text{righe}} = \sum_{\text{righe}} \left( \text{somma(riga)} - 1 \right)^2
    $$
 
2. **Penalità per colonne:** Simile alle righe, se una colonna ha una somma maggiore di 1, viene assegnata una penalità.
    
    Penalità:
    $$
    C_{\text{colonne}} = \sum_{\text{colonne}} \left( \text{somma(colonna)} - 1 \right)^2
    $$
1. **Penalità per il blocco:** Per il blocco 2×2, la somma totale dei valori deve essere pari a 2 (dato che abbiamo 2 valori 1 e 2 valori 0).
    
    Penalità:
    $$
        C_{\text{blocco}} = \left( \text{somma(blocco)} - 2 \right)^2
    $$


La funzione di costo complessiva diventa:
$$
C(x) = C_{\text{righe}} + C_{\text{colonne}} + C_{\text{blocco}}​
$$
---

### **Esempio Pratico**

Supponiamo di avere la seguente configurazione iniziale di Sudoku 2×2:
$$
\begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}
$$
#### Verifica:

1. **Regole delle righe e colonne:**
    
    - Riga 1: Somma 1+0=1(valida).
    - Riga 2: Somma 0+1=1 (valida).
    - Colonna 1: Somma 1+0=1 (valida).
    - Colonna 2: Somma 0+1=1 (valida).
2. **Regola del blocco:**
    
    - Somma totale del blocco: 1+0+0+1=2 (valida).

La configurazione è corretta, quindi C(x)=0.

---

Se invece la configurazione fosse:
$$
\begin{bmatrix} 1 & 1 \\ 0 & 1 \end{bmatrix}
$$
#### Penalità:

1. **Regole delle righe e colonne:**
    
    - Riga 1: Somma 1+1=2. Penalità: (2 - 1)^2 = 1
    - Riga 2: Somma 0+1=1(valida).
    - Colonna 1: Somma 1+0=1 (valida).
    - Colonna 2: Somma 1+1=2 Penalità: (2 - 1)^2 = 1.
2. **Regola del blocco:**
    
    - Somma totale del blocco: 1+1+0+1=3. Penalità: (3 - 2)^2 = 1.

Funzione di costo complessiva:

C(x)=1+0+1=3

---

### **Esecuzione con QAOA**

1. **Stato iniziale:** Una sovrapposizione uniforme di tutte le configurazioni binarie della griglia.
2. **Operatore costruttivo:** Si applica un'operazione basata sulla funzione di costo C(x), che penalizza gli stati con C(x) > 0.
3. **Operatore di miscelazione:** Esplora le configurazioni alternative.
4. **Ottimizzazione:** I parametri del circuito QAOA vengono ottimizzati per minimizzare C(x), favorendo configurazioni con C(x)=0.

Questo processo porta gradualmente il sistema verso configurazioni valide che rispettano tutte le regole del Sudoku 2×2.