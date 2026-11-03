# Scenario 08: Choosing Which Brewery Accounts Get a Save Play

## Company Background

**Kettleford Software** makes production software for independent breweries, cideries and small distilleries. Brewers use it to plan brew days, track what is in each tank, keep stock of grain, hops and cans, work out what each recipe costs, and prepare the monthly production and tax reports the government requires. Customers pay a subscription and use the software in a web browser; there is nothing to install.

About 2,000 of these businesses are customers (Kettleford calls each one an **account**). There are three plans:

- **Starter** ($120 a month): one site, brew planning and inventory.
- **Professional** ($270 a month): adds recipe costing and lab records.
- **Enterprise** ($620 a month): several sites, custom reports and priority support.

Every plan can connect to other systems, such as accounting, taproom sales or an online shop.

The average account pays about $300 a month. An account chooses a **monthly** contract (it can cancel at the end of any month) or an **annual** contract (it commits for a year and can leave only at its renewal date).

Kettleford's growth has stalled. Last year it lost about 650 accounts and signed about 700 new ones. Small breweries are a hard market: some close, and others move to a competing product.

## Business Problem

Every account gets a **health check** twice a year, spread evenly through the year, so about 340 accounts are checked each month. Kettleford wants a model that predicts, at each health check, whether the account will **give notice** (tell Kettleford it is leaving) within the next six months, before its next check.

For the accounts it picks, a customer-success manager (a staff member whose job is to help customers get value from the software) runs a **save play**: about four hours of work (a call with the head brewer, a review of how the brewery uses the software, setting up parts it does not use yet, retraining staff) plus 20% off the next three months. Save plays are new: in a small trial last year, the managers picked the accounts by feel, from a dashboard. None of the accounts in your data got a save play.

**Kettleford's three customer-success managers can each fit about 20 save plays a month around their other work: 60 a month**, about one health check in six. The 1,012 test rows are about three months of health checks, so the team could run **about 180** save plays on them.

Capacity is a business constraint, not part of the scored cost: your recommendation must say how your threshold fits it.

The other option is a fourth customer-success manager. One would cost about $100,000 a year and add about 20 save plays a month.

## Stakeholders

- **Ines Carvalho, VP of Customer Success** — Sponsors the project and is measured on how many accounts Kettleford keeps. She believes save plays aimed by a good model will keep **half** of the accounts that were going to leave, not 4 in 10. That would make the false-negative cost $2,320 (50% × $5,400 − $380). She wants a fourth manager.
- **Malik Johnson, Chief Financial Officer** — Sees the 20% discount as money handed to breweries that were never going to leave. He thinks 18 months is too hopeful for a saved account; his figure is **12 months** ($3,600), which would make the false-negative cost $1,060 (40% × $3,600 − $380). He wants fewer save plays, not more managers.
- **Sunniva Berg, Customer-Success Team Lead** — Runs the three managers and cannot go above 60 save plays a month. Before each call, her managers need to know why the account was chosen, so they know what to talk about with the head brewer.
- **Arjun Mehta, Head of Product** — Believes most accounts leave for reasons the product could fix. He plans to use the project's results to decide which parts of the software to improve next year.
- **Yusuf Demir, Support Manager** — His team answers customers' support requests. He worries that once breweries learn that unhappy accounts are offered 20% off, some will act unhappy to get it, and that staff will be pressed to record things differently.

## Cost Structure

**What a save play costs ($380):** four hours of a manager's time at $50 an hour, including benefits ($200), plus 20% off three months of the average $300 fee ($180). On an account that was going to stay anyway, all of it is wasted: this is the false-positive cost.

**What a lost account costs ($5,400):** Finance's figure. An account that stays now stays, on average, about another 18 months: 18 × $300 = $5,400.

**A catch is not free.** Save plays keep about 4 in 10 of the accounts that were going to leave (Kettleford measured this in a small trial last year, on accounts that are not in your data). So a catch costs the save play ($380), and 6 in 10 caught accounts leave anyway (60% × $5,400 = $3,240). The false-negative cost is $5,400 − ($380 + $3,240) = **$1,780**.

| Prediction | Actual | Outcome | Cost |
|---|---|---|---|
| No save play (0) | Stays (0) | True Negative | Not counted (normal business) |
| Save play (1) | Gives notice (1) | True Positive | Not counted (a manager works the account; about 4 in 10 such accounts stay) |
| Save play (1) | Stays (0) | **False Positive** | **$380** (a manager's time and a discount spent on an account that was staying anyway) |
| No save play (0) | Gives notice (1) | **False Negative** | **$1,780** (the expected value a save play would have kept, less what it would have cost) |

**The cost used to score your model:** (false positives x $380) + (false negatives x $1,780). True positives and true negatives add nothing.

## Target Variable

- `target = 1`: within six months of the health check, the account gave notice that it is leaving (a monthly account cancelled; an annual account said it would not renew).
- `target = 0`: the account did not give notice in those six months.

**Class balance:** About 18% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision.

| Feature | Description |
|---------|-------------|
| `account_age_months` | Months since the account signed up (1 to 120, one decimal) |
| `monthly_active_users` | Number of different people at the brewery who logged in during the past month (3 to 32) |
| `daily_active_users_avg` | Average number of people who logged in each day over the past month (0.1 to 20.6, one decimal) |
| `feature_adoption_score` | Percentage of the plan's features the account has used at least once (0 to about 97) |
| `num_support_tickets_last_month` | Number of support requests the account opened in the past month (0 to 7) |
| `avg_session_duration_minutes` | Average length of a login session, in minutes (0.5 to 190, one decimal) |
| `login_frequency_weekly` | Average number of login sessions per user per week (0 to 15) |
| `num_integrations_enabled` | Number of other systems connected to the software, such as accounting, taproom sales or an online shop (0 to 9) |
| `contract_type` | `annual` or `monthly` (see Company Background) |
| `plan_tier` | `Enterprise`, `Professional` or `Starter` (prices in Company Background) |
| `num_admin_users` | Number of users with administrator rights, who can change settings and add users (1 to 9) |
| `data_storage_used_pct` | Percentage of the plan's storage allowance in use: batch records, lab results, label files (0.3 to 98) |
| `num_reports_created_last_month` | Number of reports created in the past month, such as production, inventory or tax reports (0 to 19) |
| `last_login_days_ago` | Days since the account's main contact (usually the head brewer) last logged in (0 to 45.2, one decimal) |
| `nps_score` | Average answer from the account's users, in the latest customer survey, to "How likely are you to recommend Kettleford to another brewery?" (0 = not at all likely, 10 = extremely likely; 0.2 to 10, one decimal) |
| `onboarding_completed` | 1 = the account finished the guided setup program when it joined; 0 = it skipped the program or did not finish it |
| `training_sessions_attended` | Number of Kettleford's online training sessions the account's staff have attended (0 to 7) |
| `billing_issues_count` | Number of billing problems in the past 12 months, such as failed card payments or disputed invoices (0 to 4) |
| `competitor_mention_in_tickets` | 1 = a support request in the past six months mentioned a competing product; 0 = none did |
| `expansion_revenue_pct` | Percentage of the account's monthly fee that comes from add-ons or upgrades bought after it signed up (0 to about 82) |

## Questions for Your Analysis

1. Ines and Malik disagree about what a save is worth. Rerun your threshold choice with Ines's false-negative cost ($2,320) and with Malik's ($1,060). Does your recommendation change? Which recommendation holds up under both?
2. At your cost-best threshold (chosen on out-of-fold probabilities), how many of the 1,012 test accounts would get a save play, against the 180 the team can run? If your model wants more, which accounts should the 180 go to? Tell Malik whether a fourth manager, at about $100,000 a year for 20 more save plays a month, would pay for itself.
3. The scored cost treats every account alike, but an Enterprise account pays more than five times what a Starter account pays. Should the 60 save plays a month follow your model's score alone, or should the plan's price count too? Show in dollars what your choice changes on the test set, and say whether the scored cost can show that difference.
4. Yusuf's worry: once breweries, or Kettleford's own staff, learn how save plays are chosen, which columns could they change on purpose? What would that do to your model after a few months, and what would you recommend?
5. Sunniva's managers believe some warning signs matter only in combination, or only past some level. Test at least two such ideas and say which hold up.
6. Arjun wants to use your model to decide what to build next. Which columns describe things Kettleford could change through its product or its service, and which describe the brewery's own business? Can your model tell him that changing one of them would keep accounts? Explain to him what it can and cannot tell him.

---

*20 features. train.csv (3,036 rows) and test.csv (1,012 rows) are raw extracts; holdout_features.csv (1,000 rows) has been checked and needs no cleaning.*
