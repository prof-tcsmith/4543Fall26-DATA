# Scenario 14: Solar Permit Rejection Prediction

## Company Background

**Copperline Solar** is a residential solar installer founded in 2013. It designs and installs rooftop and ground-mounted solar systems, often with home batteries, for homeowners in six neighboring towns and counties. Copperline installs about 4,000 systems a year. The average system is about 9 kW and sells for about $24,000, and Copperline's average gross profit on an install is about $7,500.

Before a crew can install anything, the local building department must approve a permit. A Copperline designer draws a **plan set**: a site plan, a roof layout, electrical diagrams, structural sheets and equipment spec sheets. The permit coordinator submits it. The building department usually takes one to four weeks to review it, then either **approves** it or **rejects** it with a list of corrections. A rejected application must be fixed and submitted again, and the wait starts over.

Each building department has its own reviewers, its own checklist and its own habits.

## Business Problem

Copperline submits about **80 permit applications a week**. About **1 in 4 is rejected on first submission**, which is roughly 19 rejections a week. Each rejection costs about $1,200 (see the Cost Structure), so first-submission rejections cost Copperline more than $1 million a year.

Copperline has one way to catch a bad file before it goes out: its senior design engineer, **Dana Whitfield**, can **pre-review** a plan set, find the problems and fix them before submission. A pre-review takes about two hours and holds the application back by one working day. When Dana reviews a file that would have been rejected, she catches and fixes the problem in about 9 files out of 10.

Dana can pre-review **at most 16 applications a week** (32 of her 40 hours; she has other work). That is 20% of the weekly applications. The 1,000 or so applications in test.csv are about 12.5 weeks of Copperline's volume, so Dana could review about 200 of them. Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it. Today she reviews whichever 16 files reach her desk first each week, which is close to a random choice.

An outside engineering firm has offered to pre-review extra files at **$300 per file**, with the same one-day hold and the same 9-in-10 catch rate, and no weekly limit. A file sent to the firm costs Copperline $340 ($300 fee plus the $40 one-day hold), against $200 for one of Dana's reviews.

The Chief Operating Officer wants a model that, each week, ranks the applications that are ready to submit by their chance of being rejected, so that Dana spends her 16 reviews where they save the most money. The COO also wants a recommendation on whether to pay the outside firm, and for how many files.

## Stakeholders

- **Curtis Wainwright, VP of Sales:** paid on installs completed each month. Every file held for review is one more day before a customer gets an install date, and his salespeople take the angry calls. He wants as few files held back as possible, and only the ones the model is very sure about. He has also said: "If my reps learn that the rush box changes which files get held, they will tick it, or not, to suit the customer."
- **Leonie Fairbanks, Director of Engineering:** wants every file checked. "Each rejection makes us look careless to building departments we deal with every week." She wants every file Dana cannot take sent to the outside firm.
- **Dana Whitfield, Senior Design Engineer:** does the reviews. She wants a ranked list each Monday, not a yes/no for every file, and wants to know what to look at first in each flagged file.
- **Luis Ortega, Installation Operations Manager:** books crews three to four weeks ahead. A rejection usually arrives after a crew has been scheduled, so the crew calendar must be reshuffled.
- **Karen Liu, CFO:** wants the weekly savings in dollars, compared with today's random choice of 16 files and with reviewing nothing at all.
- **Tom Becker, Design Team Lead:** manages the 14 designers. He is uneasy that the model may hold back some designers' files more often than others. "Is the model judging the file or the person who drew it?"

## Cost Structure

A **pre-review** costs $200: Dana's time (2 hours at $80 an hour with benefits = $160) plus the one-day hold on the application (about $40: the permit coordinator's and crew scheduler's time to re-plan the job, plus a day's delay in payment).

A **rejection** on first submission costs about $1,200:

- $250 average resubmission fee charged by the building department
- $180 of designer and permit-coordinator time to make the corrections and resubmit
- $320 to reschedule the crew that was booked for the original install date
- $150 for the delayed final payment (about three weeks of financing cost and cash flow)
- $300 of expected lost profit from cancellations (about 1 customer in 25 whose permit is rejected cancels the job; Copperline loses the $7,500 profit on that job)

Catching a file that would have been rejected still costs something: the $200 review, plus a 1-in-10 chance that Dana misses the problem and the file is rejected anyway. So the extra cost of a miss over a catch is $1,200 − ($200 + 0.10 × $1,200) = $1,200 − $320 = $880.

| Prediction | Actual | Outcome | Cost |
| --- | --- | --- | --- |
| Submit as is (0) | Approved (0) | True Negative | Not counted (normal process) |
| Pre-review (1) | Would have been rejected (1) | True Positive | Not counted (Dana finds and fixes the problem in about 9 files in 10) |
| Pre-review (1) | Would have been approved (0) | **False Positive** | **$200** (Dana's time and a one-day hold for a file that was fine) |
| Submit as is (0) | Rejected (1) | **False Negative** | **$880** (the extra cost of a missed rejection over a pre-reviewed one: $1,200 − $320) |

**The cost used to score your model:** (false positives × $200) + (false negatives × $880). True positives and true negatives add nothing.

## Target Variable

- `target = 1`: The building department rejected the application on its first submission (it was returned with corrections required).
- `target = 0`: The building department approved the application on its first submission.

**Class balance:** About 24% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision. Each row is one application, as Copperline knew it on the day the plan set was ready to submit.

| Feature | Description |
| --- | --- |
| `jurisdiction` | The building department reviewing the application (text): `Ashcombe Township`, `Brindle Creek`, `Corvale`, `Dunfield County`, `Elmsworth`, `Fennmoor` |
| `application_month` | Month the application was submitted (text): `Jan`, `Feb`, `Mar`, `Apr`, `May`, `Jun`, `Jul`, `Aug`, `Sep`, `Oct`, `Nov`, `Dec` |
| `days_since_code_update` | Days between the jurisdiction's most recent building-code update (or change to its permit checklist) and the day this application was submitted (whole days, 0 to 730) |
| `prior_rejections_jurisdiction_90d` | Number of Copperline applications to the same jurisdiction that were rejected in the 90 days before this application was submitted (whole number, 15 to 94; counts only rejections already received by the submission date) |
| `system_size_kw` | Total panel capacity in kilowatts (kW; two decimals, 3.20 to 18.92): the number of panels times each panel's rated wattage (400 to 430 watts) |
| `num_panels` | Number of solar panels in the system (8 to 44) |
| `mount_surface` | What the panels are mounted on (text): `Asphalt shingle`, `Concrete tile`, `Clay tile`, `Metal`, `Flat membrane` (roof mounts), or `Ground mount` (a frame in the yard, not on the roof) |
| `roof_pitch_deg` | Slope of the roof section that holds the panels (whole degrees, 0 to 45; 1 to 5 for `Flat membrane`; 0 for `Ground mount`) |
| `home_age_years` | Age of the house when the application was submitted (whole years, 0 to 110; 0 = new construction) |
| `structural_letter_attached` | 1 = the plan set includes a letter stamped by a licensed structural engineer confirming the roof (or, for a ground mount, the footings) can carry the system; 0 = no letter |
| `battery_included` | 1 = a home battery is part of the job; 0 = panels only |
| `main_panel_upgrade` | 1 = the home's main electrical panel must be replaced or upgraded as part of the job; 0 = no upgrade |
| `inverter_type` | Equipment that converts the panels' power for the home (text): `Microinverters`, `String inverter`, `String inverter with optimizers` |
| `site_survey_type` | How the roof and site were measured (text): `In-person` (a technician visited), `Drone` (a drone flight, no roof walk), `Satellite imagery only` (no site visit) |
| `designer_experience_months` | Months of solar permit-design experience of the designer who drew the plan set, at Copperline and at earlier jobs (1 to 144) |
| `plan_set_pages` | Number of pages in the submitted plan set (6 to 22) |
| `design_revisions` | Number of times the design was changed at the customer's request before the plan set was finished (0 to 6) |
| `sales_rush_request` | 1 = the salesperson marked the job as a rush (the customer wants the earliest possible install); 0 = not marked |
| `hoa_member` | 1 = the home is in a homeowners' association; 0 = not in one |
| `financing_type` | How the customer is paying (text): `Cash`, `Loan`, `Lease` |

## Questions for Your Analysis

1. Dana can review 16 of about 80 files a week, and the outside firm will take more at $340 each. Curtis wants as few files held as possible; Leonie wants every file Dana cannot take sent to the firm. What would each position cost Copperline per week, compared with today's random 16 and with your recommendation? How many files a week, if any, should go to the firm, and how many does your recommendation hold back for a day?
2. `designer_experience_months` describes the person who drew the plan set, not the job. Should the model use it? If some designers' files were held more often than others', would that be unfair to them, useful training, or both? What does leaving it out cost, and what would you tell Tom?
3. Salespeople fill in `sales_rush_request`, and Curtis says his reps will change how they use it once they know the model reads it. Should the model use it? What would happen to your model once the reps change their habits?
4. Some columns describe things Copperline cannot change: the town, the house and the customer's choices. Others describe Copperline's own decisions: how the job was sold, how the site was surveyed, who drew the plan set and what went into it. Besides choosing files for Dana, what should Copperline do differently before a file reaches her, and how sure can you be from past data alone?
5. Building departments change their rules, Copperline hires new designers, and the number of applications rises and falls with the seasons. The model was trained on two years of applications. Would it still rank files well next year, and what would you watch?
6. Dana believes some warning signs matter only in combination, or only past some level. Test at least two such ideas and say which hold up. What would you tell her to look at first in a flagged file?

---

*20 features. train.csv (3,030 rows) and test.csv (1,010 rows) are raw extracts; holdout_features.csv (1,000 rows) has been checked and needs no cleaning.*
