# 📡 RIS-Assisted Wireless Communication: Research Paper Roadmap

> **Working Title:** Reconfigurable Intelligent Surfaces for Next-Generation Wireless Networks: Optimization, Channel Modeling, and Performance Analysis
>
> **Target Venue:** IEEE Transactions on Wireless Communications / IEEE Journal on Selected Areas in Communications
>
> **Status:** 🟡 In Progress

---

## 🧭 Overview

This roadmap outlines the end-to-end plan for writing, submitting, and revising a research paper on Reconfigurable Intelligent Surfaces (RIS) applied to next-generation wireless communications. It covers motivation, technical contributions, timeline, and writing milestones.

---

## 📌 Phase 1 — Literature Survey & Problem Formulation

### 1.1 Core RIS Concepts to Cover

- [ ] Passive vs. active RIS architectures
- [ ] Channel modeling for RIS-assisted links (cascaded channels)
- [ ] Beamforming and phase-shift optimization
- [ ] Near-field vs. far-field RIS propagation
- [ ] RIS vs. relay comparison (amplify-and-forward, decode-and-forward)
- [ ] Hybrid RIS-MIMO systems

### 1.2 Key Papers to Read & Cite

| # | Paper | Authors | Venue | Year | Status |
|---|-------|---------|-------|------|--------|
| 1 | Towards Smart Radio Environments... | Wu & Zhang | IEEE TWC | 2020 | ✅ Read |
| 2 | Intelligent Reflecting Surface vs. Relaying | Björnson et al. | IEEE WCL | 2020 | ✅ Read |
| 3 | Reconfigurable Intelligent Surfaces: Potentials, Applications and Challenges | Renzo et al. | IEEE JSAC | 2020 | 🔄 Reading |
| 4 | Channel Estimation for RIS-Assisted mmWave MIMO | He et al. | IEEE JSAC | 2021 | ⬜ Pending |
| 5 | Active RIS vs. Passive RIS | Long et al. | IEEE TWC | 2022 | ⬜ Pending |
| 6 | Near-Field Communications with RIS | Liu et al. | IEEE Trans. | 2023 | ⬜ Pending |

### 1.3 Research Gaps Identified

1. **Channel estimation overhead** — Most works assume perfect CSI; practical estimation for large RIS arrays is underexplored.
2. **Energy efficiency trade-offs** — Active RIS amplification noise vs. passive phase-shift limitations not well quantified.
3. **Heterogeneous deployment** — Multi-RIS multi-user scenarios with inter-RIS interference are largely open.
4. **Robust beamforming** — Imperfect phase-shift quantization effects on system capacity need deeper analysis.

---

## 📌 Phase 2 — System Model & Problem Statement

### 2.1 System Model

```
     BS (M antennas)
          |
    h_d   |   h_r1 (direct link)
          |
    +-----v------+
    |    RIS     |  (N reflecting elements)
    | Phi = diag |
    +-----+------+
          |  h_r2 (reflected link)
          v
       UE / User Equipment (K users)
```

**Key variables:**
- `M` — Number of BS transmit antennas
- `N` — Number of RIS passive elements
- `K` — Number of single-antenna users
- `Phi in C^{NxN}` — RIS phase-shift matrix (diagonal)
- `beta_n = e^{j*theta_n}`, `theta_n in [0, 2*pi)` — phase shift of element `n`

### 2.2 Channel Model

- **BS to RIS link:** `G in C^{NxM}` — Rician fading with LOS component
- **RIS to UE link:** `h_r in C^{Nx1}` — Rayleigh or Rician depending on scenario
- **Direct BS to UE link:** `h_d in C^{Mx1}` — may be blocked in NLOS scenarios
- **Cascaded channel:** `h_eff = h_d + h_r^H * Phi * G * w`

### 2.3 Optimization Problem

**Maximize weighted sum-rate subject to:**

```
max_{w, Phi}  Sum_k alpha_k * log2(1 + SINR_k)

subject to:
  |w|^2 <= P_max         (transmit power constraint)
  |beta_n| = 1, for all n (unit modulus constraint)
  theta_n in {0, 2pi/L, ...} (discrete phase shifts, L levels)
```

**Approach:** Alternating Optimization (AO) — decouple beamforming `w` and phase shifts `Phi`, solve iteratively.

---

## 📌 Phase 3 — Proposed Methodology

### 3.1 Algorithm Design

- [ ] **Step 1:** Initialize beamformer `w` with matched filter / zero-forcing solution
- [ ] **Step 2:** Fix `w`, solve for `Phi` using Majorization-Minimization (MM) or Semidefinite Relaxation (SDR)
- [ ] **Step 3:** Fix `Phi`, update `w` via WMMSE or fractional programming
- [ ] **Step 4:** Iterate until convergence (KKT conditions met or delta < epsilon)
- [ ] **Step 5:** Quantize phase shifts to discrete levels for practical implementation

### 3.2 Proposed Innovations

| Contribution | Description | Novelty |
|---|---|---|
| **C1** | Joint active-passive beamforming with discrete phase shifts | Practical quantization model vs. continuous idealization |
| **C2** | Robust design under imperfect CSI with bounded uncertainty | Worst-case optimization for reliability |
| **C3** | Low-complexity closed-form approximation for large N | Scalable solution for massive RIS arrays |
| **C4** | Energy efficiency analysis with RIS hardware power model | Realistic power consumption accounting for RIS control circuits |

### 3.3 Baseline Comparisons

- **Baseline 1:** No RIS (conventional MIMO beamforming)
- **Baseline 2:** Random phase shifts (no optimization)
- **Baseline 3:** Continuous phase shifts (upper bound)
- **Baseline 4:** AF relay with same power budget

---

## 📌 Phase 4 — Simulation & Results

### 4.1 Simulation Setup

| Parameter | Value |
|---|---|
| Carrier frequency | 28 GHz (mmWave) |
| BS antennas M | 64 |
| RIS elements N | 64–256 |
| Users K | 4–8 |
| BS–RIS distance | 50 m |
| RIS–UE distance | 10–30 m |
| Path loss model | 3GPP UMa / UMi |
| Phase shift bits | 1, 2, 3 bits |
| Monte Carlo runs | 1000 |
| SNR range | -10 to 30 dB |

### 4.2 Key Results to Generate

- [ ] **Fig. 1:** Sum-rate vs. SNR (proposed vs. baselines)
- [ ] **Fig. 2:** Sum-rate vs. number of RIS elements N
- [ ] **Fig. 3:** Convergence of AO algorithm
- [ ] **Fig. 4:** Impact of phase shift quantization bits
- [ ] **Fig. 5:** Robust vs. non-robust design under CSI errors
- [ ] **Fig. 6:** Energy efficiency vs. transmit power
- [ ] **Fig. 7:** CDF of per-user rate (fairness analysis)
- [ ] **Table I:** Complexity comparison

### 4.3 MATLAB / Python Simulation Checklist

- [ ] Channel generation module (Rician/Rayleigh, 3GPP path loss)
- [ ] WMMSE beamforming solver
- [ ] MM-based phase-shift optimizer
- [ ] SDR-based phase-shift optimizer (validation)
- [ ] Discrete phase-shift quantization module
- [ ] Monte Carlo averaging loop
- [ ] Plotting scripts (IEEE style, LaTeX fonts)

---

## 📌 Phase 5 — Paper Writing

### 5.1 Paper Structure

```
I.   Introduction
     +-- RIS motivation & background
     +-- Related work & limitations
     +-- Contributions (bulleted, clear C1–C4)

II.  System Model
     +-- Network topology & signal model
     +-- Channel model (cascaded RIS channel)
     +-- Problem formulation

III. Proposed Algorithm
     +-- AO framework overview
     +-- Beamforming subproblem (WMMSE)
     +-- Phase-shift subproblem (MM/SDR)
     +-- Convergence & complexity analysis
     +-- Discrete phase quantization

IV.  Simulation Results
     +-- Simulation setup
     +-- Convergence behavior
     +-- Performance vs. SNR / N
     +-- Effect of quantization
     +-- Robustness analysis
     +-- Energy efficiency

V.   Conclusion

     Appendices (proofs, derivations)
     References (40–60 citations)
```

### 5.2 Writing Milestones

| Section | Draft Due | Review Due | Status |
|---|---|---|---|
| Abstract | Week 2 | Week 3 | ⬜ |
| Introduction | Week 3 | Week 4 | ⬜ |
| System Model | Week 4 | Week 5 | ⬜ |
| Proposed Algorithm | Week 5 | Week 6 | ⬜ |
| Simulation Results | Week 7 | Week 8 | ⬜ |
| Conclusion | Week 8 | Week 9 | ⬜ |
| Full Draft | Week 9 | Week 10 | ⬜ |
| Revision & Polish | Week 10 | Week 11 | ⬜ |

### 5.3 Abstract Draft (v0.1)

> Reconfigurable Intelligent Surfaces (RIS) have emerged as a transformative technology for next-generation wireless networks, offering the ability to programmably reshape the radio propagation environment. In this paper, we investigate a RIS-assisted multi-user MIMO downlink system and propose a joint active-passive beamforming design to maximize the weighted sum-rate. Unlike existing works that assume ideal continuous phase shifts, we address the practical constraint of discrete phase quantization and imperfect channel state information (CSI). We formulate the optimization as a non-convex problem and develop an efficient alternating optimization (AO) algorithm that iteratively optimizes the BS precoder via WMMSE and the RIS phase-shift matrix via majorization-minimization (MM). Simulation results demonstrate that the proposed design significantly outperforms conventional baselines and achieves near-optimal performance with only 2–3-bit phase resolution.

---

## 📌 Phase 6 — Submission & Revision

### 6.1 Target Journals / Conferences

| Priority | Venue | Impact Factor | Deadline | Notes |
|---|---|---|---|---|
| 1st | IEEE Trans. Wireless Commun. | ~10.4 | Rolling | Flagship journal |
| 2nd | IEEE JSAC | ~16.4 | Rolling | Special Issue on RIS |
| 3rd | IEEE Wireless Commun. Letters | ~6.4 | Rolling | Faster turnaround |
| Conf | IEEE GLOBECOM 2026 | — | ~June 2026 | For early dissemination |

### 6.2 Submission Checklist

- [ ] Format paper to IEEE double-column template
- [ ] Check page limits (8 pages for TWC regular, 5 for letters)
- [ ] Verify all citations are in IEEE format
- [ ] Proofread for grammar and clarity (Grammarly + manual)
- [ ] Ensure all figures are high-res (300 DPI minimum)
- [ ] Write cover letter
- [ ] Prepare author bios and headshots (for final version)
- [ ] Check author order and affiliations
- [ ] Submit via IEEE Manuscript Central / ScholarOne

### 6.3 Review Response Workflow

```
Submit --> [4-12 weeks] --> Review Received
   |
   v
Categorize comments (major / minor / editorial)
   |
   v
Draft point-by-point response letter
   |
   v
Revise paper --> Highlight changes in blue
   |
   v
Re-submit --> [2-6 weeks] --> Decision
```

---

## 📌 Resources & Tools

### Software

| Tool | Purpose |
|---|---|
| MATLAB / Python (NumPy, SciPy) | Simulations |
| LaTeX (Overleaf / VS Code) | Paper writing |
| CVX / CVXPY | Convex optimization solver |
| Zotero / Mendeley | Reference management |
| Grammarly | Grammar and style checking |
| draw.io / TikZ | System diagrams |

### Key RIS Toolboxes & Datasets

- **SimRIS Channel Simulator** — MATLAB toolbox for RIS channel simulation
- **DeepMIMO Dataset** — Ray-tracing based channel data for mmWave MIMO/RIS
- **RIS-E2E** — End-to-end simulation framework (GitHub)

### Useful Formulas Quick Reference

**SINR at user k:**
```
SINR_k = |h_eff_k^H * w_k|^2 / (Sum_{j!=k} |h_eff_k^H * w_j|^2 + sigma^2)
```

**Achievable rate:**
```
R_k = log2(1 + SINR_k)   [bits/s/Hz]
```

**RIS array gain (asymptotic, N to infinity):**
```
SNR proportional to N^2  (passive RIS, optimal beamforming)
SNR proportional to N    (relay, same power)
```

---

## Master Timeline

```
Sep 2026  ████░░░░░░░░░░░░░  Literature Review + Problem Formulation
Oct 2026  ░░░████████░░░░░░  Algorithm Design + Simulation Setup
Nov 2026  ░░░░░░░████████░░  Results Generation + Analysis
Dec 2026  ░░░░░░░░░░░████░░  Paper Writing (Sec I-III)
Jan 2027  ░░░░░░░░░░░░░████  Paper Writing (Sec IV-V) + Submission
Feb 2027  ░░░░░░░░░░░░░░░██  Revision (if needed)
```

---

## Notes & Ideas

> Use this section to jot down quick ideas, concerns, or questions during the research process.

- Consider extending to multi-RIS cooperative beamforming in future work
- Check if stochastic geometry analysis is feasible for large-scale deployment modeling
- Reviewer concern preemption: justify why AO converges to at least a local optimum
- Look into federated learning for distributed RIS control (possible extension)
- Is there a closed-form solution for the single-user case? Could strengthen theory section.

---

*Last updated: September 2026 | Maintained by: Research Team*
