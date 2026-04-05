# **Can a 1-Qubit Gate Be Harder Than a 2-Qubit Gate?**

## **1. Introduction**

At first glance, a 1-qubit gate seems simpler than a 2-qubit gate: it acts on fewer resources. This report builds a technical case for why this intuition breaks down in practice in certain circumstances. We start by focusing on the implementation of Z-rotations RZ(π/2^k) and tracing the growing cost of implementing them as k increases, followed by the exploration of magic state and their use in applying T gates in practice. We then propose a brief study of the STAR Architecture to support the claim that 

![placeholder](https://markdowntoword.io/placeholder.png)

Learning the gates with the Bloch sphere and how to use the Clifford + T gate set.

## **2. The Clifford+T Gate Set**

The Clifford+T gate set {H, S, T, CNOT} is universal for quantum computing. Within this set, the gates are not equal in cost:

- H, S, CNOT : cheap, fault-tolerant friendly
- T : expensive, it cannot be applied transversally on most error-correcting codes

 Studying  for large k= {0,1,2,3,4,5} reveals an obvious relationship between the first three values of k : i.e. a simple RZ(π/4) rotation requires exactly one T gate. But we start to try to find a combination of the T gate corresponding to k= 3, we end up with . Implementing such gate is actually an approximation problem requiring an **exponentially growing number of T gates** : this is the first sign that small angles are hard.

We eventually understand that there is no unique solution, we only can approximate the T gate by using heuristics. With no idea of pattern, we try multiple combination of H, S and T. We get varying results from from simple combinations of H and T gates being repeated but nothing is below 10 % error. 

From then on we try and algorithmic approach to test the different paths instead of statically repeating H and T as we had previously observed that there were sometimes good results when we did a combination like H T T. 

The first algorithm we implement is a breadth-first search, we basically explore all combinations from every subset “branch” of the possible circuits.  We had our best result being barely under 0,04. 

![placeholder](https://markdowntoword.io/placeholder.png)

The second approach we had was using beam search, as the previous algorithm was too slow and gave limited results as we tried to go deeper, with added predefined gates in the set. This enriched gate set consisted of combinations that we observed were common. 

For 22.5 [degrees] we approached the result with a distance of 0,029. 

 ![placeholder](https://markdowntoword.io/placeholder.png)

With this algorithm we also had undeniably better results for the subsequent roots (that would otherwise either not return anything from the complexity or be above 0,04) which were the following: 

11.25 [degrees]: 0,0101

5.625 [degrees]: 0,02226

Approximation of the sqrt(T) gate with H, S andT gate. 

## **3. Synthesizing Small Rotations**

Using the Clifford+T gate set, we synthesized the family:

![placeholder](https://markdowntoword.io/placeholder.png)

The approximation quality is measured by:

![placeholder](https://markdowntoword.io/placeholder.png)

Key observations:

- For small n, exact synthesis is possible with few T gates
- For large n, the T-count grows, and approximations accumulate error
- The Solovay-Kitaev theorem guarantees approximations exist, but at a cost

## **4. When T Gates Are Forbidden on the Data Qubit**

In realistic hardware, **T gates cannot be applied directly** to the data qubit — they introduce too much noise. The solution is **T gate injection** via ancilla qubits:

- Prepare a magic state on an ancilla qubit
- Apply a CNOT between the ancilla and the data qubit
- Measure the ancilla and apply a classical correction if needed

 ![placeholder](https://markdowntoword.io/placeholder.png)

T gate apply on psy

This means that implementing a single logical $T$ gate now costs:

- 1+ ancilla qubit
- 1 CNOT (a 2-qubit gate)
- 1 measurement + classical feedforward

We verify if the state T is apply on the state psy. We apply the T^-1 gate after the T gate. As T is a unitary matrix, the application of  give the identity matrix. If we find the same state at the start and the end, we can assure that the T state if apply to the physique Qubit logic. 
We compare the density matrix of state vector à the start, after the application of T and at the end of the circuit. Finally the gate T is apply by magic one the logic Qubit. 

**A 1-qubit rotation now requires a 2-qubit gate to implement.** The further we apply T gate by magic, the further the depth, ancilla count, cnot count, measurment and feedforward growth by O(n).

 ![placeholder](https://markdowntoword.io/placeholder.png)

 

​

T gate​

T state injection​

Depth​

*O(n)*​

*O(n)*​

Ancilla count​

*O(1)*​

*O(n)*​

CNOT count​

*O(1)*​

*O(n)*​

Measurements​

*O(1)*​

*O(n)*​

Feedforward​

*O(1)*​

*O(n)*​

## **5. From Physical to Logical Qubit: The Steane Code**

The $[[7,1,3]]$ Steane code encodes 1 logical qubit into 7 physical qubits. Allowing for stabilizer extraction, with Pauli observables. The transversal application of S gates and H gates isn't impacting the stabilizers value, but the T state transversal injection is, meaning it affects the error detection and cannot be applied in fault-tolerant computing.  

![placeholder](https://markdowntoword.io/placeholder.png)

## **6. Implementing Tsim circuit from a Stim File**

Through this challenge, we implemented a library that allows us to import pre-built circuits saved in .stim file format, which we can then compile and run shots on. This approach significantly increases the flexibility of writing and testing different circuits.

Having access to the files, we wrote a Jupyter notebook that enabled us to rapidly test different star fidelities and generate data for visualization. First, we compiled the detector sample. Then, we ran shots on the sample while adding a separate_observables variable to True, and finally, we calculated the mean of the logical bit results.

I believe that if we had more time, we could have added post-selection based on the distance of the result. This would considerably improve the success rate and yield better data.

## **7. Conclusion**

The claim is true: **a 1-qubit gate can be harder than a 2-qubit gate**. The difficulty is not in the mathematical description, but in the physical implementation:

- Small-angle rotations require many T gates
- T gates cannot be applied directly, they must be teleported via ancillas and 2-qubit gates
- Moving to a logical qubit amplifies every cost

The deeper the target rotation RZ(π/2^k), the more expensive it becomes, in ancilla count, circuit depth, and error rate.

## **References**

- *"Quantum Computing, Universal Gate Sets"* — https://www.scottaaronson.com/qclec/16.pdf
- *"The Solovay-Kitaev Algorithm"* — https://arxiv.org/pdf/quant-ph/0505030
- *"Optimal ancilla-free Clifford+T approximation of z-rotations"* — https://arxiv.org/abs/1403.2975
- *"Efficient synthesis of universal Repeat-Until-Success circuits"* — https://arxiv.org/abs/1404.5320
- *"Partially Fault-tolerant Quantum Computing Architecture with Error-corrected Clifford Gates and Space-time Efficient Analog Rotations"* — https://arxiv.org/pdf/2303.13181
- *"Practical quantum advantage on partially fault-tolerant quantum computer"* — https://arxiv.org/pdf/2408.14848
- *"Transversal STAR architecture for megaquop-scale quantum simulation with neutral atoms"* — https://arxiv.org/pdf/2509.18294
