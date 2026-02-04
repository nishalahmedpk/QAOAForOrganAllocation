# QAOA for Organ Allocation Logistics

This repository contains my course project for **Quantum Architecture & Programming (Fall 2025, BITS Pilani Dubai)**, where I explored the use of the **Quantum Approximate Optimization Algorithm (QAOA)** to model the *logistic* component of an organ allocation problem.

The goal is not to claim quantum advantage, but to:
- Formulate a realistic organ allocation / routing subproblem as a **QUBO / Ising Hamiltonian**.
- Implement a **hybrid quantum–classical QAOA pipeline** using **PennyLane** (and optionally Qiskit).
- Empirically compare QAOA solutions with classical baselines on small instances.
- Reflect on scalability bottlenecks and what they imply for **distributed / LOCAL-model–inspired quantum algorithms**, which is the main reason I highlight this project in my Aalto AScI application.

> This project is work-in-progress and intentionally small-scale: it is meant as an honest exploration of current NISQ-era capabilities rather than a polished production library.

---

## Problem Overview

The real organ allocation problem is multi-objective and policy-heavy (fairness, medical urgency, geography, waiting time, etc.). In this project, I isolate a **logistics-style subproblem**:

- A set of donor organs and transplant centers.
- A graph encoding travel / routing constraints (flight legs, capacities, timing).
- A cost model that penalizes:
  - Violating capacity / matching constraints,
  - Excessive travel time,
  - Leaving available organs unused.

This is encoded as a **Quadratic Unconstrained Binary Optimization (QUBO)** problem:
- Binary variables represent assignment / routing decisions.
- Linear terms capture simple preferences.
- Quadratic terms encode constraints and pairwise interactions.
- The QUBO is converted to an **Ising Hamiltonian** suitable for QAOA.

The instances used here are intentionally small (toy), so that they are tractable on simulators and easy to inspect.

---

## Methodology

### 1. Classical Formulation

1. Define a small synthetic organ–center instance (few donors, few centers).
2. Construct the QUBO:
   - Encode constraints: one organ → at most one center, center capacities, simple routing feasibility.
   - Add soft penalties to discourage illegal solutions rather than removing them.
3. Solve the QUBO with a simple classical baseline:
   - Brute force for tiny instances, or
   - A straightforward heuristic (e.g., greedy or local search).

### 2. Quantum Formulation (QAOA)

- Map the QUBO to an **Ising Hamiltonian**.
- Implement QAOA using **PennyLane**’s `ApproxTimeEvolution` template for the cost Hamiltonian [web:54][web:52]:
  - Start from an equal superposition.
  - Alternate between cost and mixer unitaries for `p` layers.
  - Use a classical optimizer to tune the variational parameters.
- Sample bitstrings from the resulting quantum state.
- Post-process samples:
  - Select the bitstring which corresponds to least energy.
  - Decode the bitstring and compare to the classical optimum.

---

## How to Run

### Environment

- Python **3.12+**
- 
### Packages

- `pennylane`
- `numpy`
- `scipy`
- `matplotlib`
- `networkx` 

---

## Limitations and Future Work

This is a student project, and there are clear limitations:
- Instances are small and synthetic.
- Constraint modeling is simplified and does not reflect full clinical policy.
- Only shallow QAOA depths and basic optimizers are explored.
- Noise / hardware-specific effects are not thoroughly modeled.

## Academic Integrity
The initial prototype of this notebook was bootstrapped with the help of large language models (e.g., ChatGPT) for boilerplate code patterns (setting up PennyLane QAOA loops, basic plotting, etc.). 
All problem modeling choices, QUBO design, experiments, and analysis are my own, and I iteratively edited and refactored the generated code to match the specific requirements of the organ allocation setting.

I keep this repository public as an honest representation of my current level and as a base to build on in future research.

