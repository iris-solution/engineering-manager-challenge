# NOTES — what I assumed about the system as it is today

_Companion to `iris-solution.md`. The pack is silent on several things I had to take a view on. Each assumption below is stated so it can be corrected — if any of them is wrong, the part of the plan that rests on it changes._

**What the business is.** The pack never labels it, but `bets` with a stake and a payout, member wallets, third-party deposits, ~30 client brands and one deployment per region all say the same thing: B2B iGaming. We run the platform, the operators are our customers, and regions exist because licences are granted per jurisdiction.

**Licensing is not a problem I'm solving.** I assume it's handled in every region, region 4 included. My scope is the engineering side, where two requirements land as technical work:

- **Balances must be reconcilable on demand** — hence the nightly check
- **Promotions are restricted per market** — so the platform needs a per-brand switch. PR #482 has none, which is the first thing I ask about in the review

**Infrastructure.** Not described in the pack. Assumed:

- Managed cloud; one deployment and one managed Postgres per region
- A small **fixed** number of app instances, no real autoscaling — incident #2 is what fixed capacity against a fixed connection limit looks like
- Provisioning still partly by hand — incident #3, and region 3 taking four months
- No platform or SRE role in the roster, so whatever runs must be simple enough for the Payments squad part-time
- Nothing in the plan depends on which cloud. Two things follow from the shape of it: config-in-Git is the biggest lever on the region-4 date, and every background job needs its own connection budget
- **No autoscaling this quarter** — more app instances against an unchanged database limit makes #2 more likely, not less

**The nightly balance check.**

- A small CLI command in the repo, run by each region's existing scheduler
- **Not** inside the Express service and **not** inside Postgres — a periodic scan sharing the request pool is how #2 happened
- Own pool of 2–3 connections, reads a **replica**. One pass groups `wallet_txs` by wallet against `wallets.balance`; ~400k wallets is a big scan, which is why it's nightly. Needs an index on `wallet_txs(wallet_id)`
- Beside it, a cheap continuous query watches for unusual credits — that one is the tripwire, not the reconciliation
- **~3 person-days** for both: query, CLI, alert hook, scheduler config per region, and a test that plants a mismatch and checks the alert fires

**Smaller assumptions.**

- `wallet_txs` is the real record, `wallets.balance` a fast copy — that's what makes the missing-transaction comment in my PR review a correctness bug rather than a style opinion
- ~30 brands / 3 regions → ~10 brands and ~400k members per deployment, which is how I sized the unfiltered-query bug in my PR review. If brands are copied into every region, the multiplier is worse
- No global auth middleware — inferred from the diff, and flagged in the review so it can be corrected
- Ops calls the cashback endpoint once per brand, as the PR description says
- Bets can be unsettled, void or cancelled — the pack is silent, so the review asks which states should count rather than guessing
- "Money incident" = any incident producing a wrong balance, payout or regulatory report
- 10 weeks is an estimate, not a fact — hence the week-4 replan
- One open backend role can move to a test automation engineer without new budget

**Lengths.** C1 is 299 words and C2 is 298 — both inside the 300-word limit. Part A is 1,780 words and Part B a 645-word review plus a 340-word answer, but roughly 1,100 of Part A's words sit inside tables, which print about twice as dense as prose. Appendices and this file sit outside the limits.
