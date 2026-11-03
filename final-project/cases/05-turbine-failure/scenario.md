# Scenario 05: Power-Plant Machine Failure Warning

## Company Background

Merrowfield Generation is an independent power producer: it owns power plants and sells their electricity to the
regional grid operator (the organization that runs the high-voltage grid and decides which plants run when). It has
nine natural-gas plants with 26 generating units and about 9,200 megawatts (MW) of capacity, and employs about 1,100
people, 400 of them in maintenance.

Each generating unit is built around a few large machines: turbines, generators and pumps. Merrowfield
monitors about 100 of these machines with vibration, temperature and pressure sensors, monthly thermal-imaging
inspections (infrared photographs that show hot spots), and maintenance records.

Today every machine is maintained on a calendar: it is opened up on a fixed schedule, whatever its condition. Between
calendar visits, a few machines fail without warning each month. A **forced outage** (an unplanned shutdown after a
failure) costs far more than a planned one.

## Business Problem

Merrowfield wants to add condition-based maintenance to the calendar. On the first of every month, each monitored
machine will be scored for the risk of a forced outage in the next 60 days. The decision for each machine: **bring it
in for an extra planned shutdown (inspection and repair) now, or leave it running until its next calendar visit.**
Each row of the data is one machine at one monthly scoring.

A planned shutdown takes about a week to set up: a crew, the parts, and approval from the grid operator. The model looks
60 days ahead, so most failures it warns about can be headed off in time. About 15% of them come before the shutdown
can be arranged.

About 100 machines are scored each month. Merrowfield's crews and the grid operator's outage calendar allow **at most 6
extra planned shutdowns a month**, on top of the calendar visits already booked. The 1,010 test rows are about ten
months of monthly scores, so about 60 extra shutdowns. The grid operator sets its limit to keep enough power available;
Merrowfield can ask for more, but cannot simply buy more.

Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

## Stakeholders

- **Dolores Whitcombe, VP of Plant Operations.** Measured on how many hours the fleet is available to run. Every extra
  shutdown takes a unit off the market for about two days, so she wants as few as possible and a list she can trust.
- **Yaw Ofori, Reliability Engineering Manager.** Has investigated every forced outage for eight years and believes
  most gave some warning weeks ahead. He would bring in every machine with a real chance of failing, even if many turn
  out to be healthy, and wants Merrowfield to ask the grid operator for a higher limit.
- **Sigrid Thorsby, Maintenance Planning Manager.** Builds the monthly outage calendar and books the crews. She needs a
  ranked list, not just a yes or no, because she can only take the top few.
- **Bertrand Ibarra, CFO.** Runs a maintenance budget of $38 million a year. Wants dollar evidence that the model beats
  the calendar alone. He also wants to use it to choose a manufacturer for the next turbine purchase.
- **Annika Hartley, Regulatory Compliance Manager.** The state regulator reviews every forced outage longer than four
  hours. After a failure, she must explain why the machine was or was not brought in, so she wants a model whose reasons
  she can show.

## Cost Structure

The figures come from Merrowfield's maintenance and finance records.

- An extra planned shutdown costs about **$20,000**: crew time, the inspection, routine parts, and sales lost while the
  unit is down. Planned shutdowns are booked on low-demand days, so the lost sales are small.
- A forced outage costs about **$250,000** on average: emergency repair, overtime, power Merrowfield must buy to meet
  its contracts, and penalties.
- About **15%** of the failures the model catches happen before the planned shutdown.

So a planned shutdown of a healthy machine costs $20,000. A missed failure costs $250,000; a caught failure costs
$20,000 for the shutdown plus a 15% chance the machine fails first ($37,500); so FN = $250,000 - $57,500 = $192,500.

| Prediction | Actual | Outcome | Cost |
|---|---|---|---|
| Leave running (0) | No forced outage (0) | True Negative | Not counted (normal business) |
| Planned shutdown (1) | Forced outage (1) | True Positive | Not counted (the fault is found and fixed in a planned shutdown, except for the about 15% that fail first) |
| Planned shutdown (1) | No forced outage (0) | **False Positive** | **$20,000** (a planned shutdown of a machine that would have kept running) |
| Leave running (0) | Forced outage (1) | **False Negative** | **$192,500** (a forced outage at $250,000, against $57,500 expected with a planned shutdown) |

**The cost used to score your model:** (false positives x $20,000) + (false negatives x $192,500). True positives and
true negatives add nothing.

## Target Variable

- `target = 1`: The machine had a forced outage (a failure that stopped it and needed repair) within 60 days of being
  scored.
- `target = 0`: The machine had no forced outage in those 60 days.

**Class balance:** About 5% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision.

| Feature | Description |
|---|---|
| `equipment_age_years` | Years since the machine was installed: 0.5 to 40 |
| `operating_hours_total` | Hours the machine has run since it was installed: about 2,500 to 340,000 |
| `hours_since_last_maintenance` | Running hours since the last maintenance visit: 0 to about 8,500 |
| `vibration_level_mm_s` | Vibration at the latest sensor reading, in millimeters per second: 0.2 to 14.5 |
| `temperature_bearing_celsius` | Bearing temperature at the latest reading, in degrees Celsius: 40 to 126 |
| `oil_pressure_psi` | Lubricating-oil pressure, in pounds per square inch: 11 to 72 |
| `coolant_flow_rate` | Coolant flow, in liters per minute: 50 to 188 |
| `power_output_mw` | Output of the generating unit the machine serves, at the latest reading, in MW: 113 to 600 |
| `load_factor_pct` | The unit's average output over the past month, as a percentage of its rated capacity: 11 to 99.5 |
| `ambient_temperature` | Air temperature around the machine (outdoors, or in the turbine hall for indoor machines), in degrees Celsius: -8 to 62 |
| `num_previous_failures` | Forced outages this machine has had before: 0 to 6 |
| `manufacturer` | Maker of the machine: "Aldane", "Brennick", "Corvu", "Delmont" or "Easterly" |
| `equipment_type` | Kind of machine. One of "gas turbine", "steam turbine", "generator", "boiler feed pump" |
| `last_maintenance_type` | Kind of the last maintenance visit. One of "routine inspection", "minor repair", "major overhaul" |
| `humidity_pct` | Outdoor relative humidity at the plant, in percent: 20 to 95 |
| `startup_count_last_month` | Times the unit was started up in the last month: 0 to 11 |
| `operating_mode` | The unit's planned role for the coming month, agreed with the grid operator: "base load" (runs most hours), "peaking" (runs when demand is high) or "standby" (held in reserve) |
| `alarm_count_last_week` | Minor alarms raised in the last week: 0 to 8 |
| `efficiency_rating` | Current efficiency as a percentage of the machine's design efficiency: 56 to 100 |
| `thermal_image_anomaly_score` | Score from the latest thermal-imaging inspection, 0 = nothing unusual, higher = more unusual: 0 to 2.81 |

## Questions for Your Analysis

1. The limit is 6 extra shutdowns a month, about 60 in the test file. How many machines a month does your cost-best
   threshold flag? If it is more than 6, which machines should Sigrid Thorsby take first, and what does the limit cost
   Merrowfield a month in expected forced outages?
2. Yaw Ofori wants to ask the grid operator for a higher limit; Dolores Whitcombe would rather use fewer than 6 slots
   a month. Using your test-set results, put a dollar figure on each position: what would each extra monthly slot save,
   and what would giving up a slot cost?
3. Bertrand Ibarra wants to use the model to choose a manufacturer for the next turbine purchase. Can a model built to
   predict failures answer that question? Does anything else differ between the makers' machines, in this data or
   outside it, that could make the comparison unfair?
4. Once the model is live, flagged machines are repaired before they fail. What will that do to the failures recorded
   next year, and to a model retrained on them? How should Merrowfield check each month that the model still works?
5. Pick one failure in the test set that your model missed. Write the few sentences Annika Hartley could give the
   regulator about why that machine was not brought in, using only what your model and the data can show.
6. Yaw Ofori's engineers believe some warning signs matter only in combination, or only past some level. Test at
   least two such ideas and say which hold up.

---

*20 features. train.csv (3,030 rows) and test.csv (1,010 rows) are raw extracts; holdout_features.csv (1,000 rows) has
been checked and needs no cleaning.*
