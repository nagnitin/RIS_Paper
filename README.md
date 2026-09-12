# RIS-Assisted MIMO Channel Estimation Using Deep Learning and Transfer Learning

## Project Overview

This project investigates deep-learning-based channel estimation for Reconfigurable Intelligent Surface (RIS)-assisted MIMO communication systems, with particular emphasis on **when learned estimators outperform analytical baselines and whether transfer learning improves adaptation reliability under channel-distribution mismatch and limited target-domain data**.

The work combines:

- LS and LMMSE analytical channel estimation baselines
- DNN, CNN and Autoencoder channel estimators
- An LS-initialized ChannelNet estimator based on SRCNN + DnCNN
- Transfer learning from a VehA source domain to a SUI-5 target domain
- Identical-budget scratch training as the controlled baseline
- Three-seed evaluation for statistical reliability
- SNR-swept evaluation
- Pilot-pattern, multi-user and hardware-impairment extensions
- ResNet as an additional architecture comparison
- Reproducible checkpoints, datasets, metrics and figures
- A Streamlit demonstration using trained checkpoints

> **Important novelty positioning:** ChannelNet itself, CNN-based RIS channel estimation, transfer learning for wireless channel estimation, pilot-pattern optimization, multi-user RIS estimation and hardware-impairment robustness are all established research directions. The principal research contribution here is the **controlled study of RIS-specific transfer/domain adaptation of an LS-initialized ChannelNet across a VehA → SUI-5 channel-domain shift under target-data scarcity, compared with an identical-budget scratch model and evaluated across multiple random seeds**.

---

## 1. Research Questions

The project is organized around two main questions:

1. **When do deep-learning channel estimators actually beat LS/LMMSE across SNR?**
2. **Can a pretrained RIS channel estimator adapt to a new channel environment using limited target-domain data?**

The second question addresses a practical deployment scenario in which a model can be trained using a data-rich source channel environment but only a relatively small amount of labeled data is available after deployment in a different channel environment.

---

## 2. Main Contributions

### Primary contribution

**RIS-specific transfer/domain adaptation under channel-distribution mismatch and target-data scarcity.**

A pretrained LS-initialized ChannelNet is adapted from a **VehA** source channel model to a **SUI-5** target channel model.

### Secondary contribution

**Multi-seed evaluation showing that transfer learning can improve adaptation reliability, not merely average NMSE.**

The transfer and scratch models use the same target dataset, split, training budget and evaluation protocol.

### Supporting finding

**SNR-dependent crossover between learned and analytical estimators.**

The experiments show that SNR-agnostic neural estimators can be competitive at lower SNR while LS/LMMSE can become substantially better at high SNR.

### Supporting validation

The repository additionally evaluates:

- Block, comb and lattice pilot patterns
- 1, 2, 4 and 8-user settings
- 2-bit RIS phase quantization
- Amplitude/phase hardware impairments
- ResNet architecture comparison
- Runtime and parameter counts
- True/estimated/error channel heatmaps

### Reproducibility contribution

The project stores:

- Generated datasets
- Trained model checkpoints
- CSV result tables
- JSON experiment summaries
- Convergence plots
- SNR curves
- Robustness plots
- Channel heatmaps
- Experiment manifests

---

# 3. System Model

The effective RIS-assisted channel is represented as a complex channel vector/matrix and estimated from pilot observations.

The implemented observation model is:

```text
y = Xᴴ h + n
```

where:

- `y` is the received pilot observation
- `X` is the pilot matrix
- `h` is the effective complex channel
- `n` is additive noise
- `Xᴴ` denotes the Hermitian transpose

The implementation represents complex-valued channels using real and imaginary components for neural-network processing.

---

# 4. Channel Modeling

## 4.1 Saleh–Valenzuela RIS Channel

The main synthetic RIS channel generator uses a Saleh–Valenzuela-style multipath formulation.

The channel is generated as a sum of multipath components with complex path gains and spatial structure.

This dataset is used for the initial DNN/CNN/Autoencoder benchmark.

## 4.2 Standardized TDL Channel Domains

For the transfer-learning experiment, the project uses two different standardized channel domains:

### Source domain

**3GPP VehA**

- 4,000 samples
- 12 dB SNR
- 20 training epochs

### Target domain

**SUI-5**

- Exactly 2,000 samples
- 20 dB SNR
- 20 training epochs

The source model is pretrained on VehA and subsequently adapted to SUI-5.

---

# 5. Estimation Methods

## 5.1 LS Estimator

The least-squares estimate is used as an analytical baseline.

Conceptually:

```text
ĥ_LS = arg min_h ||y - Xᴴh||²
```

For suitable pilot matrices this reduces to the standard pseudo-inverse solution.

---

## 5.2 LMMSE Estimator

The LMMSE estimator incorporates channel covariance and noise statistics.

It serves as a stronger analytical reference than LS when the relevant second-order statistics are available.

---

## 5.3 DNN Estimator

The DNN receives the pilot-derived real-valued representation and predicts the real and imaginary components of the channel.

Parameter count:

```text
132,224
```

---

## 5.4 CNN Estimator

The CNN processes the pilot representation as a structured two-dimensional input.

Parameter count:

```text
26,946
```

---

## 5.5 Autoencoder Estimator

The Autoencoder architecture provides an additional neural baseline.

Parameter count:

```text
99,008
```

The repository implementation uses the pilot observation as the estimator input rather than treating the true channel as the only reconstruction target.

---

## 5.6 ChannelNet

The principal deep-learning architecture is a ChannelNet-style estimator consisting of:

```text
LS initialization
       ↓
  Low-resolution
 channel estimate
       ↓
     SRCNN
       ↓
     DnCNN
       ↓
Estimated channel
```

ChannelNet contains:

- SRCNN-based super-resolution
- DnCNN-based residual denoising
- LS initialization
- Complex-channel reconstruction

Parameter count:

```text
238,436
```

---

## 5.7 ResNet

A residual-network estimator is also included as an optional architecture comparison.

Parameter count:

```text
73,186
```

The executed repository experiment used 10 epochs on the 8,000-sample SV dataset.

---

# 6. Dataset and Experimental Configuration

## 6.1 Saleh–Valenzuela Benchmark

The main synthetic dataset contains:

```text
8,000 samples
SNR = 0, 10, 20, 30 dB
64 pilots
```

DNN, CNN and Autoencoder models were trained for:

```text
30 epochs
```

The dataset is evaluated using a held-out test set.

---

## 6.2 Transfer-Learning Experiment

### Source

```text
Channel model: VehA
Samples:       4,000
SNR:           12 dB
Epochs:        20
```

### Target

```text
Channel model: SUI-5
Samples:       2,000
SNR:           20 dB
Epochs:        20
```

### Transfer protocol

```text
VehA pretrained ChannelNet
          ↓
SUI-5 target-domain adaptation
          ↓
Early SRCNN first-convolution freezing
          ↓
Target validation checkpoint selection
```

### Scratch control

A second ChannelNet is trained from scratch on the same SUI-5 target dataset using the same overall training budget.

This is essential because comparing transfer learning against a differently configured scratch model would not isolate the effect of transfer.

---

# 7. Train / Validation / Test Protocol

The repository uses a deterministic:

```text
70% training
15% validation
15% testing
```

split.

The training pipeline uses:

- Fixed random seed
- Batch size 512
- Validation-based best-checkpoint selection
- Same target data for transfer and scratch comparisons

The multi-seed experiment uses:

```text
42
123
2025
```

This avoids drawing conclusions from one lucky initialization.

---

# 8. Main Transfer-Learning Results


## Transfer

```text
Mean MSE:          0.00857164
Std MSE:           0.00033327

Mean NMSE (dB):   -20.67155 dB
Std NMSE (dB):      0.16873 dB

Mean BER:           0
Mean training:     13.82017 s

Mean best epoch:   19.6667
```

## Scratch

```text
Mean MSE:           0.02597061
Std MSE:            0.01530303

Mean NMSE (dB):    -16.33214 dB
Std NMSE (dB):       2.44834 dB

Mean BER:            0
Mean training:      13.50053 s

Mean best epoch:    19.6667
```

### Interpretation

The average NMSE improvement is approximately:

```text
4.34 dB
```

More importantly, the scratch model exhibits substantially larger cross-seed variation.

The NMSE standard-deviation ratio is approximately:

```text
2.448 / 0.169 ≈ 14.5×
```

Thus, the strongest practical interpretation is not that transfer learning produces an enormous absolute accuracy gain, but that it can make target-domain adaptation **more stable and reliable** under the tested conditions.

> A scientifically appropriate statement is: **Transfer learning provides a modest-to-substantial improvement in mean target-domain NMSE in the executed experiment while markedly reducing run-to-run variability relative to same-budget scratch training.**

---

# 10. DNN / CNN / Autoencoder Results

The following results are from the held-out portion of the 8,000-sample Saleh–Valenzuela dataset.

| Method | MSE | NMSE (dB) | Epochs | Training Time |
|---|---:|---:|---:|---:|
| LS | 0.266013 | -5.751 | — | — |
| LMMSE | 0.145864 | -8.361 | — | — |
| DNN | 0.230490 | -6.373 | 30 | 2.598 s |
| CNN | 0.224848 | -6.481 | 30 | 7.269 s |
| Autoencoder | 0.638856 | -1.946 | 30 | 2.636 s |

The LMMSE baseline is the strongest method on this mixed-SNR aggregate evaluation.

This is important because it prevents an unsupported claim that deep learning universally dominates analytical estimators.

---

# 11. SNR-Dependent Behavior

The per-SNR results reveal an important crossover.

## 0 dB

Neural estimators are competitive with analytical methods, although all methods experience significant noise.

## 10 dB

DNN/CNN become competitive, while LS/LMMSE remain strong.

## 20 dB

LS/LMMSE become substantially stronger than the SNR-agnostic neural estimators.

## 30 dB

LS/LMMSE strongly dominate the tested DNN/CNN/Autoencoder estimators.

The observed behavior suggests that a neural estimator trained across multiple SNR values without explicit SNR conditioning can develop an approximation/generalization floor.

This motivates future work on:

- SNR-conditioned neural estimators
- noise-aware training
- curriculum learning across SNR
- mixture-of-experts estimators
- hybrid analytical + learned estimators

---


## LS

| SNR | MSE | NMSE (dB) | BER |
|---:|---:|---:|---:|
| 0 dB | 0.993708 | -0.027 | 0.15878 |
| 10 dB | 0.100851 | -9.963 | 0.000830 |
| 20 dB | 0.009928 | -20.031 | 0 |
| 30 dB | 0.001006 | -29.974 | 0 |

## LMMSE

| SNR | MSE | NMSE (dB) | BER |
|---:|---:|---:|---:|
| 0 dB | 0.499801 | -3.012 | 0.15879 |
| 10 dB | 0.091574 | -10.382 | 0.000830 |
| 20 dB | 0.009846 | -20.067 | 0 |
| 30 dB | 0.001005 | -29.979 | 0 |

## DNN

| SNR | NMSE (dB) |
|---:|---:|
| 0 dB | -2.018 |
| 10 dB | -7.961 |
| 20 dB | -10.863 |
| 30 dB | -11.352 |

## CNN

| SNR | NMSE (dB) |
|---:|---:|
| 0 dB | -2.157 |
| 10 dB | -8.219 |
| 20 dB | -10.799 |
| 30 dB | -11.060 |

## Autoencoder

| SNR | NMSE (dB) |
|---:|---:|
| 0 dB | -0.652 |
| 10 dB | -2.285 |
| 20 dB | -2.522 |
| 30 dB | -2.542 |

---

# 13. Complexity Comparison

| Model | Parameters |
|---|---:|
| DNN | 132,224 |
| CNN | 26,946 |
| Autoencoder | 99,008 |
| ChannelNet | 238,436 |
| ResNet | 73,186 |

The parameter count alone does not determine estimator quality. Runtime, memory, convergence behavior and robustness also matter.

---

# 14. Robustness Extensions

The repository includes three extension categories.

## 14.1 Pilot Pattern

Evaluated patterns:

- Block
- Comb
- Lattice

Pilot counts:

```text
16
32
48
64
```

The experiment examines whether pilot count or arrangement is the dominant factor under the tested configuration.

This is treated as a supporting robustness experiment rather than a primary novelty claim.

---

## 14.2 Multi-User Scaling

The repository evaluates:

```text
1 user
2 users
4 users
8 users
```

This tests how estimator performance changes as the multi-user setting becomes more demanding.

Again, this is validation rather than the main novelty claim because RIS multi-user channel estimation is already an established research direction.

---

## 14.3 Hardware Impairments

The extension includes:

- Ideal RIS
- 2-bit phase quantization
- Amplitude/phase mismatch

The expected effect is degradation as the RIS becomes less ideal.

Hardware-impairment robustness is therefore presented as supporting engineering evidence, not as the primary research novelty.

---

# 15. BER Metric

A QPSK BER metric is included alongside channel-estimation metrics.

The main estimation metrics are:

- MSE
- NMSE
- NMSE in dB
- QPSK BER

The BER results should be interpreted jointly with channel-estimation error rather than as a replacement for NMSE.

---

# 16. Visualization Outputs

The repository generates:

```text
mse_vs_snr_all.png
nmse_vs_snr_all.png
ber_vs_snr_all.png
transfer_vs_scratch_convergence.png
transfer_vs_scratch_train_val.png
transfer_vs_scratch_nmse.png
transfer_multiseed_nmse.png
runtime_comparison.png
parameter_comparison.png
pilot_overhead_nmse.png
multiuser.png
hardware.png
```

It also generates true / estimated / error heatmaps for:

```text
DNN
CNN
Autoencoder
ChannelNet Transfer
```

These visualizations make the estimator behavior easier to inspect beyond a single scalar metric.

---

# 17. Repository Structure

```text
RIS_Channel_Estimation/
│
├── ris_ce/
│   ├── __init__.py
│   ├── simulation.py
│   ├── core.py
│   ├── models.py
│   └── training.py
│
├── scripts/
│   ├── run.py
│   ├── evaluate.py
│   ├── neural_evaluate.py
│   ├── extensions.py
│   ├── multiseed_transfer.py
│   ├── run_resnet.py
│   └── smoke_streamlit.py
│
├── data/
│   ├── ris_dl_8000.npz
│   ├── veha12_4000.npz
│   └── sui5_20_2000.npz
│
├── checkpoints/
│   ├── dnn_sv.pth
│   ├── cnn_sv.pth
│   ├── ae_sv.pth
│   ├── channelnet_veha12_full.pth
│   ├── transfer checkpoints
│   ├── scratch checkpoints
│   └── resnet_sv.pth
│
├── results/
│   ├── metrics.csv
│   ├── snr.csv
│   ├── neural_metrics.csv
│   ├── neural_snr.csv
│   ├── transfer_vs_scratch.csv
│   ├── convergence.csv
│   ├── parameter_comparison.csv
│   ├── experiment_manifest.json
│   ├── transfer_summary.json
│   ├── transfer_multiseed.csv
│   ├── transfer_multiseed_summary.json
│   ├── resnet_metrics.csv
│   ├── baseline_metrics.csv
│   ├── baseline_snr.csv
│   ├── extensions.csv
│   ├── LEFT_OUT_WORK.md
│   └── figures/
│
├── RIS_Demo_Streamlit.py
├── requirements.txt
├── README.md
└── PROJECT_STATUS.md
```

---

# 18. Streamlit Demo

The Streamlit application is:

```text
RIS_Demo_Streamlit.py
```

The application loads the actual trained checkpoints:

```text
DNN        → checkpoints/dnn_sv.pth
CNN        → checkpoints/cnn_sv.pth
Autoencoder → checkpoints/ae_sv.pth
ChannelNet → checkpoints/channelnet_veha12_full.pth
```

The neural estimators are configured for the 64-pilot setting used during training.

A checkpoint-based inference smoke test was successfully performed.

The sandbox environment used for validation did not provide the Streamlit executable/package needed to launch a live HTTP server, although the project declares Streamlit in `requirements.txt`.

---

# 19. Reproducibility

Install dependencies:

```bash
pip install -r requirements.txt
```

Generate the synthetic dataset:

```bash
python scripts/run.py generate --samples 8000 --snr 0 10 20 30 --out data/ris_dl_8000.npz
```

Train a DNN:

```bash
python scripts/run.py train --data data/ris_dl_8000.npz --model dnn --out checkpoints/dnn_sv.pth --epochs 30
```

Train a CNN:

```bash
python scripts/run.py train --data data/ris_dl_8000.npz --model cnn --out checkpoints/cnn_sv.pth --epochs 30
```

Train an Autoencoder:

```bash
python scripts/run.py train --data data/ris_dl_8000.npz --model autoencoder --out checkpoints/ae_sv.pth --epochs 30
```

Train ChannelNet:

```bash
python scripts/run.py train --data data/veha12_4000.npz --model channelnet --out checkpoints/channelnet_veha12_full.pth --epochs 20
```

Run baseline evaluation:

```bash
python scripts/evaluate.py
```

Run neural-model evaluation:

```bash
python scripts/neural_evaluate.py
```

Run robustness extensions:

```bash
python scripts/extensions.py
```

Run the multi-seed transfer experiment:

```bash
python scripts/multiseed_transfer.py
```

Run the ResNet experiment:

```bash
python scripts/run_resnet.py
```

Run the Streamlit application after installing dependencies:

```bash
streamlit run RIS_Demo_Streamlit.py
```

---

# 20. Scientific Scope and Limitations

The results should be interpreted within the implemented simulation and dataset configuration.

Important limitations include:

1. The study is simulation-based.
2. The exact VehA → SUI-5 transfer direction is one particular domain shift.
3. The target dataset contains 2,000 samples in the executed final experiment.
4. The main transfer comparison uses three random seeds.
5. The neural estimators are not SNR-conditioned in the main benchmark.
6. The LMMSE baseline has access to channel statistics that a learned estimator may not have at deployment.
7. Pilot, multi-user and hardware experiments are supporting extensions rather than independent claims of novelty.
8. A broader set of real measured channels would strengthen the deployment argument.
9. The experiments do not establish that the proposed configuration is universally optimal.
10. Literature searches cannot prove that no identical prior work exists anywhere.

These limitations should be stated explicitly in a conference paper.

---

# 21. Novelty Positioning

## What is NOT claimed as novel

The following should not be presented as first-of-kind contributions:

- CNN for RIS channel estimation
- DNN for channel estimation
- ChannelNet architecture itself
- RIS + CNN + LMMSE comparison
- Transfer learning for wireless channel estimation in general
- Pilot-pattern optimization in general
- RIS hardware-impairment-aware estimation in general
- Multi-user RIS channel estimation in general

## What is claimed as the research contribution

The defensible contribution is:

> **A controlled RIS-specific study of transfer/domain adaptation of an LS-initialized ChannelNet across a substantial channel-distribution mismatch, from VehA to SUI-5, under target-domain data scarcity, using an identical-budget scratch control and multi-seed analysis of both mean performance and variability.**

The strongest empirical observation is that transfer learning can substantially reduce cross-run variability while improving mean target-domain NMSE under the tested configuration.

---

# 22. Literature-Based Novelty Analysis

The literature review supplied with this project identifies the following important precedents.

| Work / Direction | RIS | DL | Channel Estimation | Transfer / Domain Adaptation | Relation |
|---|---:|---:|---:|---:|---|
| Soltani et al., 2019, Deep Learning-Based Channel Estimation | No | Yes | Yes | No | ChannelNet architecture inspiration |
| Soltani et al., 2020, Pilot Pattern Design for DL Channel Estimation | No | Yes | Yes | No | Pilot-design background |
| Kundu & McKay, 2021, RIS MISO CE: LMMSE to DL | Yes | Yes | Yes | No | Very close RIS/DL baseline |
| RIS channel-estimation surveys/frameworks | Yes | Mixed | Yes | No | RIS CE landscape |
| Zhu et al., 2023, Transfer Learning in OFDM CE | No | Yes | Yes | Yes | Closest transfer-learning precedent |
| RIS DL with hardware impairments | Yes | Yes | Yes | No | Robustness precedent |
| RIS pilot-overhead DL studies | Yes | Yes | Yes | No | Pilot-efficiency precedent |
| 2025 RIS distributed neural CE | Yes | Yes | Yes | No | Pilot-efficiency overlap |
| 2025 map-based MIMO-OFDM domain adaptation | No | Yes | Yes | Yes | General domain-adaptation precedent |
| 2026 RIS-NOMA meta-learning CE | Yes | Yes | Yes | Yes/meta | Modern adaptation direction |
| **This project** | **Yes** | **Yes** | **Yes** | **Yes** | **VehA → SUI-5 LS-initialized ChannelNet transfer + scarce target data + same-budget scratch + multi-seed variance** |

The supplied literature analysis states:

> “I did not find an exact prior paper in the searches I ran that combines all of those elements in this exact RIS/ChannelNet/VehA→SUI-5 configuration.”

This should be treated as a **literature-search-based novelty assessment**, not an absolute claim of worldwide first publication.

---

# 23. Recommended Novelty Statement for the Paper

A suitable conference-paper formulation is:

> **Unlike prior RIS channel-estimation studies that primarily evaluate a fixed estimator under a fixed channel distribution, this work investigates the reliability of transfer-based adaptation of an LS-initialized ChannelNet under a cross-domain channel mismatch and target-data scarcity. A controlled VehA-to-SUI-5 transfer experiment is compared against an identically configured scratch-trained model using the same target samples, optimization budget, and multiple random seeds. The results show that transfer learning provides a meaningful improvement in target-domain NMSE while substantially reducing cross-run variability, revealing reliability benefits that can be obscured by single-seed evaluations.**

---

# 24. SNR-Crossover Finding

Another useful paper-level finding is:

> **Deep channel estimators should not automatically be assumed to dominate analytical estimators across the entire SNR range; an SNR-dependent crossover can occur when the learned estimator is trained without explicit SNR conditioning.**

This is best presented as an empirical characterization rather than a fundamental algorithmic novelty.

---

# 25. Target-Data Efficiency: Recommended Next Experiment

The strongest recommended experiment before final conference submission is a target-data-efficiency curve.

Instead of only using:

```text
2,000 target samples
```

evaluate:

```text
100
250
500
1,000
2,000
```

target-domain samples.

For each target-data budget, compare:

```text
Transfer
vs.
Scratch
```

using identical:

- Target split
- Random seeds
- Epoch budget
- Optimizer
- Model architecture
- Evaluation protocol

Plot:

```text
NMSE (dB)
    vs.
Number of target-domain training samples
```

A result showing a clear transfer advantage at small target-data budgets would substantially strengthen the practical domain-adaptation story.

---

# 26. Recommended Second Domain Experiment

A second unseen-domain transfer direction would further strengthen the claim.

For example:

```text
VehA → another channel model
```

or:

```text
SUI-5 → VehA
```

This would help demonstrate that the observed benefit is not an artifact of one particular channel-model pair.

---


# 30. Conference-Paper Positioning

This project is suitable for a conference paper focused on:

- Machine intelligence for wireless systems
- AI/ML for signal processing
- Deep learning for communication-system sensing/estimation
- Transfer learning and domain adaptation
- Intelligent radio environments
- RIS-assisted communications
- Data-efficient model adaptation

The paper should emphasize the experimental question:

> **How reliable is transfer learning for adapting a RIS channel estimator when the deployment channel distribution differs from the training distribution and target-domain labeled data are limited?**

That is a clearer and more defensible research question than claiming a new CNN architecture.

---

# 31. Suggested Paper Titles

### Option 1

**Transfer Learning for RIS-Assisted MIMO Channel Estimation Under Cross-Domain Channel Mismatch**

### Option 2

**Reliable Cross-Domain Adaptation of ChannelNet for RIS-Assisted MIMO Channel Estimation**

### Option 3

**Target-Data-Efficient Transfer Learning for RIS Channel Estimation with ChannelNet**

### Option 4

**On the Reliability of Transfer-Based RIS Channel Estimation Under Channel-Distribution Shift**

### Option 5

**Transfer vs. Scratch: Multi-Seed Evaluation of ChannelNet Adaptation for RIS-Assisted MIMO**

The strongest academic framing is likely Option 4 or Option 1 because neither overclaims architectural novelty.

---


# 35. Project Status

### Core implementation

- [x] RIS channel simulation
- [x] Saleh–Valenzuela channel generation
- [x] VehA channel generation
- [x] SUI-5 channel generation
- [x] LS estimator
- [x] LMMSE estimator
- [x] DNN estimator
- [x] CNN estimator
- [x] Autoencoder estimator
- [x] ChannelNet estimator
- [x] ResNet estimator
- [x] Deterministic train/validation/test split
- [x] Checkpoint selection
- [x] Transfer learning
- [x] Scratch baseline
- [x] Three-seed transfer analysis
- [x] SNR evaluation
- [x] Pilot-pattern extension
- [x] Multi-user extension
- [x] Hardware-impairment extension
- [x] Runtime comparison
- [x] Parameter comparison
- [x] Channel heatmaps
- [x] CSV/JSON result export
- [x] Streamlit checkpoint integration
- [x] Checkpoint inference smoke test

### Partially environment-dependent

- [x] Streamlit source implementation
- [x] Streamlit checkpoint loading
- [x] Streamlit smoke-test inference
- [ ] Live Streamlit HTTP-server validation in the sandbox environment

---

# 36. Final Recommendation Before Submission

Before submitting the paper, prioritize:

1. **Reconcile every number in the manuscript with the final repository.**
2. Add the **target-data-efficiency curve** for 100/250/500/1000/2000 target samples.
3. Add a second unseen channel-domain transfer direction if computationally feasible.
4. Keep the novelty claim focused on **cross-domain RIS adaptation + target-data scarcity + controlled scratch comparison + multi-seed reliability**.
5. Do not claim ChannelNet, CNN-based RIS CE, transfer learning, pilot optimization or hardware-impairment robustness as individually new.
6. Include the SNR crossover as an empirical finding.
7. Report mean and standard deviation rather than only the best seed.
8. Keep all checkpoints and raw result files in the repository.
9. Clearly state simulation assumptions and limitations.
10. Re-run the literature search immediately before the final manuscript submission.

---

## License / Reproducibility

This repository is intended as a reproducible research implementation. See the repository files for the applicable license and dependency information.

# 34. Key Literature

1. Soltani et al., *Deep Learning-Based Channel Estimation*, IEEE Communications Letters, 2019.
2. Soltani et al., *Pilot Pattern Design for Deep Learning-Based Channel Estimation in OFDM Systems*, IEEE Wireless Communications Letters, 2020.
3. Kundu and McKay, *Channel Estimation for Reconfigurable Intelligent Surface Aided MISO Communications: From LMMSE to Deep Learning Solutions*, IEEE Open Journal of the Communications Society, 2021.
4. Zhu et al., *Enhancing CNN-Based Channel Estimation Using Transfer Learning in OFDM Systems*, IEEE SPAWC, 2023.
5. Recent RIS channel-estimation literature on pilot overhead, multi-user estimation, hardware impairments and adaptation/domain shift should be reviewed again immediately before submission.
