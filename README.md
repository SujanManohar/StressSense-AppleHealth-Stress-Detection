# StressSense: ML-Driven Stress Profiling from Apple Health Data

StressSense is a personalized stress-detection pipeline built on Apple Watch + Apple Health data (HRV, heart rate, resting heart rate, sleep, steps, and active energy).  
Instead of using generic stress scores, StressSense creates a **rolling HRV baseline** per user and detects **physiological stress days** when HRV drops meaningfully below that baseline. Predictions are made with **Logistic Regression** (interpretable baseline) and **Random Forest** (strong performance), and explained using **SHAP**.

## Project Motivation
Wearables capture a lot of physiology, but most stress scores are:
- opaque (no “why”)
- not personalized (HRV varies massively by person)
- noisy and context-dependent

StressSense focuses on **transparent, baseline-driven** stress detection.

--------

## What This Project Does
1. Parses Apple Health export data (XML) and extracts relevant metrics (HRV, HR, RHR, sleep, steps, energy)
2. Aggregates irregular events into **daily metrics**
3. Builds a **14-day rolling HRV baseline**
4. Labels stress days as:  
   **stress = 1 if HRV is >10% below baseline, else 0**
5. Trains and evaluates:
   - Logistic Regression
   - Random Forest
6. Explains model behavior using SHAP (global + per-day explanations)

---

## Dataset
- Source: Apple Health export (iPhone + Apple Watch)
- Time span: **Sept 2022 → Dec 2025**
- Granularity: raw event-level records aggregated to daily features

**Privacy:** Raw Apple Health export data is not included in this repo.

---

## Features Engineered (examples)
Core physiology + behavior:
- `hrv` (daily HRV)
- `resting_hr`, `avg_hr`
- `sleep_hours`
- `steps`, `active_energy`

Personalized baseline + stress signal:
- `hrv_baseline` (14-day rolling mean)
- `hrv_deviation` = (hrv - baseline) / baseline
- `stress_label` = 1 if deviation < -0.10 else 0

Trends + lags:
- previous-day lags (HRV, sleep, steps, energy, resting HR)
- 7-day rolling averages (sleep, steps, HRV, RHR)
- ratios like `hrv_to_rhr`
- `cardio_strain` = avg_hr / resting_hr
- `activity_load` = steps + active_energy

---

## Modeling Approach
To avoid time leakage, data is split chronologically:
- Train / Validation / Test = **70/15/15**

Models:
- **Logistic Regression**: fast baseline + clear feature direction
- **Random Forest**: captures non-linear interactions (better performance)

Evaluation metrics:
- Precision, Recall, F1-score, ROC-AUC

---

## Results Summary (from report)
Random Forest outperformed Logistic Regression, especially for stress precision, while maintaining perfect stress recall in the test window.

---

## Interpretability (SHAP)
SHAP is used to explain:
- Which features matter most globally (bar plot)
- How features push predictions per-day (waterfall)
- Feature impact distribution (beeswarm)

Key driver: **HRV deviation** (baseline-adjusted HRV drop) is consistently the strongest signal.

---

## How to Run (local)
### 1) Create environment
Using pip:
```bash
python -m venv .venv
source .venv/bin/activate   # mac/linux
# .venv\Scripts\activate    # windows

pip install -r requirements.txt
