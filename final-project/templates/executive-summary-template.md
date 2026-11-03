# Executive Summary Template

**Purpose:** a one-page summary for executives with no machine-learning background.
**Format:** one page, converted to PDF, submitted as `TeamNN_ExecSummary.pdf` (your team's two-digit number).
**Hard limit:** one page. No jargon, no technical terms, no code.

---

## Template Structure

```
[Scenario Title]: Model Deployment Recommendation
Team NN: [members' names], [Date]

BOTTOM LINE
[2-3 sentences. What should they do, expressed as a concrete action?]

BUSINESS IMPACT
[One table or two paragraphs, quantified in dollars.]
- Expected annual impact: $X
- Estimated payback period: X months
- Comparison to the current approach: $Y better or worse per year

THE APPROACH IN PLAIN ENGLISH
[3-4 sentences. What does the model do? What information does it use?
Do NOT say "random forest" or "average precision": say "a tool that learned
from past outcomes" or "3 in 8 of the patients it flags return".]

KEY RISKS
[Bullet list of 2-3 things that could go wrong and what would happen.
Focus on business and operational risks, not technical ones.]
- Risk 1: [e.g., the patient mix changes, so the scores become less accurate]
- Risk 2: [e.g., staff do not trust the scores, so the flags are ignored]
- Risk 3: [e.g., the flags fall unevenly on one group of customers]

RECOMMENDATION
[One of: Deploy | Pilot | Wait. Explain in 2-3 sentences why.]
```

---

## Example: Hospital Readmission Risk (not offered as a project)

> *A complete example, for a hospital-readmission case. Your own summary
> should match it in tone and length: readable by a hospital board chair in under two minutes.*

---

### Hospital Readmission Risk: Model Deployment Recommendation

**Team NN: [members' names], November 2026**

**BOTTOM LINE**
We recommend a **90-day pilot** of a discharge risk list at one hospital. The list gives the care team's 300 bundles a month to the patients most likely to return within 30 days. For the same budget, it reaches about **twice as many** of the patients who will return as the bundles reach today: about 113 a month instead of 55.

**BUSINESS IMPACT**

| | Without the list | With the list | Change |
|---|---|---|---|
| Care bundles a year | 3,600 | 3,600 | none: no new spending |
| Bundles reaching a patient who would have returned, a year | about 663 | about 1,355 | about 692 more |
| Readmission costs avoided a year, if each bundle that reaches such a patient prevents the return | | | up to **$17.7M** |

The saving depends on how many returns the bundle actually prevents, which the pilot measures. The list pays for itself if the bundle prevents as few as **1 in 30** of the returns it reaches. Payback: immediate, since no new bundles are bought.

**THE APPROACH IN PLAIN ENGLISH**
The tool reads each patient's discharge record (age, number of diagnoses, length of stay, admissions and emergency visits in the past year, lab results) and gives a risk score. Each day the care team takes the highest-scoring patients, up to 300 a month. Of the patients at the top of the list, about **3 in 8** return within 30 days, against fewer than 1 in 5 patients overall. It was built from 3,000 past discharges and checked on 1,000 more it had never seen. Doctors keep the final say.

**KEY RISKS**

- **Most returns are still missed.** 300 bundles a month can reach only about 1 in 6 of the patients who will return. The tool picks better; it cannot pick more. Our numbers say a bundle is worth giving to far more patients than the care team can reach today: the board should weigh a larger team.
- **Trust.** If doctors ignore the scores, nothing is saved. Each score comes with its reasons (a one-day stay, many diagnoses, past admissions), and the pilot measures how often the scores are used.
- **Fairness.** The record includes insurance type, which partly reflects income. We will check, before rollout and every month, that the list does not favour or pass over patients by insurance.

**RECOMMENDATION**
**Pilot for 90 days** at one hospital. Measure (1) how often flagged patients return compared with similar patients who were not flagged, which tells us how many returns the bundle prevents; (2) how often doctors and the care team use the scores; (3) who is flagged, by insurance type. If the pilot confirms that the bundles reach about twice as many returning patients, extend it to all four hospitals.

---

## What Makes a Good Executive Summary

### Do

- **Lead with the action**: the reader should know what you recommend before the second paragraph.
- **Quantify every claim**: "a significant reduction" means nothing; "about 692 more returning patients reached a year" can be acted on.
- **Say what the numbers assume**: a saving that depends on the bundle working says so, and says what the pilot will measure.
- **Acknowledge risks honestly**: executives trust recommendations that engage with what could go wrong.
- **Match the vocabulary of the scenario**: a hospital scenario says "readmissions", "discharge", "care team"; a banking scenario says "fraud losses", "investigators", "customers".
- **Recommend a pilot over full deployment** when uncertainty is high: a pilot that disappoints is cheap; a full deployment that disappoints is not.

### Don't

- Name algorithms (logistic regression, random forest, XGBoost).
- Use machine-learning metrics without translating them (AUC, F1, precision, recall).
- Report numbers the reader cannot interpret: "average precision 0.277" means nothing to a board; "3 in 8 of the patients it flags return" does.
- Be so cautious that no recommendation emerges: you were hired for a decision.
- Go over one page. If it does not fit, it is not clear enough yet.
