# Scenario 11: Home Inspections Before Renewal

## Company Background

**HearthGuard Insurance** is a regional home insurer in four southeastern US states. It insures about 52,000 homes and
collects about $125 million a year in premiums, about $2,400 a home. Its policies cover fire, wind, water damage, theft
and liability. Each policy runs for one year and then comes up for renewal.

HearthGuard is losing money. Last year it spent $103 on claims and running costs for every $100 of premium it collected.
Big claims, those over $10,000, cost it about $60 million, roughly half of all premiums.

## Business Problem

HearthGuard's loss-control team can inspect a home before its policy renews. The inspector visits and lists repairs and
safety upgrades the owner must make before renewal. HearthGuard's records show that an inspection and the
repairs that follow prevent about 1 big claim in 3 among the homes that would otherwise have had one.

The decision for each policy coming up for renewal: **inspect the home first**, or **renew as usual**. Today
underwriters (the staff who decide which homes to insure, and on what terms) choose homes for inspection by
judgement. HearthGuard wants a model that scores every renewal, so that inspections go to the homes most likely to have
a big claim in the coming year.

About 1,000 policies come up for renewal each week. The four inspectors can complete about **100 inspections a week**.
The 1,010 homes in test.csv are about one week of renewals, so the team could inspect about 100 of them. The same holds
for the 1,000 homes in holdout_features.csv.

There is a cheaper option. In a **virtual inspection**, the owner walks an inspector through the home on a video call.
It costs $90, and there is no practical limit on how many can be done. In a trial it prevented about 1 big claim in 6,
half as many as a visit. Fewer owners take offense at a video call than at a visit (see the Cost Structure).

Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

## Stakeholders

- **Lorraine Castellanos, VP of Underwriting.** Big claims are why HearthGuard is losing money. She would inspect far
  more homes if she could, and she wants virtual inspections for every home the inspectors cannot reach.
- **Pieter Achterberg, Head of Customer Retention.** An inspection tells a good customer that HearthGuard does not
  trust their home, and some of them leave. Good homes are where HearthGuard makes its money. He wants as few
  inspections as possible, and none at all for customers of ten years or more.
- **Ifeoma Nwosu, Loss-Control Manager.** She runs the four inspectors. She wants a ranked weekly list her team can
  finish. She doubts that a video call finds what a visit finds.
- **Kenji Albrecht, Chief Actuary.** He must explain to the state insurance department how homes are chosen for
  inspection. The department allows insurance credit scores (scores built from an owner's credit history that
  insurers use in pricing) but may ask HearthGuard to justify using them, and he is
  uneasy about relying on one.
- **Marisol Teague, Chief Financial Officer.** Before she approves the program, she wants to know what it saves each
  week after paying for the inspections.

## Cost Structure

The figures come from HearthGuard's claims, loss-control and retention records.

- **An inspection** costs $300: the inspector's visit, travel and report.
- **Inspecting a home that would not have had a big claim** also risks losing a good customer. About 1 in 10 such owners
  move to another insurer at renewal, and a lost good customer costs about $2,000 in future profit (1 in 10 x $2,000 =
  $200). Total: $300 + $200 = $500.
- **A big claim** costs HearthGuard $18,000 on average.
- **Catching a home that would have a big claim** is not free. The inspection costs $300, and 2 big claims in 3 still
  happen after the repairs (2/3 x $18,000 = $12,000). So a catch costs $12,300, and a miss costs $18,000 - $12,300 =
  $5,700 more than a catch.

A virtual inspection costs $90 and prevents about 1 big claim in 6. Among owners of homes without a big claim, about
1 in 20 leave at renewal after a video call (rather than 1 in 10 after a visit). These figures are not in the scored
cost; use them if you recommend virtual inspections.

| Prediction | Actual | Outcome | Cost |
|---|---|---|---|
| Renew as usual (0) | No big claim (0) | True Negative | Not counted (normal business) |
| Inspect first (1) | Big claim (1) | True Positive | Not counted (the owner makes the repairs; about 1 big claim in 3 is prevented) |
| Inspect first (1) | No big claim (0) | **False Positive** | **$500** (the $300 inspection and the $200 expected cost of losing a good customer) |
| Renew as usual (0) | Big claim (1) | **False Negative** | **$5,700** (the $18,000 claim, less the $12,300 a catch would still cost) |

**The cost used to score your model:** (false positives x $500) + (false negatives x $5,700). True positives and true
negatives add nothing.

## Target Variable

- `target = 1`: The home had claims totaling more than $10,000 in the policy year after renewal (a big claim).
- `target = 0`: The home had no claim, or claims totaling $10,000 or less.

**Class balance:** About 7% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision.

| Feature | Description |
|---------|-------------|
| `property_age_years` | Age of the home (years, to one decimal, 0 to about 122) |
| `property_value` | Assessed value of the home (dollars, about $50,000 to $1.12 million) |
| `square_footage` | Living area (square feet, about 600 to 7,750) |
| `num_stories` | Number of stories (1, 2 or 3) |
| `roof_age_years` | Age of the current roof (years, to one decimal, 0 to 40) |
| `roof_material` | One of "asphalt shingle", "metal", "tile", "slate", "other" |
| `heating_type` | One of "central gas", "electric", "heat pump", "oil/wood" |
| `has_security_system` | 1 = monitored security system installed, 0 = none |
| `has_fire_alarm` | 1 = fire or smoke alarm system installed, 0 = none |
| `distance_to_fire_station_miles` | Distance to the nearest fire station (miles, 0.2 to 25) |
| `distance_to_coast_miles` | Distance to the nearest coastline (miles, 0.5 to about 390) |
| `flood_zone_risk` | Flood-zone rating from the national flood maps (0 = lowest risk to 4 = highest risk) |
| `wildfire_risk_score` | Wildfire risk score from a hazard-data supplier (0 to 10 scale; 0 to about 7.7 in these homes) |
| `crime_rate_index` | Local crime index (0 to 10 scale; 0 to about 8.4 in these homes) |
| `num_prior_claims` | Claims filed on this home in the past five years (0 to 5) |
| `years_as_customer` | Years the owner has been a HearthGuard customer (to one decimal, 0 to 40) |
| `credit_score` | The owner's insurance credit score (about 540 to 850) |
| `num_occupants` | Number of people living in the home (1 to 8) |
| `has_pool` | 1 = the property has a swimming pool, 0 = no pool |
| `has_trampoline` | 1 = the property has a trampoline, 0 = no trampoline |
| `dog_breed_risk` | Liability category of the household's dog (0 = no dog or a low-risk breed, 3 = a high-risk breed) |
| `tree_coverage_pct` | Share of the lot under tree canopy, from aerial images (percent, about 1.5 to 98) |

## Questions for Your Analysis

1. The inspectors can visit about 100 homes a week. How many a week would your cost-best threshold send them? If it is
   more than 100, should the rest get a virtual inspection, or be renewed as usual? Show the dollars per week, and give
   Marisol Teague the program's weekly saving after its costs.
2. Lorraine Castellanos and Pieter Achterberg want different things. Put each view in dollars per week at two or three
   thresholds, including the one you recommend. Then price Pieter's rule (no inspections for customers of ten years or
   more): how many big claims a week would it let through, and what would it cost?
3. Compare your model with and without `credit_score`. What does leaving it out cost in dollars per week? Some people
   see credit scores as unfair to lower-income owners. What should Kenji Albrecht tell the state insurance department?
4. An inspection can lead to repairs; it cannot move a home. Which of your model's inputs could an owner change after an
   inspection, and which come with the home or its location? What does that mean for what the inspectors look at, and
   for how the model should score a repaired home at its next renewal?
5. Your training rows come from past policy years. Storm seasons, building costs and the mix of homes HearthGuard
   insures all change. Would you expect your model to work as well next year? What should HearthGuard track to find
   out?
6. Ifeoma Nwosu's inspectors believe some warning signs matter only in combination, or only past some level. Test at
   least two such ideas and say which hold up.

---

*22 features. train.csv (3,030 rows) and test.csv (1,010 rows) are raw extracts; holdout_features.csv (1,000 rows) has
been checked and needs no cleaning.*
