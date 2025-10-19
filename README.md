# MA-RQAOA: Simulation Framework

This repository contains the Python code for the simulation and comparison of the **Majorana-Aware Recursive Quantum Approximate Optimization Algorithm (MA-RQAOA)** against other approaches, as described in the paper *"MA-RQAOA: A System-Algorithm Co-Design Framework for Coherence-Limited Topological Quantum Computing"*.

The script simulates solving Max-Cut problems on random 3-regular graphs, taking into account the physical constraints of a simulated topological quantum computer (TQC).

## 🚀 Core Concepts

The framework compares three different algorithms for solving the Max-Cut problem using QAOA:

1.  **MA-RQAOA (Proposed)**: A hardware-aware algorithm that integrates a penalty function into the classical optimization loop. This penalizes parameters that lead to quantum circuits whose execution time exceeds the hardware's coherence time. This gently guides the optimizer towards physically feasible, high-quality solutions.

2.  **Clipped RQAOA**: A simpler hardware-aware approach. It returns an infinite cost value as soon as a parameter set leads to a non-executable circuit. This creates a hard, insurmountable boundary in the optimization landscape, which can trap the optimizer in suboptimal local minima.

3.  **Naive RQAOA**: A hardware-agnostic standard approach. It optimizes solely for energy (approximation ratio) without considering the execution time. The physical feasibility is only checked after the optimization is complete, often resulting in theoretically good but practically unusable solutions.

### Simulated Hardware Model

The simulation is based on a TQC model with the following temporal parameters:

-   **Maximum Coherence Time (`T_MAX`)**: The maximum duration a quantum state can be maintained.
-   **Classical Loop Latency (`T_CLASSICAL`)**: The time required for classical processing between quantum operations.
-   **Primitive Gate Times**:
    -   `TAU_B`: Time for a fast braiding operation (1-qubit gate).
    -   `TAU_SWAP`: Time for an adjacent SWAP gate to move qubits.
    -   `TAU_F`: Time for a slow, costly fusion operation (T-gate), which is not explicitly used here but mentioned in the paper as motivation.

The effective time available for the quantum circuit is `COHERENCE_LIMIT = T_MAX - T_CLASSICAL`.

## ⚙️ Code Features

-   **Problem Generation**: Creates random 3-regular graphs for the Max-Cut problem and a 2D grid topology for the simulated hardware.
-   **Quantum Simulation with Qiskit**: Uses `qiskit_aer` for a high-performance simulation of the QAOA circuits.
-   **Exact Benchmark**: Employs the `pulp` library to calculate the exact optimal solution for the Max-Cut problem to determine the approximation ratio (`alpha`).
-   **Automated Simulation & Analysis**: Runs the simulation for various problem sizes and QAOA depths (`p_depth`), saves the raw data, and automatically generates the plots shown in the paper.
-   **Reproducibility**: The script is designed to reproduce the key results of the paper, including:
    -   Approximation ratio vs. problem size
    -   Feasibility rate vs. problem size
    -   Optimization trajectory for a challenging instance

## 📦 Installation

Ensure you have Python 3.8+ installed. Clone the repository and install the required dependencies using pip:

```bash
pip install numpy networkx scipy matplotlib tqdm pandas pulp qiskit-aer
```

## ▶️ Execution

The script can be run directly from the command line. It executes all steps sequentially: simulation, trajectory capture, and plot generation.

```bash
python <script_name>.py
```

The workflow is as follows:

1.  **`run_and_save_full_simulation()`**: Executes the main simulation. This may take some time depending on the configuration (especially `N_RUNS` and `problem_sizes`). The results are saved to `multi_p_simulation_results_qiskit1.1.csv`.
2.  **`capture_and_save_trajectory()`**: Searches for a "challenging" problem instance (where the execution time is close to the coherence limit) and records the optimization path for each algorithm. The data is saved to `improved_trajectory_results_qiskit1.1.csv`.
3.  **`plot_results_from_csv()`**: Reads the two CSV files and generates a multi-page PDF file (`multi_p_MA_RQAOA_Simulation_Results_Qiskit1.1.pdf`) with all relevant plots.

## 🔧 Configuration

All important simulation and hardware parameters can be adjusted at the beginning of the script in the section `1. SIMULATED HARDWARE- AND SIMULATIONSPARAMETER`:

```python
# Hardware Parameters
T_MAX = 1.0
T_CLASSICAL = 0.1
TAU_B = 0.001
TAU_SWAP = 0.003
TAU_F = 0.2

# Simulation Parameters
P_DEPTHS = range(1, 6)      # QAOA depths to test
PENALTY_MU = 10.0           # Penalty strength for MA-RQAOA
N_RUNS = 50                 # Number of runs per configuration
OPTIMIZER_MAX_EVALS = 300   # Max evaluations for the COBYLA optimizer

# Output File Paths
RESULTS_CSV_PATH = "multi_p_simulation_results_qiskit1.1.csv"
TRAJECTORY_CSV_PATH = "improved_trajectory_results_qiskit1.1.csv"
OUTPUT_PDF_PATH = "multi_p_MA_RQAOA_Simulation_Results_Qiskit1.1.pdf"
```

## 📄 Output Files

After a complete run, the script generates the following files:

-   **`multi_p_simulation_results_qiskit1.1.csv`**: Contains the detailed results of each individual simulation run, including problem size, algorithm, approximation ratio (`alpha`), feasibility, and calculated execution time.
-   **`improved_trajectory_results_qiskit1.1.csv`**: Stores the proposed circuit times at each optimizer iteration for the special trajectory analysis.
-   **`multi_p_MA_RQAOA_Simulation_Results_Qiskit1.1.pdf`**: A comprehensive PDF file containing all result plots for analysis and comparison of the algorithms.
