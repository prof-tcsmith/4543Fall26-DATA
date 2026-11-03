# Scenario 04: Payroll Portal Account Takeover

## Company Background

Corvanne Payroll Services runs payroll for about 1,150 mid-size employers, each with 50 to 1,000 staff. Together they
employ about 380,000 people, most of them paid every two weeks. Corvanne has about 900 staff of its own and is based in
Columbus, Ohio.

Every employee gets a login to Corvanne's self-service portal. There they view payslips, download tax forms, choose the
bank account their pay goes into, request time off and update their address. Managers also use it to approve time off
and run reports for their teams. Employees sign in through a web browser, Corvanne's mobile app, their employer's own
company login (called single sign-on), or an outside app they have linked to the portal.

Over the past two years, a fraud called **account takeover** has grown fast. An attacker gets hold of an employee's
sign-in details, often through a fake "your payslip is ready" email or a password the employee reused on another site.
The attacker signs in as the employee, changes the bank details, and the next paycheck lands in an account the attacker
controls. The employee finds out only when the pay does not arrive. Corvanne's contracts promise to make the employee
whole when a takeover came through its portal.

## Business Problem

Every portal session that saves a change to bank details, or that trips one of the portal's basic alert rules, goes into
a review queue. A session is one visit to the portal, from sign-in until it ends. About 300 sessions a week land in the
queue. Today the security review team works the queue in the order sessions arrive and gets through about 45 a week. The
rest are released without review before the next payroll run.

Corvanne wants a model that scores every session in the queue, so the team reviews the riskiest ones first. In a review,
an analyst checks the session's record and phones the employee at the number on file. While the review is open, any
bank-detail change from that session is held, and pay keeps going to the old account until the employee confirms the
change by phone. If the session turns out to be a takeover, the change is cancelled and the account is locked until the
employee sets a new password.

**Capacity.** Two analysts share the reviews alongside their other security work. Together they can finish about **45
reviews a week**. The 1,212 rows in test.csv are about four weeks of the queue, so about **180 reviews** for the test
file. An outside fraud-review firm can take extra reviews at $140 each, with no weekly limit, but its staff do not know
Corvanne's clients. Capacity is a business constraint, not part of the scored cost: your recommendation must say how
your threshold fits it.

## Stakeholders

- **Imogen Thackeray, Chief Information Security Officer:** Wants to stop as many takeovers as possible. She would send
  every session the model rates risky to review, and pay the outside firm for whatever the team cannot finish. "A
  takeover we miss is an employee who cannot pay rent this month."
- **Rafael Quintero, Director of Payroll Operations:** Hears from employers' HR teams every time a real employee's new
  bank account is held. Some of those employees have already closed the old account and get paid late. He wants holds
  kept to the sessions the team is nearly sure about.
- **Mei-Ling Cho, Security Review Team Lead:** Runs the two analysts. She will not let the queue grow past what her team
  can finish before payroll. Her analysts want to see why a session ranks high before they phone an employee, and she
  doubts the outside firm's reviews will be as careful as her team's.
- **Gareth Pemberton, Chief Financial Officer:** Takeover reimbursements and incident costs passed $3 million last year. He
  wants to know what a model saves compared with today's first-come, first-served queue, and whether outside reviews at
  $140 each pay for themselves.
- **Fatima Haddad, Director of Client Relations:** Worries about whose pay gets held. Employees who share a family
  computer or use an older phone may look unusual to the portal. If the model holds their pay more often, the cost falls
  on people who are often among the lowest paid, and their employers will notice.

## Cost Structure

The figures come from Corvanne's finance team.

- **A review** takes an analyst about 45 minutes, including the call to the employee, at $100 an hour including
  benefits and overhead: $75.
- **A false alarm** (a review of a session that was the employee's own) wastes that review. The hold also delays pay
  into the employee's new account. In about one false alarm in four, the old account is already closed, and Corvanne
  must make a special payment outside the normal payroll run, which costs $180. That averages $45 per false alarm. A
  false alarm costs $75 + $45 = **$120**.
- **A missed takeover** sends the next paycheck to the attacker. A takeover session that changes nothing costs about
  the same: the attacker has the password and comes back to change the bank details, or takes tax forms. The average
  net paycheck in the queue is $2,250. The receiving bank returns about one fifth of stolen pay, so Corvanne's
  reimbursement costs $1,800. Investigating, sending a replacement payment and reporting to the employer add $300. A
  miss costs **$2,100**.
- **A caught takeover** is not free. The review costs $75. About one catch in twenty comes too late, after the payroll
  file has already gone to the bank, so 5% of $1,800 ($90) is lost anyway. A catch costs $165.
- The false-negative cost is the extra cost of a miss compared with a catch: $2,100 - ($75 + $90) = **$1,935**.

| Prediction | Actual | Outcome | Cost |
|---|---|---|---|
| Release without review (0) | Employee's own session (0) | True Negative | Not counted (normal business) |
| Review and hold (1) | Takeover (1) | True Positive | Not counted (the change is cancelled and the account locked before payday) |
| Review and hold (1) | Employee's own session (0) | **False Positive** | **$120** (a wasted review, and some employees paid late) |
| Release without review (0) | Takeover (1) | **False Negative** | **$1,935** (stolen pay reimbursed and the incident handled, less what a catch costs) |

**The cost used to score your model:** (false positives × $120) + (false negatives × $1,935). True positives and true
negatives add nothing.

## Target Variable

- `target = 1`: The session was later confirmed as an account takeover: someone other than the employee signed in with
  the employee's details.
- `target = 0`: The session was the employee's own; no takeover was found.

**Class balance:** About 11% of the rows in train.csv have `target = 1`.

## Features

All features are known at the time of the decision. A session's **network** is the block of internet addresses (IP
addresses) one internet provider, office or mobile carrier uses; many people can sign in from the same network.

| Feature | Description |
|---------|-------------|
| `avg_request_size_bytes` | Average size of the requests sent to the portal from this session's network in the hour before the session began, in bytes (100 to 50,000) |
| `session_duration_sec` | Time from the session's first request to its last, in seconds (0.1 to 282) |
| `num_requests` | Number of requests the session's browser or app sent to the portal (27 to 85) |
| `bytes_uploaded` | Data sent to the portal during the session, such as form entries and uploaded documents, in bytes (100 to 500,000) |
| `bytes_downloaded` | Data the portal sent back in reports and exports, in bytes; pages, payslips and tax forms are not counted (100 to 500,000) |
| `client_type` | How the session signed in. One of "web browser" (the portal's sign-in page), "mobile app" (Corvanne's app), "API" (an outside app the employee linked, such as a budgeting app), "single sign-on" (the employer's own company login) |
| `portal_section` | The section the session opened first after sign-in. One of "dashboard", "payslips", "tax forms", "bank details", "benefits", "time off", "profile", "documents", "messages", "reports", "team admin", "help" |
| `session_end` | How the session ended. One of "logged out" (the user signed out), "timed out" (closed by the portal after a spell with no activity), "browser closed" (the window or app closed without signing out), "forced logout" (the portal ended the session itself), "error" (a technical fault ended it), "still open" (not yet ended when the record was taken) |
| `from_new_device` | 1 = the portal did not recognize the device from this account's earlier sign-ins, 0 = a device this account has used before |
| `supported_client` | 1 = the session's software identified itself as a current, supported version (browser, app or linked app), 0 = an outdated, unknown or absent identification |
| `failed_logins_before` | Failed sign-in attempts on this account in the hour before the session began (0 to 4) |
| `temp_password_login` | 1 = the session signed in with a temporary password from a password reset, 0 = with the employee's regular password |
| `bank_detail_changes` | Number of times the session saved a change on the bank-details form, such as an account number or how pay is split between accounts (0 to 9) |
| `payslip_downloads` | Number of payslips downloaded in the session (0 to 4) |
| `sessions_same_ip_1h` | Number of portal sessions from the same network in the hour before this session began (2 to 26) |
| `timeout_share` | Of the portal sessions from the same network in the 24 hours before this session began, the share that ended by timing out (0.00 to 0.63) |
| `blocked_share` | Of the portal sessions from the same network in the 24 hours before this session began, the share that the portal blocked at a security check (0.00 to 0.51) |
| `same_section_share` | Of the portal sessions from the same network in the 24 hours before this session began, the share that opened the same portal section first as this session (0.14 to 1.00) |
| `share_other_accounts` | Of the portal sessions from the same network in the 24 hours before this session began, the share that signed in to a different employee's account from this session's (0.00 to 0.78) |
| `accounts_tried_from_ip` | Number of different employee accounts that sessions from this network signed in to, or tried to, in the 24 hours before this session began (10 to 57) |

## Questions for Your Analysis

1. Gareth wants to know what a model is worth. Today the team reviews sessions in the order they arrive. How would you
   estimate the cost of that practice on the test set, and how does your model at your threshold compare, in dollars a
   week?
2. Your cost-best threshold may flag more sessions than the team can review. How many a week does it flag? Compare
   reviewing only what the team can finish, sending the overflow to the outside firm at $140 a review, and any other plan
   you think sensible. What do you recommend to Imogen and Gareth, and why?
3. Rafael wants holds only when the team is nearly sure; Imogen wants every risky session reviewed. Choose a threshold
   that fits each view on out-of-fold probabilities (cross-validation on the training rows; the test set only reports
   your final model, once), and put each in dollars a week. What does Rafael's position cost, and what does it buy?
4. Mei-Ling's analysts believe some warning signs matter only in combination, or only past some level. Test at least two
   such ideas and say which hold up.
5. Fatima worries that employees on shared computers or older phones will have their pay held more often. Which of your
   features could have that effect? Does each one earn its place in the model, and what would you tell her?
6. Once the model is live, attackers will learn which sessions get held. Which of your model's inputs could an attacker
   change cheaply, and which could they not? What should Corvanne watch to know whether the model still works next year?

---

*20 features. train.csv (7,676 rows) and test.csv (1,212 rows) are raw extracts; holdout_features.csv (1,200 rows) has
been checked and needs no cleaning.*
