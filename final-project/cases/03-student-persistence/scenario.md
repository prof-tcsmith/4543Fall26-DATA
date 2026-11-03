# Scenario 03: Mid-Degree Persistence Coaching

## Company Background

Larkfield University is a private, nonprofit university with about 9,000 undergraduates. Its largest school, the
Halvorsen School of Business, enrolls about 3,800 of them. Students take a common core of courses first, then apply to
a major (accounting, finance, marketing, management or information systems), usually after their first year.

Larkfield's listed price is high, and most students receive large aid packages. Many still face costs their aid does not
cover, and many work while they study.

The school loses most of its students in the middle of the degree, in the second and third years. Of the Halvorsen
students who reach this stage, a little more than one in five leaves Larkfield without a degree. Each one takes with
them about $28,000 of net tuition (tuition after aid) that Larkfield would have collected for the rest of the degree.

## Business Problem

Last year the school piloted a **Persistence Plan**. A student on a plan is assigned a success coach (a staff member who
checks in every week and helps with registration, study planning and aid appeals) for the academic year, plus a small
completion grant when money is the obstacle. Today a student gets a plan only by asking for one, or when an instructor
raises a concern.

The Dean wants plans to reach students before they ask. In the fifth week of each fall term, the school will review every
student who has earned between about 23 and 75 credit hours, about 1,000 students a year. The decision for each one:
**offer a Persistence Plan, or not.** Each row of the data is one student at one fall review.

The school has six success coaches. Each coach is capped at 40 students from the review on a plan at a time, so the
**capacity is 240 plans a year**. The 1,011 test rows are about one fall's review, so the coaches could take about 240 of
them. Students who ask for a plan, or whom an instructor refers, still get one; the coaches hold places for them outside
the 240.

An outside coaching firm has offered to take extra students at $2,600 each for the year, completion grants included,
and says its coaches save about as many students as Larkfield's do. Larkfield has not yet decided whether to use it.

Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

## Stakeholders

- **Dr. Halina Zawadzki, Dean of the Halvorsen School.** Sponsors the plan. The school publishes its completion rate in
  its admissions material, and she answers to the Provost for it. She will ask for more coaches if the dollar case
  holds up.
- **Gideon Feld, Director of Student Success.** Manages the six coaches. He treats the cap of 40 as firm: past that,
  coaches stop doing weekly check-ins and the plan stops working. He believes some students who are very likely to leave
  have already decided, and that a coach's time is better spent on students "still on the fence".
- **Aparna Venkatesan, Associate Vice President for Finance.** Sees the plan as an investment and wants its return shown
  in dollars. She would pay the outside firm, but only for students where the expected saving is clearly above its
  price.
- **Esteban Alvarado, Dean of Students.** Insists that a plan is only ever extra support, never a mark against a student.
  He is uneasy about a model that uses first-generation status, Pell eligibility or age, because coaches would receive a
  list that may sort students by background. He would accept a somewhat costlier model to avoid that.
- **Mi-rae Seo, Senior Analyst, Institutional Research.** Built the data from students reviewed in past falls whose
  outcomes are now known. She points out that Larkfield changed how it packages financial aid last year, and asks
  whether a model trained on earlier students will hold for new ones.

## Cost Structure

The figures come from Larkfield's finance office and from the pilot.

- A Persistence Plan costs about **$1,400** per student for the year: about $1,000 of coach time and an average of $400
  in completion grants.
- A student who leaves costs Larkfield about **$28,000** of net tuition.
- In the pilot, coaches kept about **3 in 10** of the students who would otherwise have left. The other 7 in 10 left
  anyway.

So a plan for a student who would have stayed anyway costs $1,400. A missed student costs $28,000; a caught student
costs $1,400 for the plan plus a 70% chance the student still leaves ($19,600); so FN = $28,000 - $21,000 = $7,000.

| Prediction | Actual | Outcome | Cost |
|---|---|---|---|
| No plan (0) | Stays (0) | True Negative | Not counted (normal business) |
| Plan (1) | Leaves (1) | True Positive | Not counted (the student works with a coach; about 3 in 10 such students stay) |
| Plan (1) | Stays (0) | **False Positive** | **$1,400** (a plan for a student who would have stayed anyway) |
| No plan (0) | Leaves (1) | **False Negative** | **$7,000** (the student leaves with no coach: $28,000 lost, against $21,000 expected with a plan) |

**The cost used to score your model:** (false positives x $1,400) + (false negatives x $7,000). True positives and true
negatives add nothing.

## Target Variable

- `target = 1`: The student left Larkfield without a degree: they stopped enrolling and had not re-enrolled two
  years later.
- `target = 0`: The student graduated, or was still enrolled at Larkfield. A student who moved from the business school
  to another Larkfield school counts as 0.

**Class balance:** About 22% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision.

| Feature | Description |
|---|---|
| `gpa_first_semester` | GPA in the student's first semester at Larkfield (0 to 4 scale): 0.17 to 4.00 |
| `gpa_current` | Cumulative GPA through the last completed term (0 to 4 scale): 1.34 to 4.00 |
| `credit_hours_completed` | Credit hours earned so far: 23 to 75 |
| `credit_hours_attempted` | Credit hours attempted so far, including courses failed (courses withdrawn from are not counted): 26 to 78 |
| `course_withdrawal_count` | Courses the student has withdrawn from so far: 0 to 6 |
| `financial_aid_amount` | Financial aid this academic year (grants, scholarships and loans), in dollars and cents: about $100 to $65,000 |
| `unmet_financial_need` | Cost of attendance this year not covered by aid or the family's expected contribution, in dollars and cents: $1 to about $28,000 |
| `is_first_generation` | 1 = neither parent has a four-year degree, 0 = otherwise |
| `is_pell_eligible` | 1 = eligible for a federal Pell grant (a need-based grant for students from low-income families), 0 = not |
| `campus_housing` | 1 = lives in university housing, 0 = lives off campus |
| `extracurricular_count` | Registered clubs and activities the student takes part in: 0 to 8 |
| `advisor_meetings_per_semester` | Meetings with an academic advisor per semester, on average: 0 to 6 |
| `declared_major` | 1 = admitted to a business major, 0 = not yet admitted to a major |
| `distance_from_home_miles` | Distance from the student's home address to campus, in miles: 5 to 3,000 |
| `age_at_enrollment` | Age when the student first enrolled at Larkfield, in years: 17 to 31 |
| `high_school_gpa` | High school GPA (0 to 4 scale): 1.50 to 4.00 |
| `sat_score` | SAT score at admission (out of 1,600): 600 to 1,600 |
| `num_absences_current_semester` | Class absences recorded so far this term: 0 to 11 |
| `part_time_work_hours` | Hours of paid work per week, as the student reported it this term: 0 to 40, with decimals |
| `library_visits_per_month` | Library entries per month, on average: 0 to 12 |

## Questions for Your Analysis

1. The coaches can take 240 students a year, about 240 of the test rows. How many students does your cost-best
   threshold flag? If it is more than 240, which 240 would you choose, and what does the cap cost Larkfield in a year?
   Then advise Aparna Venkatesan: for which of the remaining students, if any, is the outside firm's $2,600 worth paying?
   Show the arithmetic.
2. Gideon Feld wants plans to go to students "still on the fence", not to those most likely to leave. Your model ranks
   students by how likely they are to leave, not by how much a coach would help them. What can your analysis say about
   his view, what can it not say, and what would Larkfield need to record to settle it?
3. Fit your final model with and without `is_first_generation`, `is_pell_eligible` and `age_at_enrollment`, and compare
   the two by cross-validation on the training rows (the test set only reports your final model, once). What does
   leaving them out cost per student, and how does the list of flagged students change? What do you advise Esteban
   Alvarado?
4. Mi-rae Seo plans to retrain the model every year. Which columns will the Persistence Plan itself change for the
   students it reaches, and what will that do to next year's training data? Given last year's change to aid packaging,
   how would you check that the model still works for new students?
5. Gideon Feld's coaches believe some warning signs matter only in combination, or only past some level. Test at least
   two such ideas and say which hold up.

---

*20 features. train.csv (3,035 rows) and test.csv (1,011 rows) are raw extracts; holdout_features.csv (1,002 rows) has
been checked and needs no cleaning.*
