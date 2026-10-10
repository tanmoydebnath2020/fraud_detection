# Fraud Shield — Financial Fraud Detection

A machine learning pipeline and interactive Tableau dashboard for scoring financial
transactions by fraud risk, built around a documented Software Requirements Specification
(SRS v1.0) and deliberately scoped down to a static-data analytical project.

> Scored transaction data in → risk score, Low/Medium/High band, and an honest, gate-checked
> evaluation out.

---

## What this project does

Given a transaction (amount, merchant, payment method, device, location, customer history),
the pipeline predicts how likely it is to be fraudulent, using a chronologically-trained and
honestly-evaluated machine learning model — then surfaces the results in an interactive
Tableau dashboard for a fraud analyst to review.

- **5,000 transactions**, Jan 2023 – Feb 2024, 9.6% labeled fraud.
- **Three models compared**: Logistic Regression, Random Forest, XGBoost.
- **Chronological train/validation/test split** — no future data ever leaks into training.
- **Honest evaluation gate**: checked against recall ≥ 80% / precision ≥ 30% / ≥ 100 confirmed
  fraud cases in the test set — reported as **demonstration-only** since the gate isn't met,
  rather than the bar being quietly relaxed.
- **Risk bands** (Low < 0.40, Medium 0.40–0.70, High ≥ 0.70) — the High band alone captures a
  fraud rate ~4x the overall baseline, a useful triage signal even under demonstration-only
  status.
- **26-chart interactive visualization gallery** built in Plotly (5 genuinely frame-animated).
- **Interactive Tableau dashboard** — a Fraud Risk / Transaction Risk operational page and a
  Model Overview page.

---

## Project structure

```
├── fraud_detection_notebook.ipynb     # Full pipeline: EDA, features, models, evaluation, export
├── financial_fraud_detection_dataset.csv   # Raw input data (5,000 transactions)
├── scored_transactions.csv            # Notebook output — feeds the Tableau dashboard
├── model_comparison_validation.csv    # Accuracy/precision/recall/F1/ROC AUC/PR AUC per model
├── threshold_gate_results.csv         # SRS gate check (recall≥80%, precision≥30%) per model
├── final_test_metrics.csv             # Held-out test set metrics for the selected model
├── feature_importance.csv             # Global feature importances, selected model
├── Fraud_Shield_Project_Report.docx / .pdf   # Full write-up (architecture, testing, results)
├── tableau_package/                   # Tableau build guide + background + data exports
│   └── Tableau_Dashboard_Guide.md
└── powerbi_package/                   # (Optional/legacy) Power BI build guide + data exports
    └── PowerBI_Layout_Guide_v2.md
```

---

## Getting started

### 1. Install Python + Jupyter

```bash
python3 -m pip install --upgrade pip
python3 -m pip install notebook
```

### 2. Install project dependencies

```bash
python3 -m pip install pandas scikit-learn xgboost matplotlib seaborn plotly scipy
```

> On Linux systems with externally-managed Python environments, add `--break-system-packages`
> to the command above.

### 3. Run the notebook

Put `fraud_detection_notebook.ipynb` and `financial_fraud_detection_dataset.csv` in the same
folder, then:

```bash
jupyter notebook
```

Open the notebook in the browser tab that launches, and run all cells top to bottom
(**Kernel → Restart Kernel and Run All Cells**). This regenerates every chart and re-exports
all the CSVs listed above.

### 4. Build the dashboard

Open Tableau (Desktop or the free Tableau Public), connect to `scored_transactions.csv`, and
follow `tableau_package/Tableau_Dashboard_Guide.md` for the calculated fields, worksheet
list, and layout coordinates.

---

## Pipeline summary

1. **Load & validate** — 5,000 rows, 14 columns, zero missing values.
2. **EDA** — class balance, amount distributions, category/time fraud-rate breakdowns, plus a
   26-chart Plotly gallery.
3. **Feature engineering** — `hour`, `weekday`, `amount_vs_avg_ratio` added; identifiers and
   the label itself excluded from the feature set (leakage prevention).
4. **Chronological split** — 70% train / 15% validation / 15% test, ordered by transaction date.
5. **Model training** — Logistic Regression, Random Forest, XGBoost, each in a scikit-learn
   pipeline (preprocessing fit only on training data).
6. **Evaluation** — accuracy, precision, recall, F1, ROC AUC, PR AUC, always shown next to a
   dummy "always legitimate" baseline (90.9% accuracy) so raw accuracy is never read alone.
7. **Gate check** — against recall ≥ 80% / precision ≥ 30% / ≥ 100 test fraud cases.
8. **Risk scoring** — every transaction gets a 0–1 score and a Low/Medium/High band.
9. **Export** — `scored_transactions.csv` and supporting metric CSVs, ready for Tableau.

---

## Key results

| Model | Accuracy | Precision | Recall | ROC AUC | PR AUC |
|---|---|---|---|---|---|
| Logistic Regression *(selected)* | 0.779 | 0.253 | 0.735 | 0.863 | **0.310** |
| Random Forest | 0.803 | 0.262 | 0.647 | 0.870 | 0.286 |
| XGBoost | 0.845 | 0.278 | 0.441 | 0.863 | 0.289 |
| *Dummy "always legitimate" baseline* | *0.909* | – | *0.000* | – | – |

- **Selected model**: Logistic Regression — not the highest accuracy, but the best PR AUC and
  recall, which matters more on this imbalanced (9.6% fraud) data.
- **Test set**: 750 transactions, 63 confirmed fraud cases → below the 100-case gate
  threshold, so results are **demonstration-only**, not production-ready.
- **Practical signal despite the gate**: the High-risk band (13% of transactions) contains a
  39% actual fraud rate — about 4x the 9.6% baseline.

---

## Tech stack

Python · pandas · NumPy · scikit-learn · XGBoost · Plotly · Matplotlib · Seaborn ·
Jupyter Notebook · Tableau

---

## Known limitations

- Static dataset only — no live ingestion, streaming, or retry/idempotency behavior.
- No database or API layer — CSV exports are the only persistence.
- No authentication or case-management workflow (alert lifecycle, role-based access).
- `amount_vs_avg_ratio` approximates true rolling-window features using a pre-aggregated
  average, since no raw event stream is available.
- Dashboard refresh is manual/periodic, not live.

See `Fraud_Shield_Project_Report.docx` for the full limitations and future-enhancements list.

---

## License

Internal project artifact — not licensed for redistribution.
