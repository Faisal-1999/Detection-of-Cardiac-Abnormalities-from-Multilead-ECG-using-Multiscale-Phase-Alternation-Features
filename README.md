# Detection of Cardiac Abnormalities from Multilead ECG using Multiscale Phase Alternation Features.

The approach extracts 48-dimensional **Multiscale Phase Alternation (PA)** features from complex wavelet sub-bands of 12-lead ECG signals via **Dual-Tree Complex Wavelet Transform (DTCWT)** and classifies them using **Fuzzy K-Nearest Neighbors (Fuzzy KNN)**

---

## 1. Project Background & Clinical Motivation

Cardiovascular diseases, including Myocardial Infarction (MI), Bundle Branch Block (BBB), and Heart Muscle Diseases (HMD such as Cardiomyopathy and Hypertrophy), are leading causes of life-debilitating cardiac events and sudden cardiac arrest. 

* **Bundle Branch Block (BBB):** Delays in the heart's ventricular conduction system manifest as wide QRS-complexes, wide R-waves and S-waves in leads V1 and V6, absence of septal Q-waves, and T-wave inversions.
* **Myocardial Infarction (MI):** Manifests as morphological variations in low-frequency segments across specific leads, including ST-segment elevation/depression, pathological Q-waves, and deep T-wave inversion.
* **Heart Muscle Disease (HMD):** Hypertrophy and cardiomyopathy exhibit ST-segment depression and prominent T-wave inversions across leads I, II, aVL, aVF, and precordial leads V4–V6.

Because visual inspection across continuous 24-hour multilead recordings is labor-intensive and prone to human error, this framework provides an automated classification system using phase transitions across wavelet decomposition levels without requiring prior QRS onset/offset point detection.

---

## 2. Theoretical Overview & Mathematical Modeling

Multilead ECG (12-Lead, 4096 Samples per frame)
                                  │
                                  ▼
         Preprocessing (0.5 Hz Butterworth HPF + Denoising)
                                  │
                                  ▼
               6-Level DTCWT Sub-band Decomposition
                  (cA6, cD6, cD5, cD4 Sub-bands)
                                  │
                                  ▼
              Sub-band Phase Extraction (Eq. 1 & 2)
                                  │
                                  ▼
          Multiscale Phase Alternation (PA) Computation
                  (48-Dimensional Feature Vector)
                                  │
                                  ▼
             Feature Selection (ANOVA / SD Scoring)
                                  │
                                  ▼
                 Fuzzy KNN Classification (K = 5)
                                  │
           ┌──────────────────────┼──────────────────────┬──────────────────────┐
           ▼                      ▼                      ▼                      ▼
  Bundle Branch Block   Heart Muscle Disease   Myocardial Infarction    Healthy Control
         (BBB)                  (HMD)                  (MI)                  (HC)

---

### Step 1: Preprocessing & Multiscale Decomposition
1. **Filtering:** Low-frequency baseline wander is removed using a zero-phase Butterworth high-pass filter with a cutoff frequency of 0.5 Hz. High-frequency noise is suppressed using discrete wavelet thresholding.
2. **Segmentation:** Continuous 12-lead ECG signals are framed into non-overlapping segments of size $4096 \times 12$.
3. **DTCWT Decomposition:** A 6-level Dual-Tree Complex Wavelet Transform decomposes each lead $m$ into real and imaginary wavelet coefficients using dual-tree filter banks with approximate Hilbert transform symmetry.:
   $$cA_L^m(k) = \tilde{cA}_L^m(k) + j\,\overline{cA}_L^m(k)$$
.
   $$cD_l^m(k) = \tilde{cD}_l^m(k) + j\,\overline{cD}_l^m(k)$$
.
   where $l \in \{1, 2, \dots, 6\}$ denotes the detail decomposition level and $L=6$ represents the approximation scale.
4. ### Sub-band Phase Evaluation

The phase angle $\phi(k)$ is extracted for the diagnostically significant sub-bands ($cA_6$, $cD_6$, $cD_5$, and $cD_4$):

$$
\phi_{cA_L}^m(k) = \tan^{-1}\left( \frac{\overline{cA}_L^m(k)}{\tilde{cA}_L^m(k)} \right)
$$

$$
\phi_{cD_l}^m(k) = \tan^{-1}\left( \frac{\overline{cD}_l^m(k)}{\tilde{cD}_l^m(k)} \right)
$$
.

---

### Step 2: Multiscale Phase Alternation (PA) Feature Extraction
For sub-band $t \in \{cA_6, cD_6, cD_5, cD_4\}$ of length $n_t$ along lead $m$, the temporal phase vectors are split into adjacent shifts.:
$$\phi 1_t(i) = \phi_t(1 : n_t - 1), \quad \phi 2_t(i) = \phi_t(2 : n_t)$$
.
The Phase Ratio ($PR$) sequence is calculated as.:
$$PR_t(i) = \frac{\phi 1_t(i)}{\phi 2_t(i)}, \quad 1 \le i \le n_t - 1$$
.
The Multiscale Phase Alternation feature $PA_t^m$ is defined as the total number of indices where $PR_t(i) < 0$.:
$$PA_t^m = g_t = \sum \mathbb{I}\left(PR_t(i) < 0\right)$$
.
Aggregating 4 sub-bands ($cA_6, cD_6, cD_5, cD_4$) across all 12 ECG leads produces a **48-dimensional feature vector** for each frame.

---

### Step 3: Feature Selection
To eliminate redundant components, two feature selection metrics are evaluated.:

1. **One-Way ANOVA:** Computes the variance ratio across class distributions. Features with $p\text{-value} < 0.001$ are selected for classification (eliminating 4 non-significant features).
2. **Statistical Dependency (SD) Score:** Quantifies joint information between quantized feature instances $z^r$ and class labels $y$.:
   $$SD^r = \sum_{z^r} \sum_{y} P(z^r, y) \frac{P(z^r, y)}{P(y) P(z^r)}$$
.
   Features satisfying $SD^r > 0.035$ are retained.

---

### Step 4: Fuzzy K-Nearest Neighbor (Fuzzy KNN) Classifier
Unlike hard KNN, Keller's Fuzzy KNN assigns class membership vectors $u_i(x)$ to a query instance $x$ based on Euclidean distance weighting over its $K$ nearest neighbors.:

$$u_i(x) = \frac{\sum_{j=1}^K u_{ij} \left( \Vert{}x - x_j\Vert{}^{-2 / (m - 1)} \right)}{\sum_{j=1}^K \left( \Vert{}x - x_j\Vert{}^{-2 / (m - 1)} \right)}$$

* **Parameters:** Neighborhood size $K = 5$.
* **Validation:** Evaluated using 5-fold cross-validation.
* **Decision Rule:** The predicted class label corresponds to the maximum membership value $\arg\max_i u_i(x)$.

---

## 3. Dataset Information

This project uses the **PTB Diagnostic ECG Database**:
* **Source:** [PhysioNet PTB Diagnostic ECG Database](https://physionet.org/content/ptbdb/1.0.0/)[cite: 1, 2] | [Kaggle Dataset Mirror](https://www.kaggle.com/datasets/faisalziyadahmed/ptb-ecg-dataset)
* **Sampling Frequency:** 1000 Hz
* **Leads:** Standard 12-lead configurations (I, II, III, aVR, aVL, aVF, V1, V2, V3, V4, V5, V6)[cite: 1, 2]
* **Recordings:** Continuous recordings divided into non-overlapping frames of $4096 \times 12$ samples.
* **Target Classes:**
  * **Healthy Control (HC):** 16 subjects / 352 frames.
  * **Bundle Branch Block (BBB):** 16 subjects / 352 frames.
  * **Myocardial Infarction (MI):** 16 subjects / 352 frames.
  * **Heart Muscle Disease (HMD):** 20 subjects (13 cardiomyopathy, 7 hypertrophy) / 420 frames.
  * **Total Evaluated Instances:** 1,476 frames.

---

## 4. Benchmark Performance

Experimental results using 5-fold cross-validation reported in the research paper.:

| Feature Set | Classifier | HC Sens. (%) | HMD Sens. (%) | MI Sens. (%) | BBB Sens. (%) | Overall Acc. (%) |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **ANOVA Selected**. | **Fuzzy KNN**. | **92.33**. | **80.90**. | **94.31**. | **78.12**. | **86.09**. |
| **SD Selected**. | **Fuzzy KNN**. | 90.63. | 78.86. | 95.17. | 77.27. | 85.09. |
| **All 48 Features**. | **Fuzzy KNN**. | 89.46. | 78.86. | 94.60. | 78.41. | 84.96. |
| **ANOVA Selected**. | Standard KNN. | 87.14. | 81.82. | 92.86. | 76.06. | 84.28. |
| **SD Selected**. | Standard KNN. | 95.71. | 77.27. | 91.55. | 75.71. | 84.61. |
| **All 48 Features**. | Standard KNN. | 90.00. | 77.27. | 94.37. | 77.14. | 84.29. |

---

## 5. Repository Structure

```text
├── result                                 #
├── Detection of Cardiac Abnormality       # Main Jupyter Notebook
├── README.md                              # Project documentation
