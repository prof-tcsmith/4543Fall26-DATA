# Scenario 07: Choosing Which Insured Restaurants Get a Safety Visit

## Company Background

**Stockpot Mutual Insurance** is a regional insurer that covers only sit-down restaurants. It insures about 12,000 of them, from diners, pizzerias and family restaurants to steakhouses and fine dining. A Stockpot policy pays when a customer claims they were made ill by the food, when stock spoils after a fridge fails, and when the restaurant loses income because the city health department has closed its kitchen. Policies last a year, so about 1,000 come up for renewal each month.

Stockpot has a **loss-control** team: 12 consultants who visit kitchens and help owners fix risks before they turn into claims. On a visit, the consultant spends a few hours in the kitchen and storerooms, checks food temperatures, storage, cleaning and pest control, and writes up what they find.

Food-safety claims are a large share of what Stockpot pays out. Most restaurants never make one. A few make very large ones: last year a single illness claim against one insured restaurant cost Stockpot $310,000.

## Business Problem

Until now, consultants have visited restaurants in a fixed rotation, by policy number, about 240 a month, whatever the restaurant's risk. Each visit's result was recorded. Your data are these past rotation visits, so the restaurants in them were not chosen by risk.

Stockpot now wants to **choose** which restaurants get a visit. Each month, it will look at the roughly 1,000 restaurants whose policies renew the following month and send a consultant to some of them before renewal. When a visit finds a serious problem, the owner gets a written plan to fix it, and a consultant comes back to check. The policy renews on normal terms only once the fix is made.

**The 12 consultants can make about 240 visits a month** (20 each; the rest of their time goes to follow-up visits and new customers). That is about one renewing restaurant in four. The 1,010 test rows are past rotation visits, about as many restaurants as renew in a month; because the rotation ignored risk, they stand in for one month's renewals, so the consultants could visit **about 240** of them.

Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

There is one other option. An outside inspection firm will make extra visits at **$700 each**, as many as Stockpot orders.

## Stakeholders

- **Delphine Moreau, VP of Loss Control** — Sponsors the project and runs the consultants. Wants the 240 monthly visits to go where they prevent the most claims, and must present the new plan to Stockpot's board.
- **Tomás Reyes, Senior Loss-Control Consultant** — Has walked into more than 3,000 kitchens. He trusts what he sees at the back door more than a spreadsheet. He wants every chosen visit to come with reasons he can check, and he proposes keeping **40 of the 240 monthly visits** for consultants to choose themselves.
- **Gwen Halloran, Claims Director** — Thinks the $7,500 miss cost below is too low, because a single outbreak can cost hundreds of thousands of dollars, as last year's did. Her estimate is **$12,000** per serious problem left unfound, which makes the false-negative cost $8,850 ($12,000 − $500 − $250 − 20% × $12,000). She wants the outside firm hired for extra visits.
- **Lionel Odell, Head of Agency Sales** — Restaurants buy their policies through independent agents. Agents tell him that owners lose half a day to a visit, and that owners who are visited and found clean feel accused and shop around at renewal. His estimate makes a visit that finds nothing serious cost **$900**, not $500 (the false-negative cost would stay $5,250). He wants as few unnecessary visits as possible.
- **Celeste Ambrose, Compliance Counsel** — State insurance regulators forbid unfair discrimination: treating customers differently for reasons that are not tied to risk, or that stand in for the owner's race or ethnicity. She asks whether `neighborhood_income_level` and `cuisine_risk_category` may be used to choose visits, and she needs every visit decision to be explainable to a regulator.
- **Owen Pratt, Chief Financial Officer** — Wants the answer in dollars: what the model saves compared with the rotation, and whether buying extra visits from the outside firm would pay for itself.

## Cost Structure

**What a visit costs ($500):** half a day of a consultant's time, including salary, benefits and car ($400), plus travel ($100). When the visit finds nothing serious, that money did not need to be spent: this is the false-positive cost.

**What a miss costs:** Stockpot's claims records show that a serious problem left unfound leads, on average, to **$7,500** of extra claims in the next policy year. Most such restaurants have no claim; a few have very large ones.

**A catch is not free.** It costs the visit ($500), a follow-up visit to check the fix ($250), and about one owner in five does not fully fix the problem (20% × $7,500 = $1,500). So the false-negative cost is $7,500 − ($500 + $250 + $1,500) = **$5,250**.

| Prediction | Actual | Outcome | Cost |
|---|---|---|---|
| No visit this month (0) | No serious problem (0) | True Negative | Not counted (normal business) |
| Visit (1) | Serious problem (1) | True Positive | Not counted (the consultant finds the problem; the owner fixes it under a written plan, checked at a follow-up visit) |
| Visit (1) | No serious problem (0) | **False Positive** | **$500** (a consultant's half day and travel, spent where nothing serious was wrong) |
| No visit this month (0) | Serious problem (1) | **False Negative** | **$5,250** (the expected extra claims from a problem left unfound, less what a catch would still have cost) |

**The cost used to score your model:** (false positives x $500) + (false negatives x $5,250). True positives and true negatives add nothing.

## Target Variable

- `target = 1`: the visit found at least one serious food-safety or hygiene problem, for example food kept at unsafe temperatures, signs of pests, raw and ready-to-eat food stored together, or no working hand sink.
- `target = 0`: the visit found only minor problems, or none.

**Class balance:** About 15% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision.

| Feature | Description |
|---------|-------------|
| `days_since_last_check` | Days since the last food-safety check at the premises: a Stockpot consultant's visit or a city health inspection, whichever was more recent (10 to 745) |
| `prior_minor_findings` | Number of minor food-safety findings at the restaurant's past checks (0 to 9) |
| `prior_serious_findings` | Number of serious food-safety findings at the restaurant's past checks (0 to 4) |
| `restaurant_type` | Style of sit-down restaurant: `bistro`, `casual dining`, `diner`, `family dining`, `fine dining`, `pizzeria`, `seafood`, `steakhouse` |
| `years_in_operation` | Years the restaurant has operated at this location (0.5 to 40, one decimal) |
| `num_employees` | Number of employees (2 to 25) |
| `seating_capacity` | Number of indoor seats (10 to 300) |
| `has_liquor_license` | 1 = licensed to serve alcohol; 0 = not |
| `complaint_count_last_6months` | Number of food-safety complaints about the restaurant that members of the public made to the city health department in the past six months (0 to 7) |
| `avg_health_score_history` | Average score at the restaurant's past city health inspections, on the city's 0-to-100 scale (53 to 100, one decimal) |
| `owner_change_last_year` | 1 = the restaurant changed owners in the past 12 months; 0 = it did not |
| `renovation_last_year` | 1 = major renovation in the past 12 months; 0 = none |
| `chain_restaurant` | 1 = part of a chain; 0 = independent |
| `neighborhood_income_level` | Median yearly household income in the restaurant's census tract (the small area the census reports on), in dollars ($15,000 to about $122,000) |
| `visit_month` | Month of the visit, 1 = January to 12 = December (for a new decision, the month the visit would take place) |
| `food_handler_certifications_pct` | Percentage of staff with a current food-handler certificate, as the owner estimated it on the policy application (11 to 100, one decimal) |
| `cuisine_risk_category` | Stockpot's rating of how complex the menu's food handling is, from 0 (simplest) to 4 (most complex) |
| `delivery_service_partner` | 1 = takes orders through outside delivery services; 0 = does not |
| `outdoor_seating` | 1 = has outdoor seating; 0 = does not |
| `kitchen_size_sqft` | Kitchen floor area in square feet (69 to about 2,330) |

## Questions for Your Analysis

1. At your cost-best threshold (chosen on out-of-fold probabilities), how many of the 1,010 test restaurants would your model send a consultant to, and how does that compare with the 240 the team can visit? Recommend what Stockpot should do about any gap, and tell Owen what 100 extra visits a month from the outside firm, at $700 each, would be worth.
2. Gwen and Lionel disagree about the costs in opposite directions. Rerun your threshold choice with Gwen's false-negative cost ($8,850) and with Lionel's false-positive cost ($900). Does your recommendation change? Which recommendation holds up under both?
3. Celeste's question: what does your model lose without `neighborhood_income_level`, without `cuisine_risk_category`, and without both? Would you keep either one? How would you explain to a regulator why one restaurant was visited and its neighbour was not?
4. Once agents and owners learn how visits are chosen, which columns could an owner change on paper, or report differently, without anything changing in the kitchen? What would that do to your model over a year, and what could Stockpot check?
5. Tomás's consultants believe some warning signs matter only in combination, or only past some level. Test at least two such ideas and say which hold up.
6. Your data come from rotation visits that were not chosen by risk. If the model chooses most visits from now on, what will next year's data look like, and could the model be checked and retrained on it? Compare Tomás's proposal with other ways of spending a few of the 240 visits, and say what each would cost.

---

*20 features. train.csv (3,030 rows) and test.csv (1,010 rows) are raw extracts; holdout_features.csv (1,000 rows) has been checked and needs no cleaning.*
