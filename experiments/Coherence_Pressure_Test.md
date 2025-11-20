# Coherence pressure test

# Compressivity-Driven Interference Test — Protocol (v0.1)

---

## 0) One-line summary

Test whether a weak "coherence pressure" biases experimental histories toward more compressible logs by adaptively toggling an interferometer between interference (which-path erased) and which-path modes to minimize the description length of the *entire* event journal.

---

## 1) Hypothesis & prediction

**H₀ (Standard QM / Control):** Given a controller whose future settings depend only on *past* data (no look-ahead), the joint distribution of outcomes and settings equals the composition of the controller’s policy with ordinary quantum probabilities. Therefore, the compressibility of the full log is fully explained by the policy; no extra bias appears.

**H₁ (Coherence-pressure / SHG):** Histories that reduce the *incremental description length* of the joint log (settings + detections) are slightly favored. Under a closed-loop “compressivity-driven” controller, the realized logs show a statistically significant *excess* compressibility beyond what the same policy would predict under H₀.

**Primary prediction:** In the compressivity-driven mode, the normalized compression ratio of the full log exceeds a pre-registered null distribution built from policy-matched Monte Carlo or permutation baselines.

---

## 2) Intuition (plain language)

Give the experiment choices between two lawful regimes:

- **Interference ON:** no which-path info; produces structured fringe patterns.
- **Interference OFF:** which-path marked; no fringes; outcomes look more i.i.d. across positions.

If a tiny “coherence pressure” exists, then a controller that always tries to *minimize the growth of description length* of the whole journal might experience a small extra tilt toward sequences that are easier to describe than chance alone would allow. If nothing special exists, you get exactly what your policy implies—no leftover effect.

---

## 3) Setup (hardware)

- **Source:** Heralded single photons (SPDC) or attenuated laser (classical baseline).
- **Interferometer:** Mach–Zehnder or double-slit with controllable which-path marking/erasure.
- **Fast switch:** Electro‑optic or acousto‑optic modulator to toggle ON/OFF within the inter-photon interval.
- **Detectors:** Single‑photon detectors (e.g., SPADs/EMCCDs) with nanosecond timing.
- **Which‑path module:** Polarization or path marker + eraser element (e.g., polarizers + QWPs + PBS as needed).
- **Timing & control:** FPGA or real-time PC for timestamping and issuing switch commands.
- **Randomness sources:**
    - Quantum RNG (for RANDOM regime and for fair tie‑breaks).
    - Presealed scripted sequence (for SCRIPTED regime).
- **Environment:** Temperature and vibration control; beam-path stabilization.

---

## 4) Event journal (what is logged)

For each detection event k:

- Timestamp tₖ.
- Controller mode Mₖ ∈ {ON (interference), OFF (which‑path)} applied just *before* emission/window.
- Detector channel / position Xₖ (pixel, bin, or output arm).
- Optional: which‑path tag bit when OFF.
- Session metadata (laser power, alignment, temperatures).

The *journal string* J is the serialized sequence of (Mₖ, Xₖ, tₖ, meta) with a fixed, pre‑declared encoding.

---

## 5) Control regimes (all three are required)

1. **RANDOM:** Mₖ sampled i.i.d. from a QRNG. (Gold-standard null.)
2. **SCRIPTED:** Mₖ follows a pre-generated, sealed sequence designed to be obviously compressible (e.g., long blocks ON…ON, then OFF…OFF). (Checks that the controller’s structure alone doesn’t fake the effect.)
3. **COMPRESSIVITY‑DRIVEN (closed loop):** After each block of N events, compute a *compressivity score* C(J₁:ₖ). Choose the next block’s mode to *minimize* the *incremental description length* ΔDL = DL(J₁:ₖ+Δ) − DL(J₁:ₖ), subject to guardrails (below).

**Guardrails for (3):**

- The decision for block k+1 must depend only on data up to block k (no look‑ahead).
- Fixed block size N (pre‑registered).
- Max switch rate (e.g., no more than one change per B blocks) to limit trivial alternation artifacts.
- Tie‑breaks with QRNG.

---

## 6) Compressivity metric (pre-register this!)

Use two independent metrics (both pre-specified):

- **C₁ (ZIP ratio):** C₁ = |ZIP(J)| / |RAW(J)| using a fixed compressor and encoding.
- **C₂ (Lempel–Ziv complexity):** normalized LZ score on the same serialized J.

**Incremental objective:** Prefer settings that minimize ΔDL ≈ Δ|ZIP(J)| for the next block, estimated from recent statistics (no peeking at future outcomes).

---

## 7) Procedure

1. **Calibration:** Align interferometer; measure fringe visibility V in ON; verify no fringes in OFF. Equalize losses across modes (or log and model them).
2. **Baseline runs:** Collect large datasets in RANDOM and SCRIPTED regimes.
3. **Closed-loop runs:** Execute COMPRESSIVITY‑DRIVEN policy with multiple (pre‑declared) N and controller hyperparameters.
4. **Classical baseline:** Repeat all regimes with classical light (attenuated laser) to ensure no spurious “effects” from hardware feedback alone.
5. **Blinding:** Randomly permute labels of some sessions or encrypt logs for analysis after pipeline is frozen.

---

## 8) Analysis plan (pre-registered)

**Primary test:**

- Build a *policy-matched null* for COMPRESSIVITY‑DRIVEN by simulating outcome sequences under standard QM with the *exact same* realized mode sequence Mₖ (or by time-shuffling within blocks), then computing the same C metrics.
- Compute the observed C for the real data; compare to the null distribution (per-session and aggregated). Report p-values / Bayes factors as pre-declared.

**Secondary checks:**

- Compare C across RANDOM vs SCRIPTED vs COMPRESSIVITY‑DRIVEN.
- Runs tests and KS tests on (Mₖ) and (Xₖ) to detect over-regularity beyond policy.
- Sensitivity analysis: different encodings of J (e.g., omit timestamps) to rule out trivial sources of compressibility.

**Multiple comparisons:** Control via pre-listed metrics; adjust if additional, exploratory metrics are reported.

---

## 9) Artefacts & how to kill them

- **Differential losses / detector bias:** Recalibrate frequently; monitor counts per mode; include loss model in null.
- **Controller-induced periodicity:** SCRIPTED baseline quantifies the maximal compressibility achievable by policy structure alone.
- **Drift (thermal, alignment):** Interleave regimes; shuffle block order; track visibility V over time.
- **Time-correlation leaks:** Ensure the controller cannot access any proxy of the *next* photon (hardware isolation; fixed decision cadence).
- **p-hacking:** Full pre-registration; sealed code; preregistered stopping rule.

---

## 10) Pre-registration checklist

- [ ]  Exact hardware and alignment tolerances.
- [ ]  Block size N, max switch rate, tie-break method.
- [ ]  Journal encoding format (bit layout) and compressors.
- [ ]  Primary & secondary metrics; statistical thresholds.
- [ ]  Null construction method (simulation/permutation) and seeds.
- [ ]  Blinding plan; data-sharing plan.
- [ ]  Replication plan (second interferometer or second lab).

---

## 11) Minimal controller pseudocode (illustrative)

```
# Inputs: past journal J, block size N, guardrails
while running:
    C_now = Compress(J)
    # Predict next-block DL growth for ON vs OFF using recent statistics only
    DL_ON  = PredictDeltaDL(J, mode='ON',  N)
    DL_OFF = PredictDeltaDL(J, mode='OFF', N)
    if abs(DL_ON - DL_OFF) < epsilon:
        next_mode = QRNG()
    else:
        next_mode = 'ON' if DL_ON < DL_OFF else 'OFF'
    ApplyMode(next_mode)
    AcquireBlock(N)  # append N events to J

```

---

## 12) Variants (optional)

- **Quantum eraser variant:** Decide erase/not-erase via the same compressivity policy.
- **Two-arm MZ variant:** Switch relative phase vs path-marking to trade off pattern sharpness and outcome entropy.
- **Bell-test flavored variant (advanced):** Apply policy independently on wings A/B based on shared *past* logs; verify no-signaling while testing for compressivity excess.

---

## 13) Expected outcomes & interpretation

- **Null result:** Compressivity in COMPRESSIVITY‑DRIVEN matches policy‑matched nulls → no evidence for coherence pressure at tested scales.
- **Positive result:** Stable, replicable excess compressibility beyond null across encodings/metrics/hardware. Immediate actions: independent replication; adversarial artefact hunts; preregistered extensions.

**Either way:** The method yields a new *operational* probe: adaptively steer settings by an information‑theoretic objective and test whether reality “leans” with you.

---

## 14) Practical notes

- Aim for per-session event counts high enough that ZIP/LZ metrics stabilize (pilot with synthetic logs).
- Keep ON/OFF optical losses within ~1% or model them explicitly.
- Version-control the controller code; hash all binaries; store raw and compressed logs.
- Publish both data and exact journal encoding to enable independent recompression.

---

## 15) Versioning

- **v0.1** — First public draft (compressivity-driven MZ / double-slit protocol, three regimes, analysis & guardrails).