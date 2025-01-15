Il [codice]([QC_project/relazione/claudio/sudoku_2x2_solver.ipynb at main · ariannacipolla/QC_project](https://github.com/ariannacipolla/QC_project/blob/main/relazione/claudio/sudoku_2x2_solver.ipynb)) di riferimento mostra un risolutore di un Sudoku, più precisamente un quadrato latino, 2x2 introducendo un modello di rumore che prevede la depolarizzazione, il rilassamento termico e l'errore sulla misura.

Per ottenere tutte le possibili soluzioni (2 in questo caso), il codice implementa l'algoritmo di Grover precedentemente trattato. 
## 3.1 Panoramica generale
Il codice modella in modo non modulare la risoluzione di un sudoku $S$ con $n=2$ celle per lato  con un alfabeto: 
$$\Sigma = \{0,1\}$$
Nella dashboard utente vi sono dei flags che permettono di configurare il risolutore cambiando i rumori che si vogliono introdurre, le iterazioni di calcolo e iterazioni di Grover.

*Per Grover è possibile inserire $0$ per lasciare calcolare il numero di iterazioni suggerite al software secondo la legge (riferimento).*

Quindi, dopo aver preparato $n \times n$ qubits che rappresentano le celle, e quindi lo spazio di una  possibile combinazione del sudoku $Qubits=\{q_1​, q_2​, q_3, q_4\}$ e 4 ancille $Ancille=\{an_1,an_2,an_3,an_4\}$ che rappresentano i risultati delle condizioni dell'oracolo, il circuito inizializza i qubits in superposition tramite la porta $H$ e procede attraverso l'algoritmo di Grover sottoponendo i qubit all'oracolo.
$$S=\begin{pmatrix}
a & b\\
c & d\\
\end{pmatrix}$$
L'oracolo, procede confrontando tutte le possibile coppie $(ab,bc,cd,bd)$ attraverso una $MCX$ ovvero una porta "$XOR$", sostanzialmente determinando se i valori delle coppie sono diversi.
```python
# Compare two qubits using an ancilla qubit
def compare(qc, control_qubit, target_qubit, ancilla_qubit):
    """Perform controlled comparison between two qubits."""
    qc.x(target_qubit)
    qc.mcx([control_qubit, target_qubit], ancilla_qubit)
    qc.x(target_qubit)
    
# Perfomr a XOR operation to two given qubits and returns result in ancilla
def XOR(qc, qubit_1, qubit_2, ancilla):
    """Perform XOR operation using ancilla qubits."""
	compare(qc, qubit_1[0], qubit_2[0], ancilla[0])
    compare(qc, qubit_2[0], qubit_1[0], ancilla[1])
    qc.cx(ancilla[0], ancilla[1])
    qc.barrier()

# Define the oracle for Grover's algorithm
def oracle(qc):
    """Mark the desired states using XOR gates."""
    XOR(qc, qubits[0], qubits[1], ancilla_ab)
    XOR(qc, qubits[0], qubits[2], ancilla_ac)    
    XOR(qc, qubits[1], qubits[3], ancilla_bd)
    XOR(qc, qubits[2], qubits[3], ancilla_cd)
    ```
Questa operazione viene svolta avendo cura di invertire i qubits tramite porte $CNOT$ all'interno delle funzioni `compare()` per garantire la reversibilità delle operazioni (proprietà fondamentale dei circuiti quantici) forzando sostanzialmente un refresh dei qubits.

C(s)=⋀(Sudoku Rules Validity for State s)

In questa fase vengono quindi testate tutte le possibili combinazioni $N_k$ che appartengono allo spazio delle combinazioni $N$, con $k$ equivalente al numero di combinazioni possibili.
$$k = n!\cdot(n-1)!$$
$$N_k \in N \Leftrightarrow N_k \cap \{ N-N_k\} = \emptyset  $$
$$esempio: N = \{N_0=\begin{pmatrix}
0 & 0\\
0 & 0\\
\end{pmatrix},N_1=\begin{pmatrix}
0 & 0\\
0 & 1\\
\end{pmatrix}...N_N=\begin{pmatrix}
1 & 1\\
1 & 1\\
\end{pmatrix}\}$$
tuttavia, essendo interessati solo allo spazio delle soluzioni accettabili $M$, condizioniamo i valori tramite un diffusore, il quale praticherà un'inversione rispetto la media, che andrà ad amplificare le misurazioni delle soluzioni accettabili.

P′(s)={P(s)+ΔP(s)−Δ​if C(s)=Trueotherwise.​

```python
# Perform inversion about the mean
def inversion_about_mean(qubits, target_qubit):
    """Perform inversion about the mean operation."""
    qc.h(target_qubit[0])
    qc.mcx([qubit for qubit in qubits if qubit != target_qubit], target_qubit[0])
    qc.h(target_qubit[0])
    
# Diffuser operation for Grover's algorithm
def diffuser(qc):
    """Apply the diffuser transformation for Grover's algorithm."""
    apply_superposition(qubits)
    flip_qubits(qubits)

    inversion_about_mean(qubits, qubits[0])

    flip_qubits(qubits)
    apply_superposition(qubits)

    qc.barrier()
```
Questo è ottenuto sfruttando una porta di $MCX$ dopo aver opportunamente preparato i tutti i qubits specificatamente quello target.
## 3.2 Rumore
Siccome l'obbiettivo è quello di creare una simulazione di un caso reale, prima della stampa dei risultati viene costruito e applicato un modello di rumore per simulare le condizioni reali di un circuito quantistico.
### 3.2.1 Modello dell'errore
Il codice di riferimento, permette all'utente di scegliere un modello puro, quindi senza errori o scegliere fra depolarizzazione tramite una probabilità, che rappresenta una variazione casuale di uno stato di un Qbit per via delle imperfezioni hardware delle porte; oppure un rilassamento T1 e T2 che rappresentano rispettivamente qubits che perdono lo stato di eccitazione o fluttuazioni dovute all'interazione con l'ambiente circostante.
```python
# --- noise: DEPOLARIZATION ---
if depolarization_noise_on:
    one_qubit_error = depolarizing_error(param=0.02, num_qubits=1)
    two_qubit_error = depolarizing_error(param=0.03, num_qubits=2)

    custom_noise_model.add_all_qubit_quantum_error(one_qubit_error, ['x', 'h', 'u3'])
    custom_noise_model.add_all_qubit_quantum_error(two_qubit_error, ['cx'])

# --- noise: RELAXATION ---
if relaxation_noise_on:
    # Apply relaxation error only to 1-qubit gates (not multi-qubit gates)
    T1s = np.random.normal(50e3, 10e3, 4)
    T2s = np.random.normal(70e3, 10e3, 4)
    
    T2s = np.array([min(T2s[j], 2 * T1s[j]) for j in range(4)])

    # Instruction times (in nanoseconds)
    time_u1 = 0   # virtual gate
    time_u2 = 50  # (single X90 pulse)
    time_u3 = 100 # (two X90 pulses)
    time_cx = 300
    time_reset = 1000  # 1 microsecond
    time_measure = 1000 # 1 microsecond

    # QuantumError objects
    errors_reset = [thermal_relaxation_error(t1, t2, time_reset) for t1, t2 in zip(T1s, T2s)]
    errors_measure = [thermal_relaxation_error(t1, t2, time_measure) for t1, t2 in zip(T1s, T2s)]
    errors_u1  = [thermal_relaxation_error(t1, t2, time_u1) for t1, t2 in zip(T1s, T2s)]
    errors_u2  = [thermal_relaxation_error(t1, t2, time_u2) for t1, t2 in zip(T1s, T2s)]
    errors_u3  = [thermal_relaxation_error(t1, t2, time_u3) for t1, t2 in zip(T1s, T2s)]
    errors_cx = [[thermal_relaxation_error(t1a, t2a, time_cx).expand(
                thermal_relaxation_error(t1b, t2b, time_cx))
                for t1a, t2a in zip(T1s, T2s)]
                for t1b, t2b in zip(T1s, T2s)]

    for j in range(4):
        custom_noise_model.add_quantum_error(errors_reset[j], "reset", [j])
        custom_noise_model.add_quantum_error(errors_measure[j], "measure", [j])
        custom_noise_model.add_quantum_error(errors_u1[j], "u1", [j])
        custom_noise_model.add_quantum_error(errors_u2[j], "u2", [j])
        custom_noise_model.add_quantum_error(errors_u3[j], "u3", [j])
        for k in range(4):
            custom_noise_model.add_quantum_error(errors_cx[j][k], "cx", [j, k])
```
Inoltre, è possibile aggiungere ai modelli precedenti, un errore relativo alla lettura degli stati dei qubits che si verificano per imperfezioni hardware.
```python
# --- noise: READOUT ---
if measurement_error_on:
    readout_error_a = ReadoutError([[0.90, 0.10], [0.05, 0.95]])
    readout_error_b = ReadoutError([[0.90, 0.10], [0.05, 0.95]])
    readout_error_c = ReadoutError([[0.90, 0.10], [0.05, 0.95]])
    readout_error_d = ReadoutError([[0.90, 0.10], [0.05, 0.95]])

    # Apply readout errors to measurements for each qubit
    custom_noise_model.add_readout_error(readout_error_a, [0])
    custom_noise_model.add_readout_error(readout_error_b, [1])
    custom_noise_model.add_readout_error(readout_error_c, [2])
    custom_noise_model.add_readout_error(readout_error_d, [3])
```
### 3.2.2 Comportamento del Rumore
Prenderemo ora in esame alcuni risultati notevoli al fine di mostrare la variazione sull'output che gli errori provocano sui dati. 
```python
# Flags for enabling specific errors noises
depolarization_noise_on = False # Toggle depolarization noise
relaxation_noise_on = False     # Toggle relaxation noise
measurement_error_on = False    # Toggle measurement error

# Simulation configurations
simulation_shots = 1024       # Number of shots for the simulation
grover_iterations = 2         # Set 0 to use the suggested optimal value
```
![[relazione/claudio/IMG/Histo_test_1.png]]
```python
# Flags for enabling specific errors noises
depolarization_noise_on = False # Toggle depolarization noise
relaxation_noise_on = False     # Toggle relaxation noise
measurement_error_on = True     # Toggle measurement error

# Simulation configurations
simulation_shots = 1024       # Number of shots for the simulation
grover_iterations = 2         # Set 0 to use the suggested optimal value
```
![[relazione/claudio/IMG/Histo_test_4.png]]
```python
# Flags for enabling specific errors noises
depolarization_noise_on = False # Toggle depolarization noise
relaxation_noise_on = True      # Toggle relaxation noise
measurement_error_on = True     # Toggle measurement error

# Simulation configurations
simulation_shots = 1024       # Number of shots for the simulation
grover_iterations = 2         # Set 0 to use the suggested optimal value
```
![[relazione/claudio/IMG/Histo_test_3 1.png]]
```python
# Flags for enabling specific errors noises
depolarization_noise_on = True # Toggle depolarization noise
relaxation_noise_on = False    # Toggle relaxation noise
measurement_error_on = True    # Toggle measurement error

# Simulation configurations
simulation_shots = 1024       # Number of shots for the simulation
grover_iterations = 2         # Set 0 to use the suggested optimal value
```
![[relazione/claudio/IMG/Histo_test_2.png]]
Risulta evidente che il caso peggiore lo troviamo quando errori di misura e di depolarizzazione si combinano. Ovviamente nel nostro caso il risultato non è compromesso, ma è necessario prenderne atto.

### 3.2.2 Contromisure
Esistono diverse librerie che introducono tecniche di mitigazione avanzate che permettono di combattere anche specifici tipi di rumore. Tuttavia vale la pena considerare semplici accorgimenti che essi stessi fungono da mitigatori.

**Incremento degli shots:**
Aumentare il numero di shots, quindi iterazioni dell'esperimento, permettono di limare gli errori statistici. Infatti testiamo nuovamente la depolarizzazione con più iterazioni:
```python
# Flags for enabling specific errors noises
depolarization_noise_on = True # Toggle depolarization noise
relaxation_noise_on = False    # Toggle relaxation noise
measurement_error_on = True    # Toggle measurement error

# Simulation configurations
simulation_shots = 10024       # Number of shots for the simulation
grover_iterations = 2         # Set 0 to use the suggested optimal value
```
![[relazione/claudio/IMG/Histo_test_5.png]]
Per quanto possa sembrare poco diverso da prima, si nota una riduzione del 6% della distanza fra il conteggio della massima soluzione e il conteggio della massima non soluzione. In altre parole, abbiamo "allontano" la più probabile non soluzione dalle soluzioni.

**Ripetizione delle misure:**
Svolgere la misura di un singolo conteggio più volte riduce gli errori casuali che sporcano la lettura. Trasformando ogni conteggio in un valor medio di $j$ ripetizioni, riduciamo l'influenza di questo rumore.
```python
# Flags for enabling specific errors noises
depolarization_noise_on = True # Toggle depolarization noise
relaxation_noise_on = False    # Toggle relaxation noise
measurement_error_on = True    # Toggle measurement error

# Simulation configurations
simulation_shots = 10024       # Number of shots for the simulation
num_repeats = 100              # Random reduction mitigation repeats
grover_iterations = 2          # Set 0 to use the suggested optimal value

...

if reduce_random:
    # Run the transpiled circuit with noise model
    all_counts = []
    for _ in range(num_repeats):
        result = backend.run(transpiled_circuit, shots=simulation_shots, noise_model=custom_noise_model).result()
        all_counts.append(result.get_counts())
        
    average_counts = Counter()
    for counts in all_counts:
        average_counts.update(counts)
        
    # Normalize the counts
    total_shots = sum(average_counts.values())
    raw_data = {key: value / total_shots for key, value in average_counts.items()}
```
![[relazione/claudio/IMG/Histo_test_7.png]]Essendo le soluzioni del nostro problema non polarizzate, ovvero equiprobabili, migliorare la qualità delle misure avvicina questi valori fra loro rendendoli decisamente evidenti.

## 3.3 Output
Oltre agli istogrammi che abbiamo visto nella sezione precedente, il codice produce anche l'immagine di un circuito e ci stampa la soluzione.
Nel nostro caso:
```python
Solution 1:
[['1' '0'] 
 ['0' '1']]
 
 Solution 2:
 [['0' '1']
 ['1' '0']]
```
![[relazione/claudio/IMG/Circuit_test_1 1.png]]
