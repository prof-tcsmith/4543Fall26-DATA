# Scenario 13: Moving Job Overrun Prediction

## Company Background

**Pellbrook Movers** is a family-owned household moving company founded in 2009. From two depots, it serves one metro area and the towns around it. It runs about 40 moves a day, six days a week (Monday to Saturday), or about 12,000 moves a year. Most moves are within 25 miles; a few go as far as 120 miles. Pellbrook only takes moves that one crew can finish in one day.

Every move starts with a quote. An estimator looks at the home (in person, on a video call, or from the customer's description on the phone), lists the furniture, and works out how many movers the job needs and how long they will take. The quote is in **labor hours**: movers × hours. A 3-mover crew quoted for 6 hours is an 18.0-labor-hour job. The customer's price is built from those hours. Quotes are prepared either by a **sales rep** or by an **operations estimator** (a former crew lead).

Customers are billed for the hours the crew actually works. They book on the strength of the quote, though, and when the bill comes in far above it, many of them dispute it. Last year about one move in five ran more than 20% over its quoted labor hours. At $1,100 per overrun (see Cost Structure), that is roughly $2.6 million a year.

## Business Problem

The crew for each move is set when the move is booked, and nobody revisits it. Rosa Delgado, the dispatcher, builds the next day's crew plan at 3 PM each day. Today she sends extra help on gut feel alone.

Pellbrook keeps **5 movers on call each day**. If Rosa flags a job by 3 PM the day before, she can add one on-call mover to that job's crew. Experience shows that an extra mover prevents about **4 in 5** of the overruns it is sent to. Once the 5 on-call movers are assigned, there is no more help for that day. With about 40 moves a day, **at most 1 job in 8 (12.5%) can be flagged**. The roughly 1,000 jobs in test.csv are about 25 working days of moves, so the on-call pool could cover about **125 of them**. Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

Ellen Marsh, the owner, wants a model that ranks tomorrow's moves by their risk of running more than 20% over the quote, so that Rosa sends the on-call movers where they will do the most good.

The data is a sample of moves from the last two years, before any flagging began. Every job in it ran with the crew it was booked with.

## Stakeholders

- **Ellen Marsh, Owner and General Manager:** Wants the cost of overruns down. She must decide whether the on-call program pays for itself and whether it should grow.
- **Rosa Delgado, Dispatcher:** Has 5 on-call movers a day and about 40 jobs to choose from. She needs tomorrow's jobs in order of risk by 3 PM, not a yes-or-no answer on each one.
- **Tyler Brooks, Sales Manager:** His reps compete with other movers on price, and some quotes are lowered to match a competitor's bid. He argues that a price-matched job barely makes money, so a $200 extra mover turns it into a loss. He also worries that if the model uses who prepared the quote, it will be used to blame his team.
- **Marcus Bell, Director of Operations:** His crews are worn out by overtime, and turnover is rising. He says flagging jobs the day before treats the symptom rather than the cause, and would rather spend the money on more accurate quotes. He would also double the on-call pool to 10 movers a day if the numbers support it.
- **Lakshmi Nair, Customer Experience Lead:** Handles the disputes. Most of the company's one-star reviews mention a bill far above the quote.

## Cost Structure

Costs per job, from the last two years' records:

- **An overrun costs $1,100 on average.** That is $250 in overtime premium (crews past the end of their shift are paid time-and-a-half), $360 in goodwill discounts (about 6 in 10 customers with an overrun dispute the bill, and the average discount is $600), $240 in knock-on costs (the crew's next job starts late, or a job is pushed to another day), and $250 in lost future business (customers whose bill ran far over the quote rarely refer friends, and many post a one-star review; this is Lakshmi Nair's estimate from the company's referral records).
- **An extra mover costs $200.** That is a 6-hour minimum call-in shift at $26 an hour ($156), plus $44 in payroll taxes and workers' compensation insurance. Pellbrook pays for the extra mover itself; the customer is not billed for it. On-call movers are paid only when they are called in.
- **A caught overrun still costs $420:** the $200 extra mover, plus the 1 in 5 flagged overruns that still happen (0.2 × $1,100 = $220). So the extra cost of a miss over a catch is $1,100 − $420 = $680.

| Prediction | Actual | Outcome | Cost |
| --- | --- | --- | --- |
| No action (0) | Finishes within 20% of the quote (0) | True Negative | Not counted (the move runs with the crew it was booked with) |
| Flag: add a mover (1) | Would have overrun (1) | True Positive | Not counted (the extra mover prevents about 4 in 5 of these overruns) |
| Flag: add a mover (1) | Would have finished within 20% of the quote (0) | **False Positive** | **$200** (an extra mover the job did not need) |
| No action (0) | Overruns (1) | **False Negative** | **$680** (the extra cost of a missed overrun over a caught one: $1,100 − $420) |

**The cost used to score your model:** (false positives × $200) + (false negatives × $680). True positives and true negatives add nothing.

## Target Variable

- `target = 1`: The move's actual labor hours were **more than 20% above** the quoted labor hours (for example, quoted at 20.0 labor hours, it took more than 24.0).
- `target = 0`: The move finished within 20% of the quote, or under it.

**Class balance:** About 20% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision (3 PM the day before the move).

| Feature | Description |
| --- | --- |
| `quoted_labor_hours` | Labor hours in the quote: movers booked × hours quoted for the job (labor hours, one decimal; 6.0 to 92.0). Each mover's time is quoted in half-hour steps, with a 3-hour minimum |
| `crew_size` | Movers booked for the job (2 to 8) |
| `home_size` | Size of the home being moved: Studio, 1 bedroom, 2 bedrooms, 3 bedrooms, 4+ bedrooms |
| `furniture_items` | Pieces of furniture and large appliances on the quote's inventory list (count, 5 to 133; boxes are not counted) |
| `heavy_items` | Specialty heavy items on the inventory list, such as pianos, gun safes and pool tables (count, 0 to 3) |
| `origin_building_type` | Building at the pickup address: House, Townhouse, Apartment, Condo |
| `dest_building_type` | Building at the delivery address: House, Townhouse, Apartment, Condo |
| `stair_flights` | Flights of stairs the crew must carry items up or down, both addresses combined (count, 0 to 6) |
| `elevator_booked` | 1 = an elevator is reserved for the move at one or both addresses, 0 = no elevator at either address |
| `long_carry_ft` | Walking distance from where the truck can park to the door, in feet, at whichever address is farther (10 to 300, to the nearest 5) |
| `drive_miles` | Driving distance between the two addresses, in miles (one decimal; about 1 to 120) |
| `packing_service` | Packing sold with the move: Customer packs (the customer packs everything), Partial (Pellbrook packs the kitchen and breakables), Full (Pellbrook packs everything). Pellbrook's packing time is included in the quoted hours |
| `survey_type` | How the estimator saw the home before quoting: In-person (a visit to the home), Video call (the customer walks the estimator through the home on video), Phone (the customer describes the home on a call) |
| `quote_prepared_by` | Who prepared the quote: Sales rep, Operations estimator |
| `price_matched` | 1 = the quote was lowered to match a competitor's bid, 0 = it was not |
| `season` | Season of the move date: Winter (December to February), Spring (March to May), Summer (June to August), Fall (September to November) |
| `day_of_week` | Day of the move: Monday, Tuesday, Wednesday, Thursday, Friday, Saturday (Pellbrook does not move on Sundays) |
| `start_slot` | When the crew is booked to arrive: Morning (8 AM) or Afternoon (1 PM) |
| `days_booked_ahead` | Days between the booking and the move date (1 to 120; bookings open 120 days ahead) |
| `confirmation_call` | Result of the office's call to the customer two days before the move (or on the day of booking, for later bookings): Ready (the customer says everything is ready to load), Not ready (the customer says packing or other preparation is unfinished), No answer (no one was reached) |
| `repeat_customer` | 1 = the customer has moved with Pellbrook before, 0 = first move with Pellbrook |

## Questions for Your Analysis

1. **Five movers, forty jobs.** Your cost-best threshold may flag more jobs than Rosa has movers for, or fewer. How will you set the cutoff when cost and capacity point to different answers, and what should Rosa do on a day when more than five jobs look worth helping?
2. **Does the program pay, and should it grow?** Ellen must decide whether the on-call program pays for itself, and Marcus would double the pool to 10 movers a day. What, in dollars a year, would the second five movers be worth? What would better quotes have to achieve, in overruns prevented a year, to be worth more?
3. **Sales versus operations.** Two columns describe how the quote was made rather than the home itself: `quote_prepared_by` and `price_matched`. Should the model use them? Tyler will object if it does, and a pattern learned from today's quoting habits may not hold if Ellen changes who prepares quotes. Put Tyler's price-match argument in dollars, take a position, and defend it.
4. **Once the model is live.** Some columns record what people say or choose: the confirmation call, a price match, who prepared the quote. Which of them could customers or staff change, on purpose or not, once an extra mover depends on them? And once flagged jobs get an extra mover, what happens to the data the model would be retrained on next year?
5. Marcus Bell's crew leads believe some warning signs matter only in combination, or only past some level. Test at least two such ideas and say which hold up.

---

*21 features. train.csv (3,030 rows) and test.csv (1,010 rows) are raw extracts; holdout_features.csv (1,000 rows) has been checked and needs no cleaning.*
