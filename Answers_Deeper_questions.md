# Answers to Deeper Questions and Judging Criteria

This document covers our detailed answers to the "Deeper questions on both tracks" and explains how our project addresses all points of the judging criteria listed in the QuantumVerse 4.0 Hackathon prompt.

---

## Part 1: Answers to Deeper Questions

### 1. How many qubits and what circuit depth are required to get a good score with your circuit?

* Track 1 (Standard Bomb Tester):
  * Qubits: Exactly 2 qubits (1 path qubit to carry the photon superposition, and 1 bomb qubit to act as the target)
  * Circuit Depth: Depth of 4 (`ry` -> `cx` -> `ry` -> `measure`). It is very compact, so there are minimal gate errors.

* Track 1 (Quantum Zeno Setup):
  * Qubits: For an N-step measurement, it uses N + 1 qubits (1 path qubit and N ancilla qubits for repeated bomb interactions).
  * Circuit Depth: Depth of 2N + 1. For example, with N = 12 steps, depth is 25.

* Track 2 (Quantum Radar for Battleship):
  * Qubits: Exactly 2 qubits per measurement line. 
  * Circuit Depth: Depth of 4.
  * Why this design: Some implementations try to encode the whole 2D board at once using 5 to 8 qubits and multi-controlled Toffoli gates. But decomposing a 4-control Toffoli gate on real hardware creates dozens of CNOT gates, driving the depth over 50 and quickly degrading coherence. By using 2 qubits to probe whole rows and columns tomographically, our circuit depth stays at 4, keeping gate fidelity high.

---

### 2. Does changing the rotation angle, or adding more qubits and gates, improve the score?

* Changing the Rotation Angle (theta):
  * Yes, it directly improves the score. 
  * In a single-stage Mach-Zehnder setup, the interaction-free score formula is:
    `Score = P_det / P_exp = cos^2(theta / 2)`
  * When theta = pi/2 (the standard 50:50 beam splitter), `Score = cos^2(pi / 4) = 50%`.
  * When we tune theta to pi/3 (60 degrees), `Score = cos^2(pi / 6) = 75%`.
  * If you decrease theta even further toward 0, the single-pass score approaches 100%. However, the actual chance of a detection per single shot drops as `sin^2(theta / 2)`, so you need more shots to get a clear measurement.

* Adding More Qubits and Gates (Quantum Zeno Effect):
  * Yes, this is how you break past the 100% single-pass limit.
  * By splitting the rotation into N small steps of `theta = pi / N` and measuring after each step, the chance of exploding the bomb on any individual step scales as `(1/N)^2`. Across all N steps, the total explosion probability drops as `O(1/N)`.
  * Meanwhile, the photon is safely guided into the detected port. In our simulation with N = 12 steps, the score reached ~440%, proving that adding gates and ancilla qubits significantly improves the score.

---

### 3. For Track 2, how many measurements does your radar need compared with a classical brute-force search, and how does that scale with board size?

A classical player has to check the board cell by cell. Our quantum radar sweeps 1D superpositions across rows and columns.

| Board Size (N x N) | Classical Brute-Force (N^2) | Quantum Radar Queries (2N) | Efficiency Multiplier (N / 2) |
| :---: | :---: | :---: | :---: |
| 4x4 | 16 queries | 8 queries (4 rows + 4 cols) | 2.0x faster |
| 6x6 | 36 queries | 12 queries (6 rows + 6 cols) | 3.0x faster |
| 8x8 | 64 queries | 16 queries (8 rows + 8 cols) | 4.0x faster |
| 10x10 (Official Battleship) | 100 queries | 20 queries (10 rows + 10 cols) | 5.0x faster |

* Mathematical Scaling:
  * Classical search scales quadratically: `O(N^2)`.
  * Quantum Radar scales linearly: `O(2N)`.
  * Speedup factor is `N^2 / 2N = N / 2`.
  * On a full 10x10 Battleship board, the radar eliminates 80 out of 100 queries (an 80% reduction in measurements) while preserving interaction-free detection.

---

### 4. How long does it take to run your circuit?

* Simulator Run Time:
  * On Google Colab using `AerSimulator`, running an 8-measurement sweep with 5,000 shots per line (40,000 total shots) takes under 0.25 seconds.
* Real Quantum Hardware Estimation (IBM Quantum QPU):
  * On an IBM Quantum processor (like Heron or Eagle architecture):
    * Single-qubit `ry` gate pulse duration: ~25 to 35 nanoseconds.
    * Two-qubit `cx` gate pulse duration: ~250 to 400 nanoseconds.
    * Measurement / Readout pulse: ~700 to 1000 nanoseconds.
  * The total circuit execution time per shot is around 1.5 microseconds.
  * Adding qubit reset time (~1 microsecond), a full 5,000-shot job on a real QPU takes about 15 to 20 milliseconds per line.

---

### 5. How many parameters does it have?

Our setup uses three main parameters:
1. Rotation Angle (`theta`): The primary physical parameter. It controls the beam-splitter ratio, allowing us to balance detection sensitivity against explosion risk (`theta = pi/2` for 50%, `theta = pi/3` for 75%).
2. Recombination Phase (`-theta`): Ensures destructive interference at the dark port so clean ocean lines produce zero dark-port clicks.
3. Detection Threshold (`tau`): A classical post-processing threshold (set around 10% of total shots) to clearly distinguish genuine quantum collapse signals from background physical noise or dark counts.

---

### 6. Is your circuit tolerant to quantum noise?

Yes, the circuit shows strong resilience against both gate depolarizing errors and readout errors.

* Track 1 Noise Performance:
  * At 0% noise, the score is 74.8%.
  * At 8% physical error rate, the score smoothly drops to ~66.5%.
  * It remains well above the 50% baseline even under high noise levels.

* Track 2 Battleship Score Performance:
  * At 0% noise, the Battleship Score is 1.47.
  * At 8% noise, the score remains at ~1.34.
  * It easily outperforms the classical baseline score of 0.00.

* Why is it so tolerant?
  Because the circuit has a depth of only 4 and contains only a single two-qubit CNOT gate. Typical modern superconducting processors have two-qubit gate error rates around 0.5% to 1.0%, so our circuit runs well within the coherence limits of current NISQ hardware.

---

## Part 2: Addressing the Judging Criteria

### 1. Technical Aspects

* Accuracy and Safety:
  * We verified both the standard 50% baseline (`theta = pi/2`) and the improved 75% target (`theta = pi/3`). Our simulation numbers closely match the theoretical predictions (50.51% and 74.82%).
  * In Track 2, we showed that the radar pinpointed the true hidden ship coordinates while correctly identifying empty rows and columns.

* Algorithm Optimization and Scalability:
  * We avoided over-complicated multi-qubit Toffoli cascades and instead applied a tomographic 1D sweep across rows and columns. This reduces the classical query count from `O(N^2)` to `O(2N)` while keeping the quantum circuit depth at 4.
  * We demonstrated how this scales up to a regulation 10x10 Battleship board (reducing measurements from 100 to 20).

* Qiskit Ecosystem Usage:
  * The implementation uses Qiskit 1.x circuit building tools, the `transpile` pipeline, `qiskit_aer.AerSimulator`, and realistic noise modeling using `NoiseModel`, `depolarizing_error`, and `ReadoutError`.

* Classical Benchmarking:
  * We built a classical brute-force player side-by-side with the quantum radar. The classical search always hit every ship it queried (0% IF score), whereas our quantum radar achieved an IF score of ~73% alongside a 2x measurement reduction.

---

### 2. Originality and Uniqueness

* Tomographic Reconstruction in Battleship:
  * Instead of treating Battleship as a point-by-point guessing game, we reframed it as an interaction-free tomographic imaging problem, drawing inspiration from White et al. (1998).
  * Probing orthogonal line superpositions allows the radar to locate 2D coordinates through geometric line intersection with fewer measurements.

* Quantum Zeno Integration:
  * In Track 1, we implemented the multi-step Quantum Zeno circuit based on Kwiat et al. (1995/1999), demonstrating an IF score above 400%.

---

### 3. Usefulness and Practical Real-World Parallels

* Real-World Relevance:
  * While Battleship is a game, the underlying physics of interaction-free measurement has concrete practical applications:
    1. Imaging delicate biological specimens (like living cells or light-sensitive proteins) that would be damaged or altered by regular optical illumination.
    2. Inspecting photosensitive lithography masks or semiconductor components during manufacturing without exposing them.
    3. Low-Probability-of-Intercept (LPI) quantum radar concepts, where a sensor detects the presence of a stealth target without alerting the target to the probe signal.
    4. Non-destructive readout for quantum memory registers without collapsing stored states.

* Accessibility and Demonstrability:
  * The code runs out of the box in Google Colab without custom drivers or complex setups.
  * It includes clear ASCII grid visualizations showing the true fleet versus the radar-detected candidates, making it easy to demonstrate and explain to technical and non-technical audiences alike.