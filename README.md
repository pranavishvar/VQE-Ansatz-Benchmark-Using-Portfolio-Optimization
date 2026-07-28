# VQE Portfolio Optimization

Solves portfolio optimization using VQE on qubits. Benchmarks a hardware-efficient ansatz against a UCCSD-inspired ansatz versus exact diagonalization, comparing ground-state accuracy and runtime across problem sizes. Finds UCCSD recovers the exact solution while hardware-efficient degrades.

## Overview

Portfolio selection is framed as a constrained combinatorial optimization problem: choose a subset of `k` assets out of `m` that minimizes risk while maximizing expected return. This is mapped onto a system of qubits using an Ising-style encoding, where each qubit represents whether an asset is included (`1`) or excluded (`0`) in the portfolio.

The resulting cost Hamiltonian has three components:

```
H = H_risk − λ · H_returns + penalty · H_constraint
```

- **H_risk** — quadratic term built from the asset covariance matrix
- **H_returns** — linear term built from expected asset returns
- **H_constraint** — penalty term enforcing the cardinality constraint (exactly `k` assets selected), implemented as `(Σx_i − k)²`

The ground state of `H` corresponds to the optimal (or near-optimal) portfolio.

## Methods compared

| Method | Description |
|---|---|
| **Exact diagonalization** | `numpy.linalg.eigh` on the full Hamiltonian matrix — ground truth, but scales exponentially with the number of assets |
| **VQE — hardware-efficient ansatz** | Generic layered `Ry` rotations + `CX` entangling gates, no problem structure baked in |
| **VQE — UCCSD-inspired ansatz** | Built from single/double excitation (ladder) operators that conserve particle number, respecting the cardinality constraint by construction |

Both VQE variants are optimized with `scipy.optimize.minimize` (COBYLA) against a statevector-based energy expectation, then sampled (1000 shots, `qasm_simulator`) to extract the most probable bitstring as the final answer.

## Results

Benchmarked across `m = 3, 4, 5, 6` assets (seed = 42):

- **UCCSD** recovered the exact ground-state bitstring and ground energy at every problem size tested.
- **Hardware-efficient** matched classical results at `m = 3`, but began to diverge from `m = 4` onward — both in bitstring and in energy (~5% off at `m = 4`).
- **Classical diagonalization** was faster than both VQE methods at every size tested, as expected — exact diagonalization is trivial at this qubit count, and this project isn't claiming quantum advantage, just comparing solution quality and optimizer behavior between ansätze.

This suggests that ansatz expressibility isn't the only thing that matters — respecting the problem's underlying symmetry (particle-number conservation, in this case) meaningfully improves solution quality on constrained combinatorial problems, even at small scale.

## Repo structure

```
.
├── Portfolio_optimization_using_VQE.ipynb   # full notebook: Hamiltonian construction, both VQE ansätze,
│                                             # classical baseline, benchmarking, and plots
└── README.md
```

## Requirements

```
qiskit[all]
qiskit-aer
numpy
scipy
matplotlib
```

Install with:

```bash
pip install qiskit[all] qiskit-aer numpy scipy matplotlib
```

## Running

Open `Portfolio_optimization_using_VQE.ipynb` in Jupyter or Google Colab and run all cells. The benchmarking loop at the bottom iterates over `m_list = [3, 4, 5, 6]`; edit this list to test other portfolio sizes (note: exact diagonalization becomes infeasible well before ~20 qubits).

## Limitations & future work

- Energy is evaluated on the exact statevector during optimization but read out via shot-based sampling — a hybrid setup useful for demonstration but not fully representative of real hardware noise.
- Results are from a single seed / single optimizer run per problem size; no variance across multiple seeds or optimizer restarts.
- Natural next steps: add a noise model to test whether UCCSD's advantage holds under realistic hardware noise, compare against QAOA, and evaluate scaling behavior with more assets using approximate (rather than exact) classical baselines.

## License

MIT
