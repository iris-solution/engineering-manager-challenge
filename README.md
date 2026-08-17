# Engineering Manager Take-Home — Submission

_Parts A, B and C, then supporting appendices and the AI disclosure. What I assumed about the system as it stands is in `NOTES.md`._

---

# Part A — 90-Day Plan

## 1. Diagnosis

Eleven incidents and a four-month launch are the signs. Underneath sit three problems — in the resourcing, the system, and the way we run — and they feed each other.

**Resource — the load sits on two individuals, and QA is at the wrong end of the line**

- S. is the only person who can review the wallet; T. is the only one who knows the pipeline, and #10–11 happened _because_ T. was away
- Two QA people cannot hand-check ~30 brands, and they only see the work at the end — so quality shows up as delay rather than as a gate

**The system — nothing in the app or the database enforces the rules**

- No guard against a repeated payment callback (#1); float maths on money (#4). Every incident was caught by a human, never by a machine
- Nothing limits what a batch job can take, and nothing tests under load: #2 drained the connection pool and froze withdrawals for six hours

**How we run — on memory rather than on anything repeatable**

- Regions are stood up by hand: #3 lost six hours on launch day to credentials copied from region 1
- No incident process ("whoever notices, fixes") and no review after region 3. Tickets, deploys and git history all exist, but nothing pulls them together and the one shared view is a burndown nobody trusts — so every decision rests on whoever tells the story best, mine included

**Two of these feed themselves:**

> **Knowledge:** S. is the only reviewer → reviews everything → runs out of hours → reviews get sharp → juniors stop asking → **he stays the only one**
>
> **Testing:** bugs found late → releases slip → batches grow → **more bugs, found later still**

**Which is why the CEO's two commitments — region 4 in 10 weeks, zero money incidents — share one cause, not two.**

- **Region 4** breaks like 2 and 3 did: hand-copied credentials (#3), four months against six weeks, no review afterwards — and launch day loads the pools that failed in #2
- **Money bugs** return like #1 and #4 did: the rule lived only in someone's head. Three of five smaller incidents came from code that _passed review_ (#5–9), and PR #482 sits on my desk carrying both again

**One fix serves both: put the rules inside the system.** Anything needing a tired person to remember it goes first under pressure.

## 2. Priorities and trade-offs

**What "zero money incidents" can honestly mean.** Not zero bugs. Stop most early, find the rest inside a day rather than waiting for a player to complain, correct balances safely. That's the version the CEO gets.

| When                             | What happens _(owner)_                                                                                                                                                                                                                                                                                                                                                                              |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Wk 0**<br>_Observe_            | • 1:1s and skip-levels<br>• **Run the region-3 review nobody ever ran**<br>• Baseline the numbers<br>• Name a region-4 launch lead<br>• Leads shape this plan, not just hear it                                                                                                                                                                                                                     |
| **Wks 1–3**<br>_Two tracks_      | **Money** _(S. + 2 mids)_<br>• Name the code that touches money<br>• Checks into the build<br>• Nightly balance check + unusual-credit alert<br><br>**Region 4** _(launch lead)_<br>• Config into Git: diff the live regions, split out secrets, refuse to start on a wrong-region value<br><br>**Me**<br>• Backup trained for T.<br>• Both roles opened — **they help next quarter, not this one** |
| **Wks 4–8**<br>_Build on it_     | • Payment integrations, D. building inside the checks _(Payments)_<br>• Region 4 stood up from Git<br>• Load and smoke tests as launch gates<br>• **Wk 6: rehearsal with real credentials** — the step region 2 skipped _(launch lead)_<br>• QA moves earlier _(L. + QA)_<br>• **Wk 4: honest replan to the CEO** _(me)_                                                                            |
| **Wks 9–10**<br>_Launch_         | • Go / no-go on the checklist<br>• Staged rollout, each stage gated on smoke<br>• Nobody on call alone                                                                                                                                                                                                                                                                                              |
| **Wks 11–13**<br>_Make it stick_ | • Review region 4 (unlike region 3)<br>• Compare to the Wk-0 baseline<br>• **Drop what didn't pay for itself**                                                                                                                                                                                                                                                                                      |

**Why config-in-Git comes first.** It's the biggest lever on the date: region 3 took four months because a region is stood up by hand, and region 2 fell over the same way (#3). Week 6 tests it — **stand a region up from the repo and the 10 weeks are real.** If not, the CEO hears it in week 4.

**The trade-off.** Weeks 1–3 spend ~30% of Payments off the region-4 plan, funded by freezing the backlog. Wrong my way, region 4 slips a week or two; wrong the other way, we lose real money in a new regulated market. I take the schedule risk.

**Explicitly not this quarter**

| Not doing                                          | Why                                                                                                             |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Reshuffle the squads, or rebuild the wallet engine | Structure isn't the problem, and the wallet doesn't get opened up mid-launch                                    |
| Clear the bug backlog                              | Frozen — money and regulator bugs only. I report the rest rather than hide it                                   |
| Get the whole team onto the wallet and pipeline    | Too big for a quarter. **Two backups for S., one for T.** — enough that a holiday no longer stops work (#10–11) |
| Automate the whole test suite                      | Money paths first                                                                                               |
| The CRM & Data roadmap                             | Pushed back — and **I** tell T.'s stakeholders, not T.                                                          |
| Hire two junior backend engineers                  | Nobody is free to mentor them                                                                                   |

## 3. Process changes

Four, and each takes something that currently depends on a person remembering and hands it to the system.

| Change                                                                                                                                                                                                      | The problem it attacks                                                           | How I'll know it's working                                                                                            |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Automatic money checks on every code change** — no rounded numbers, full-precision columns, balance and ledger written together or not at all, every read naming its brand, two approvals with one not S. | #1, #4, the 3-of-5 that slipped through review, our reliance on one person       | Run them over 90 days of merged PRs: if they'd have caught #1 and #4, that's my proof — their history, not my opinion |
| **Nightly balance check, plus a continuous alert on unusual credits**                                                                                                                                       | Today a customer tells us; "zero incidents" means nothing if we can't notice one | We plant a mismatch and the next run alerts on it; the first real one is found by the job, not support                |
| **A one-page incident process** — severity levels, one person in charge, no-blame review within 5 working days                                                                                              | Rising fix times, "whoever notices fixes it", no reviews                         | Fix times stop rising, then fall. Every serious incident produces one named, dated action that closes                 |
| **Launch readiness review** — config in Git, credentials checked, load and smoke tests, staged rollout, go/no-go                                                                                            | #2, #3, the region-3 slip                                                        | We launch in the window, no config or load incidents                                                                  |

**Testing moves earlier behind these.** Six layers run before code ships, and two on live data matter more still — because the six only catch what someone thought to test for, and PR #482 is the proof of that. **The only tool I buy is paging**: a check firing at 3am that wakes nobody is worthless. Appendices 2–3.

The rule I hold myself to: **if I can't say how I'd notice a change isn't working, I don't make it.**

## 4. Metrics

Five numbers, none needing a new tool — each falls out of a change in §3, not a dashboard:

| The number                                    | Why it earns its place                                                                | Target                                     |
| --------------------------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------ |
| **Money bugs reaching production**            | The CEO's ask, made countable. If it isn't zero, nothing else matters                 | **0**                                      |
| **Time to notice a wrong balance**            | The early warning for the above; today the honest answer is "when a player complains" | Under a day                                |
| **How often a money change breaks something** | Whether the checks work, or I've just added paperwork                                 | Baseline first — nothing collects it today |
| **Release size and lead time**                | Weekly slipping to fortnightly is a batch-size problem                                | Batch size falling                         |
| **People who can change each money path**     | The biggest structural risk, and one of the few fixable in a quarter                  | 3 wallet, 2 pipeline                       |

**SLOs, but not in week 1.** Our ~30 operators almost certainly hold contractual **SLAs**; incident #2 blocked withdrawals six hours and nobody can say whether that breached one. We run no **SLOs**, so there's no warning short of a breach. Wk 0 baselines; **wk 4** sets two off real data — _drift found within 24h_, _withdrawals 99.x%_ — each with an owner and one action when missed.

**What I refuse to measure:** velocity, story points, per-person PR counts, the burndown. They count activity, and nobody here is short of activity.

## 5. People

| Who                                                | Needs                            | What I do                                                                                                                                                                                                                                                                                               |
| -------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **S.** — only wallet expert, only real gate        | Scope change, challenge          | Written goal: **wallet from 1 person to 3**, so teaching counts as staff work. Co-owns the money checks — gatekeeping turned into leverage — and advises on region 4 without building it. I read "LGTM, nice and simple" as out of hours, not careless, and say so. Tone is a separate talk, days later |
| **D.** — wrote PR #482                             | Protection, then coaching        | They built what they were told to. Pairs with S. on the fix, then owns the money checklist: the person who got burned guards the gate                                                                                                                                                                        |
| **T.** — sole pipeline owner, on call              | Protection, urgently             | The easiest risk here to undo. I push back the CRM roadmap and tell T.'s stakeholders myself; second engineer and runbooks in six weeks; off the critical path                                                                                                                                           |
| **L.** — best people-manager, mentoring informally | Challenge, scope                 | The most under-used person here. Owns the release process and the "test earlier" work across all three squads, and co-presents at the wk-4 CEO check-in                                                                                                                                                 |
| **2 mids / 3 juniors**                             | Coaching, protection             | Mids pair with S. to learn the wallet — how one expert becomes three. Juniors own no money change alone this quarter                                                                                                                                                                                    |
| **QA (2)**                                         | Scope change, not headcount      | Two people can't hand-check ~30 brands and a third wouldn't change that. They move from _test the release_ to _make the squads able to test their own work_                                                                                                                                             |
| **Two open roles**, empty 3 months                 | A decision from me               | The role is wrong, so I reshape it rather than wait: one **senior payments engineer** — the second wallet expert, the highest-value hire here — and one **test automation engineer**. Neither moves the 10-week clock, and I say so                                                                     |
| **Me**                                             | To stop being a liability at 2am | I don't know this codebase. Wks 0–3 I read the wallet with S. — not to review PRs forever, but to make a real call on a Saturday night                                                                                                                                                                  |

---

# Part B — PR #482

## B.1 — The review, as I would post it

**Top comment**

> Requesting changes — please don't merge tomorrow. The service/route split is clean, and writing a payout row separately from the ledger entry is exactly right. But this moves real money across ~30 brands: one question before the code, then seven correctness bugs, three of them incidents #1, #2 and #4 happening again. D., almost none of this is a judgement call you got wrong — it's what our tooling should catch before a human reads the code, and fixing that is my job. Free to pair today.

**Before the code — `blocking:` has compliance signed this off, per market?** Cashback on net losses is a restricted promotion in some of our markets, and this runs across every brand at once — with no per-brand switch, no exclusion for self-excluded or flagged players, and no cap. I can't let it run until I know which markets allow it and who can switch it off. D. — this one isn't on you.

**1. `blocking:` we take `accountId` and never use it** — `cashbackService.ts:78`, and `Bet.findAll` is unfiltered too. `Member.findAll()` returns all ~400k members in this deployment, not the ~40k in the brand we're running. Ops runs it once per brand, so **everyone gets paid about ten times** and brand A's promo budget pays brand B's players — a contract problem with our operators, not just a data leak.

**2. `blocking:` money is a float in four places** — `Sequelize.FLOAT` in the migration, `DataTypes.FLOAT` in the model, `parseFloat` in the maths, `String(cashback)` on the way out. House rule is `DECIMAL(36,18)` through `dec()`. So we lose precision on save as well as in the maths, and `String()` on a small number emits `1e-7`, which the column won't take. _(incident #4, but crediting wallets rather than a report)_

**3. `blocking:` nothing stops a second run paying the same week twice.** No unique key on `(account_id, member_id, week_start)`, no check before crediting — and `weekStart = Date.now() - 7d` slides, so a 9am run and an 11am re-run cover different windows. _(incident #1)_

**4. `blocking:` the three writes aren't in one transaction, and the balance update can be lost.** A failure between them leaves the wallet credited with no ledger row, and balance must stay rebuildable from `wallet_txs`. And `balance = String(parseFloat(balance) + cashback)` reads, adds in memory, then writes — quietly wiping any bet or deposit landing mid-run.

**5. `blocking:` the failure paths are silently wrong.** `payout` is NULL on unsettled bets → `parseFloat(null)` is `NaN` → `NaN <= 0` is `false` → we write `"NaN"` into a money column, with no bet-status filter anywhere — which states should count? The `catch` only logs while the endpoint answers `{ ok: true }`, so money can leave with no record and nobody paged.

**6. `blocking:` `/admin` has no auth** — no middleware, no rate limit, no record of who triggered a run, on an endpoint that credits wallets. (If auth is global somewhere I've missed, point me at it.) I also want a **payout cap per run** and a **dry-run mode** before this touches real money.

**7. `blocking (tests):` every model is mocked, so the suite passes with all six bugs above still in place** — and `pays cashback to losing members` never checks the amount: 100 staked against 40 returned must give exactly `3.000000000000000000`. Before merge, three tests on a real database — exact balance as a string, two brands not affecting each other, the job run twice leaving the balance unchanged.

**Before the cron (not blocking):** ~400k × ~5 queries on the request thread will time out, ops will retry, and per #3 the retry pays again — incident #2 again. Make it async and chunked. One product question: is the week boundary UTC or brand-local? It changes the amounts.

_(Appendix 1 lists all 15 findings with file, line and the fix I'd take for each.)_

## B.2 — The conversation, and what I change

**S. first, privately, before the comments go up** — so this is something we agreed, not a day-9 manager overruling a staff engineer in public.

- **What I open with:** he's the only person who can review this code, so he reviews all of it, and "LGTM, nice and simple" is what reviewing looks like out of hours. That's a system I own now, not a flaw in him
- **The direct part:** _"You're our best money reviewer, and that's exactly why I want to change what we ask of you. Catching this by eye every time doesn't scale and makes you the bottleneck. Let's put these rules in the build, so your reviews go to design and teaching."_
- **Two asks:** co-own the money checks, and own getting the wallet from 1 person to 3
- **Not today:** his review tone. Piling "you're too hard on juniors" onto "you missed seven things" is how you lose a staff engineer in month one

**D. next: cover them, then hand them something.** The tooling should have caught every one of these. _And_ this — idempotency, decimals, brand filtering, transactions — is a craft I want them to own. They're blocked, I pair with them to land it, and they help write the checklist, so this ends as learning instead of a telling-off.

**What changes so this doesn't reach me again.** Me reading this PR isn't protection, it's luck.

- **Money checks in the build**, so notes 1–4 become machine comments instead of mine
- **A "done" definition for money code:** idempotency key, transaction, real-database test with exact amounts, and a runbook line on how to reverse it
- **Two approvals, one of them not S.** — that's how wallet knowledge actually spreads
- **I run the checks over 90 days of merged PRs.** If they'd have caught #1 and #4, that's my argument, and it comes from the team's own history
- **PR #482 becomes our first no-blame review** — nothing broke, so it's the cheapest place to start the habit

---

# Part C

## C1 — Saturday, 01:40

**Answer the CEO within five minutes, leading with what I've already done.** _"Confirmed — balances are wrong in region 2. I'm freezing withdrawals and payouts there now, so nothing inflated can leave. I'll have the size of it, and whether money is already gone, in 30 minutes — then every 30 minutes after."_ I don't give a CEO a number I haven't checked. I run the incident and own all comms; S. leads the technical side. My first question to him isn't "what's the bug" — it's "what's the fastest way to stop money going out?"

**Stop the bleeding first (0–15 min).** Freeze with the region flag; if there isn't one, pause the payout job and stop deploys. A blocked withdrawal we can apologise for; money that's gone, we can't. No engineer should have to make that call alone at 2am — it's mine. At the same time: check whether the same code is live in regions 1 and 3, and snapshot the data before anyone starts "fixing" it.

**Answer what the CEO actually asked.** Split "balance shows too high" from "money actually withdrawn". Only the second is a real loss; the first is limited and reversible. That difference _is_ the answer.

**Hours 1–6.** S. plus one more engineer — nobody works alone on code that writes money. T. stays on leave; if we truly need T., that's my decision and my phone call. Compliance joins by hour two — we're regulated, and I'd rather be early. Past six hours I hand over and rest S.; tired people on money code at 4am is how the second incident starts.

**Hours 6–24.** Fix balances with a reviewed, reversible script, never by hand. Reopen withdrawals only after we've checked. Written timeline throughout, and I book the no-blame review before I sleep.

## C2 — Week 6, sprint planning

**The first sixty seconds.** I don't defend the plan, win the argument, or move it to a private chat. Our strongest lead said it in the room, so it gets answered there: _"Fair question, and I'd rather you say it here than in a DM. Which items feel like theatre? Name them and I'll cut the weak ones by end of day."_ Then one line I won't move on — about content, not rank: _"The four money items — idempotency, decimals, brand filtering, transactions — are incidents #1 and #4 in disguise, so those stay. Everything else is open, and we change it out loud rather than quietly skipping it."_

**Why not overrule L.** The most respected lead here is an asset, not an opponent. Winning by rank makes an enemy and makes me look nervous. Publicly cutting the weak items earns me the right to keep the four that matter.

**Afterwards.** Same day, a 1:1 — not to correct L., but to learn what they know that I don't. They helped shape this plan in week 0, so something changed: either items were added without them, or the real subject is launch pressure. I ask. If they're right, I cut those items that week and say publicly that L. was right — the cheapest credibility I'll ever buy, and proof that pushing back is safe. If they're right about the cost but wrong about the value, the fix is automation, not persuasion: a checklist a person ticks is theatre; one the build runs is a gate.

What I never do: punish L., raise it again in a group, or go round them to their squad. You don't announce that it's safe to disagree — people learn it by watching what happens to the first person who tries.

---

# Appendices

## 1. PR #482 — all 15 findings

_Supporting detail for Part B. Every finding is in the posted review; this adds file, line and the fix I'd take._

| #   | Finding                                                                                                    | Where                       | What happens                                                                 | Fix                                                                                                      |
| --- | ---------------------------------------------------------------------------------------------------------- | --------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| 1   | `accountId` accepted but never used; `Bet.findAll` unfiltered too                                          | `cashbackService.ts:78, 86` | Every member paid ~10x; brand A's budget pays brand B's players              | `where: { accountId }` on both, plus a repository guard so it can't be left out again                    |
| 2   | `Sequelize.FLOAT` / `DataTypes.FLOAT` for money                                                            | migration, model            | Precision lost when saving, separate from the math                           | Migrate both to `DECIMAL(36,18)`                                                                         |
| 3   | `parseFloat`, `*`, `+` on money; `dec()` never used                                                        | `:92, 99, 106`              | Wrong payouts, regulatory exposure (incident #4)                             | All money math through `dec()`, strings end to end                                                       |
| 4   | `String(cashback)` gives scientific notation for small values                                              | `:106, 118`                 | `1e-7` written into a DECIMAL column                                         | Comes free with #3 — `dec().toFixed()`, never `String()`                                                 |
| 5   | No idempotency check, no unique key                                                                        | migration, service          | Pays twice on any re-run (incident #1)                                       | Unique key on `(account_id, member_id, week_start)`; insert the payout row first, inside the transaction |
| 6   | `weekStart` is a sliding timestamp, not a fixed boundary                                                   | `:76`                       | Bets near the edge counted twice or missed; no stable key for the constraint | Pass a fixed week boundary in as a parameter                                                             |
| 7   | Three writes outside a transaction                                                                         | `:106–119`                  | Wallet credited with no ledger row; a gap nobody can reconcile               | Wrap all three in one transaction                                                                        |
| 8   | Read-add-write on `wallet.balance`                                                                         | `:106`                      | A concurrent bet or deposit silently disappears                              | `UPDATE wallets SET balance = balance + :amount`                                                         |
| 9   | `parseFloat(bet.payout)` where payout is NULL                                                              | `:92`                       | `NaN` spreads; `"NaN"` written into a money column                           | Filter to settled bets before summing                                                                    |
| 10  | No bet status filter (void / cancelled / unsettled)                                                        | `:86`                       | Unknown — needs a product answer, not a guess                                | Ask product which states count, then filter explicitly                                                   |
| 11  | `catch` logs and carries on; `ok: true` regardless                                                         | `:122`, `admin.ts:149`      | Silent partial failure; money out with no record                             | Fail the run, structured log, alert                                                                      |
| 12  | `/admin` mounted with no auth, rate limit or audit log                                                     | `admin.ts`, `app.ts`        | Anyone who can reach it can credit wallets                                   | Auth middleware, rate limit, audit row per run, payout cap, dry-run mode                                 |
| 13  | Synchronous loop and a query per member, on the request thread                                             | `:83–125`                   | Timeout → retry → pays twice; drains the pool (incident #2)                  | Async job in chunks, with a status endpoint and capped concurrency                                       |
| 14  | Schema gaps — no `account_id`, no index on `(member_id, week_start)`, no unique key, no FK to `wallet_txs` | migration                   | Can't filter, can't enforce, can't report per brand                          | Add all four in the same migration                                                                       |
| 15  | Tests mock the whole data layer; no amount checked; no multi-brand or repeat-run case                      | `test/cashback.test.ts`     | The suite stays green with every problem above still present                 | Three integration tests on real Postgres: exact string balance, brand isolation, run-twice               |

## 2. AI in the QA and delivery workflow

_Supporting detail for Part A §3._

| Use                                                   | Why it fits here                                                                                   | The limit I put on it                                                                                                     |
| ----------------------------------------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Draft integration test cases from the money-path spec | Closes a gap two people cannot close by hand across ~30 brands                                     | A person signs off every money assertion. Tests written against current behaviour would lock today's bugs in as "correct" |
| A PR review bot that knows our money rules            | Would have flagged most of PR #482 before S. opened it, freeing senior review for design questions | Advisory only. It never counts as one of the two required reviewers                                                       |
| Fuzz and property tests on the money math             | Rounding and precision bugs are exactly what fuzzing finds and people miss                         | The reference answer comes from `dec()`, never from the code being tested                                                 |
| Run only the tests a change affects                   | Shortens releases, which is our cadence problem                                                    | The money-path suite always runs in full, no exceptions                                                                   |
| Draft the incident timeline and the review write-up   | Time is the scarcest thing at 2am, and the writing is what usually kills the review habit          | The person running the incident checks the facts before it's shared                                                       |
| Diff region-4 config against regions 1–3              | Incident #3 was a hand-copied config mistake                                                       | Schema validation is the actual gate; AI is just a second pair of eyes                                                    |

## 3. The test layers in full

_Supporting detail for Part A §3._

| Layer                                      | What it covers                                                                                       | Stops               | Runs                                |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------- | ------------------- | ----------------------------------- |
| **Unit**                                   | Money math through `dec()`, plain logic                                                              | #4                  | Every commit                        |
| **Integration** (real database, not fakes) | Exact amounts; brands can't touch each other; run twice and the balance doesn't move; rollback works | #1, #4, PR #482     | Every money PR                      |
| **Smoke**                                  | ~10 key journeys per region: login, balance, deposit, withdrawal, payout                             | #3                  | After deploy; blocks the next stage |
| **Regression**                             | Money paths automated first                                                                          | Late bugs, QA queue | Nightly, pre-release                |
| **Load / soak**                            | Batch jobs and database connections at region-4 volume, held for hours                               | #2                  | Launch gate                         |
| **Stress**                                 | Connections running out, provider timeouts, the same callback twice                                  | #1, #2              | Pre-launch, quarterly               |
| **Balance check in production**            | Does the ledger still add up to the balance?                                                         | Slow detection      | Nightly, on a replica               |
| **Unusual-credit alert**                   | Credits above a threshold, or a sudden jump in credit volume                                         | Fast-moving money   | Continuously                        |

The last two rows are the cheapest things in the whole plan — about three days of work for both (see `NOTES.md`).

## 4. Where my thinking comes from

_Jargon kept out of the answer itself; for the record, these are the ideas behind it._

| Where    | Idea                                               | Why this one                                                                                                                                                                                  |
| -------- | -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A.1      | **Layers of defence** (the "Swiss cheese" picture) | Every incident got through because there's only one layer and it's a person. It names the problem without blaming anyone in it                                                                |
| A.1      | **Feedback loops**                                 | Three of the four problems feed themselves. Fixing a loop once and walking away is wasted work                                                                                                |
| A.2–3    | **Mistake-proofing** (_poka-yoke_, from Toyota)    | The core bet of the quarter: move correctness out of people's attention and into the system, because attention is the first thing to go under pressure                                        |
| A.2      | **Theory of Constraints**                          | There are two real bottlenecks — S. and QA. Improving anywhere else buys nothing. It's also why I don't add juniors                                                                           |
| A.3      | **Test pyramid / shift left**                      | A bug costs more the later you find it, so two people at the very end is the most expensive place they could stand                                                                            |
| A.4      | **DORA metrics**                                   | Well-tested and a good fit for this failure shape. Two of the four are in §4 (change failure rate, lead time); fix time sits in §3. Deploy frequency on its own would be a vanity number here |
| A.5      | **Bus factor**                                     | The biggest structural risk in the pack, and one of the few you can both measure and improve inside one quarter                                                                               |
| B.1      | **Blocking vs non-blocking labels**                | Removes any doubt about what actually stops a merge — which matters when the reviewer is the author's new manager                                                                             |
| B.2, A.5 | **Talk about behaviour and effect, not character** | Keeps the S. and D. conversations about what happened and what it caused                                                                                                                      |
| B.2, A.3 | **No-blame reviews after failures**                | This team has never turned an event into a change. PR #482 is a free first try, because nothing actually broke                                                                                |
| C1       | **One person runs the incident**                   | It answers the constraint in the question directly: I don't know the code, so my value is coordination and communication, not debugging                                                       |
| C1       | **Prefer the reversible cost**                     | When you don't know yet, take the loss you can undo over the one you can't                                                                                                                    |
| C2       | **Safety to speak up**                             | What the room learns from L.'s challenge matters more than the checklist. The line I hold is about skipping quietly, not about disagreeing                                                    |

---

# AI usage disclosure

_As required by the rules._

**What I used an AI assistant for**

- Reading `cashback-payout.diff` and walking me through each method, so I was working from a clear picture of what the code actually does
- Turning my notes and decisions into this document — structure, wording, and keeping it inside the page limits

**What is mine**

- The diagnosis: which patterns sit under the eleven incidents, and why three of them get worse on their own
- Judging which of those code behaviours are real defects, how severe each one is, and which of them block a merge
- The strategy, the sequencing, the trade-offs, the metrics and the people decisions

The assistant helped me read faster and write tighter. It did not decide what the problem is or what to do about it. I own every recommendation here and can defend each one.

Separately, §3 of Part A proposes using AI _inside_ the team's workflow. Limits are written next to each use rather than treated as free capacity: on money code an AI check can inform a person, but never replaces a check the database enforces. The full list is in Appendix 2.
