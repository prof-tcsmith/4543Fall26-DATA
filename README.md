# ISM4543 Course Data — Fall 2026

Public datasets for **ISM4543 Applied Data Science**, Fall 2026 (University of
South Florida, Dr. Tim Smith).

These files are loaded directly by the lecture, assignment, and practice notebooks
in the course. Notebooks reference them by raw GitHub URL, so they work in Google
Colab with no download step:

```python
import pandas as pd

DATA_URL = "https://raw.githubusercontent.com/prof-tcsmith/4543Fall26-DATA/main/data"
df = pd.read_csv(f"{DATA_URL}/week03/sales_transactions.csv")
```

**All data in this repository is synthetic.** Companies, people, transactions, and
measurements are fictional and were generated for teaching. Any resemblance to
real businesses or persons is coincidental.

## Layout

```text
data/
├── week01/   Course Introduction — Bay Harbor Coffee daily sales
├── week02/   ML Foundations & Python — Northwind Fitness membership data
├── week03/   Data Preparation — Horizon Coffee transactions, customers, orders
├── week05/   Logistic Regression — churn, loan default, employee attrition
├── week06/   K-Nearest Neighbors — insurance churn, trial conversions
├── week07/   Support Vector Machines — SecureBank fraud transactions
└── week09/   Ensemble Methods — insurance claims fraud

final-project/
├── cases/    20 binary-classification scenarios (01–20)
│   └── NN-name/  scenario.md, train.csv, test.csv
└── templates/    submission templates and exemplars
```

Folder numbers are **Fall 2026 week numbers**. Weeks 4, 8 and 10 have no folder
— those notebooks use scikit-learn built-in datasets or generate their data inline.

## Datasets added for Fall 2026

New for Fall 2026. Generated deterministically by `scripts/build_class_data.py` in
the course repo, so regenerating them produces byte-identical files.

| File | Rows | What it is |
|---|---:|---|
| `week01/bay_harbor_daily_sales.csv` | 90 | Daily café sales — transactions, ticket size, temperature, staffing. Clean by design; Week 1 is about workflow, not cleaning. |
| `week02/northwind_members.csv` | 1,218 | Gym membership records — plan, tenure, visits, support tickets, churn flag. **Deliberately messy:** 85 missing values, 18 duplicate rows, and inconsistent capitalisation in `region`. Finding those defects is part of the assignment. |
| `week07/fraud_transactions.csv` | 3,000 | SecureBank card transactions, 5% fraud. The Spring Week 7 notebook referenced this file but it was never published, so the notebook silently fell back to generating data inline; it is now published so the URL path works. |

The Week 2 defects are intentional. Please do not "fix" them.

## Final-project cases

Twenty binary-classification scenarios, each with a business narrative, an explicit
false-positive / false-negative cost structure, and its own train/test split:

```python
BASE = "https://raw.githubusercontent.com/prof-tcsmith/4543Fall26-DATA/main/final-project/cases"
train = pd.read_csv(f"{BASE}/04-telecom-churn/train.csv")
test  = pd.read_csv(f"{BASE}/04-telecom-churn/test.csv")
```

Holdout labels and reference solutions are **not** in this repository — they stay
in the private course repo.

## Stability guarantee

**Files are never moved, renamed, or altered once the semester starts.** Student
notebooks depend on these URLs resolving for the whole term. Corrections, if ever
required, are published as a new file alongside the original.

## Provenance

Migrated from the Spring 2026 ISM6251 data repo (`prof-tcsmith/ism6251s26-data`),
which was cross-listed with the undergraduate ISM4543. Spring weeks 1–10 map 1:1
onto Fall weeks 1–10. Dropped in the migration: Spring's Week 11 clustering data
(the topic is not taught this term) and final-project cases 21–30 (the graduate
track is not offered).

---

© Dr. Tim Smith, 2026. Provided for educational use in ISM4543.
