# Scenario 02: Car-Share Damage Claim Inspections

## Company Background

**Fernway Carshare** is a peer-to-peer car-sharing platform. Private owners and businesses, called **hosts**, list their cars in the Fernway app. Drivers, called **guests**, book a car by the hour or the day, collect it from the host and bring it back at the end of the trip. Fernway keeps about a quarter of each trip's price, and the host gets the rest.

Fernway began in 2000 as a neighborhood car club in which members lent each other their own cars. Some of its first members still host with it. Today it works in four metro areas, with about 24,000 hosts, 39,000 listed cars and 2.1 million trips a year, and it employs about 300 people.

Every host is covered by Fernway's **host-protection program**, which is funded by a protection fee guests pay on each trip. If a car comes back damaged, or something is stolen during a trip, the host files a damage claim in the app, with photos and a repair estimate. The program pays the claim, less the host's deductible, so the host does not have to chase the guest for the money. Hosts file about 12,000 claims a year, and the program paid out about $55 million last year.

## Business Problem

Most claims are genuine. Some are not. A host may claim for damage that was there before the trip, pad a repair estimate, blame a guest for damage done during the host's own use, or report a theft that did not happen.

Last year, to measure the problem, Fernway paid an independent appraisal firm to inspect a random sample of about 5,000 claims before they were paid, whatever they looked like. About 8 claims in 100 turned out to be inflated or fraudulent. On average, each one asked for about $2,500 more than the real damage justified. Across 12,000 claims a year, that is about $2.4 million.

Fernway now wants to inspect claims as a matter of routine. Every claim gets a desk review by the claims team within two days of being filed. At the end of that review the team makes one decision: **pay the claim**, or **hold it for an in-person inspection** by a field adjuster before any money is paid. An adjuster is a claims specialist who meets the host, examines the car, compares it with the photos from the trip and checks the repair estimate with the shop. The model will score each claim at the end of the desk review, so that the team knows which claims to hold. Today the team holds claims on instinct, and nobody has measured how well that works.

Hosts file about **1,000 claims a month**. Fernway has four field adjusters, one in each metro area. Each can inspect about 30 claims a month: an inspection takes about four hours with travel, and the adjusters also handle the program's largest losses and disputes with guests. Together they can inspect **about 120 claims a month**. The 1,009 test rows are about as many claims as Fernway receives in a month, so the adjusters could inspect about 120 of them.

The appraisal firm that ran the study has offered to inspect up to 60 more claims a month, at $300 each.

Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

The rows in train.csv and test.csv come from the earlier months of the study; the holdout rows come from its last months.

## Stakeholders

- **Teresa Ostrowski, Director of Host Protection:** owns the program's budget. Payouts grew twice as fast as trips last year. She wants all 120 inspection slots used every month on the claims most likely to be inflated, and she wants the appraisal firm hired for the claims that do not fit.
- **Corinne Dufresne, Chief Financial Officer:** welcomes every dollar the program stops overpaying, but she has signed vendor contracts that never paid back. Before she accepts the appraisal firm's offer, she wants evidence that the extra claims it would inspect are inflated often enough to cover $300 a visit.
- **Nikhil Barua, Head of Host Community:** hosts are Fernway's supply; without their cars there are no trips. A held claim means the host waits about a week longer for the money and has a stranger examine a claim the host believes is honest. He estimates that about 1 genuine host in 20 whose claim is held stops hosting within a year, and an average host brings Fernway about $2,400 a year in commission. By his count, a wrong inspection costs about $120 more than the cost table says. He has no data to back the estimate. He wants claims held only when the model is fairly sure.
- **Kofi Mensah-Boateng, Field Inspection Manager:** runs the four adjusters. He wants a ranked list each week that his team can work through, not more claims than they can visit. He also warns that hosts trade tips in online forums: once they learn which claims get held, the hosts who pad claims will change how they file.
- **Joaquín Ferreira, Trust and Safety Counsel:** Fernway's terms let a host ask why a claim was held, and he must give an answer the host would accept as fair. He will not defend a hold based on a host's age. He is also uneasy about the trust-and-safety score in the data: hosts cannot see it or challenge it.

## Cost Structure

Where the figures come from:

- **An inspection costs $200:** about four hours of an adjuster's time, travel included ($160), plus mileage ($40).
- **A genuine claim that is held costs $50 more.** Fernway promises hosts payment within five days, and a held claim is paid about a week later. So a host whose held claim turns out to be genuine gets a $50 credit on their next fees. A false positive costs $200 + $50 = $250.
- **An inflated claim paid without an inspection overpays by about $2,500** on average, as the study found.
- **A catch costs the $200 inspection.** The adjuster finds the inflated part, and the program pays only for the real damage. So a miss costs $2,500 - $200 = $2,300 more than a catch.

The $250 does not include any host who quits after a wrong inspection. Nikhil Barua thinks it should.

| Prediction | Actual | Outcome | Cost |
|---|---|---|---|
| Pay the claim (0) | Genuine claim (0) | True Negative | Not counted (normal business) |
| Hold for an inspection (1) | Inflated claim (1) | True Positive | Not counted (the adjuster finds the inflated part, and the program pays only for the real damage) |
| Hold for an inspection (1) | Genuine claim (0) | **False Positive** | **$250** (the $200 inspection plus the $50 credit for the late payment) |
| Pay the claim (0) | Inflated claim (1) | **False Negative** | **$2,300** (about $2,500 overpaid, less the $200 inspection a catch would have cost) |

**The cost used to score your model:** (false positives x $250) + (false negatives x $2,300). True positives and true negatives add nothing.

## Target Variable

- `target = 1`: the study's inspection found the claim inflated or fraudulent: damage that was there before the trip, a repair estimate padded beyond the real damage, damage from the host's own use, or a loss that did not happen.
- `target = 0`: the inspection found the claim genuine: the damage or loss happened during the trip, and the amount claimed was fair.

**Class balance:** About 8% of the rows in train.csv have `target = 1`.

## Features

Each row is one claim, as Fernway knew it at the end of the desk review.

All features are known at the time of the decision.

| Feature | Description |
|---|---|
| `claimed_repair_amount` | Amount the host claimed, in dollars and cents: the repair estimate, or for a theft the cost of replacing what was stolen. From $200 to about $80,400 |
| `host_tenure_months` | Months since the host's Fernway account was opened (whole months, 1 to 300) |
| `host_age` | Age of the host in years, or for a business host the age of the person named on the account (18 to 80) |
| `prior_claims` | Earlier claims by this host, in the past five years, that were paid: at Fernway, or at other car-sharing and rental companies through the claims-history service they share (whole number, 0 to 7) |
| `prior_claims_denied` | Earlier claims by this host, in the past five years and from the same sources, that were denied. Counted separately from `prior_claims` (whole number, 0 to 3) |
| `days_to_report` | Whole days from the end of the trip to the filing of the claim (0 = the same day; 0 to 27) |
| `return_photos_uploaded` | 1 = photos of the car taken at the end of the trip (by the guest at drop-off, or by the host at hand-back) are in the app; 0 = no return photos |
| `guest_statements` | Written statements about the trip received from the guest and anyone else on it (additional drivers, passengers) by the end of the desk review (whole number, 0 to 6) |
| `car_age_years` | Age of the car in whole years (0 = under a year old; up to 25) |
| `claim_type` | What the claim is for (text): `collision` (damage to the body from a crash or a scrape), `interior damage` (seats, carpets, stains, smoke, pets), `mechanical` (engine, transmission, tires or other parts the host says the guest's driving damaged), `theft` (parts of the car or items in it stolen during the trip, or the car itself, later recovered) |
| `damage_severity` | Severity the host chose on the claim form: 0 = cosmetic, 1 = minor, 2 = moderate, 3 = severe |
| `trip_end_hour` | Hour of the day the trip ended (0 to 23; 0 = midnight to 12:59 am) |
| `is_weekend` | 1 = the trip ended on a Saturday or Sunday; 0 = a weekday |
| `is_holiday_adjacent` | 1 = the trip ended on a public holiday or the day before or after one; 0 = not |
| `protection_deductible` | The host's deductible: the part of each paid claim the host bears, in dollars (250, 500, 1,000 or 2,000). Hosts choose it separately from their plan |
| `protection_plan` | The host's protection plan (text): `basic`, `standard`, `premium`. A higher plan costs the host a larger share of each trip's price, in return for extras such as towing and a rental car while the host's car is repaired |
| `host_type` | Kind of host (text): `individual` (a private owner listing their own car), `small fleet` (a person or small business listing a few cars), `dealer` (a car dealership listing cars from its stock), `rental partner` (a local rental company listing part of its fleet) |
| `host_account_risk_score` | Score from Fernway's trust-and-safety system for the host's account on the day the claim was filed, from 0 to 1. It rises with unusual account activity, such as a change of payout bank account, sign-ins from new devices or payment disputes. Hosts cannot see it |
| `repair_shop_is_partner` | 1 = the estimate comes from one of Fernway's partner repair shops (shops that have agreed prices with Fernway); 0 = a shop the host chose |

## Questions for Your Analysis

1. Compare the number of test claims your cost-best threshold would hold with the 120 the adjusters can inspect. If the two differ, which claims should fill the 120 slots, and what does the limit cost Fernway in dollars a month? Would the appraisal firm's 60 extra inspections at $300 each pay for themselves? Show how you know.
2. Nikhil Barua puts the cost of a wrong inspection at $250 plus about $120 in hosts who leave. Redo your threshold choice with his figure. How far does the threshold move, how many fewer claims are held, and how much more do inflated claims cost Teresa Ostrowski's program? His figure is a guess: what would you tell the two of them?
3. Joaquín Ferreira must be able to tell a host why a claim was held. Should `host_age`, `host_type` and `host_account_risk_score` be in the model? Decide for each, and show what leaving it out costs.
4. Kofi Mensah-Boateng's adjusters believe some warning signs matter only in combination, or only past some level. Test at least two such ideas and say which hold up.
5. Kofi Mensah-Boateng expects hosts to learn which claims get held. Which columns could a host change on purpose once the model is live, and which could they not? What would that do to your model's results, and what should Fernway do about it?
6. The study inspected claims at random. Once the model is live, the adjusters will inspect only the claims it ranks highest. How would Fernway know next year whether the model still works, and what should it keep doing so that it can check and retrain it?

---

*19 features. train.csv (3,029 rows) and test.csv (1,009 rows) are raw extracts; holdout_features.csv (1,002 rows) has been checked and needs no cleaning.*
