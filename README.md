# Duress-Aware Banking: AI-Driven Detection of Unsafe User Behaviour through Multimodal Anomaly Analysis
> MSc Dissertation — Distinction | Teesside University, 2025 | Formatted for IEEE Access submission

---

## Overview

Traditional fraud detection systems identify unauthorised access and unusual transactions — but they fail to detect a critical gap: **legitimate users acting under coercion or psychological duress**. If a user is forced to make a bank transfer at gunpoint, their credentials are valid, their device is real, and the transaction appears intentional. Standard fraud models cannot help them.

This dissertation proposes a **duress-aware multimodal detection framework** (CFG-Fusion) that goes beyond fraud classification to ask a fundamentally different question: *"Is this verified user safe?"*

The framework combines two detection streams:
- A **supervised transactional fraud model** trained on the IEEE-CIS Fraud Detection dataset (Random Forest, XGBoost, LightGBM)
- An **unsupervised behavioural anomaly detector** trained exclusively on normal keystroke dynamics — detecting deviations in typing rhythm that may signal stress or coercion

These streams are fused into a single duress risk score, evaluated under strict safety constraints — prioritising low false positives to avoid disrupting legitimate users in already-dangerous situations.

---

## Research Question

> *To what extent can integrating transactional fraud signals and behavioural keystroke dynamics support the detection of potentially risky or coercive user activity in digital banking systems under strict low false-positive constraints?*

---

## System Architecture

The full pipeline consists of four components:

```
Data Collection
    ├── IEEE-CIS Fraud Dataset → Data Preprocessing (encoding, scaling, SMOTE)
    │                         → Transactional Models (RF, XGBoost, LightGBM)
    │                         → Fraud Probability Score
    │
    └── Keystroke Dynamics Dataset → Autoencoder (trained on normal sessions only)
                                   → Reconstruction Error → Anomaly Score
                                   
Both streams → Fusion Layer (Logistic Regression) → Combined Duress Risk Score
                                                  → Decision Alert Engine
                                                  → Normal | Suspicious | Duress Alert
                                                  → User Interface (Ipywidgets prototype)
```

---

## Datasets

| Dataset | Size | Description |
|---------|------|-------------|
| IEEE-CIS Fraud Detection | ~590k transactions | Anonymised online transactions with numerical, categorical, device, and time features. Fraud class = 3.5% (severe imbalance). Source: Kaggle |
| Keystroke Dynamics Benchmark | 20,400 sessions | Fine-grained typing timing features: key-hold duration (H), down-down latency (DD), up-down latency (UD). Split: 16,320 train / 4,080 validation |

---

## Models & Methods

### Stream 1 — Transactional Fraud Detection (Supervised)

Four model configurations trained and compared:

| Model | Accuracy | Precision | Recall | F1-Score | PR-AUC |
|-------|----------|-----------|--------|----------|--------|
| LightGBM (Class-Weight) | 0.870 | 0.1800 | 0.7800 | 0.3000 | 0.5562 |
| LightGBM (SMOTE) | 0.970 | 0.6873 | 0.5168 | 0.5900 | 0.6071 |
| XGBoost | 0.976 | 0.7203 | 0.5217 | 0.6051 | 0.6273 |
| **Random Forest** | **0.979** | **0.7260** | **0.5930** | **0.6528** | **0.6812** |

**Random Forest** was selected as the primary transactional baseline — highest F1-score and PR-AUC, best balance between fraud detection and false alarm control.

**Why PR-AUC over ROC-AUC?** ROC-AUC is misleading on highly imbalanced datasets. PR-AUC provides a more realistic measure of classifier performance when the negative class dominates.

**Class imbalance strategy:** SMOTE was preferred over class weighting. Class weighting achieved high recall (0.78) but extremely low precision (0.18) — too many false positives for a safety-critical user-facing system.

### Stream 2 — Behavioural Anomaly Detection (Unsupervised)

A **feedforward autoencoder** trained exclusively on normal keystroke sessions learns a compact representation of typical typing behaviour. Anomalous sessions produce high reconstruction error.

- Anomaly threshold: **95th percentile** of training reconstruction errors = **0.7231**
- Anomalies detected: **204 out of 20,400 sessions (~1%)** — indicating effective modelling of stable typing patterns
- The autoencoder acts as both a duress detector and a behavioural "liveness" indicator — micro-temporal latencies measured in milliseconds are driven by neuromuscular patterns impossible to replicate under mimicry attacks

### Fusion Layer

Transactional fraud probability and behavioural anomaly scores are combined via logistic regression into a single duress risk score.

### Safety-Critical Evaluation: Recall@1% FPR

Standard accuracy metrics are insufficient for this task. The key evaluation metric is **Recall at 1% False Positive Rate (Recall@1% FPR)** — reflecting real-world banking constraints where only a tiny proportion of sessions can be flagged without disrupting legitimate users.

| Model | Stream | Threshold | Recall@1% FPR |
|-------|--------|-----------|---------------|
| Random Forest (RF) | Transactional | 0.2300 | **0.6402** |
| Fusion (RF + Autoencoder) | Multimodal | 0.1840 | 0.5625 |

**Key finding:** Naive score-level fusion did not outperform the transactional baseline under strict false-positive constraints. The behavioural signal introduces additional variance that dilutes the transactional model's concentrated signal at the 1% threshold. This is a research insight, not a failure — it demonstrates that successful multimodal fusion requires temporal alignment, feature-level integration, or attention-based gating rather than simple score combination.

---

## Explainability — SHAP Analysis

SHAP (SHapley Additive Explanations) was applied to the fused duress-risk model to ensure transparency and support regulatory compliance in banking contexts.

Key findings from SHAP:
- Transactional anomalies account for the majority of the overall risk signal
- Behavioural (keystroke) irregularities provide a secondary but meaningful contribution
- Duress risk is highest when **high transactional risk combines with atypical typing behaviour** — validating the multimodal design rationale

---

## Prototype Interface

An interactive **Ipywidgets prototype** demonstrates real-time duress risk assessment:
- Adjustable transactional and behavioural risk inputs
- Continuous risk score output (not binary) — Low / Moderate / High
- Operates on abstracted risk scores only — no raw keystroke data stored, aligned with GDPR data minimisation principles

---

## Privacy, Ethics & GDPR

This project includes a comprehensive **Data Protection Impact Assessment (DPIA)** and ethics analysis:

- Raw keystroke sequences are never stored — only timing-based aggregate features are processed
- The system supports safety alerts rather than automatic enforcement — duress risk guides secondary verification or discreet support, not transaction blocking
- Fairness consideration: global anomaly thresholds may disadvantage users with neurodegenerative conditions, age-related motor changes, or disabilities. Future work should implement **personalised behavioural baselines** rather than fixed global thresholds
- Aligned with EU GDPR, EU AI Act principles, and Trustworthy AI frameworks

---

## Project Structure

```
duress-aware-banking/
│
├── S3293279_CIS4055_DuressAwareBanking.ipynb   # Main dissertation notebook
├── S3293279_CIS4055_DuressAwareBanking.pdf     # Submitted dissertation (IEEE Access format)
├── README.md                                    # Project documentation
```

**Datasets** (not included — available via Kaggle):
- [IEEE-CIS Fraud Detection Dataset](https://www.kaggle.com/c/ieee-fraud-detection)
- [Keystroke Dynamics Benchmark Dataset](https://www.kaggle.com/datasets/carnegiecylab/keystroke-dynamics-benchmark)

---

## How to Run

1. Clone the repository:
```bash
git clone https://github.com/TY-GITH/duress-aware-banking.git
cd duress-aware-banking
```

2. Install dependencies:
```bash
pip install numpy pandas matplotlib seaborn scikit-learn imbalanced-learn xgboost lightgbm tensorflow keras shap ipywidgets
```

3. Download datasets from Kaggle and place in the project directory

4. Open the notebook:
```bash
jupyter notebook S3293279_CIS4055_DuressAwareBanking.ipynb
```

Run all cells top to bottom. Sections are clearly labelled: data preprocessing, transactional modelling, behavioural autoencoder, fusion layer, SHAP analysis, and prototype interface.

---

## Technologies Used

- **Python 3**
- **Scikit-learn** — Random Forest, Logistic Regression fusion, evaluation metrics
- **XGBoost / LightGBM** — gradient boosting transactional models
- **TensorFlow / Keras** — feedforward autoencoder for behavioural anomaly detection
- **SHAP** — model explainability and feature contribution analysis
- **imbalanced-learn** — SMOTE implementation
- **Pandas / NumPy** — data preprocessing pipelines
- **Matplotlib / Seaborn** — visualisation, confusion matrices, PR curves
- **Ipywidgets** — interactive prototype interface

---

## Key Contributions

1. **Novel problem framing** — redefines duress detection as a user safety challenge rather than a fraud classification problem, shifting the question from "Is this the right user?" to "Is this verified user safe?"

2. **Multimodal framework** — first integration of IEEE-CIS transactional fraud detection with keystroke dynamics anomaly scoring for duress-aware banking

3. **Safety-oriented evaluation** — adoption of Recall@1% FPR as the primary metric, reflecting realistic banking deployment constraints rather than offline accuracy optimisation

4. **Critical fusion insight** — demonstrates that naive score-level fusion does not outperform strong unimodal baselines under conservative operating conditions, pointing toward the need for feature-level or attention-based integration

5. **Explainability and compliance** — SHAP analysis, DPIA, and GDPR-aligned prototype design demonstrating responsible deployment standards

---

## Future Work

- **Feature-level fusion** — replace score-level integration with joint feature representations to exploit cross-modal dependencies
- **Temporal alignment** — link transactional and keystroke datasets at the user-session level for genuine multimodal modelling
- **Personalised behavioural baselines** — adaptive per-user anomaly thresholds to ensure fairness for users with disabilities, neurological conditions, or age-related motor variation
- **Attention-based gating** — suppress behavioural signal during high-confidence transactional events; activate only when transactional evidence is ambiguous
- **Sequence-aware architectures** — RNNs or temporal autoencoders to capture behavioural evolution within a session rather than relying on aggregated features
- **Controlled duress simulation studies** — ethically structured user studies to generate labelled duress ground truth data

---

## Academic Context

**Degree:** MSc Artificial Intelligence with Advanced Practice  
**Institution:** Teesside University, UK  
**Grade:** Distinction  
**Year:** 2025  
**Module:** CIS4055 Advanced Practice  
**Format:** Formatted for IEEE Access submission

---

## Author

**Omotoyosi Ogunmola**  
MSc Artificial Intelligence with Advanced Practice (Distinction) — Teesside University  
[GitHub](https://github.com/TY-GITH) · [LinkedIn](https://linkedin.com/in/omotoyosi-o-74835722a) · abikedaisy@gmail.com
