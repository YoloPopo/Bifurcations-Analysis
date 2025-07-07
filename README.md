# Bifurcation Analysis of EEG Signals and the 2D Bratu Problem

## Overview

This repository contains a Jupyter Notebook implementing a bifurcation analysis on two distinct problems. The project serves as a practical application and exploration of numerical methods, particularly Newton's method, for identifying qualitative changes in dynamic systems.

1.  **EEG Signal Analysis**: The primary objective is to identify the bifurcation point in a time-series EEG signal, which corresponds to the moment a human subject transitions from an active problem-solving state to a resting state. The signal is modeled as a sum of two Gaussians, and the bifurcation is detected by analyzing the stability of this model's parameters over time.

2.  **2D Bratu Problem**: As a more complex, academic exercise, the notebook performs a bifurcation analysis on the 2D Bratu problem. It employs the Generalized Iterative Kantorovich Method (GIKM) on a Taylor-approximated version of the Bratu PDE to trace the solution curve and identify the critical turning point.

## Project Structure

```
.
├── Bifurcations Analysis.ipynb   # The main Jupyter Notebook with all the analysis and code.
├── Brain_EEG.npy                 # The dataset containing EEG power data.
└── README.md                     # This documentation file.
```

## Dataset: EEG Brain Activity

- **File**: `Brain_EEG.npy`
- **Source**: Electroencephalography (EEG) data from a human subject.
- **Description**: The data represents the power spectrum of brain activity over time. It was recorded during a two-stage experiment:
    1.  **Active State**: The subject is mentally solving a mathematical problem.
    2.  **Resting State**: The subject has finished the problem and is at rest.
- **Properties**:
    - **Shape**: (700, 79694), representing 700 frequency bins and 79,694 time samples.
    - **Sampling Frequency**: 500 Hz.
    - **Duration**: ~159 seconds.

---

## Part 1: EEG Bifurcation Analysis Methodology

The core task is to pinpoint the exact time of the cognitive state transition. This is framed as a bifurcation problem where the signal's qualitative nature changes.

### 1.1. Signal Characterization and Modeling

- The EEG power spectrum at any given time is modeled as a sum of two Gaussian functions. This simplification assumes that the brain activity is dominated by two primary frequency components.
- The objective function is the squared L2-norm of the difference between the real signal and the two-Gaussian model:
  $$ J(\omega_1, \omega_2) = \int_{0}^{\Omega_{max}} \left( (G(\omega, \omega_1, \sigma) + G(\omega, \omega_2, \sigma)) - S_{real}(\omega) \right)^2 d\omega $$
- To find the optimal mean frequencies (ω₁, ω₂) that best fit the data, we solve the system of non-linear equations derived from the first-order optimality condition, ∇J = 0.

### 1.2. Custom Newton's Method Implementation

- A Newton solver was implemented from scratch in Python to find the roots of the system ∇J = 0.
- The Jacobian of the system (i.e., the Hessian of the objective function J) is approximated numerically using finite differences.
- This solver is first validated on a synthetic two-Gaussian signal with known parameters to ensure correctness.

### 1.3. Bifurcation Point Detection

- The analysis proceeds by applying the Newton solver within a sliding window across the time-series data.
- At each window, the determinant of the Jacobian matrix (det(J)) is computed at the optimal solution.
- **The bifurcation point is identified as the time where det(J) approaches zero.** A near-singular Jacobian indicates that the model parameters are becoming ill-defined, which often happens when the two Gaussian components merge or their roles become indistinguishable—a sign of a qualitative change in the system.

### 1.4. Key Findings for EEG

- The bifurcation point was successfully identified at approximately **t = 59.25 seconds**.
- **Before Bifurcation**: The signal spectrum is distinctly **bimodal**, with two prominent peaks, likely corresponding to the complex cognitive load of problem-solving.
- **At and After Bifurcation**: The spectrum transitions to a **unimodal** shape, with a single dominant peak. This suggests a shift to a more stable, less complex brain state, consistent with the transition to rest.
- The convergence of the two Gaussian means (ω₁ and ω₂) towards each other around the bifurcation point further validates this finding.

---

## Part 2: 2D Bratu Problem Analysis

This section serves as an advanced application of the bifurcation analysis framework to a well-known nonlinear PDE.

### 2.1. Problem Formulation

- The 2D Bratu problem is defined by the PDE $ \Delta u + \lambda e^u = 0 $ on a unit square with zero Dirichlet boundary conditions.
- **Approximation**: To simplify the problem, the exponential term is replaced with its 3rd-order Taylor series expansion: $ e^u \approx 1 + u + \frac{u^2}{2} + \frac{u^3}{6} $. This is a significant simplification that affects the quantitative accuracy of the results.

### 2.2. Solution Methodology: IGKM & Bifurcation Analysis

- The PDE is reduced to a system of coupled, nonlinear Ordinary Differential Equations (ODEs) using the **Generalized Iterative Kantorovich Method (GIKM)** with a one-term approximation ($u(x,y) = h(x)g(y)$).
- Two different approaches for bifurcation analysis were implemented and compared:
    1.  **Direct Operator-Based IGKM**: Uses a Newton-Shooting method to solve the ODEs and a pseudo-arc-length continuation algorithm to trace the solution branch around the turning point. The bifurcation is detected where the determinant of the shooting method's Jacobian becomes zero.
    2.  **Variational Method**: Uses IGKM to find the optimal *shapes* of h(x) and g(y). The problem is then reduced to a simple quadratic equation for the solution's amplitude, *A*. The bifurcation is identified where the second derivative of the potential functional with respect to the amplitude is zero ($d^2\Xi/dA^2 = 0$).

### 2.3. Key Findings for Bratu Problem

- Both methods successfully identified the classic "S-shaped" bifurcation curve with a distinct turning point (fold bifurcation).
- **Direct Operator Method**: Predicted a critical value of **λ_crit ≈ 7.18**.
- **Variational Method**: Predicted a critical value of **λ_crit ≈ 7.80**.
- **Full PDE Solver (Benchmark)**: A provided arc-length continuation solver on the *full* (non-approximated) PDE yields **λ_crit ≈ 6.80**.

The discrepancy between the methods and the benchmark highlights the impact of the approximations made. The operator-based method is more accurate for the approximated PDE, while the variational approach is conceptually simpler but less precise.

## How to Run the Analysis

1.  **Prerequisites**: Ensure you have a Python environment (e.g., Anaconda) with the following libraries installed:
    ```
    pip install numpy matplotlib numba scipy mne tqdm
    ```
2.  **Dataset**: Make sure the `Brain_EEG.npy` file is in the same directory as the notebook.
3.  **Execution**: Open `Bifurcations Analysis.ipynb` in Jupyter Lab or Jupyter Notebook and run the cells sequentially from top to bottom.

## Critical Analysis and Future Work

While this project successfully demonstrates the application of bifurcation theory, several areas could be improved:

- **EEG Model**: The two-Gaussian model is a strong simplification. A Gaussian Mixture Model (GMM) with a variable number of components and learnable variances could provide a more accurate fit and potentially a clearer bifurcation signal.
- **Bratu Approximation**: The Taylor series approximation is the main source of inaccuracy in the Bratu analysis. The methods could be extended to handle the full exponential nonlinearity, possibly by incorporating it directly into the Newton-Raphson solver for the ODEs.
- **Numerical Stability**: The custom Newton solver is functional but lacks the robustness of heavily optimized library functions (e.g., from `scipy.optimize`). For a more general tool, replacing the custom implementation would be beneficial.
- **Bifurcation Detection**: The method of finding the "first local minimum" of the Jacobian determinant is heuristic. More rigorous methods, such as tracking the eigenvalues of the Jacobian and identifying where one crosses zero, would provide a more robust detection mechanism.

## License

This project is open-source and available under the MIT License.
