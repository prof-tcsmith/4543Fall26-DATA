# Scenario 12: Unsold Auction Lot Prediction

## Company Background

**Harrow & Finch Auctioneers** is a family-owned regional auction house founded in 1961. It sells art, antiques and collectibles for private collectors, estates, dealers and institutions, through ten departments: paintings, prints, jewelry, watches, furniture, silver, ceramics, books, coins and sports memorabilia. It holds about 40 sales a year, some live in its saleroom (an auctioneer on the rostrum, with phone and online bidders) and some timed online-only sales. A typical sale has about 320 lots, so the house offers about 12,800 lots a year.

The house earns money only when a lot sells. The winning bidder pays a **buyer's premium** of 25% on top of the hammer price (the winning bid), and the consignor (the owner who sends the lot) pays a **seller's commission** of 10% of the hammer price. Every lot also has a confidential **reserve**: the lowest price the consignor will accept, agreed in the consignment contract. House rules never allow a reserve above the low end of the published estimate. If bidding stops below the reserve, the auctioneer "buys in" the lot: it goes back to the consignor unsold. Under its "no sale, no fee" promise, Harrow & Finch charges nothing for a lot that does not sell and pays to ship it back.

Last year about 77% of lots sold (the trade calls this the **sell-through rate**). That means roughly 2,950 lots went back unsold, at a cost of more than $5 million in lost premium and commission, wasted handling and return shipping.

## Business Problem

Consignments for a sale close 42 days before the sale date, and the printed catalogue goes to the printer 28 days before. In the two weeks in between, a specialist can **rework** a lot that looks likely to go unsold: research recent prices again, call the consignor to agree a lower reserve (and sometimes a lower estimate), and add targeted marketing, such as a featured spot in the department's email to past buyers and paid online ads. A rework costs about $250. When a lot that would have gone unsold is reworked, about 6 in 10 of them then sell.

The house has 26 specialists across its ten departments, and the rework window is short: they also have to finish cataloguing the whole sale. Between them they can rework **at most 80 lots per sale**, about one lot in four (25%). The 1,000 or so lots in test.csv are about as many as three sales, so the specialists could rework about 250 of them. Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

Today the department heads choose which lots to rework at a Monday meeting, from memory of what failed at the last sale. That catches some obvious cases and misses many others. The Managing Director wants a model that, on the day consignments close, uses what the house knows about each lot at that moment to say which lots are likely to be bought in, so the specialists spend their 80 reworks where they matter most.

## Stakeholders

- **Eleanor Harrow, Managing Director:** the founder's granddaughter. After every sale the trade press prints each house's sell-through rate. Harrow & Finch's 77% trails the 82% of its two regional rivals, and she wants it above 80% within a year.
- **Raymond Achebe, Chief Financial Officer:** counts dollars, not lots. Unsold lots cost the house more than $5 million last year. He wants the rework program to show a clear return and expects all 80 rework slots to be used at every sale.
- **Marguerite Delacroix, Director of Consignor Relations:** every flagged lot means a phone call asking a consignor to accept a lower reserve. Long-standing collectors and the executors of estates can feel their property is being talked down, and some take their next, often more valuable, consignment to a rival. She believes about 1 in 20 unnecessary calls costs the house a future consignment, though she has no data to show it. She wants lots flagged only when the model is fairly sure, and she objects to the model judging a lot by its consignor's own history with the house: "We would be singling out our most loyal clients."
- **Tobias Lindqvist, Head of Specialists:** runs the 26 specialists and wants a ranked list for each sale that his team can trust. He is uneasy that the data records each specialist's years of experience, because a model that uses it could be read as a verdict on individual colleagues' estimates. He also points out that the reserve the model sees is the very number his team would renegotiate.

## Cost Structure

A **rework** costs about $250: about $150 of specialist time and $100 of marketing. Spent on a lot that would have sold anyway, that money is wasted, and the consignor has had an unnecessary call asking for a lower reserve.

A lot that goes **unsold** costs the house about $1,800: about $1,400 of buyer's premium and seller's commission lost on an average lot (one that would hammer at about $4,000), $110 of photography and catalogue space, $90 of handling and storage, and $200 of return shipping and insurance.

Catching a lot that would have gone unsold still costs something: the $250 rework, plus the 4 in 10 reworked lots that go unsold anyway. So the extra cost of a miss over a catch is $1,800 − ($250 + 0.4 × $1,800) = $1,800 − $970 = $830.

| Prediction | Actual | Outcome | Cost |
| --- | --- | --- | --- |
| No rework (0) | Sells (0) | True Negative | Not counted (normal business: the lot sells as catalogued) |
| Rework (1) | Would have gone unsold (1) | True Positive | Not counted (the lot is reworked, and about 6 in 10 such lots then sell) |
| Rework (1) | Would have sold anyway (0) | **False Positive** | **$250** (specialist time and marketing spent on a lot that would have sold, and an unnecessary call to its consignor) |
| No rework (0) | Bought in (1) | **False Negative** | **$830** (the extra cost of a missed unsold lot over a reworked one: $1,800 − $970) |

**The cost used to score your model:** (false positives × $250) + (false negatives × $830). True positives and true negatives add nothing.

## Target Variable

- `target = 1`: The lot was bought in: bidding at the sale did not reach the reserve, so the lot went back to the consignor unsold.
- `target = 0`: The lot sold at the auction, at or above its reserve.

**Class balance:** About 23% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision. Each row is one lot, as the house knew it on the day consignments for its sale closed.

| Feature | Description |
| --- | --- |
| `category` | Sale department of the lot (text): `paintings`, `prints`, `jewelry`, `watches`, `furniture`, `silver`, `ceramics`, `books`, `coins`, `sports_memorabilia` |
| `low_estimate_usd` | Low end of the published pre-sale estimate, in US dollars (a round figure such as 800, 1,500 or 2,000; 200 to 120,000) |
| `high_estimate_usd` | High end of the published pre-sale estimate, in US dollars (a round figure, always above the low estimate; 250 to 250,000) |
| `reserve_pct_of_low_estimate` | Confidential reserve (the lowest price the consignor will accept) as a percentage of the low estimate (whole percent, 40 to 100); house rules do not allow a reserve above the low estimate |
| `condition` | Specialist's condition grade for the object (text): `excellent`, `good`, `fair`, `poor` |
| `maker_identified` | 1 = the artist, maker or brand is identified with confidence; 0 = unattributed or uncertain |
| `provenance_documented` | 1 = a documented ownership history (receipts, earlier sale records, exhibition labels) comes with the lot; 0 = none |
| `object_age_years` | Approximate age of the object, from the specialist's dating (whole years, 3 to about 2,200) |
| `previously_offered_at_auction` | 1 = the same object was offered at a public auction (any house) in the past three years; 0 = not found |
| `requires_freight_shipping` | 1 = the lot is too large or fragile for parcel shipping and must go by crated freight; 0 = parcel shipping |
| `consignor_type` | Who consigned the lot (text): `private_collector`, `estate` (the executor of a deceased person's estate), `dealer`, `institution` (a museum, library or company collection) |
| `consignor_prior_lots` | Lots this consignor placed with Harrow & Finch in the past five years (whole number, 0 to 80; 0 = first consignment) |
| `consignor_prior_unsold_lots` | Of those earlier lots, how many were bought in (whole number, 0 to 46; never more than `consignor_prior_lots`) |
| `consignor_raised_estimate` | 1 = the consignor asked for a higher estimate than the specialist first proposed and the estimate was raised; 0 = the specialist's first estimate stood |
| `comparable_sales_2y` | Comparable lots (same maker or type, similar quality) sold at public auction anywhere in the past two years, from the specialist's price-database search (whole number, 0 to 28) |
| `category_sell_through_pct` | Share of lots in this category that sold at the four most recent Harrow & Finch sales completed before consignments closed (percent, one decimal, about 62 to 95) |
| `sale_format` | How the sale is run (text): `live_saleroom` (auctioneer on the rostrum, with phone and online bidding) or `online_only` (timed online sale) |
| `sale_season` | Season of the sale date (text): `winter` (Dec to Feb), `spring` (Mar to May), `summer` (Jun to Aug), `fall` (Sep to Nov) |
| `lots_in_sale` | Total number of lots in the sale this lot is catalogued in (whole number, 141 to 517) |
| `days_consigned_before_sale` | Days between the signed consignment agreement and the sale date (whole days, 43 to 200) |
| `specialist_years_experience` | Years of auction experience of the specialist who catalogued the lot and set its estimate (whole years, 2 to 33) |

## Questions for Your Analysis

1. The specialists can rework at most 80 lots per sale. How many lots per sale would your cost-best threshold flag? How should the 80 slots be filled, what does the house give up, in dollars, by working within the limit, and would more rework capacity pay for itself?
2. Raymond wants all 80 slots used at every sale; Marguerite wants lots flagged only when the model is fairly sure. She believes about 1 in 20 unnecessary calls costs a future consignment. What would a lost consignment have to be worth for her concern to change your threshold, and how would your total cost change if it did? Whose position do you recommend?
3. Eleanor's sell-through rate counts every lot the same, a $300 print or a $40,000 watch. Raymond counts dollars, and the cost table uses one average figure per lot. Which goal does your model serve, and where could the two pull in different directions?
4. `specialist_years_experience` describes the house's own staff. `consignor_prior_unsold_lots` and `consignor_type` treat consignors differently according to their history and who they are. Should each one be in the model? What does leaving it out cost, and what would you tell Tobias and Marguerite?
5. Some columns describe the object, the consignor or the market. Others, such as the estimate, the reserve and the sale format, are the house's own decisions, and a rework changes the reserve. Once specialists act on the model's flags, what does a flag mean, and how could the house tell next year whether the model is still working?
6. Tobias's specialists believe some warning signs matter only in combination, or only past some level. Test at least two such ideas and say which hold up.

---

*21 features. train.csv (3,030 rows) and test.csv (1,010 rows) are raw extracts; holdout_features.csv (1,000 rows) has been checked and needs no cleaning.*
