# Track 2: Quantum Radar for Battleship

This folder contains the complete design, implementation, game demonstration, and scaling analysis for Track 2 of the QuantumVerse 4.0 Hackathon[cite: 30, 31].

---

## What is Track 2 Trying to Solve?

In standard Battleship, two players place ships on a grid (often 10x10) and take turns firing at coordinates[cite: 30]. The opponent calls out "hit" or "miss"[cite: 30].

In Track 2, the rules are turned around:
1. Every ship is treated like a delicate bomb[cite: 30]. If you hit a cell with a ship, it explodes[cite: 30].
2. Your goal is to find ALL hidden ships on the board while minimizing the chance of hitting them and minimizing the total number of measurements[cite: 30].

A classical player has no choice but to test cells one by one[cite: 30]. On a 4x4 board, that takes 16 measurements[cite: 30]. Every time a classical shot lands on a ship, it is a hit (explosion)[cite: 30]. That means the classical interaction-free detection score is strictly 0%[cite: 30].

A quantum player can use an Elitzur-Vaidman interferometer to probe cells in superposition[cite: 30]. Even better, instead of probing one cell at a time, we can probe an entire row or column at once[cite: 30].

---

## Why Our Algorithm is Kept Simple (and Why That Matters)

When designing a quantum radar, it is tempting to build huge circuits with lots of index qubits, multi-controlled Toffoli gates, and complex quantum state preparation. We intentionally did not do that, and here is why:

1. Circuit Depth and Gate Errors:
   A multi-controlled Toffoli across 4 or 5 qubits decomposes into dozens of 2-qubit CNOT gates on real hardware. That huge depth kills quantum coherence, causes depolarizing errors, and makes noise simulations blow up.
2. The Tomographic Principle (White et al. 1998):
   In optical interaction-free imaging, researchers do not address individual pixels with complicated quantum gates[cite: 32]. Instead, they send 1D probe beams across slices of the object (rows and columns) and reconstruct the image from where the beams were blocked[cite: 30, 32].

Our design probes each row and each column using a clean 2-qubit interferometer (Qubit 0 = probe path, Qubit 1 = ship in that line)[cite: 29, 30]. 
* Circuit depth is only 4.
* It uses only one CNOT gate per line probe.
* On a 4x4 board, it needs just 8 measurements (4 rows + 4 columns)[cite: 30].
* On a 10x10 board, it needs just 20 measurements (10 rows + 10 columns), compared to 100 classical measurements[cite: 30, 39].

This keeps the quantum algorithm simple, fast, and tolerant to noise, which directly answers the technical judging criteria[cite: 31, 32].

---

## How the Quantum Radar Locates Hidden Ships

Here is the exact step-by-step logic:

### Step 1: Probe Each Row
For each row `r`:
* We run the radar circuit with `theta = pi/3`[cite: 29, 30].
* If no ship is in row `r`, the photon undergoes constructive interference and 100% of counts land in bright port '00'[cite: 29]. No explosion occurs, and no detection clicks happen[cite: 29].
* If any cell in row `r` contains a ship, the photon has a chance to collapse safely and trigger the dark port '01'[cite: 29].
* If dark port counts exceed our detection threshold (or if an explosion occurs), row `r` is flagged as containing a ship[cite: 30].

### Step 2: Probe Each Column
For each column `c`:
* We run the exact same circuit along the vertical axis[cite: 30].
* If column `c` contains a ship segment, it gets flagged[cite: 30].

### Step 3: Intersect Flagged Lines
Any cell `(r, c)` where both row `r` was flagged AND column `c` was flagged is marked as a candidate ship cell[cite: 30]. 

For example, on our test 4x4 board:
* Ships were placed at: `(1, 1)` and `(3, 2), (3, 3)`
* The radar cleanly identified Rows 1 and 3 as active, and Columns 1, 2, and 3 as active[cite: 37].
* Rows 0 and 2, and Column 0 were identified as clean ocean with zero false alarms[cite: 37].
* The candidate intersection pinpointed:
  `(1, 1), (1, 2), (1, 3)`
  `(3, 1), (3, 2), (3, 3)`
All 3 true ship squares were detected inside this candidate set[cite: 37].

---

## Evaluation: The Official Battleship Score

The hackathon guidelines define the Battleship Score as:
`Battleship score = (Detection / Explosion) * (Brute-force Queries / Actual Queries)`[cite: 31]

This metric rewards two things:
1. How safely you find the ships (Detection / Explosion ratio)[cite: 31].
2. How efficiently you search compared to classical brute-force (16 / 8 = 2.0x efficiency gain on a 4x4 grid)[cite: 30, 31].

### Benchmark Results on 4x4 Fleet (5000 shots per line probe)
* Classical Brute-Force:
  * Queries: 16[cite: 30, 37]
  * Detections: 0 (classical measurements cannot be interaction-free)[cite: 30, 37]
  * Explosions: 3[cite: 37]
  * IF Score: 0.00[cite: 30, 37]
  * Battleship Score: 0.00[cite: 37]

* Quantum Radar (Standard theta = pi/2):
  * Queries: 8[cite: 30, 37]
  * Safe Detections: 6379 shots[cite: 37]
  * Explosions: 12407 shots[cite: 37]
  * IF Score: 0.51 (Matches 50% baseline)[cite: 29, 30, 37]
  * Efficiency Gain: 2.0x (16 / 8)[cite: 30, 37]
  * Battleship Score: 0.51 * 2.0 = 1.03[cite: 31, 37]

* Quantum Radar (Improved theta = pi/3):
  * Queries: 8[cite: 30, 37]
  * Safe Detections: 4600 shots[cite: 37]
  * Explosions: 6273 shots[cite: 37]
  * IF Score: 0.73 (Matches ~75% target)[cite: 29, 30, 37]
  * Efficiency Gain: 2.0x (16 / 8)[cite: 30, 37]
  * Battleship Score: 0.73 * 2.0 = 1.47[cite: 31, 37]

---

## Noise Study: Track 2 Resilience

We tested the radar with depolarizing gate noise and readout errors from 0% to 8% error rate[cite: 31, 38]. 
Because our radar uses a shallow 2-qubit circuit with only one CNOT per line, the score stays high:
* At 0% noise: Score = 1.47[cite: 37, 38]
* At 4% noise: Score = 1.39[cite: 38]
* At 8% noise: Score = 1.34[cite: 38]

Even under heavy 8% noise, the radar still scores 1.34, easily outperforming classical brute-force (0.00) while keeping the 2x speedup intact[cite: 30, 37, 38].

---

## Scalability Analysis: Larger Board Sizes

A key question asked in the prompt is: *"For Track 2, how many measurements does your radar need compared with a classical brute-force search, and how does that scale with board size?"*[cite: 31]

Because classical brute-force checks every square, it scales as `O(N^2)`[cite: 30, 39].
Our quantum radar checks rows and columns tomographically, so it scales as `O(2N)`[cite: 30, 39].

| Board Size (N x N) | Classical Queries (N^2) | Quantum Queries (2N) | Efficiency Multiplier (N / 2) |
| :---: | :---: | :---: | :---: |
| 4x4 | 16 | 8 | 2.0x[cite: 30, 39] |
| 6x6 | 36 | 12 | 3.0x[cite: 39] |
| 8x8 | 64 | 16 | 4.0x[cite: 39] |
| 10x10 (Official Battleship) | 100 | 20 | 5.0x[cite: 30, 39] |

On a regulation 10x10 Battleship board, our quantum radar needs only 20 measurements instead of 100, cutting the search effort by 80% while retaining the ~75% interaction-free detection ratio[cite: 30, 39].

---

## Detailed Walkthrough of the Track 2 Code

The file `track2_quantum_radar.py` is organized into 7 clear sections:

* Section 1: Installs required packages and initializes the Qiskit Aer simulator.
* Section 2 (`build_radar_line_circuit`): Defines the 2-qubit line-probe interferometer circuit using `ry(theta)`, a conditional `cx(0, 1)` if a ship is in that line, and `ry(-theta)` to complete interference[cite: 29].
* Section 3 (`BattleshipBoard`, `run_classical_baseline`): Creates the grid, places ships, displays the board in ASCII, and runs the classical brute-force search baseline[cite: 30].
* Section 4 (`run_quantum_radar`): Executes the 8 row and column quantum measurements, filters out empty ocean to prevent phantom counts, calculates the true IF score, and intersects flagged lines to pinpoint candidates[cite: 30, 37].
* Section 5: Runs the complete head-to-head comparison between Classical, Quantum (theta = pi/2), and Quantum (theta = pi/3), rendering the candidate overlay grid and the final scorecard table[cite: 30, 31, 37].
* Section 6: Injects depolarizing and readout noise models across 0% to 8% error rates and plots the physical degradation curve[cite: 31, 38].
* Section 7: Computes query counts across 4x4, 6x6, 8x8, and 10x10 boards, tabulates the efficiency multipliers, and plots the `O(2N)` vs `O(N^2)` scaling curves[cite: 30, 39].