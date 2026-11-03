# Scenario 15: Catching Risky Software Releases Before They Reach Clinics

## Company Background

**Quarry Labs** is a software-as-a-service company founded in 2014. Software-as-a-service means customers pay a monthly subscription and use the software in a web browser; there is nothing to install. Quarry Labs sells appointment-scheduling software to about 2,600 outpatient clinics: physical therapy practices, dental offices, dermatology and family medicine clinics. A clinic pays $420 a month on average. Front-desk staff use the software all day to book, move and check in patients, and patients get their appointment reminder texts and emails from it.

The software is built by about 110 engineers in 14 teams. They do not save up changes for one big release. Instead, each team puts small batches of changes live several times a week. Putting a new version of the software live for every clinic is called a **deployment**, and Quarry Labs makes about **60 deployments a week**. Before a deployment goes live, at least one other engineer reads the change and approves it (a **code review**), and **automated tests** (small programs that check the software still does what it should) run by themselves every time the code changes.

These checks are not perfect. Over the last 19 months, a little under one deployment in five caused a **production incident**: something broke for real clinics, and the deployment had to be rolled back (undone by switching back to the previous version) or clinics saw errors or an outage. Most incidents are small and fixed within an hour. Some are not: on one Monday morning last spring, 300 clinics could not book patients for 40 minutes.

## Business Problem

Quarry Labs wants to **hold** the riskiest deployments before they go live. A held deployment gets two extra steps:

1. **Senior review:** one of the company's staff engineers (its most experienced engineers) reviews the change again, looking for what the first review missed.
2. **Staged ("canary") rollout:** the new version goes to 5% of clinics first and is watched for four hours before it goes to everyone. The name comes from the canaries miners once carried to warn them of bad air. If something breaks, only a few clinics see it, and the change is pulled back before the rest do.

Holds are not free, and there are not many of them. **Four staff engineers share the release review rotation, and between them they can take at most 12 held deployments a week** (three each; each hold takes about three hours of a staff engineer's time for the senior review, while an on-call engineer watches the staged rollout). With about 60 deployments a week, that is at most one deployment in five. The 1,000 or so deployments in test.csv are about 17 weeks of work, so the rotation could hold about 200 of them. Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

Deployments arrive one at a time through the week, not as a list to rank at the end of it. Today, a team lead asks for a hold when a change makes them uneasy, and the rotation takes what it is sent: about one deployment in nine. Incidents keep happening anyway, and the reviewers suspect they are holding some of the wrong ones.

The Chief Technology Officer has asked for a model that scores each deployment **before it goes live**, using only what is known at that moment, and a clear rule for which deployments to hold.

## Stakeholders

- **Renata Alvarez, Chief Technology Officer:** sponsors the project and will choose the policy. She wants fewer incidents without slowing the company down.
- **Ingrid Solberg, Head of Reliability:** measured on incidents per month and on the 99.9% uptime that Quarry Labs promises clinics. She thinks the $5,000 incident cost below is too low, because clinics hit by repeated outages leave: her estimate is **$9,000 per incident** once lost clinics are counted. She wants every one of the 12 weekly holds used, and is asking for a fifth staff engineer on the rotation.
- **Desmond Pryce, VP of Product:** measured on shipping the features clinics were promised, on the date they were promised. He thinks the $440 delay cost below is far too low, because a held release can miss a sales demo or a promised date: his estimate makes a hold cost **$2,000**, not $1,000. He wants **no more than one deployment in ten** held.
- **Elena Ortiz, Staff Engineer, leads the release review rotation:** cannot go above 12 holds a week. She warns that when most held deployments turn out to be fine, reviewers start to skim, and the holds that matter get less attention.
- **Hannah Lowe, Director of Engineering:** responsible for hiring and training. She worries that a model that holds deployments partly because of who wrote them will make some engineers feel singled out and watched. She wants to know what the model would lose without `author_months_at_company`.
- **Jordan Kim, CFO:** wants the answer in dollars: how much the model saves compared with holding nothing, and what three more holds a week (a fifth staff engineer on the rotation) would be worth.

## Cost Structure

**What a hold costs ($1,000):**

- Senior review: 3 hours of a staff engineer at $120 an hour = $360
- Staged rollout: about 2 hours of an on-call engineer's attention (the engineer on duty to respond to problems) at $100 an hour = $200
- One-day delay: the change reaches clinics one business day later. The product team's agreed estimate of a one-day slip (sales and customer-success time spent on clinics waiting for a promised change) = $440

**What an incident costs ($5,000):**

- Engineering response: on-call engineers find the problem, roll back, fix it and write up what happened; about 20 engineer-hours at $100 an hour = $2,000
- Service credits: refunds owed to affected clinics under the 99.9% uptime promise, $1,800 on average
- Support load: extra calls and tickets from clinic front desks, $900
- Customer-success follow-up calls with affected clinic managers, $300

**When a held deployment would have caused an incident:** the hold still costs $1,000. The senior review and the staged rollout stop about 4 in 5 of these problems before they reach most clinics, and fixing them then costs about $250 of engineering time. The other 1 in 5 still cause an incident, but only for the 5% of clinics in the staged rollout, at about $1,500. So a caught incident costs $1,000 + 0.8 × $250 + 0.2 × $1,500 = $1,500, and the extra cost of a miss over a catch is $5,000 − $1,500 = $3,500.

| Prediction | Actual | Outcome | Cost |
| --- | --- | --- | --- |
| Ship normally (0) | Fine (0) | True Negative | Not counted (the deployment goes live as usual) |
| Hold (1) | Would have caused an incident (1) | True Positive | Not counted (the problem is mostly caught before it reaches most clinics) |
| Hold (1) | Would have been fine (0) | **False Positive** | **$1,000** (senior review, rollout watch and a day's delay, for nothing) |
| Ship normally (0) | Causes an incident (1) | **False Negative** | **$3,500** (the extra cost of a missed incident over a held one: $5,000 − $1,500) |

**The cost used to score your model:** (false positives × $1,000) + (false negatives × $3,500). True positives and true negatives add nothing.

## Target Variable

- `target = 1`: The deployment caused a production incident: within 48 hours of going live it was rolled back, or clinics saw errors or an outage traced to it.
- `target = 0`: No incident was traced to the deployment within 48 hours.

**Class balance:** About 18% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision. Each row is one deployment, as recorded before it went live.

| Feature | Description |
| --- | --- |
| `service_area` | Part of the product the deployment changes (text): `scheduling` (booking and the appointment calendar), `billing` (invoices and card payments), `messaging` (reminder texts and emails to patients), `reports` (clinic dashboards and exports), `integrations` (links to other systems such as health-record and insurance software), `settings` (clinic account and user settings) |
| `change_type` | Kind of change (text): `new_feature` (adds something new), `bug_fix` (corrects something that was wrong), `refactor` (reorganizes existing code without changing what it does), `config` (changes settings files only, no program logic) |
| `is_hotfix` | 1 = an urgent fix shipped outside the normal schedule, usually for a problem clinics already have; 0 = a normal deployment. Only hotfixes go live at weekends or outside 7:00 to 21:59 |
| `lines_changed` | Lines of code added plus lines removed (1 to about 5,800) |
| `files_changed` | Number of code files changed (1 to 121) |
| `commits_in_deploy` | Number of commits (separately saved changes) bundled into the deployment (1 to 19) |
| `touches_db_schema` | 1 = changes the structure of the database (the schema: it adds, removes or alters tables or columns); 0 = does not |
| `feature_flagged` | 1 = the new code sits behind an on/off switch (a "feature flag") that can turn it off without another deployment; 0 = no switch |
| `customer_facing` | 1 = clinic staff or patients will see something change on screen; 0 = a behind-the-scenes change |
| `dependencies_upgraded` | Number of outside software libraries (code written by other companies or open-source projects that Quarry Labs uses) upgraded to a new version in this deployment (0 to 5) |
| `test_coverage_pct` | Percentage of the changed lines that the automated tests actually run (0.6 to 97, one decimal) |
| `test_reruns` | Number of times the automated test run had to be restarted before it passed (0 = passed the first time; up to 4) |
| `build_minutes` | Minutes the automated build-and-test run took (3.0 to 26.2, one decimal) |
| `num_reviewers` | Number of engineers who approved the code review (1 to 4) |
| `review_hours` | Hours from asking for a code review to receiving approval (one decimal, 0.1 to about 345; a few waited more than a week) |
| `author_months_at_company` | Months the engineer who wrote the change had worked at Quarry Labs (1 to 128) |
| `deploy_day` | Day of the week the deployment went live (text): `Monday`, `Tuesday`, `Wednesday`, `Thursday`, `Friday`, `Saturday`, `Sunday` |
| `deploy_hour` | Hour of the day the deployment went live, 24-hour clock, company time (0 to 23; for example, 14 means 2:00 to 2:59 p.m.) |
| `other_deploys_same_day` | Number of other deployments that had already gone live earlier the same day (0 to 21) |
| `days_since_last_incident` | Days since the last major incident in the same service area (one that set off a company-wide alert; most incidents do not); 0 to 680, where 0 = a major incident earlier the same day |

## Questions for Your Analysis

1. The rotation can take at most 12 holds a week, and deployments arrive one at a time. How many holds a week would your cost-best threshold want? How would you turn your model's scores into a rule that holds about the right number each week, and what would three more holds a week be worth to Jordan?
2. Ingrid says a missed incident costs $9,000, not $5,000. Desmond says a hold costs $2,000, not $1,000, and wants no more than one deployment in ten held. Does your recommended rule change under either figure? Which recommendation holds up under both, and what would you tell each of them?
3. `author_months_at_company` is known before every deployment. What, if anything, does the model gain from it, and what would leaving it out cost the company? How would you explain your choice to Hannah, and to an engineer whose deployment was held?
4. A hotfix usually fixes a problem clinics already have. Holding it for a senior review and a four-hour staged rollout delays the fix, a cost the table does not include. Should hotfixes be scored like any other deployment, held only in extreme cases, or never held? What does your choice cost?
5. An engineer whose deployment is held will ask "why mine?", and Elena's reviewers want to know where to look. Of the deployments your rule holds, how many would have been fine, and what would you tell Elena about keeping reviewers alert? Would you accept a slightly costlier model whose holds come with reasons engineers can act on?
6. Elena's reviewers believe some warning signs matter only in combination, or only past some level. Test at least two such ideas and say which hold up.

---

*20 features. train.csv (3,030 rows) and test.csv (1,010 rows) are raw extracts; holdout_features.csv (1,000 rows) has been checked and needs no cleaning.*
