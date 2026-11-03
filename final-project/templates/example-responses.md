# Example Responses: A-Level and C-Level

This document answers the same prompts at different levels of quality, using a hospital-readmission case that is not offered as a project. Use it to calibrate your team's writing.

**The pattern:** C-level answers *describe what was done*. A-level answers *explain why it was chosen, what else was considered, what was traded off, and what the team would do differently.*

---

## Example 1: Part 1.2, Cost Analysis

**Prompt:** *"Analyze the asymmetry between false positives and false negatives. How should it influence your choice of evaluation metric?"*

### C-Level Response
> The FN cost is $25,000 and the FP cost is $500. FN is more expensive, so we will use F1 score since it balances precision and recall.

*(Two sentences, no reasoning, and the wrong conclusion: F1 weighs precision and recall equally, which contradicts the asymmetry the answer has just identified.)*

### B-Level Response
> The FN:FP ratio is 50:1: one missed readmission costs as much as 50 unneeded bundles. So we should not use accuracy (it counts both mistakes the same) or F1 (it weighs precision and recall equally). We will use average precision, because it measures how well the model ranks patients across every threshold.

*(Identifies the problem and picks a sound metric, but does not say why average precision beats the other candidates, such as recall or ROC AUC, or how the costs will be used once a model is chosen.)*

### A-Level Response
> The FN:FP ratio is 50:1: one missed readmission costs the hospital what 50 unneeded bundles cost. That rules out accuracy (82% of patients do not return, so "nobody returns" scores 82% and helps no one) and F1 (it weighs precision and recall equally, and our two mistakes are fifty times apart). Recall alone fails too: flagging everyone maximises it, and the care team can give only 300 bundles a month, about 8% of discharges.
>
> That limit decides the metric. MercyFirst will act on the top 8% of a ranked list, so we need the model whose highest-scored patients really are the ones who return. ROC AUC also measures ranking, but it gives as much credit to ordering the low-risk majority as to the top of the list; the precision-recall curve shows the top of the list, and **average precision** summarises it. So we compare and tune models by cross-validated average precision.
>
> The costs come back in at the second step: we choose the threshold on out-of-fold chances to minimise the error cost (`FP × $500 + FN × $25,000`), within the care team's 8%. Keeping the two steps apart lets us choose the model by how well it ranks, and the threshold by what MercyFirst can afford and deliver.

*(Rules out each candidate with a reason tied to this scenario, lets the business constraint decide the metric, and separates ranking from thresholding, with a reason for doing so.)*

---

## Example 2: Part 4.2, Final Model Selection

**Prompt:** *"Defend your final model choice in depth. Why this model over the runner-up?"*

### C-Level Response
> Logistic regression had the highest average precision (0.302), so we chose it. At threshold 0.35 the confusion matrix has 32 true positives, 53 false positives and 152 false negatives.

*(Reports numbers without interpreting them. It never names the runner-up, does not say how the threshold was chosen, and offers no trade-off.)*

### B-Level Response
> Our top three models (logistic regression, a random forest and XGBoost) were all within 0.015 of one another in cross-validated average precision. We chose logistic regression because it scored best and is easy to explain. The forest was very close, and another team might reasonably pick it.

*(Sees the near-tie and names the runner-up, but does not measure the gap against the spread, does not check the models on the decision the hospital will actually make, and does not say who would choose differently, or why.)*

### A-Level Response
> A strong defense:
> - compares the lead (0.004 over the forest) with the spread across folds (0.014 to 0.024) and across random starts (0.295 to 0.300), and so says "scored best", not "proven better";
> - checks the finalists on the decision itself: readmissions among each model's top 8% of patients (83, 83 and 80 of 553), where the two best are level;
> - states the runner-up's case honestly: the logistic regression leans on columns the team built by hand, and a forest can find such cut-offs itself;
> - names who would choose differently and why (the analytics team might take the forest; the hospitalists would want the model whose reasons they can read);
> - recommends one model, with the alternative and a pilot to settle the difference.

*(This is what "defend in depth" means: find the real decision, the real trade-offs, and the reasonable disagreements, and put numbers on them.)*

---

## Example 3: Part 5.1, Algorithm Suitability

**Prompt:** *"For each model type you tried, explain in 2-3 sentences why it was or wasn't well-suited to this problem."*

### C-Level Response
> **Logistic regression:** It's a linear model. The data was non-linear so it didn't work well.
> **Random forest:** It's an ensemble of decision trees. It worked okay.
> **XGBoost:** It's the best model for tabular data. It worked best.

*(Says nothing. "Non-linear data" is vague, "worked okay" is not an explanation, and "best for tabular data" is a slogan, not reasoning. Two of the three claims are also contradicted by the team's own results.)*

### A-Level Response
> **Logistic regression:** It draws a straight boundary, so on the original columns it cannot learn that both short and long stays raise the risk (the U shape), or that age matters most together with many diagnoses. Once we gave it those as columns of their own, it went from 0.273 to 0.302, the best score we had: what was left of the pattern was broad and simple, which suits a straight boundary.
>
> **Random forest:** Averaging hundreds of trees, each on a bootstrap sample and a random choice of columns, smooths out a single tree's steps and lets many small effects add up. With unlimited depth it scored 0.285; limited to depth 4 it drew level with logistic regression (0.298), which suggests the deep trees were following individual patients rather than the broad pattern.
>
> **XGBoost:** It builds small trees one after another, each fixing what the earlier ones left. At its default depth of 6 it scored only 0.252; its best settings used fewer, shallower trees (50 trees of depth 3) and reached 0.287, still below logistic regression. Boosting's strength is building up combinations of columns; in data this noisy, with the main combinations already given to logistic regression as columns, there was little more for it to find.
>
> **The vote:** Logistic regression, a forest and a tree did not beat their best member, because the members were not equally good: the weaker two pulled the vote down. It also gives a yes or no, not a chance, so it could not produce the ranked list the hospital needs.

*(Each model gets an explanation of how it works, connected to this data and backed by the team's own numbers. That is what holds up when someone asks "why?".)*

---

## Example 4: Part 5.3, "What if FN cost were 10 times higher?"

### C-Level Response
> If FN cost were higher we would lower the threshold to catch more positives.

*(True but trivial. It does not say by how much, and misses what the change would mean for the business.)*

### A-Level Response
> The ratio would go from 50:1 to 500:1. A bundle would then pay for itself whenever more than about 1 in 501 of the patients who get one would otherwise return. The lowest chance our model gives any patient is 0.024, so the cost sweep, which already says "flag everyone" at 50:1, would say it even more firmly.
>
> Our threshold would not move, though, because it is set by the care team's 300 bundles a month, not by the costs. What would change is the recommendation around it. Each patient the care team cannot reach would cost ten times as much, so the case for a larger care team, or for a cheaper, lighter bundle for the patients just below the cut-off, would become the headline. At 500:1 the most valuable thing we could do is not a better model; it is an organisational change.

*(Works through the first effect, sees why the threshold does not move, and follows the change to the decision that matters. The last sentence reframes the problem, which is what senior-level analysis looks like.)*

---

## Writing Guidelines Summary

| If your answer is... | ...it is probably: |
|---|---|
| 1–3 sentences describing what you did | C-level or below |
| A paragraph describing what you did and naming one trade-off | B-level |
| Paragraphs that weigh alternatives, put numbers on trade-offs, and say what a reasonable team might choose differently | A-level |

### The "Why?" Test

When writing Part 5, imagine someone stopping **any one of you** mid-sentence and asking *"why?"*. If that member can answer with specifics (names, numbers, the alternatives considered), the answer is A-level. If they would have to stall or generalise, it is not. Every member presents part of the team video, and every member should be able to explain every answer.

### Common Phrases That Signal AI-Generated Text

These phrases appear in AI chat tools' answers far more often than in student writing. Using one does not mean you used AI, but in context markers find them a reliable warning sign:

- "It is important to note that..."
- "In this section, we will..."
- "By leveraging..."
- "This highlights the importance of..."
- "In conclusion..."
- "Notably..."
- "Crucially..."
- "A holistic approach..."
- Repeated three-item parallel lists ("rigor, clarity, and precision")

**Write like yourselves.** Part 5 is marked on your team's own voice, not a polished one. A slightly clumsy sentence that shows real engagement will score higher than a fluent one that shows rehearsed patterns.
