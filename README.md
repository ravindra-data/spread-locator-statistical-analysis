# Spread Locator – Statistical Distribution Analysis

Statistical analysis of 220 payment transactions (January 2023) using probability distributions, Q-Q plots, Box-Cox transformation and Z-scores, written in Python. The project answers one practical question: **which probability distribution best describes the transaction data, and what does that mean for decision-making?**

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Dataset](#dataset)
3. [Project Structure](#project-structure)
4. [Setup and Installation](#setup-and-installation)
5. [How to Run](#how-to-run)
6. [Methodology (Part B Tasks)](#methodology-part-b-tasks)
7. [Key Results](#key-results)
8. [Business Insights](#business-insights)
9. [Assumptions and Limitations](#assumptions-and-limitations)
10. [Theory Guide (Part A)](#theory-guide-part-a)
11. [Troubleshooting](#troubleshooting)

---

## Project Overview

| Part | What it contains | Where |
|---|---|---|
| **Part A – Theory** | Definitions, formulas and examples for 11 statistical concepts, plus glossary and cheat sheet | `Part_A_Theory_Concepts.pdf` |
| **Part B – Analysis** | Python code that fits and tests distributions on the dataset (8 tasks) | `Spread_Locator_Statistical_Analysis_.ipynb` / `Part_B_Simple_Code.py` |

**Tools:** Python, NumPy, Pandas, SciPy, Statsmodels, Matplotlib, Seaborn.

## Dataset

File: `spread_locator_dataset.xlsx` – 220 rows, no missing values, no duplicate rows, dates from 2023-01-01 to 2023-01-31.

| Field | Type | Description |
|---|---|---|
| `transaction_id` | String (UUID) | Unique ID of each transaction |
| `customer_id` | String (UUID) | Unique ID of each customer |
| `transaction_amount` | Float | Transaction amount in ₹ (range ₹804 – ₹20,463) |
| `transaction_date` | Date | Date of the transaction |
| `transaction_count` | Int | Number of transactions by the customer in a given week |
| `region` | String | North, South, East or West |
| `transaction_status` | String | `Success` or `Fail` |

## Project Structure

```
spread-locator/
├── README.md                                   # this file
├── requirements.txt                            # Python packages needed
├── spread_locator_dataset.xlsx                 # input data
├── Spread_Locator_Statistical_Analysis_.ipynb  # Part B – main notebook (with outputs)
├── Part_B_Simple_Code.py                       # Part B – same code as a plain script
└── Part_A_Theory_Concepts.pdf                  # Part A – theory guide with definitions
```

> The code reads `spread_locator_dataset.xlsx` from the **same folder** it is run from, so keep the files together.

## Setup and Installation

Requires **Python 3.9 or newer**.

```bash
# 1. (optional but recommended) create a virtual environment
python -m venv venv
venv\Scripts\activate          # Windows
source venv/bin/activate       # macOS / Linux

# 2. install the packages
pip install -r requirements.txt
```

## How to Run

**Option 1 – Jupyter Notebook (recommended for learning)**
```bash
jupyter notebook Spread_Locator_Statistical_Analysis_.ipynb
```
Then choose *Kernel → Restart & Run All*. Each cell prints its result and shows its chart.

**Option 2 – Python script**
```bash
python Part_B_Simple_Code.py
```
Charts open one after another; close each window to continue.

**Option 3 – Google Colab:** upload the notebook and the `.xlsx` file, and run `%pip install statsmodels` in the first cell if needed.

## Methodology (Part B Tasks)

| # | Task | Method used |
|---|---|---|
| 1 | Bernoulli and Binomial | Success/Fail coded as 1/0 gives p; weekly count compared with Binomial(n = 9, p = mean/n) by comparing mean and variance |
| 2 | Poisson | Transactions counted per day; λ = mean daily count; observed vs Poisson probabilities |
| 3 | Log-Normal and Power law | Normal, Log-Normal and Pareto fitted by maximum likelihood; compared using the Kolmogorov-Smirnov (KS) test |
| 4 | Q-Q plot | Raw and log-transformed amounts plotted against the Normal; Shapiro-Wilk test |
| 5 | Box-Cox | Optimal λ found by maximum likelihood; skewness and normality compared before and after |
| 6 | Z-scores | z = (x − mean)/std; P(amount > ₹5,000) by the Normal method, the Log-Normal method and the real data; outliers where \|z\| > 3 |
| 7 | PDF and CDF | Histogram vs fitted Log-Normal PDF; empirical CDF vs fitted Log-Normal CDF |
| 8 | Conclusion | Best distribution chosen and insights listed |

## Key Results

**Transaction amount – the Log-Normal distribution fits best** (μ ≈ 8.0, σ ≈ 0.47, median ≈ ₹2,983).

| Model | KS distance | KS p-value | Verdict |
|---|---|---|---|
| Normal | 0.140 | 0.0003 | Rejected |
| **Log-Normal** | **0.038** | **0.90** | **Best fit** |
| Power law (Pareto) | 0.322 | ≈ 0 | Rejected |

| Check | Result |
|---|---|
| Skewness of raw amount | 3.73 (strongly right-skewed) |
| Shapiro-Wilk p-value, raw / log amount | ≈ 2×10⁻¹⁸ / 0.106 |
| Box-Cox optimal λ | −0.18 (close to 0, i.e. effectively a log transform) |
| Skewness after Box-Cox | ≈ 0 (Shapiro p = 0.64) |
| P(amount > ₹5,000) – Normal / Log-Normal / actual data | 20.5% / 13.9% / 11.4% |
| Outliers with \|z\| > 3 | 3 transactions (up to ₹20,463) |

**Other variables**

| Variable | Model | Result |
|---|---|---|
| `transaction_status` | Bernoulli | P(Success) = 0.445, P(Fail) = 0.555 |
| Transaction occurrence (≥ 1 in the week) | Bernoulli | p = 0.932 |
| Weekly `transaction_count` | Binomial (n = 9) | Poor fit – data variance 3.23 vs Binomial 1.95 (over-dispersed) |
| Transactions per day | Poisson | λ ≈ 7.1 per day; broadly consistent; P(≥ 10 in a day) ≈ 18% |

## Business Insights

1. **Use the median (≈ ₹3,000), not the mean (≈ ₹3,365),** as the typical transaction size – a few very large payments pull the mean up.
2. **Analyse amounts on the log scale** (or after Box-Cox) before running t-tests, regression or control limits.
3. **Set alert limits with Log-Normal quantiles**, not mean ± 3σ. A Normal assumption overstates the share above ₹5,000 by almost 2×.
4. **Review the 3 outlier transactions** (|z| > 3) manually, for example for fraud checks or key accounts.
5. **Plan capacity with Poisson:** about 7 transactions per day, with roughly 6% of days at 12 or more.
6. **About 55% of transactions fail** – the largest business issue in the data and worth a root-cause study by region and amount band.

## Assumptions and Limitations

- Only one month (Jan 2023) of 220 records – seasonality and trends cannot be assessed.
- The number of trials *n* for the Binomial is not in the dataset; **n = 9 (the maximum observed weekly count) is an assumption**.
- "Transaction occurrence" is interpreted as *at least one transaction in the week* (`transaction_count ≥ 1`).
- A few customers appear in more than one row, so rows are not perfectly independent.
- Only three observations exceed \|z\| = 3, so conclusions about the extreme tail are indicative only.
- The Poisson check on 31 daily counts has low statistical power.

## Theory Guide (Part A)

`Part_A_Theory_Concepts.pdf` (10 pages) explains, with a definition, formula, key properties, example and link to this project:

1. Statistical distributions · 2. Q-Q plot · 3. Discrete vs continuous · 4. Bernoulli · 5. Binomial · 6. Log-Normal · 7. Power law · 8. Box-Cox transform · 9. Poisson (with worked example) · 10. Z-score probability · 11. PDF vs CDF

It ends with a **glossary**, a **formula cheat sheet** and a **"which distribution should I use?"** table.

## Troubleshooting

| Problem | Fix |
|---|---|
| `ModuleNotFoundError: No module named 'statsmodels'` | `pip install statsmodels` (or `python -m pip install statsmodels`; in a notebook use `%pip install statsmodels` and restart the kernel) |
| `FileNotFoundError: spread_locator_dataset.xlsx` | Run the code from the folder that contains the dataset |
| `ImportError: Missing optional dependency 'openpyxl'` | `pip install openpyxl` |
| Charts do not appear when running the script | Close each chart window to see the next one, or use the notebook |
| Slightly different decimals | Different SciPy versions can change the last digits of fitted values |
