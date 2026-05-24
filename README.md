# 🔐 E-XAI: Evaluating Explainability of Tree-Based Classifiers for 5G/6G Network Intrusion Detection

> *Can we trust what a model says — and trust why it says it?*  
> This project answers that question for six tree-based classifiers applied to 5G/6G network intrusion detection, using a custom multi-dimensional explainability evaluation framework built on TreeSHAP.

---

## Overview

Modern 5G/6G networks are high-stakes environments where intrusion detection systems (IDS) must be not only accurate, but *explainable*. Security analysts need to understand *why* a model flagged a flow as malicious — not just that it did.

This project trains and evaluates **six tree-based classifiers** on the **5G-NIDD dataset** for binary intrusion detection (Benign vs. Malicious), then applies a custom **E-XAI (Explainable AI Evaluation) framework** to measure the quality of TreeSHAP explanations across seven interpretability dimensions, revealing that models with near-identical accuracy can differ dramatically in how trustworthy, efficient, and robust their explanations are.

---

## Key Results at a Glance

| Model | Accuracy | ROC-AUC | Completeness ↑ | Robustness ↑ | SHAP Time (s) ↓ |
|---|---|---|---|---|---|
| Decision Tree | 97.04% | 0.9981 | **0.9758** | 0.0000 | **1.49** |
| Random Forest | 97.04% | 0.9982 | 0.4105 | 0.0000 | 487.22 |
| Gradient Boosting | 96.94% | 0.9980 | 0.6872 | 0.0434 | 9.16 |
| XGBoost | 97.04% | 0.9982 | 0.6401 | 0.0702 | 14.88 |
| LightGBM | 97.04% | 0.9982 | 0.8451 | 0.0699 | 43.67 |
| **CatBoost** ⭐ | **97.04%** | **0.9982** | 0.7220 | **0.9681** | 4.81 |

> All six models achieve near-identical predictive performance. The E-XAI framework reveals where they truly differ.

---

## The E-XAI Framework

A custom evaluation suite that measures **seven explainability dimensions** for any tree-based model via TreeSHAP:

| Metric | What it measures |
|---|---|
| **Descriptive Accuracy (DA AUC)** | How critically predictions depend on top-ranked features |
| **Sparsity** | How concentrated feature importance is across the feature space |
| **Stability** | Consistency of top-SHAP features across different random seeds |
| **Efficiency** | How SHAP computation time scales with sample size |
| **Completeness** | Whether top-5 SHAP features are sufficient to drive prediction changes |
| **Robustness** | Resistance to adversarial noise feature injection |
| **Hijack Rate** | Rate at which irrelevant features infiltrate top-5 SHAP rankings |

Each model receives a **6-panel dashboard** visualising all metrics, plus cross-model heatmap and radar chart comparisons.

---

## Dataset

**5G-NIDD DATASET** — 5G Network Traffic (BS1 Attack Scenarios)

- **728,316** total network flow records
- **Benign:** 406,959 samples (55.9%)
- **Malicious:** 321,357 samples (44.1%)
- **46 features** retained after IDS-domain whitelist feature selection
- **80/20 stratified train-test split**
- Feature selection via `RandomForestClassifier` + `SelectFromModel` (median threshold)

---

## Models Evaluated

- `DecisionTreeClassifier`
- `RandomForestClassifier`
- `GradientBoostingClassifier`
- `XGBClassifier` (XGBoost)
- `LGBMClassifier` (LightGBM)
- `CatBoostClassifier`

All models trained with `random_state=42` and 100 estimators (where applicable) for reproducibility.

---

## Project Structure

```
📦 root
 ┣ 📓 TreeSHAP_IDS_EXPERIMENTS.ipynb       # Main notebook — full pipeline end-to-end
 ┣ 📊 exai_dashboard_*.png        # Per-model E-XAI dashboards (6 models)
 ┣ 📊 exai_heatmap_comparison.png # Cross-model metric heatmap
 ┣ 📊 exai_radar_comparison.png   # Cross-model radar chart
 ┗ 📄 README.md
```

---

## Pipeline

```
Raw CSV Files (BS1 Attack Dataset)
        │
        ▼
  Data Loading & Inspection
        │
        ▼
  IDS-Domain Feature Whitelist (46 features)
        │
        ▼
  Encoding + Imputation + Preprocessing
        │
        ▼
  Feature Selection (RF + SelectFromModel)
        │
        ▼
  Train / Test Split (80 / 20, stratified)
        │
        ▼
  Model Training × 6 classifiers
        │
        ▼
  Classification Evaluation
  (Accuracy, Precision, Recall, F1, ROC-AUC)
        │
        ▼
  E-XAI Evaluation (TreeSHAP)
  DA AUC · Sparsity · Stability · Efficiency
  Completeness · Robustness · Hijack Rate
        │
        ▼
  Per-Model Dashboards + Cross-Model Comparison
```

---

## Findings

- **All six models achieve ~97% accuracy and ROC-AUC ≈ 0.998** — predictive performance alone cannot differentiate them.
- **Stability was unanimous** (1.0 for all models) — TreeSHAP produces perfectly consistent rankings across seeds for tree-based models.
- **Robustness was the most polarising metric** — Decision Tree and Random Forest both failed completely under adversarial feature injection (hijack rate: 1.0), while CatBoost was nearly immune (hijack rate: 0.0001).
- **Random Forest is impractical for real-time explainability** — its SHAP computation time of 487s over 200 samples makes it unsuitable for any latency-sensitive IDS deployment.
- **CatBoost is the recommended model for 5G/6G IDS deployment**, offering the best robustness, second-fastest SHAP time among ensembles, and strong classification performance.
- **LightGBM is the best alternative** where explanation completeness is the priority.

---

## Requirements

```bash
pip install numpy pandas scikit-learn matplotlib seaborn shap xgboost lightgbm catboost
```

| Library | Version tested |
|---|---|
| Python | 3.12 |
| scikit-learn | ≥ 1.3 |
| shap | ≥ 0.43 |
| xgboost | ≥ 2.0 |
| lightgbm | ≥ 4.0 |
| catboost | ≥ 1.2 |

---

## Running the Notebook

1. Download the **5G-NIDD DATASET** from [Kaggle]((https://www.kaggle.com/datasets/humera11/5g-nidd-dataset)) and update the data path in Cell 4:
   ```python
   path = "/your/path/to/BS1_each_attack_csv/*.csv"
   ```
2. Run all cells top to bottom. The E-XAI evaluation loop (Cell 36) will generate all dashboards automatically.
3. Dashboard images are saved to the working directory as `exai_dashboard_<ModelName>.png`.

> ⚠️ **Note:** Random Forest SHAP computation takes ~8 minutes over the full dataset. A 200-row sample is used by default for speed. Adjust `n=200` in the sampling cell to increase coverage.

---

## Citation

If you use this framework or findings in your work, please cite:

```bibtex
@misc{exai_5g_ids,
  title   = {E-XAI: Evaluating Explainability of Tree-Based Classifiers for 5G Network Intrusion Detection},
  author  = {Favour Akpan},
  year    = {2026},
  note    = {GitHub repository},
  url     = {[https://github.com/FavourAKPAN/XAI-TREE-BASED-MODELS-IDS-EVALUATION]}
}
```

---

## License

This project is released under the MIT License.
