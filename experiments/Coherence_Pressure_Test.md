# Coherence Pressure Test

This document describes a **compressivity-driven interference test** inspired by the Seamless Hourglass Model (SHG).

> Goal: test whether realized histories exhibit a small bias toward globally more compressible (shorter-description) outcome sequences, beyond what is predicted by standard quantum mechanics plus a known controller policy.

## 1. Experimental setup (conceptual)

- Interferometer: Mach–Zehnder or double-slit with fast, switchable which-path marking.
- Two configurations:
  - **Interference ON**: which-path information is erased, high-visibility interference expected.
  - **Interference OFF**: which-path information is marked, no interference fringes, particle-like statistics.
- Detector array records impact positions or equivalent outcome data for each trial.

## 2. Closed-loop controller

- A classical controller chooses the configuration for each new trial (ON/OFF) based only on **past recorded data**.
- Objective: minimize the incremental description length (e.g. under a fixed compressor) of the entire experiment log.
- The controller has *no access to future outcomes*; its policy is fully defined and reproducible.

## 3. Competing hypotheses

- **H₀ (Standard QM)**:  
  Outcome statistics and compressibility are fully explained by:
  - the controller policy,
  - quantum probabilities for each configuration,
  - finite-sample fluctuations.

- **H₁ (Compression-pressure / SHG-inspired)**:  
  Histories that reduce global description length of the log are **slightly favored**, yielding:
  - excess compressibility beyond policy-matched null simulations,
  - subtle deviations in configuration/outcome correlations that cannot be absorbed into controller details.

## 4. Data analysis sketch

1. Define a fixed, open-source compressor (e.g. LZ-based) and a standard encoding for the experiment log.
2. Run the closed-loop protocol to produce a long sequence of (configuration, outcome) pairs.
3. Generate many synthetic null logs under H₀ using the same controller policy and quantum probabilities.
4. Compare the real log's compressed length to the distribution from null logs.
5. Look for statistically significant excess compressibility (or other deviations) that survive robustness checks.

## 5. Status

This protocol is **conceptual** and serves as a heuristic probe for one of SHG's motivating ideas (compression bias in realized histories). A null result remains fully compatible with standard quantum mechanics and does not falsify SHG as an interpretation, but a robust positive signal would require intense scrutiny and independent replication.
