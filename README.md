# Golay-Complementary-Sequences
Implementation and analysis of Golay Complementary Sequences for applications in signal processing, radar and communication systems.
# Golay Complementary Sequences — DSP Project

> A complete Python implementation covering sequence generation, PAPR reduction in OFDM, and channel estimation using Golay complementary pairs.


## Table of Contents
- What are Golay Complementary Sequences?
- Key Results
- Project Structure
- How to Run
- Section 1 - Sequence Generation
- Section 2 - PAPR Analysis
- Section 3 - Channel Estimation
- Requirements



## What are Golay Complementary Sequences?

A **Golay complementary pair** is a pair of binary sequences `a` and `b` (values ±1) of length `N = 2ⁿ` such that the sum of their aperiodic autocorrelations is a perfect impulse:


autocorr(a)[k] + autocorr(b)[k] = 2N · δ(k)


This means all off-peak sidelobes cancel exactly when summed.  
In the frequency domain this gives a **perfectly flat combined power spectrum**:


|A(f)|² + |B(f)|² = 2N   for all f


This single property is the source of two powerful DSP results:
- **PAPR ≤ 3.010 dB** when used as OFDM subcarrier symbols (provable, deterministic bound)
- **Optimal channel estimation** with uniform noise enhancement across all frequencies


## Key Results

### Sequence Generation — Complementary Property Verified

All sidelobes cancel to exactly zero when autocorrelations of `a` and `b` are summed.

![Autocorrelation sum](figures/plot2_autocorrelation.png)

| n | Length (2ⁿ) | Max sidelobe (sum) | Status |
|---|------------|-------------------|--------|
| 1 | 2   | 0.000 | ✅ PASSED |
| 2 | 4   | 0.000 | ✅ PASSED |
| 3 | 8   | 0.000 | ✅ PASSED |
| 4 | 16  | 0.000 | ✅ PASSED |
| 5 | 32  | 0.000 | ✅ PASSED |
| 6 | 64  | 0.000 | ✅ PASSED |
| 7 | 128 | 0.000 | ✅ PASSED |


### PAPR Analysis — Golay OFDM vs Random BPSK OFDM

Golay-coded OFDM achieves exactly **3.010 dB PAPR** — the theoretical maximum — while random BPSK OFDM averages 5.5 dB with peaks reaching 11+ dB.

![Time-domain comparison](figures/plot3_timedomain.png)

![CCDF of PAPR](figures/plot4_ccdf_papr.png)

| Metric | Golay OFDM | Random BPSK OFDM |
|--------|-----------|-----------------|
| PAPR (measured) | **3.010 dB** | 5.50 dB (mean) |
| PAPR (worst case) | **3.010 dB** | 11.80 dB |
| Theoretical bound | **≤ 3.010 dB** | Unbounded |
| Pr(PAPR > 6 dB) | **0%** | ~28% |
| Pr(PAPR > 9 dB) | **0%** | ~4% |


### Channel Estimation — Golay vs Least Squares

The Golay estimator achieves **~22× lower Mean Square Error** than a Least Squares baseline using random pilots at SNR = 20 dB.

![True vs estimated channel](figures/plot5_channel_estimate.png)

![MSE vs SNR](figures/plot6_mse_vs_snr.png)

The flat combined spectrum `|A(f)|² + |B(f)|² = 2N` ensures uniform estimation quality at all frequencies — random pilots have spectral nulls that amplify noise unevenly.



# How to Run

### Option A — Google Colab (easiest)

1. Upload `Golay_DSP_Project.ipynb` to [colab.research.google.com](https://colab.research.google.com)
2. Click **Runtime → Run all**
3. All 6 plots will appear inline. No setup required — NumPy and Matplotlib are pre-installed.

### Option B — Run scripts locally


# Clone the repo
git clone https://github.com/sarlapraghna-07/Golay-Complementary-Sequences.git
cd Golay-Complementary-Sequences

# Install dependencies
pip install -r requirements.txt

# Section 1 — Sequence generation and verification
python golay_pairs.py

# Section 2 — PAPR analysis
python papr_analysis.py

# Section 3 — Channel estimation
python channel_estimation.py
```

### Option C — Jupyter locally

pip install -r requirements.txt
jupyter notebook Golay_DSP_Project.ipynb


## Section 1 — Sequence Generation & Verification

File: `golay_pairs.py`

Implements the Golay-Rudin-Shapiro recursive construction**:

Base:       a = [1],  b = [1]
Recursion:  a_new = [a,  b]
            b_new = [a, -b]


At each step the length doubles. After `n` steps both sequences have length `2ⁿ` and the complementary property is preserved.

Plots produced:

| Plot | Description |
| Plot 1 | Stem plots of sequences `a` and `b` — values are ±1 |
| Plot 2 | Autocorrelations of `a` and `b` individually + their sum (perfect impulse) |

![Sequences](figures/plot1_sequences.png)


## Section 2 — PAPR Analysis in OFDM

File: `papr_analysis.py`

OFDM maps subcarrier symbols to a time-domain signal via IFFT:


x[n] = IFFT( X[k] )  ×  √N

PAPR is defined as:

PAPR (dB) = 10 · log₁₀( max|x[n]|² / mean|x[n]|² )


When a Golay sequence is used as the subcarrier vector `X[k]`, the flat power spectrum `|A(f)|² + |B(f)|² = 2N` mathematically bounds PAPR to **≤ 3.010 dB**.

The experiment generates **5000 random BPSK OFDM symbols** and plots their PAPR distribution against Golay — showing the full gap.

Plots produced:

| Plot | Description |
| Plot 3 | Time-domain magnitude — Golay stays flat, random spikes |
| Plot 4 | CCDF of PAPR — the standard DSP comparison plot |



## Section 3 — Channel Estimation

File:`channel_estimation.py`

Simulates a 5-tap multipath channel with complex Gaussian gains and exponential power decay. Both sequences `a` and `b` are transmitted and received as `y_a`, `y_b`.

Golay estimator (frequency domain):

H_est(f) = ( Y_a(f)·conj(A(f))  +  Y_b(f)·conj(B(f)) ) / 2N

The LS baseline uses a single random BPSK pilot:

H_est(f) = Y(f) / X(f)

Random pilots can have spectral nulls where `|X(f)| ≈ 0`, causing noise amplification at those frequencies. Golay pairs have no nulls by design.

The experiment runs **200 Monte Carlo trials** at each SNR from 0 to 30 dB.

Plots produced:

| Plot | Description |
| Plot 5 | Bar + stem: True channel vs Golay estimate vs LS estimate |
| Plot 6 | MSE vs SNR (log scale) — Golay vs LS random pilot |


## Requirements

numpy>=1.24
scipy>=1.10
matplotlib>=3.7
jupyter>=1.0


Install with:

pip install -r requirements.txt




## References

1. Golay, M. J. E. (1961). Complementary series. *IRE Transactions on Information Theory*, 7(2), 82–87.
2. Davis, J. A., & Jedwab, J. (1999). Peak-to-mean power control in OFDM, Golay complementary sequences, and Reed-Muller codes. *IEEE Transactions on Information Theory*, 45(7), 2397–2417.
3. Tarokh, V., & Jafarkhani, H. (2000). On the computation and reduction of the peak-to-average power ratio in multicarrier communications. *IEEE Transactions on Communications*, 48(1), 37–44.
4. Proakis, J. G., & Manolakis, D. G. (2007). *Digital Signal Processing* (4th ed.). Prentice Hall.

---

<p align="center">
  Made as a DSP course project &nbsp;|&nbsp; Python · NumPy · Matplotlib
</p>

