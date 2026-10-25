```markdown
# Track 1: Interaction-Free Bomb Tester

This folder contains the complete implementation, testing, and noise study for Track 1 of the QuantumVerse 4.0 Hackathon[cite: 29].

---

## What is Track 1 About?

The core problem here is simple to state: imagine you have a box that might contain a bomb so sensitive that if even a single photon touches it, it detonates[cite: 29]. In classical physics, there is no way to verify the bomb is inside without touching it, which means it will always blow up if it is there[cite: 29].

In 1993, physicists Avshalom Elitzur and Lev Vaidman showed that quantum mechanics provides a way out using an optical interferometer (a Mach-Zehnder interferometer)[cite: 29]. By splitting a photon into a superposition of two paths (one path going through the bomb chamber and one path going around it), quantum interference allows us to detect the bomb without detonating it a certain percentage of the time[cite: 29].

Our task in Track 1 was to:
1. Build the clean Mach-Zehnder circuit with no bomb and show the output is 100% deterministic[cite: 29].
2. Build the bomb circuit and report the probabilities for detection, explosion, and inconclusive results[cite: 29].
3. Improve the score from the 50% baseline to ~75% by tuning the rotation angle[cite: 29, 30].
4. Push the score even higher using repeated measurements (the Quantum Zeno effect)[cite: 30].
5. Test how real physical quantum noise impacts the score[cite: 29].

---

## How the Quantum Circuit Works

We represent the entire setup with two qubits:
* Qubit 0 (Path Qubit): Represents the photon's path. State |0> means the reference arm (the path that avoids the bomb), and state |1> means the probe arm (the path that goes through the bomb)[cite: 29].
* Qubit 1 (Bomb Qubit): Represents the bomb status. State |0> means intact/dormant, and state |1> means exploded[cite: 29].

### 1. The Clean Circuit (No Bomb)
First, a photon hits a beam splitter. We model a beam splitter using a rotation gate around the Y-axis: `ry(theta, 0)`.
For a standard 50:50 beam splitter, theta = pi/2[cite: 29]. This puts the photon into an equal superposition:
`|psi> = (|0> + |1>) / sqrt(2)`

When there is no bomb in the arm, the photon paths recombine at the second beam splitter. To make the paths interfere properly, the second beam splitter applies `ry(-theta, 0)`. 
Because `ry(-theta) * ry(theta) = Identity`, the rotation completely undoes itself:
`ry(-theta) ry(theta) |0> = |0>`

The photon always ends up in state |0> (Detector 0, the bright port)[cite: 29]. It never ends up in state |1> (Detector 1, the dark port)[cite: 29]. This is 100% deterministic destructive interference[cite: 29].

### 2. What Happens When a Bomb is Added?
If a bomb is placed in the probe arm, we model the interaction with a CNOT gate (`cx(0, 1)`), where Qubit 0 is the control and Qubit 1 is the target[cite: 29]. 
* If the photon takes the reference path (|0>), the bomb qubit stays |0> (no explosion)[cite: 29].
* If the photon takes the probe path (|1>), the bomb qubit flips to |1> (bomb explodes!)[cite: 29].

The bomb acts like an involuntary measurement that collapses the photon's superposition[cite: 29]:

1. Bomb Explodes (50% probability at theta = pi/2):
   The photon took the probe path and triggered the bomb. Qubit 1 measures as 1[cite: 29].

2. Superposition Collapses Safely (50% probability at theta = pi/2):
   The photon took the reference path. The bomb did not go off, but the wavefunction collapsed to |0> on Qubit 0[cite: 29]. Now, this photon continues to the second beam splitter `ry(-theta, 0)`. 
   Because there is no longer a probe-arm photon to interfere with it, the photon has a 50/50 chance of exiting through either port:
   * It hits Detector 1 (Dark Port, outcome '01'): This happens with 25% total probability (50% * 50%)[cite: 29]. But remember: Detector 1 could NEVER click if the arm was empty[cite: 29]! The moment Detector 1 clicks, we know for 100% sure that a bomb is present, and yet the bomb did not explode[cite: 29]. This is our Interaction-Free Detection[cite: 29].
   * It hits Detector 0 (Bright Port, outcome '00'): This happens with 25% total probability (50% * 50%)[cite: 29]. This is inconclusive, because Detector 0 clicks even when no bomb is present[cite: 29].

### 3. Calculating the Interaction-Free Score
The evaluation metric given in the hackathon prompt is:
`Score = Detection Probability / Explosion Probability`[cite: 29]

For the standard setup (theta = pi/2):
* Detection = 25%[cite: 29]
* Explosion = 50%[cite: 29]
* Score = 25% / 50% = 50% (0.50 baseline)[cite: 29, 30]

---

## How We Improved the Score

### Method A: Tuning the Rotation Angle (theta = pi/3)
If you do the math for any arbitrary beam splitter angle theta, the probabilities turn out to be:
* Explosion probability: `P_exp = sin^2(theta / 2)`[cite: 29]
* Detection probability: `P_det = cos^2(theta / 2) * sin^2(theta / 2)`[cite: 29]

So the score is:
`Score = P_det / P_exp = cos^2(theta / 2)`[cite: 29, 34]

If we change the angle from theta = pi/2 down to theta = pi/3 (which is 60 degrees):
* Explosion = `sin^2(pi / 6) = (1/2)^2 = 25%`[cite: 29]
* Detection = `cos^2(pi / 6) * sin^2(pi / 6) = (3/4) * (1/4) = 18.75%`[cite: 29]
* Score = `18.75% / 25% = 75% (0.75)`[cite: 29, 30]

Our simulation results matched this: we got 24.99% explosion, 18.70% safe detection, and an IF score of 74.82%[cite: 33].

### Method B: The Quantum Zeno Effect (Beating 100% Score)
In a single-pass circuit, `cos^2(theta / 2)` can never exceed 1.0 (100%), even if theta gets close to zero[cite: 29, 34]. 
To get past 100%, we followed the research paper by Kwiat et al. (1995/1999) on the Quantum Zeno effect[cite: 30]. 

Instead of rotating by a big angle all at once, we rotate the path qubit by a tiny angle `theta = pi / N` across N steps[cite: 5, 30]. After each small rotation, the photon interacts with a fresh ancilla qubit representing the bomb[cite: 5]. Because the rotation was tiny, the probability of triggering the bomb on each step is proportional to `(1/N)^2`[cite: 5]. Across all N steps, the total explosion probability drops as `O(1/N)`[cite: 5, 30]. Meanwhile, the photon is gently guided into the detected state[cite: 5].

In our code, running N = 12 steps gave:
* Safe Detection: ~81.5%
* Explosion: ~18.5%
* IF Score: ~440% (4.4x more detections than explosions!)[cite: 35]

---

## Noise Study: What Happens on Real Hardware?

We added realistic physical noise using Qiskit Aer's `NoiseModel`[cite: 29, 36]:
* 1-qubit depolarizing error on `ry` gates[cite: 5, 36]
* 2-qubit depolarizing error on `cx` gates (set to 2x the 1-qubit rate)[cite: 5, 36]
* Readout error (measuring 0 as 1, or 1 as 0)[cite: 5, 36]

When we swept physical error rates from 0% up to 8%, the score decayed smoothly from 74.8% down to ~66.5%[cite: 36]. The circuit showed good noise resilience because it has a depth of only 4 and uses a single two-qubit gate, meaning it stays well above the classical 0% line even with high noise levels[cite: 30, 36].

---

## Detailed Walkthrough of the Track 1 Code

The file `track1_bomb_tester.py` is divided into 6 clear sections:

* Section 1: Installs libraries and imports Qiskit, AerSimulator, numpy, and matplotlib.
* Section 2 (`build_clean_interferometer`): Builds the 1-qubit circuit with `ry(theta)` followed by `ry(-theta)`. Verifies that 100% of counts land in Detector 0[cite: 29].
* Section 3 (`build_bomb_tester`, `evaluate_bomb_circuit`): Implements the 2-qubit bomb circuit with `cx(0, 1)`. Runs 30,000 shots and prints the official probability table comparing theta = pi/2 against theta = pi/3[cite: 29, 30, 33].
* Section 4: Sweeps theta from 0.25 to pi/2 rad over 35,000 shots per point. Plots the simulation points against the theoretical `cos^2(theta / 2)` curve and marks the 50% and 75% reference lines[cite: 29, 30, 34].
* Section 5 (`run_quantum_zeno_tester`): Builds an (N+1)-qubit circuit that repeatedly rotates by `pi/N` and measures the bomb[cite: 5]. Plots the score scaling from N=2 up to N=12 steps[cite: 35].
* Section 6: Applies depolarizing and readout noise to the improved circuit (theta = pi/3) across error rates from 0% to 8% with 40,000 shots per point, plotting the noise decay curve[cite: 29, 30, 36].