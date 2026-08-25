# From Prediction Toward Action: Explainable Risk Modeling and Exploratory Uplift Analysis for Patient No-Show Management

<p align="center">
  <strong>A decision-oriented machine-learning framework for moving from no-show risk prediction to transparent, resource-aware intervention analysis.</strong>
</p>

<p align="center">
  <img alt="Python 3.10" src="https://img.shields.io/badge/Python-3.10-blue.svg">
  <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-yellow.svg">
  <img alt="Task: Healthcare ML" src="https://img.shields.io/badge/Task-Healthcare%20ML-green.svg">
  <img alt="Status: Research" src="https://img.shields.io/badge/Status-Research%20Prototype-lightgrey.svg">
</p>

> **Research status.** This repository accompanies a research manuscript on patient no-show management. The framework is intended for reproducible research and decision-support experimentation—not for direct clinical deployment. The uplift results are exploratory and must **not** be interpreted as identified causal effects.

---

## Overview

Missed outpatient appointments create operational inefficiency, delay care, and can worsen access disparities. Most no-show studies stop at **risk prediction**: *Who is likely to miss an appointment?* This work extends the analysis toward a second decision question: *Who may benefit from an intervention such as an SMS reminder, and when would intervention be operationally worthwhile?*

The study integrates three distinct decision layers:

1. **Risk** — estimate the probability of a no-show.
2. **Responsiveness** — explore heterogeneity in estimated response to SMS reminders.
3. **Treatment priority** — connect estimated responsiveness to intervention cost and operational benefit.

The core contribution is not a new classifier. It is a **decision-oriented analytical workflow** that combines predictive modeling, explainability, graph-based learning, exploratory uplift estimation, and cost-aware targeting while making the assumptions and deployment limitations explicit.

---

## Framework

<p align="center">
  <img src="proposed_framework.png" alt="Decision-oriented analytical framework" width="900">
</p>

The pipeline covers:

**Kaggle data → preprocessing → train/test split → base-model benchmarking → Optuna tuning → GCN modeling → ensemble modeling → SHAP/LIME explanations → exploratory uplift analysis → cost-aware decision framing**.

---

## Research question

> **How can no-show risk prediction and heterogeneous responsiveness estimates be integrated to support targeted reminder decision-making under realistic resource constraints?**

This framing deliberately separates predictive accuracy from intervention value: a high-risk patient is not necessarily the patient with the largest expected response to outreach.

---

## Dataset

The experiments use the public **No-show Appointments** dataset by J. Arroba on Kaggle:

- **Source:** https://www.kaggle.com/datasets/joniarroba/noshowappointments
- **Setting:** outpatient appointments in Brazil
- **Observations:** **110,527** appointments
- **No-show prevalence:** **20.2%**
- **Raw variables:** 14 appointment/patient attributes in the source dataset
- **Target:** `No-show`, with no-show treated as the positive class

Representative variables include age, gender, hypertension, diabetes, alcoholism, handicap status, scholarship/welfare enrollment, neighbourhood, scheduling date, appointment date, and SMS reminder receipt.

`PatientId` and `AppointmentID` are used for data management and excluded from the predictive feature space.

> **Data licensing.** The repository's MIT License applies to repository code and original repository materials only. The Kaggle dataset remains subject to its own source terms/license and is **not relicensed** by this repository.

---

## Preprocessing and feature engineering

The manuscript describes the following pipeline:

- harmonize variable names and types;
- remove invalid observations such as negative age;
- derive appointment day-of-week, appointment month, and waiting time;
- set negative waiting-time values to zero;
- create second-degree polynomial terms for age and waiting time;
- add selected interactions, including:
  - `Age_x_Hypertension`,
  - `SMSReceived_x_Waiting_Time_Days`,
  - `Gender_x_Alcoholism`;
- target-encode high-cardinality categorical variables using **training data only**;
- one-hot encode lower-cardinality categorical variables;
- standardize continuous features;
- apply **SMOTE to the training partition only**;
- fit all transformations inside the training partition and apply them unchanged to the test partition.

### Important temporal-design caveat

`SMS_received` is used as a retrospective predictive covariate in the current analysis but also serves as the treatment in the intervention analysis. A production targeting model should therefore be retrained using **pretreatment covariates only**, excluding the treatment and any post-treatment information.

---

## Models

### Base classifiers

- Random Forest (RF)
- XGBoost (XGB)
- LightGBM (LGBM)
- CatBoost
- HistGradientBoosting
- Multilayer Perceptron / DNN with hidden layers `(100, 50, 25)`

### Graph model

A two-layer **Graph Convolutional Network (GCN)** is built from a feature-similarity k-nearest-neighbor graph with `k = 5`, ReLU activations, and dropout. The reported GCN experiment is trained for 200 epochs.

### Ensembles

The study evaluates multiple:

- voting ensembles,
- stacking ensembles,
- hybrid bagging–boosting configurations.

### Hyperparameter optimization

Base models are tuned with **Optuna**, using ROC AUC as the optimization objective, first with a validation split and then with **10-fold cross-validation** on the training data.

---

## Evaluation protocol

- **Split:** stratified 80/20 train/test split at the appointment level
- **Primary minority-class metric:** F1-score for the no-show class
- **Additional metrics:** precision, recall, accuracy, ROC AUC
- **Default classification threshold:** 0.5 unless otherwise stated
- **Ranking metric:** ROC AUC

Because some patients contribute multiple appointments, the appointment-level split can place records from the same patient in both training and test sets. Patient-grouped and temporal validation are therefore required before claims of prospective deployment performance.

---

## Headline results

### Predictive performance

| Model / setting | Test accuracy | Precision | Recall | F1 | ROC AUC | Interpretation |
|---|---:|---:|---:|---:|---:|---|
| Random Forest, baseline | 66% | 32% | **60%** | **41%** | 0.71 | Strongest baseline sensitivity–F1 trade-off |
| XGBoost, baseline | 76% | 35% | 21% | 26% | 0.71 | Higher accuracy, weaker no-show sensitivity |
| LightGBM, baseline | 76% | 35% | 23% | 28% | 0.71 | Higher accuracy, weaker no-show sensitivity |
| CatBoost, baseline | 73% | 34% | 42% | 37% | 0.71 | More balanced than XGB/LGBM baseline |
| DNN, baseline | 72% | 33% | 39% | 36% | 0.69 | Competitive but below best tree models |
| HistGradientBoosting, baseline | 72% | 34% | 42% | 37% | 0.71 | Similar to CatBoost baseline |
| GCN | 72% | 34% | 43% | 38% | 0.69 | No improvement over best tabular baseline |
| LightGBM, tuned | 79% | 43% | 16% | 21% | **0.73** | Joint-best tuned ranking performance |
| CatBoost, tuned | 79% | 42% | 12% | 19% | **0.73** | Joint-best tuned ranking performance |

**Key result:** tuned gradient-boosting models reach the strongest ranking performance (**ROC AUC up to 0.73**), while Random Forest provides the most useful sensitivity–precision balance for detecting the minority no-show class.

### Ensemble behavior

Voting ensembles achieve superficially high test accuracy near the majority-class share, but minority detection collapses:

- recall: **0.6%–3.7%**;
- F1: **< 7%**;
- ROC AUC: **0.61–0.63**.

Stacking recovers usable recall in some configurations, but ranking remains around **ROC AUC 0.62–0.63**, below the tuned individual models. The strongest reported stacking F1 is **38%**, with recall reaching approximately **51%–52%** in selected stacks.

**Takeaway:** for imbalanced healthcare operations, high accuracy and model complexity should not be treated as evidence of useful minority-class performance.

---

## Explainable AI

The study applies **SHAP** and **LIME** to examine global and local model behavior.

The most influential correlates include:

1. **waiting time / scheduling lead time**,
2. **age**,
3. **SMS reminder status**.

Operational interpretation distinguishes between feature roles:

- **modifiable:** waiting time, reminder strategy;
- **non-modifiable stratification variables:** age, gender, chronic conditions;
- **policy-sensitive attributes:** welfare/socioeconomic and neighbourhood information, which require explicit equity review before use in targeting.

This distinction is essential: feature importance does not imply that every influential variable is an appropriate intervention lever.

---

## Exploratory uplift analysis

The intervention analysis treats SMS receipt as `T ∈ {0,1}` and no-show as `Y = 1`. Three heterogeneous-effect estimators are examined:

- T-Learner
- Causal Forest DML
- Neural uplift network

### Estimated average treatment effects

| Estimator | Estimated ATE on no-show probability |
|---|---:|
| T-Learner | +0.0428 |
| Causal Forest DML | **−0.0442** |
| Neural uplift network | +0.0141 |

A negative effect corresponds to an estimated reduction in no-show probability. The estimators **disagree in sign**, so the manuscript treats the uplift outputs as **inconclusive, hypothesis-generating decision-support signals**, not causal effects.

### Why the causal claim is intentionally limited

Reminder assignment is strongly associated with scheduling lead time, creating substantial confounding risk. The observational dataset alone cannot verify unconfoundedness or overlap. Credible intervention-effect estimation requires additional diagnostics and, ideally, randomized or quasi-experimental variation.

---

## Cost-aware targeting rule

The manuscript formalizes an operational decision rule that separates risk from treatment priority.

Let:

- `τ̂(x)` = estimated effect of the reminder on no-show probability,
- `B` = operational benefit of preventing a missed appointment,
- `C` = intervention cost.

An intervention is worthwhile when:

```text
-τ̂(x) · B - C > 0
```

Under a budget constraint, patients can be ranked by estimated benefit and treated until the intervention budget is exhausted.

The natural policy baselines are:

- treat all,
- target by predicted no-show risk,
- target by estimated responsiveness,
- random assignment.

> The paper formalizes this policy but does **not** claim that the policy has already been prospectively validated. Quantitative policy evaluation requires credible uplift estimates, explicit benefit/cost values, uncertainty analysis, and budget-constrained evaluation.

---

## Reproducibility

### Reported software stack

The manuscript reports **Python 3.10** with:

- `scikit-learn`
- `imbalanced-learn`
- `xgboost`
- `lightgbm`
- `catboost`
- `torch` / PyTorch
- `torch-geometric`
- `econml`
- `shap`
- `lime`
- `optuna`

Reported reference hardware: **NVIDIA RTX 3090 GPU** and **32 GB RAM**.

### Reproducibility status

| Item | Status from manuscript |
|---|---|
| Public dataset | ✅ Available on Kaggle |
| Dataset size / target prevalence | ✅ Reported |
| Train/test protocol | ✅ Reported |
| Model families | ✅ Reported |
| Core feature engineering | ✅ Reported |
| Evaluation metrics | ✅ Reported |
| Main numerical results | ✅ Reported |
| Exact package versions | ⚠️ Not specified in manuscript |
| Random seeds | ⚠️ Not specified in manuscript |
| Complete executable source code | ⚠️ Not included in the materials provided here |
| Patient-grouped / temporal validation | 🔬 Identified as future work |
| Probability calibration | 🔬 Identified as future work |
| Bootstrap uncertainty intervals | 🔬 Identified as future work |
| Prospective causal validation | 🔬 Required before deployment |

### Recommended repository layout

The following structure is recommended when releasing the executable code:

```text
.
├── README.md
├── LICENSE
├── assets/
│   └── proposed_framework.png
├── configs/
│   ├── base_models.yaml
│   ├── optuna.yaml
│   └── uplift.yaml
├── data/
│   └── README.md              # download instructions; do not redistribute if terms prohibit it
├── notebooks/
│   └── exploratory_analysis.ipynb
├── src/
│   ├── data.py
│   ├── features.py
│   ├── train_baselines.py
│   ├── train_gcn.py
│   ├── train_ensembles.py
│   ├── explain.py
│   ├── uplift.py
│   └── evaluate.py
├── tests/
├── requirements.txt
└── reproduce.sh
```

For a publication-grade release, pin package versions, set and document random seeds, save all Optuna search spaces/configurations, and generate manuscript tables/figures from scripted outputs rather than manual transcription.

---

## Limitations and deployment requirements

The current study is a **proof-of-concept research analysis**. Important limitations include:

- one observational dataset from a single healthcare context and period;
- possible unmeasured confounding;
- appointment-level rather than patient-grouped or temporal splitting;
- no probability-calibration analysis;
- no utility-based threshold optimization;
- cost-aware targeting formalized but not quantitatively evaluated;
- feature-similarity GCN may not capture richer longitudinal/relational structure;
- reported model metrics are point estimates without patient-level bootstrap confidence intervals.

Before operational use, the manuscript recommends a staged validation path:

1. retrain baseline risk models on **pretreatment-only** covariates;
2. perform **patient-grouped and temporal validation**;
3. evaluate **probability calibration** and utility-based thresholds;
4. conduct propensity, overlap, balance, doubly robust, and uncertainty diagnostics for treatment-effect estimation;
5. preferably obtain randomized or quasi-experimental reminder assignment;
6. evaluate budget-constrained policies against treat-all, risk-based, and random baselines;
7. report net-benefit metrics such as prevented no-shows per contact and number needed to remind;
8. perform subgroup fairness and external-validation analyses.

---

## Responsible use

This repository should not be used as an autonomous clinical decision system.

- Predictions are statistical estimates, not clinical judgments.
- Uplift estimates from the current observational analysis are not established causal effects.
- Sensitive or policy-relevant attributes require fairness and equity review.
- Any deployment should include human oversight, prospective validation, monitoring, calibration, and institution-specific governance.

---

## Citation

If you use this work, please cite the accompanying manuscript. Because the supplied manuscript is anonymized, author metadata is intentionally omitted here.

```bibtex
@article{prediction_toward_action_noshow,
  title   = {From Prediction Toward Action: Integrating Explainable Risk Modeling and Exploratory Uplift Analysis for Patient No-Show Management},
  author  = {Authors withheld for review},
  year    = {2026},
  note    = {Manuscript}
}
```

> **Anonymous-review note:** if this repository is used during double-blind peer review, keep author names, institution identifiers, personal accounts, and commit metadata consistent with the venue's anonymity policy. Replace the placeholder citation after de-anonymization/publication.

---

## License

Repository code and original repository materials are released under the **MIT License**. See [`LICENSE`](LICENSE).

The external Kaggle dataset is not covered by the repository MIT License; consult the dataset page for its applicable terms.

---

## Acknowledgment of scope

The project is designed to move the discussion from **“Who is high risk?”** toward the more operationally meaningful sequence:

> **Who is high risk? → Who is likely to respond? → Who should be prioritized under constraints?**

That distinction—between **risk**, **responsiveness**, and **treatment priority**—is the central organizing principle of the framework.
