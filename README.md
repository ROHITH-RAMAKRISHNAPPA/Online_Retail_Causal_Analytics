# Causal Purchase Dynamics Engine
### Online Retail II - B2B Wholesale Analytics

> **"Which customers should a B2B wholesaler contact and which should it never contact?"**

This project applies causal inference, probabilistic customer health modelling, and LLM-assisted hypothesis generation to the Online Retail II dataset (UCI/Kaggle). The result is a ranked intervention list, a sleeping dog blacklist, and a fully auditable analytics pipeline.

---

## Key Results

| Metric | Value |
|---|---|
| Recoverable revenue (top-50 campaign) | £85,855 |
| Spend protected (Sleeping Dog suppression) | £4.88M |
| Campaign ROI at £7/contact | 24,439% |
| P(profitable) across all simulations | 100% |
| Model AUC (enriched XGBoost, 5-fold CV) | 0.887 |
| Customers scored | 5,852 |

---

## Project Structure

```
.
├── data/
│   └── online_retail_II.csv          # Raw dataset (NOT committed - see .gitignore)
│
├── notebooks/
│   ├── 00_verify_setup.ipynb         # Dependency check - run this first
│   ├── 01_data_engineering.ipynb     # Load, clean, feature engineering -> 27 CSVs
│   ├── 02_eda_visualisations.ipynb   # 9 publication-quality charts
│   ├── 03_rq1_b2b_identity.ipynb    # RQ1: 3 formal B2B statistical tests
│   ├── 04_rq2_probabilistic_health.ipynb  # RQ2: BG/NBD, Hawkes, PMI network
│   ├── 05_rq3_causal_uplift.ipynb   # RQ3: T-Learner + IPTW causal model
│   └── 06_llm_integration.ipynb     # LLM: HypoGeniC, CAAFE, XAIstories
│
└── outputs/
    ├── data/                         # 27 auditable CSVs (every chart backed by CSV)
    ├── charts/                       # 17 publication-quality PNGs
    └── llm/                          # Prompt log, hypotheses, narrative outputs
```

---

## Research Questions

| RQ | Question | Method |
|---|---|---|
| RQ1 | Is this B2B wholesale data? | 3 statistical tests (Poisson, Mann-Whitney U, Chi-squared) |
| RQ2 | Who is still commercially alive? | BG/NBD + Gamma-Gamma, Hawkes Process, Cox PH, PMI Network |
| RQ3 | Who should we contact — and who should we never contact? | T-Learner + IPTW causal uplift, Qini validation, Monte Carlo ROI |

---

## RQ1 - B2B Proof (3 Tests, All p < 0.001)

| Test | Statistic | p-value | Verdict |
|---|---|---|---|
| Saturday Anomaly (Poisson ratio) | 231:1 ratio, CI [209, 253] | 0.000 | B2B confirmed |
| Guest vs Named AOV (Mann-Whitney U) | £920 vs £476, r = 0.128 | 8.38e-31 | B2B confirmed |
| Business-hours concentration (Chi-squared) | 96.8% in 08:00-18:00, x2 = 54,814 | 0.000 | B2B confirmed |

---

## RQ3 - Four-Quadrant Segmentation

| Segment | Count | CATE | Action |
|---|---|---|---|
| Persuadable | 708 (12.1%) | +0.28 | CONTACT |
| Sure-Thing | 290 (5.0%) | +0.019 | Skip - saves budget |
| Sleeping Dog | 1,380 (23.6%) | -0.017 | DO NOT CONTACT |
| Lost Cause | 3,474 (59.4%) | -0.362 | Deprioritise |

> **CATE**: positive = contacting increases purchase likelihood. Negative = contacting reduces it.

---

## How to Run

### 1. Get the data

Download `online_retail_II.xlsx` from [UCI/Kaggle](https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci) and convert to CSV. Place at:

```
data/online_retail_II.csv
```

### 2. Install dependencies

```bash
pip install pandas numpy scipy scikit-learn xgboost lifetimes lifelines \
            plotly matplotlib seaborn networkx joblib
```

For LLM integration (NB06), install Ollama and pull the model:

```bash
# Install Ollama: https://ollama.ai
ollama pull llama3.1
```

### 3. Verify setup

```bash
jupyter notebook notebooks/00_verify_setup.ipynb
```

Expected output: all 12 dependencies shown as `ok`.

### 4. Run notebooks in order

```
00_verify_setup.ipynb   -> confirms environment
01_data_engineering.ipynb  -> creates outputs/data/ (27 CSVs)
02_eda_visualisations.ipynb -> creates outputs/charts/ (9 charts)
03_rq1_b2b_identity.ipynb  -> B2B proof charts and CSVs
04_rq2_probabilistic_health.ipynb -> probabilistic model outputs
05_rq3_causal_uplift.ipynb -> CATE scores, blacklist, ROI
06_llm_integration.ipynb   -> LLM outputs (requires Ollama running)
```

> NB06 requires `ollama serve` running in a separate terminal before execution.

---

## LLM Integration

All LLM use is local — no customer data is transmitted externally.

| Prompt | Framework | Result |
|---|---|---|
| P1 | HypoGeniC (arXiv 2404) | 5 hypotheses generated. H_entropy confirmed (p ~ 0.000). H_UK not confirmed (p = 0.9897) - honest null result. |
| P2 | CAAFE (Hollmann 2024) | 25 products labelled with semantic features. AUC delta = 0.0 - null result honestly reported. |
| P3 | XAIstories (2025) | CEO narrative for Customer 16446 (£168k spend, CATE +0.457). All figures verified against CSV. |

**Human-in-the-loop**: every LLM output verified line-by-line against CSV ground truth before use.

Full prompt log: `outputs/llm/06_prompt_log.csv`

---

## Output Files

### Key CSVs for business use

| File | Purpose |
|---|---|
| `outputs/data/05_intervention_priority.csv` | Top-50 Persuadables ranked by counterfactual CLV |
| `outputs/data/05_sleeping_dog_blacklist.csv` | 1,380 customers - DO NOT CONTACT |
| `outputs/data/05_uplift_cate.csv` | All 5,852 customers with CATE scores and segment |
| `outputs/data/05_monte_carlo_roi.csv` | ROI at each contact cost £2-£15 |
| `outputs/data/05_qini_curve.csv` | Qini validation curve (AUUC = 0.1511) |

### Key numbers traceable to CSVs

Every number in the presentation is backed by a named CSV. See source attributions in each notebook cell.

---

## Dependencies

| Package | Version | Purpose |
|---|---|---|
| pandas | 2.3.3 | Data manipulation |
| numpy | 2.2.6 | Numerical computing |
| scipy | 1.17.1 | Statistical tests |
| scikit-learn | 1.6.1 | ML utilities |
| xgboost | 3.2.0 | T-Learner base model |
| lifetimes | 0.11.3 | BG/NBD + Gamma-Gamma |
| lifelines | 0.30.1 | Cox PH survival analysis |
| plotly | 6.5.2 | Interactive charts |
| matplotlib | 3.10.7 | Static charts |
| seaborn | 0.13.2 | Statistical plots |
| networkx | 3.4.2 | PMI co-purchase network |
| joblib | 1.5.3 | Parallelized Hawkes MLE |

Python version: 3.13.13

---

## Reproducibility

- All outputs are deterministic (`random_state=42` throughout)
- Every chart is backed by a named CSV in `outputs/data/`
- Full audit trail: 7 notebooks, 27 CSVs, 17 charts
- LLM prompt log: `outputs/llm/06_prompt_log.csv`
- Model: `llama3.1:latest` via local Ollama

---

## Dataset

**Online Retail II** (UCI Machine Learning Repository)  
Chen, D. (2015). Online Retail II. UCI Machine Learning Repository.  
1,067,371 transactions. Dec 2009 - Dec 2011. UK-based B2B wholesale retailer.

The raw dataset is not committed to this repository due to file size (~150MB).  
Download from: https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci

---

## References

- Fader & Hardie (2005) - BG/NBD model. *Marketing Science*
- Guidotti et al. (2015) - Behavioral entropy. *IEEE DSAA*
- Newman (2006) - PMI network communities. *PNAS*
- Lee & See (2004) - Algorithmic trust calibration. *Human Factors*
- Hollmann et al. (2024) - CAAFE framework. *NeurIPS*
- Zhao et al. (2024) - HypoGeniC. *arXiv 2404.04326*
- XAIstories (2025) - SHAP narrative translation. *ScienceDirect*
