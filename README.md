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
├── week04/   Linear Regression — assignment/: Pelican Row Growers tray trial;
│             practice/: Ridgeline Facilities building energy
├── week05/   Logistic Regression — churn, loan default, employee attrition
├── week06/   K-Nearest Neighbors — insurance churn, trial conversions
├── week07/   Support Vector Machines (Week 8) — SecureBank fraud transactions, the practice data
├── week08/   Support Vector Machines — First Republic Financial complaints, the assignment data
└── week09/   Ensemble Methods — insurance claims fraud

final-project/
├── cases/    20 binary-classification scenarios (01–20)
│   └── NN-name/  scenario.md, train.csv, test.csv
└── templates/    submission templates and exemplars
```

Folder numbers are **Fall 2026 week numbers**, with one exception: `week07/` holds the
Week 8 practice data. The Support Vector Machines week moved from Week 7 to Week 8 on
October 6, and a published file path is never renamed. Week 10 has no folder — its
notebooks use scikit-learn built-in datasets or generate their data inline.
(Week 4's lecture demo also generates its data inline, on purpose: seeing the hidden
relationship is the lesson. Its assignment and practice data live here.)

## Datasets added for Fall 2026

New for Fall 2026. Generated deterministically by `scripts/build_class_data.py` in
the course repo, so regenerating them produces byte-identical files.

| File | Rows | What it is |
|---|---:|---|
| `week01/bay_harbor_daily_sales.csv` | 90 | Daily café sales — transactions, ticket size, temperature, staffing. Clean by design; Week 1 is about workflow, not cleaning. |
| `week02/northwind_members.csv` | 1,218 | Gym membership records — plan, tenure, visits, support tickets, churn flag. **Deliberately messy:** 85 missing values, 18 duplicate rows, and inconsistent capitalisation in `region`. Finding those defects is part of the assignment. |
| `week04/assignment/pelican_row_trays.csv` | 500 | Pelican Row Growers growing trial, one row per harvested microgreen tray — tray ID, nutrient concentrate added (mL), grow-room temperature (°C), relative humidity (%), the tray's position along its rack (m), seed sown (g), seed-lot age (days), and harvest weight (g, the target). No missing values. The graded Week 4 assignment asks students to recover how the inputs relate to harvest weight. |
| `week04/practice/ridgeline_tower_energy.csv` | 50 | One office tower's daily electricity use (kWh) and that day's average outdoor temperature (°F). Used in the ungraded Week 4 practice assignment. |
| `week04/practice/ridgeline_portfolio_energy.csv` | 420 | Ridgeline Facilities building portfolio, one row per building — building type, floor area (sq ft), age (years), weekly operating hours, LED retrofit flag, cooling capacity (tons), number of floors, parking spaces, and annual electricity use (kWh, the target). Used in the ungraded Week 4 practice assignment. |
| `week07/fraud_transactions.csv` | 3,000 | SecureBank card transactions, 5% fraud. The Spring Week 7 notebook referenced this file but it was never published, so the notebook silently fell back to generating data inline; it is now published so the URL path works. Used in the ungraded Week 8 practice assignment. |
| `week08/first_republic_complaints.csv` | 4,000 | First Republic Financial customer complaints, one row per complaint — numbers the bank's intake system extracts from each message (length in words, counts of fee, rate, payment and treatment words, amount disputed), what the customer holds (credit card, mortgage, an open application), account age, prior complaints, when the complaint arrived, and the team that resolved it (`Cards`, `Deposits`, `Mortgage` or `Fair Lending`, the target). No missing values. Used in the graded Week 8 assignment. |

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
