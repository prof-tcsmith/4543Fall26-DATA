# Scenario 10: Warranty Claim Investigation

## Company Background

**DuraHome Appliances** is a mid-sized maker of refrigerators, washers, dishwashers, ovens and HVAC units (heating and
air-conditioning). It sells about 400,000 appliances a year through big-box retailers, online stores and its own
website. Every appliance carries a manufacturer's warranty of two to three years, depending on the product, and most buyers add DuraHome's extended plan, which runs to five
years.

Customers file warranty claims online: about 26,000 a year, or 500 a week. Most claims are honest. A part fails, and
DuraHome pays for the repair, a replacement or a refund. Some claims are not honest: damage the owner caused, use the
warranty does not cover, or a fault that does not exist.

Finance estimates that about 1 claim in 9 is illegitimate. Paying them costs DuraHome about $2 million a year.

## Business Problem

Every claim is approved unless it is picked for an investigation. In an investigation, a specialist checks the claim
against DuraHome's records, asks for any documents the customer did not send, and may phone the customer or send a
technician. It adds about a week to the claim. Most illegitimate claims that are investigated are denied.

Today a set of fixed rules, written years ago, picks about 40 claims a week. Only about 1 in 4 of them turns out to be
illegitimate. DuraHome wants a model that scores each claim as it arrives, so that investigations go to the claims most
likely to be illegitimate. The decision for each claim: **investigate it**, or **approve it as usual**.

The claims-integrity team (three investigators) can complete about **50 investigations a week**, out of about 500 new
claims. The 1,012 claims in test.csv are about two weeks of claims, so the team could investigate about 100 of them.
The same holds for the 1,000 claims in holdout_features.csv.

An outside claims-review firm has offered to take up to 40 more claims a week, at $130 per investigation (an in-house
investigation costs $60). Its investigators deny illegitimate claims about as often as DuraHome's own.

Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

## Stakeholders

- **Adaeze Okafor, VP of Customer Experience.** An honest customer who is investigated waits a week longer for a
  repair, and many of them say so in public reviews. She wants as few honest customers investigated as possible.
- **Graham Lockwood, Chief Financial Officer.** He sees $2 million a year paid on claims that should have been denied.
  He wants more of them caught, and he will pay the outside firm if the numbers work.
- **Meera Kapoor, Head of Claims Integrity.** She runs the three investigators. If the weekly list is longer than her
  team can finish, they pick from it by instinct, and the model's ranking is wasted. She wants a list her team can
  finish, in order.
- **Daniel Osei, Legal Counsel.** A wrongful denial can turn into a consumer-protection complaint. He wants DuraHome to
  be able to explain why any claim was investigated. He worries that a model could single out some kinds of honest
  customers more often than others.
- **Hye-jin Park, Director of Warranty Operations.** She wants the score to appear the moment a claim arrives, with no
  extra work for the claims processors. She doubts a model trained on past claims will keep working once people learn
  what gets a claim investigated.

## Cost Structure

The figures come from DuraHome's finance and claims teams.

- **Investigating an honest claim** costs $60 of investigator time, plus about $40 for the customer's delay (goodwill
  credits from Customer Experience and some lost repeat sales). Total: $100.
- **Paying an illegitimate claim** costs $700 on average: about $330 for the repair, replacement or refund, and about
  $370 for the technician's trip or shipping, and processing.
- **Catching an illegitimate claim** is not free. The investigation costs $60, and about 15% of investigated
  illegitimate claims are paid anyway because DuraHome cannot prove enough to deny them (15% x $700 = $105). So a
  catch costs $165, and a miss costs $700 - $165 = $535 more than a catch.

| Prediction | Actual | Outcome | Cost |
|---|---|---|---|
| Approve as usual (0) | Legitimate (0) | True Negative | Not counted (normal business) |
| Investigate (1) | Illegitimate (1) | True Positive | Not counted (the team investigates; most of these claims are denied) |
| Investigate (1) | Legitimate (0) | **False Positive** | **$100** (investigator time and an honest customer's delay) |
| Approve as usual (0) | Illegitimate (1) | **False Negative** | **$535** (the $700 claim is paid, less the $165 a catch would still cost) |

**The cost used to score your model:** (false positives x $100) + (false negatives x $535). True positives and true
negatives add nothing.

## Target Variable

- `target = 1`: The claim was illegitimate (damage the owner caused, use the warranty excludes, or a fault that did not
  exist).
- `target = 0`: The claim was legitimate (a genuine fault covered by the warranty or the extended plan).

The labels come from an audit: DuraHome's auditors re-examined a random sample of past claims in depth after they were
settled.

**Class balance:** About 11% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision.

| Feature | Description |
|---------|-------------|
| `product_age_months` | Age of the appliance when the claim was filed (months, with decimals, 0 to 60) |
| `warranty_remaining_months` | Months left on the manufacturer's warranty (0 = the claim rests on the extended plan; months, with decimals, 0 to about 34) |
| `claim_amount` | Amount claimed for the repair, replacement or refund (dollars, about $20 to $2,630) |
| `product_category` | One of "refrigerator", "washer", "dishwasher", "oven", "HVAC", "other" |
| `purchase_channel` | Where the appliance was bought: one of "big-box retailer", "online", "direct" (from DuraHome), "other" |
| `num_previous_claims` | Earlier warranty claims by this customer (0 to 5) |
| `days_since_last_claim` | Days since the customer's previous claim (0 to about 840; 999 = no previous claim) |
| `issue_description_length` | Length of the customer's written description of the fault (characters, 7 to about 1,350) |
| `photo_submitted` | 1 = the customer sent photos of the fault, 0 = no photos |
| `receipt_provided` | 1 = the customer provided proof of purchase, 0 = no receipt |
| `registered_product` | 1 = the appliance was registered with DuraHome at purchase, 0 = not registered |
| `customer_account_age_months` | How long the customer has had a DuraHome account (months, with decimals, 0 to 120) |
| `claim_filed_hour` | Hour of the day the claim was filed (0 to 23) |
| `claim_filed_day_of_week` | Day of the week the claim was filed (0 = Monday, 6 = Sunday) |
| `product_price_original` | Original retail price of the appliance (dollars, $50 to about $4,400) |
| `repair_type_requested` | What the customer asks for: 0 = in-home repair, 1 = ship to a service center, 2 = replacement, 3 = refund |
| `distance_to_service_center` | Distance from the customer to the nearest authorized service center (miles, 0 to about 210) |
| `serial_number_verified` | 1 = the serial number matched DuraHome's records, 0 = it could not be matched |
| `customer_satisfaction_history` | The customer's average rating in earlier contacts with DuraHome (0 to 10 scale) |

## Questions for Your Analysis

1. The team can investigate about 50 claims a week. How many a week would your cost-best threshold send them? If it is
   more than 50, what should DuraHome do with the rest: approve them, or pay the outside firm? Show the dollars per
   week.
2. Adaeze Okafor and Graham Lockwood want different things. Put each view in dollars per week (honest customers
   delayed, illegitimate claims paid) at two or three thresholds, including the one you recommend. What would you tell
   each of them?
3. The cost table uses one figure, $700, for every paid illegitimate claim, but claims range from about $20 to over
   $2,000. Should the team work through its list in a different order than your model's ranking? Your model is still
   scored with the table: say what you would do in practice, and what it would cost or save.
4. Some columns describe the customer rather than the claim. Name them, then check whether your model sends honest
   customers in some groups to investigation more often than others. Should DuraHome use those columns? What would
   Daniel Osei say?
5. Once the model is live, people who file false claims will learn what gets a claim investigated. Which of your
   model's inputs could a claimant change easily, and which could they not? What should Hye-jin Park check in the
   months after launch to know whether the model still works?
6. Meera Kapoor's staff believe some warning signs matter only in combination, or only past some level. Test at least
   two such ideas and say which hold up.

---

*19 features. train.csv (3,036 rows) and test.csv (1,012 rows) are raw extracts; holdout_features.csv (1,000 rows) has
been checked and needs no cleaning.*
