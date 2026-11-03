# Scenario 06: Late Critical Supplier Deliveries

## Company Background

Vantry Automotive Components makes brake systems, suspension parts and drive shafts for three car and truck makers. It
has four plants in Michigan and Ohio and about 6,500 employees. It buys from about 180 suppliers in 22 countries and
places about 12,000 purchase orders a month.

Vantry's plants run **just-in-time**: parts arrive shortly before they are needed, so little money is tied up in stock.
The price of that is exposure. When a part runs out, an assembly line stops, and Vantry's customers charge a penalty for
every hour their own lines then wait.

About 250 of Vantry's orders a month are **critical**: parts for which the plant holds only about five days of buffer
stock. A critical delivery more than five days late stops a line. The data covers these critical orders.

## Business Problem

Today a buyer judges each critical order's risk by experience, and a supplier audit happens once a year at most. Vantry
wants a model that scores each critical order **when it is placed**, so that the team has the whole lead time to act.
The decision for each order: **mitigate it, or leave it to arrive as planned.** To mitigate, the expediting team places a
small backup order with another supplier, books faster freight for part of the quantity, or builds extra safety stock.
Each row of the data is one critical order.

Vantry places about 250 critical orders a month. The expediting team of four buyers can mitigate **at most 25 orders a
month**. The 1,012 test rows are about four months of critical orders, so the team could mitigate about 100 of them.

An outside expediting firm has offered to handle extra orders for about $14,000 each, all costs included, against about
$8,000 when Vantry's own team does it. Vantry has not yet decided whether to use it.

Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

## Stakeholders

- **Deepa Krishnan, VP of Supply Chain.** Sponsors the project. She is measured on on-time delivery of critical parts,
  with a target of 98%, and has to show the board that disruption costs are falling.
- **Callum Mills, Procurement Director.** Owns the supplier relationships. Each backup order moves business away from
  the original supplier and tells it Vantry has doubts, so he wants mitigation used sparingly, with reasons he can
  explain to suppliers.
- **Ignacio Benavides, Plant Manager, Toledo.** His plant pays the customer penalties when a line stops. He wants every
  order the model has real doubt about covered, and would pay the outside firm to do it.
- **Leila Farouk, CFO.** Every mitigation ties up cash in extra stock or premium freight. She wants the team to mitigate
  only the orders it is confident will be late, and doubts the outside firm is worth its price.
- **Oksana Petrenko, Supplier Quality Manager.** Keeps the supplier scorecards. She wants to show each supplier its own
  risk scores in its quarterly review, to push it to improve.

## Cost Structure

The figures come from Vantry's finance and logistics records.

- Mitigating an order costs about **$8,000**: the premium on the backup order, faster freight, carrying extra stock, and
  the buyer's time. It is spent whether or not the original order would have been late.
- A late critical delivery with no mitigation costs about **$120,000** on average: the line stoppage, customer
  penalties and rush freight.
- Mitigation prevents the stoppage in about **4 cases out of 5**. In the fifth, the line stops anyway.

So mitigating an order that would have arrived on time costs $8,000. A missed late order costs $120,000; a caught one
costs $8,000 for the mitigation plus a 20% chance the line still stops ($24,000); so FN = $120,000 - $32,000 = $88,000.

| Prediction | Actual | Outcome | Cost |
|---|---|---|---|
| No action (0) | On time (0) | True Negative | Not counted (normal business) |
| Mitigate (1) | Late (1) | True Positive | Not counted (backup supply covers the gap in about 4 cases out of 5) |
| Mitigate (1) | On time (0) | **False Positive** | **$8,000** (backup supply for an order that arrived on time) |
| No action (0) | Late (1) | **False Negative** | **$88,000** (a late delivery with no backup at $120,000, against $32,000 expected with mitigation) |

**The cost used to score your model:** (false positives x $8,000) + (false negatives x $88,000). True positives and true
negatives add nothing.

## Target Variable

- `target = 1`: The order arrived more than five days after its promised delivery date.
- `target = 0`: The order arrived on time, or no more than five days late.

**Class balance:** About 8% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision.

| Feature | Description |
|---|---|
| `supplier_reliability_score` | Score from Vantry's latest audit of the supplier's delivery processes, 0 to 100 (higher = more reliable): 30 to 100 |
| `lead_time_days` | Days from placing the order to the date the supplier promised to ship it (transport time not included): 1 to 120, with decimals |
| `order_quantity` | Units ordered: 23 to about 35,100 |
| `supplier_country_risk` | Risk rating of the supplier's country: 0 (low) to 4 (very high) |
| `transportation_mode` | How the order ships: "truck", "rail", "ocean" or "air" |
| `days_since_last_disruption` | Days since this supplier last reported a disruption at its own operations (a fire, a strike, a breakdown, a shortage of material), whether or not a delivery was late: 0 to 1,402 |
| `num_disruptions_last_year` | Deliveries from this supplier that arrived more than five days late in the past 12 months, whatever the cause: 0 to 6 |
| `supplier_financial_health_score` | The supplier's financial strength from a credit-rating agency, scored 0 to 100 (higher = healthier): 14 to 100 |
| `commodity_price_volatility` | How much the price of the order's main raw material has moved over the past year, as an index (higher = more movement): 0 to 1.23 |
| `weather_risk_index` | Forecast weather risk along the shipping route, on a 0 to 10 scale: 0 to 7.5 |
| `geopolitical_risk_score` | Political-instability rating of the supplier's region, on a 0 to 10 scale: 0 to 9.1 |
| `port_congestion_index` | Congestion at the ports and freight hubs on the route, on a 0 to 10 scale: 0 to 9 |
| `num_alternative_suppliers` | Other suppliers already approved to make this part: 0 to 12 |
| `contract_type` | "spot" (a one-off purchase), "short-term" (a contract of a year or less) or "long-term" (a contract of more than a year) |
| `order_complexity_score` | How customized or complex the order is, on a 0 to 10 scale: 0.2 to 9.8 |
| `seasonal_demand_index` | Seasonal demand for the part when the order is placed: 0 (low season) to 1 (peak) |
| `supplier_capacity_utilization` | Share of the supplier's production capacity in use at its last review, in percent: 13 to 100 |
| `distance_km` | Shipping distance, in kilometers: 30 to about 14,000 |
| `customs_complexity_score` | Customs and paperwork complexity for the shipment, on a 0 to 10 scale: 0 to 9 |

## Questions for Your Analysis

1. The team can mitigate 25 orders a month, about 100 of the test rows. How many orders does your cost-best threshold
   flag? If it is more than 100, which orders go first, and what does the limit cost Vantry a month? For which of the
   remaining orders, if any, is the outside firm's $14,000 worth paying? Show the arithmetic.
2. Ignacio Benavides wants every order the model has real doubt about covered; Leila Farouk wants only the orders the
   team is confident will be late. Turn each view into a threshold and price both on out-of-fold probabilities
   (cross-validation on the training rows; the test set only reports your final model, once), and say what you would
   tell them.
3. Oksana Petrenko wants to show each supplier its risk scores. Which columns could a supplier change to look safer
   without becoming more reliable, and what would that do to the model over time?
4. Some columns describe choices Vantry makes when it orders (how it ships, the contract, the quantity, the lead time);
   others it cannot control. Can this model tell Vantry how to order, for example whether to ship by air, or only which
   orders to watch? Explain from what the course taught about what a predictive model can and cannot show.
5. What changes in the world could make the model stop working next year, and what should Deepa Krishnan's team track
   each month to notice?
6. Deepa Krishnan's buyers believe some warning signs matter only in combination, or only past some level. Test at
   least two such ideas and say which hold up.

---

*19 features. train.csv (3,036 rows) and test.csv (1,012 rows) are raw extracts; holdout_features.csv (1,000 rows) has
been checked and needs no cleaning.*
