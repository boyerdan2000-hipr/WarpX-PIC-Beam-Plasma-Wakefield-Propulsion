# PyWarpX: Macroscopic Ambipolar Towing for Aneutronic Fusion Ignition

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/boyerdan2000-hipr/WarpX-PIC-Beam-Plasma-Wakefield-Propulsion/blob/main/hipr_picmi_dynamic_gradient.ipynb)

## Overview
This repository contains a specialized PyWarpX Particle-in-Cell (PIC) simulation framework developed to model macroscopic ambipolar towing in a heavy-ion plasma. The primary objective is to evaluate the kinematic conditions required for continuous beam-to-plasma momentum transfer, specifically targeting the $300 \text{ keV}$ to $600 \text{ keV}$ relative center-of-mass velocity threshold necessary for $p-^7Li$ aneutronic fusion ignition.

## Physics Parameters
The baseline simulation models the continuous injection of an 80-packet sawtooth ion train into a uniform cylindrical plasma target over 320,000 computational time steps.

*   **Driver Beam:** $Au^{51+}$ (Gold), $4.0 \text{ A}$ peak current, $500 \text{ \mu m}$ radius
*   **Beam Velocity:** $v_b = 0.2c$
*   **Target Plasma:** $^7Li^+$ (Lithium) at a density of $1.0 \times 10^{19} \text{ m}^{-3}$
*   **Fusion Target:** $300-600 \text{ keV}$ astronomical S-factor plateau
*   **Domain:** 2D Cylindrical (RZ) geometry, $256 \times 2048$ cells

## Computational Methodology: OOM Crash Bypass
Executing a 320,000-step high-fidelity PIC simulation reliably exceeds the 24GB VRAM hardware limits of single-node cloud GPUs (e.g., Google Colab L4). This codebase implements a custom in-memory compression and Drive-checkpointing pipeline to bypass these constraints:

1.  **Direct AMReX Extraction:** Macroscopic field data ($E_z$) is pulled directly from the C++ backend at 4,000-step intervals to prevent rendering overhead.
2.  **In-Memory Compression:** Extracted arrays undergo a 10-cell moving average for anti-aliasing, a 10x spatial downsampling stride, and are cast to lightweight 32-bit floats.
3.  **Drive Checkpointing:** The compressed `.npy` chunks and raw peak gradient statistics are serialized directly to Google Drive.
4.  **VRAM Purging:** The massive original AMReX arrays are forcefully deleted from RAM after each packet injection, preventing out-of-memory (OOM) runtime crashes and surviving overnight session disconnects.

## Execution Instructions

### 1. Cloud Execution (Recommended)
Click the "Open in Colab" badge above to launch the interactive notebook. 
1. Mount your Google Drive when prompted to authorize directory creation.
2. Execute **Cell 1** to compile the WarpX environment.
3. Execute **Cell 2** to run the 320,000-step execution loop. Progress is automatically saved to Drive every 4,000 steps.
4. If the session disconnects during an overnight run, restart the notebook, run **Cell 1**, and skip directly to **Cell 3**. The script will automatically load the preserved checkpoints from disk.

### 2. Post-Processing & Visualization
*   **Cell 3:** Reconstructs the 1D macroscopic DC field using a 4th-order Butterworth low-pass filter to reveal the ambipolar towing envelope.
*   **Cell 4:** Renders the 2D spatiotemporal heatmap of the longitudinal wakefield evolution.
*   **Cell 5:** Extracts the longitudinal phase space ($z$ vs $p_z$) directly from the active AMReX backend to verify bulk momentum transfer into the target ions. *(Note: Cell 5 must be executed immediately following Cell 2 in an active session; massive particle distributions are not checkpointed).*

## License
Distributed under the Apache 2.0 License. See `LICENSE` for more information.
