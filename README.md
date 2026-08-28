# Detection-of-Cardiac-Abnormalities-from-Multilead-ECG-using-Multiscale-Phase-Alternation-Features

A complete implementation of the automated cardiac pathology detection methodology proposed by R. K. Tripathy and S. Dandapat (*Journal of Medical Systems*, Springer, 2016)[cite: 1]. The system extracts 48-dimensional **Multiscale Phase Alternation (PA)** features from complex wavelet sub-bands of 12-lead ECG signals via **Dual-Tree Complex Wavelet Transform (DTCWT)** and classifies them using **Fuzzy K-Nearest Neighbors (Fuzzy KNN)**[cite: 1].

---

## 1. Project Background & Clinical Motivation

Cardiovascular diseases, including Myocardial Infarction (MI), Bundle Branch Block (BBB), and Heart Muscle Diseases (HMD such as Cardiomyopathy and Hypertrophy), are leading causes of life-debilitating cardiac events and sudden cardiac arrest[cite: 1]. 

* **Bundle Branch Block (BBB):** Delays in the heart's ventricular conduction system manifest as wide QRS-complexes, wide R-waves and S-waves in leads V1 and V6, absence of septal Q-waves, and T-wave inversions[cite: 1].
* **Myocardial Infarction (MI):** Manifests as morphological variations in low-frequency segments across specific leads, including ST-segment elevation/depression, pathological Q-waves, and deep T-wave inversion[cite: 1].
* **Heart Muscle Disease (HMD):** Hypertrophy and cardiomyopathy exhibit ST-segment depression and prominent T-wave inversions across leads I, II, aVL, aVF, and precordial leads V4–V6[cite: 1].

Because visual inspection across continuous 24-hour multilead recordings is labor-intensive and prone to human error, this framework provides an automated classification system using phase transitions across wavelet decomposition levels without requiring prior QRS onset/offset point detection[cite: 1].

---

## 2. Theoretical Pipeline & Mathematical Modeling
