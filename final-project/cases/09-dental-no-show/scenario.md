# Scenario 09: Empty Chairs at Maplewick Dental

## Company Background

**Maplewick Dental Partners** is a regional chain of 22 dental offices in small cities and suburbs. Its 140 dentists and
hygienists offer everyday care (cleanings, exams, fillings), larger treatments (crowns, root canals, extractions),
orthodontic checks and whitening. The offices are open on weekdays and book about 5,000 appointments a week.

Patients pay in different ways. Most have a dental plan, either through an employer or one they bought themselves.
Others are on Medicaid, pay for themselves, or belong to a discount plan: a yearly membership that lowers the chain's
prices.

Each appointment holds a chair: a hygienist's chair for a cleaning, a dentist's chair for treatment. Chair time that
goes unused cannot be stored and sold later.

## Business Problem

Last year, a little over one appointment in four (about 27%) ended with an empty chair: the patient neither came nor
cancelled in time. The chain counts a cancellation as "in time" when it arrives at least 24 hours before the visit.
That is early enough to offer the slot to a patient on the office's waiting list. Later than that, the slot usually
stays empty. At about 1,000 appointments a weekday, the chain loses roughly $65,000 of chair time each weekday.

The chain already sends automatic text reminders. Its managers now want to add a **personal call** from the office's
front desk, two working days before the visit, for the appointments most likely to end with an empty chair. On the call,
the front desk confirms the visit or offers to move it. If the patient cancels, the slot goes to the waiting list. The
data are last year's appointments, from before any personal calls were made.

**Volume and capacity.** The chain books about 1,000 appointments each weekday. Each office's front desk can fit about
14 personal calls a day around check-ins and the phones: about **300 calls a weekday** across the 22 offices, or
roughly 3 appointments in 10. More calls would need more front-desk hours, at about the same cost per call. The 1,010
test rows are about one weekday's appointments across the chain, so the front desks could call about 300 of them.

Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

## Stakeholders

- **Dr. Imani Sorensen, Chief Dental Officer:** Wants as many at-risk appointments called as possible. Every empty
  chair is a longer wait for a patient who needs treatment. She is asking for extra front-desk hours so the offices can make
  more calls.
- **Wendell Okoye, Chief Financial Officer:** Counts every call at $15 and points out that the automatic texts cost
  almost nothing. He wants calls made only where the patient is more likely than not to miss. He wants the savings in
  dollars a day.
- **Danuta Kowalczyk, front-desk lead at the largest office:** Speaks for the front desks. Fourteen calls a day per office
  is the limit; past that, the phones go unanswered and check-ins slow down. She wants a short list each morning,
  riskiest first, not a yes or no for every visit.
- **Theo Marchetti, Director of Patient Experience:** Worries that a call list could single out patients by their age
  or by how they pay. A family called before every visit may feel distrusted. He wants to know what the model would
  lose without such columns.
- **Noor Rahimi, Scheduling Systems Manager:** Runs the booking system and the automatic texts, and her team changes
  the text settings from time to time. She points out that once the calls start, next year's data will not look like
  last year's.

## Cost Structure

**An empty chair costs about $240.** That is the average fee the slot would have earned across the mix of visits, less
the supplies it would have used. Staff are paid either way.

**A personal call costs about $15.** That is about 20 minutes of front-desk time (looking up the visit, up to two tries,
a voicemail, a note on the file, and rebooking the slot when the patient cancels) at $45 an hour, counting benefits.

**A call does not save every visit.** Of the patients who would have left the chair empty, about half come after the
call, or cancel in time for the waiting list to fill the slot. The other half still leave the chair empty. So a caught
case costs $15 + 50% × $240 = $135, and the extra cost of a miss over a catch is $240 − $135 = $105.

| Prediction | Actual | Outcome | Cost |
| --- | --- | --- | --- |
| No call (0) | Comes or cancels in time (0) | True Negative | Not counted (normal business) |
| Personal call (1) | Would leave the chair empty (1) | True Positive | Not counted (the call saves about half of these visits) |
| Personal call (1) | Comes or cancels in time (0) | **False Positive** | **$15** (front-desk time on a call that was not needed) |
| No call (0) | Leaves the chair empty (1) | **False Negative** | **$105** (an empty chair, less what a call would have cost and saved: $240 − $135) |

**The cost used to score your model:** (false positives × $15) + (false negatives × $105). True positives and true
negatives add nothing.

## Target Variable

- `target = 1`: The patient did not come to the appointment and did not cancel or move it at least 24 hours ahead. The
  chair sat empty.
- `target = 0`: The patient came, or cancelled or moved the appointment at least 24 hours ahead.

**Class balance:** About 27% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision.

| Feature | Description |
| --- | --- |
| `lead_time_days` | Days from the day the appointment was booked to the appointment (0 to 120, with decimals); some appointments were booked too late for a call two working days ahead |
| `patient_age` | Patient's age in years (2 to 95); a parent or guardian books for a child |
| `num_previous_appointments` | Appointments booked before this one, at any of the chain's offices, including missed ones (1 to 17) |
| `num_previous_no_shows` | Of those, how many the patient missed without cancelling in time (0 to 7) |
| `no_show_rate_historical` | Share of previous appointments missed: `num_previous_no_shows` divided by `num_previous_appointments` (0 to 1) |
| `appointment_hour` | Hour the appointment starts, on a 24-hour clock (7 to 18) |
| `day_of_week` | Day of the appointment (0 = Monday to 4 = Friday) |
| `is_new_patient` | 1 = the patient's first appointment with this dentist or hygienist, 0 = has seen them before. First-ever visits to the chain are not in the data |
| `visit_type` | One of "cleaning", "exam", "filling", "crown", "root canal", "extraction", "orthodontic check", "whitening", "follow-up" |
| `dental_coverage` | How the visit is paid for: one of "employer plan", "individual plan", "Medicaid", "self-pay", "discount plan" |
| `distance_to_office_miles` | Distance from the patient's home to the office, in miles (about 0.5 to 60) |
| `reminder_sent` | 1 = the automatic reminder text was sent, 0 = not sent (the patient opted out or has no mobile number) |
| `num_reminders_sent` | Follow-up automatic texts sent after the first one, up to the day of the call (0 to 5; 0 when no reminder was sent) |
| `rescheduled_count` | Times this appointment has already been moved to another date (0 to 4) |
| `weather_forecast_severity` | Weather forecast for the appointment day, as known two working days ahead (0 = clear to 3 = severe) |
| `ongoing_treatment_plan` | 1 = the patient is partway through a treatment plan of several visits (for example a crown, an implant or braces), 0 = no |
| `procedures_last_year` | Treatments (fillings, crowns, extractions, root canals and so on) on the patient's chart for the last 12 months; cleanings and exams are not counted (0 to 8) |
| `patient_portion_due` | Amount the patient is asked to pay at check-in for this visit, in dollars; any balance is billed later ($0 to $85) |
| `wait_last_visit_minutes` | Minutes the patient waited past the booked time at their last visit (0 to 90, with decimals) |
| `app_account_active` | 1 = the patient (or, for a child, a parent) uses the chain's booking app, 0 = does not |

## Questions for Your Analysis

1. Choose the threshold with the lowest cost per appointment, using out-of-fold predictions on the training rows. How
   many calls a weekday would it ask for across the chain, against the 300 the front desks can make? If it asks for
   more, which appointments should get the 300 calls, and what does the limit cost the chain each weekday?
2. Imani wants more front-desk hours; Wendell wants calls only where a miss is more likely than not. Price both
   positions in dollars a weekday. What would the next 100 calls a day be worth, and would you hire for them?
3. Danuta's staff believe some warning signs matter only in combination, or only past some level. Test at least two such
   ideas and say which hold up.
4. Theo does not want the list to single out patients by age or by how they pay. Build your model with and without
   `patient_age` and `dental_coverage`. What does leaving them out cost in dollars, and what would you advise him?
5. Some columns describe the patient; others describe what the chain itself does, such as the reminder texts, which
   Noor's team can change. Which of your findings would change if the text settings changed, or once the personal calls
   start? Will your model still work next year, and how would the chain find out?

---

*20 features. train.csv (3,030 rows) and test.csv (1,010 rows) are raw extracts; holdout_features.csv (1,000 rows) has
been checked and needs no cleaning.*
