# dandelion-optimizer_MPPT
Dandelion Optimizer MPPT (DO-MPPT): A simple MATLAB implementation of the Dandelion Optimizer (DO) algorithm for Maximum Power Point Tracking (MPPT) in photovoltaic systems. This project aims to find the optimal duty cycle (D) to maximize power output from solar panels.

This project is licensed under GNU GPL v3 with the following additional restrictions:

✅ You may modify and use this code, but any changes must be shared back with the community (copyleft).

❌ You may not distribute it for personal use without sharing it back.

❌ You may not sell or resell this code in any form.

By using this code, you agree to comply with these rules.

Disclaimer: Use at your own risk—this is a proof-of-concept, not battle-tested code.

## Key Features
Dandelion Optimizer: Inspired by the wind-dispersed flight of dandelion seeds, balancing exploration and exploitation.

Three Adaptive Stages:

Rising Stage: Global exploration under "weather conditions".

Decline Stage: Exploitation around promising regions.

Landing Stage: Levy-flight-driven convergence to the elite solution.

Adaptive Parameters: Self-tuning coefficients (alpha, beta, delta) for dynamic optimization.

## How It Works
Initialization: Generates a population of random duty cycles.

Fitness Evaluation: Measures power output (currently a dummy function; replace with real PV measurements).

Iterative Optimization: Updates candidate solutions across three stages, merges populations, and selects elites.

Output: Returns the best duty cycle D after Max_iterations.

## Limitations & Known Flaws
Dummy Fitness Function: The current evaluateMPPT is a placeholder (maximizes at D = 0.5). Replace this with real hardware integration!

Parameter Tuning: Coefficients (beta, delta, etc.) may need tuning for your system.

Convergence: No early stopping criteria—uses fixed iterations.

Code Efficiency: Not optimized for real-time embedded systems.

## How You Can Help
Test with Real Data: Replace the dummy evaluateMPPT and share results!

Improve the Algorithm: Suggest better parameter adaptation or convergence strategies.

Optimize the Code: Port to C/Python or reduce computational overhead.

## Contribution & Contact
Open to critiques, bug reports, and collaborations! Feel free to open an issue for discussions.
