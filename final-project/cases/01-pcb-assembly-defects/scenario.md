# Scenario 01: Circuit Board Re-inspection at Kestrine

## Company Background

**Kestrine Circuit Assembly** builds high-value circuit boards for other companies: mostly control boards for factory
equipment and power systems, and also engine and brake controllers for car makers and patient monitors for
medical-device makers. The customer sends the design. Kestrine buys the bare boards and the parts, fixes the parts to
the boards, and tests the result. Its boards sell for about $1,500 each, and its sales are about $450 million a year. It
employs about 450 people in one plant.

The plant runs three assembly lines around the clock on weekdays, in day, evening and night shifts. On each line,
placement machines put the parts on the board (eight machines across the three lines). A reflow oven (an oven that melts
solder paste to fix the parts in place) then carries the board through three heated zones. Finally, a camera and an
electrical test check every board. The plant builds about 1,200 boards a day.

About 6 boards in 100 have a fault that these automatic checks pass. Most of those faults show up later, at the
customer's plant or in the field. Kestrine's largest customer, a maker of car electronics, has warned that it will move
a large share of its orders to another supplier unless fewer faulty boards reach it.

## Business Problem

After the automatic checks, a board can be pulled for **re-inspection**: a technician X-rays it, runs a full functional
test, and repairs any fault found. Today the plant pulls a random 1 board in 10. Kestrine wants a model that scores each
board as it leaves the automatic checks, so that the re-inspection benches see the boards most likely to be faulty.

**Volume and capacity.** The plant builds about 1,200 boards a day, about 400 a shift. The re-inspection area has
three benches, staffed on every shift. A re-inspection takes about 30 minutes, so after breaks and set-up the area can
take about **40 boards a shift, or 120 a day**: 1 board in 10. The 1,214 test rows are about one day's boards, drawn
from all three shifts, so the benches could take about 120 of them.

A test lab nearby can take extra boards at $160 a board. Boards sent there ship two days later than planned.

Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

## Stakeholders

- **Ruth Abernathy, VP of Manufacturing:** Measured on boards shipped on time. Every pulled board leaves the line,
  and every board sent to the lab ships two days late. She wants the benches used on the riskiest boards only, and
  nothing sent to the outside lab.
- **Kwame Asante, Quality Director:** Runs the re-inspection area. He would rather check a good board than ship a faulty
  one. He wants every doubtful board checked, with whatever the benches cannot take sent to the lab.
- **Xiaowen Zhou, Customer Quality Engineer:** Works with the car-electronics customer. The number she must show the
  customer is faulty boards delivered per 10,000 shipped. She wants that number for the plan, not a model score.
- **Bruno Ferrante, Controller:** Tracks the cost of quality. He wants the savings in dollars a day compared with
  today's random 1 in 10, and he doubts the outside lab pays for itself.
- **Paula Brandt, Production Manager, Line 2:** Wants to know which settings her crews can change, not only which boards
  to pull. She does not want the model's list turned into a scorecard for her crews.

## Cost Structure

**Re-inspecting a board costs about $90:** 30 minutes of a technician's time at $120 an hour with benefits and overhead
($60), plus $30 of X-ray time, test fixtures and handling. On a good board, that money buys nothing. A board sent to the
outside lab costs $160 instead; the table uses the in-house figure.

**A faulty board that ships costs about $2,000** on average: a replacement board, the labour to swap it at the
customer's plant or in the field, freight, and the quality charge the contracts set for each faulty board delivered.
For a board in a car or a medical device the cost can be far higher; $2,000 is the average across customers.

**A caught board still costs something.** The re-inspection costs $90. The technician finds the fault on about 9 boards
in 10 and repairs it for about $200, or $180 on average ($200 × 90%). The other 1 in 10 faults ship anyway
($2,000 × 10% = $200). So a caught board costs $90 + $180 + $200 = $470, and the extra cost of a miss over a catch is
$2,000 − $470 = $1,530.

| Prediction | Actual | Outcome | Cost |
| --- | --- | --- | --- |
| Ship without re-inspection (0) | Good (0) | True Negative | Not counted (normal production) |
| Re-inspect (1) | Faulty (1) | True Positive | Not counted (the fault is usually found and repaired before the board ships) |
| Re-inspect (1) | Good (0) | **False Positive** | **$90** (a re-inspection that finds nothing) |
| Ship without re-inspection (0) | Faulty (1) | **False Negative** | **$1,530** (the extra cost of a faulty board shipped over one caught: $2,000 − $470) |

**The cost used to score your model:** (false positives × $90) + (false negatives × $1,530). True positives and true
negatives add nothing.

## Target Variable

- `target = 1`: The board was faulty. The fault was found at re-inspection (today's random 1 in 10), at the customer's
  incoming test, or in a board returned from the field.
- `target = 0`: No fault was found.

**Class balance:** About 6% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision.

| Feature | Description |
| --- | --- |
| `solder_temperature` | Peak temperature the solder reached in the reflow oven, in °C (216 to 274) |
| `solder_duration_sec` | Time the solder stayed melted in the oven, in seconds (30 to 90, with decimals) |
| `board_thickness_mm` | Board thickness, in millimetres (0.8 to 2.4) |
| `copper_weight_oz` | Copper in each layer of the board, in ounces per square foot (0.5, 1 or 2) |
| `num_layers` | Copper layers in the board (2, 4, 6 or 8) |
| `component_count` | Parts placed on the board (48 to 119) |
| `humidity_pct` | Relative humidity on the factory floor, in % (10 to 90) |
| `ambient_temperature` | Air temperature on the factory floor, in °C (10 to 34) |
| `machine_id` | Placement machine that built the board (an ID number, 0 to 7) |
| `operator_shift` | Shift that built the board: one of "day", "evening", "night" |
| `time_since_maintenance_hours` | Hours the machine had run since its last scheduled maintenance (0 to 1,320) |
| `production_speed_units_per_hour` | Line speed when the board was built, in boards per hour (15 to 90, with decimals) |
| `reflow_zone1_temp` | Temperature of the reflow oven's first zone, in °C (212 to 280) |
| `reflow_zone2_temp` | Temperature of the reflow oven's second zone, in °C (219 to 283) |
| `reflow_zone3_temp` | Temperature of the reflow oven's third zone, in °C (207 to 271) |
| `visual_inspection_score` | Score from the automatic camera inspection, out of 100 (53 to 100, with decimals) |
| `electrical_test_voltage` | Voltage measured on the board's supply line in the electrical test, in volts (2.55 to 4.00) |
| `pin_alignment_variance` | Spread of the measured pin positions around their targets, as a variance in mm² (0 to 0.79) |
| `flux_concentration` | Concentration of flux (the chemical in the solder paste that cleans the metal so the solder sticks), in % (5 to 26) |

## Questions for Your Analysis

1. Find the threshold with the lowest cost per board, choosing it on out-of-fold probabilities (cross-validation on the
   training rows); the test set only reports your final model, once. How many boards a day would it send to
   re-inspection, against the 120 the benches can take? If it sends more, which boards should get the benches, and is
   the outside lab worth $160 a board for the rest? Give Ruth and Kwame one recommendation, in dollars a day.
2. Today the benches take a random 1 board in 10. Compare that with your model's riskiest 1 in 10: in dollars a day for
   Bruno, and in faulty boards delivered per 10,000 shipped for Xiaowen.
3. Kwame's technicians believe some warning signs matter only in combination, or only past some level. Test at least two
   such ideas and say which hold up.
4. Some columns are settings the plant controls; others it cannot change, such as the board designs customers send.
   Which of your findings could Paula's crews act on? Why might changing a setting not change the fault rate the way
   your model suggests?
5. Some columns name a particular machine or crew. Should the model use them? What happens to your model when a machine
   is rebuilt or crews change shifts, and how would the plant notice that it has stopped working?

---

*19 features. train.csv (7,691 rows) and test.csv (1,214 rows) are raw extracts; holdout_features.csv (1,200 rows) has
been checked and needs no cleaning.*
