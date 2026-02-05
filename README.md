# QAOA for Organ Allocation

A hybrid quantum-classical approach to solving the logistic optimization problem in organ transportation networks.

## Overview

This project applies Quantum Approximate Optimization Algorithm (QAOA) to a real-world problem: routing organs through interconnected hub networks while minimizing cost and respecting time constraints. The idea is classical computers struggle with combinatorial optimization, but quantum computers can explore exponentially many solutions simultaneously. By encoding the problem cleverly and letting quantum interference work in our favor, we can find good solutions on near-term quantum hardware.

## The Problem

Organ allocation is inherently a routing problem. You have an organ at point A, a patient at point B, and a limited time window. You need to find the cheapest path through available flights while ensuring connection times are feasible and you actually reach your destination. Add in multiple hubs and multiple potential paths, and this becomes NP-hard, meaning classical algorithms struggle as the network grows.

This is where quantum computing comes in. QAOA lets us encode the entire problem as a Hamiltonian and use quantum superposition to explore many solutions at once.

## How It Works

### Step 1: Encode as QUBO

The problem is reformulated as a Quadratic Unconstrained Binary Optimization (QUBO). Each flight is a binary variable (selected or not), and we define penalties for violating constraints:

- One flight must depart the origin
- One flight must arrive at the destination  
- Flow is balanced at intermediate hubs
- Connection times are at least 1.5 hours
- Total cost is minimized

The QUBO matrix Q encodes all of this. Solving QUBO is NP-hard, but it's perfect for QAOA.

### Step 2: Convert to Ising Hamiltonian

We map QUBO variables to spin states and construct a quantum Hamiltonian. This is the objective function that the quantum computer tries to minimize. The low-energy states of this Hamiltonian correspond to good solutions.

### Step 3: Run QAOA

The quantum circuit:
1. Starts in a superposition (all qubits in equal superposition)
2. Applies problem Hamiltonian (encodes the constraints)
3. Applies mixer Hamiltonian (explores the solution space)
4. Repeats these for 2 layers with tunable angles
5. Measures the qubits, collapsing to a bitstring

We optimize the angles using a classical optimizer (Adam) to minimize the expected energy. After 60 iterations, we sample 2048 times and decode the best bitstrings.

### Step 4: Validate & Compare

The bitstring tells us which flights to select. We decode it, check if it's actually a valid solution (respects all constraints), and compare against a classical baseline.

## What I Learned

The main insight is that problem encoding matters a lot. How you weight the constraints directly affects solution quality. Setting the mixer right also makes a difference because we use X rotations to explore the space efficiently.

One thing that surprised me: even on just 16 qubits with 2 layers, QAOA found competitive solutions. Of course, we're still in the NISQ era (Noisy Intermediate-Scale Quantum), so perfect solutions aren't guaranteed. Quantum hardware has coherence issues and gate errors. But the algorithm is robust enough to handle real noise and still produce meaningful results.

The hybrid approach is the real win here. We use quantum for the hard part (exploring huge solution spaces) and classical computing for validation and comparison. Neither alone is enough.

## Implementation

Everything lives in `final_qaoa.ipynb`. The notebook is structured as:

- **Cells 1-2:** Import libraries (PennyLane, NumPy, Pandas, Matplotlib, NetworkX)
- **Cells 3-5:** Generate a flight network and implement a classical baseline (label-correcting algorithm)
- **Cells 6-7:** Build the QUBO matrix by encoding all constraints
- **Cells 8-9:** Convert QUBO to an Ising Hamiltonian with proper Pauli operators
- **Cells 10-11:** Define the QAOA circuit and run optimization using Adam
- **Cells 12-13:** Decode the quantum results and validate against the classical solution

The entire pipeline is in one notebook for clarity and reproducibility.

## Tech Stack

- **PennyLane** for quantum circuits and optimization
- **NumPy & Pandas** for numerical work and data handling
- **Matplotlib & NetworkX** for visualization
- **Jupyter** for interactive development

## Running It

```bash
pip install pennylane numpy pandas matplotlib networkx jupyter
jupyter notebook final_qaoa.ipynb
```

Then just run the cells top to bottom. The notebook generates a flight network, solves it classically first, then runs QAOA and compares results.

## What's Next

Right now this works on a simulator with 16 qubits. Real applications would need:

- **Scaling to larger networks:** More qubits, better parameter tuning
- **Real hardware:** Testing on IBM Quantum or AWS Braket
- **Error correction:** Current quantum computers are too noisy for complex problems
- **Hybrid classical-quantum:** Maybe use neural networks to predict good starting angles for QAOA

The long-term goal is to show that quantum algorithms can actually help with real logistical problems, not just toy examples.

## On AI & Development

This project used AI assistance at specific points, mostly where it was actually useful rather than a shortcut:

- **Scaffolding:** Boilerplate imports, basic NumPy operations, and the initial structure of the flight network generator were accelerated with AI. These are the repetitive parts that don't require novel thinking.
- **Bug fixing:** When the constraint penalty weights weren't working right, I described the symptom and got debugging suggestions that pointed in the right direction (though I still had to verify and adjust the actual values).
- **Documentation:** The README structure came from a template, but the actual insights are hand-written from experience running the code.
- **Sanity checks:** I used AI to validate PennyLane syntax and make sure the Ising Hamiltonian mapping was correct.

What wasn't AI: the problem formulation, the decision to use 2 layers vs 3, tuning the penalty weights through trial and error, realizing that connection time constraints needed a different encoding, and the actual learning about what works and what doesn't. That's all manual iteration.

The takeaway is that AI fills the gaps where writing boilerplate by hand is tedious, but actual problem-solving still requires running code, interpreting results, and making informed choices. It's a productivity tool, not a substitute for thinking.


