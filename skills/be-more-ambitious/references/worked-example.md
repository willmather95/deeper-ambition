# Worked example - weekly ads report

The numbers below are illustrative, not real account data. Working notes come first. The final answer, in the exact output format from SKILL.md, is at the end; copy that layout, not the notes.

## Working notes

### Step 0 - access

| Need | Status |
|---|---|
| Meta Ads data, daily, ad level | present (connector) |
| Store orders with UTM tags (utm_content = ad ID) | missing - store export needed |
| Two years of spend and revenue history | present (CSV archive) |
| Hand-labelled list of known bad weeks | missing - human provides |
| A place to run scheduled jobs and keep state | present (cron + repo) |

### Step 1 - altitude

- Rung 0: a table and a summary for Monday. Needs: Meta data.
- Rung 1: know which ads to scale, hold or kill this week, and why. Needs: Meta data + UTM-tagged orders.
- Rung 2: a standing system that flags a dying ad or a broken pixel within a day, so Monday's meeting only covers decisions. Needs: all of the above + scheduler + labelled bad weeks for the judge.

Rung 2 has an outside check (did it flag the bad weeks we already know about?). Chosen, with the two missing items as first deliverables. "Maximize profit" sits higher, but no check can confirm it in one cycle; cut.

Signal: this lives in a weekly meeting, so it wants altitude.

### Step 2 - depth

1. Expert delta: a strong media buyer checks Meta's reported purchases against the store's own UTM-attributed orders, separates creative fatigue from audience saturation, and ignores day-to-day noise.
2. Rawest material: ad-level daily spend, impressions, frequency and purchases, plus store orders with UTM tags.
3. Instruments: a small SQLite database, a per-ad fatigue curve (CTR against cumulative frequency), a control chart for cost per purchase, a daily Meta-vs-store reconciliation per ad.
4. Evidence: replay the last 26 weeks and check whether the system would have caught the three known bad weeks earlier than the humans did.

Hard target: flag a broken pixel within 24 hours, and catch all 3 known bad weeks in the replay with at most 1 false alarm a month.

Assumption to break: "our best ad is the one with the lowest CPA in Ads Manager." Test it against UTM-attributed store orders.

New knowledge: a fatigue threshold for this account, fitted on 18 months and checked on the last 6.

### Step 3 - range

Verb rewrite: "a report" becomes "catch money leaking from ads early and get a busy owner to act on it in five minutes."

Crew (an adversary is included because the result moves ad spend):
- Analyst - builds the database, fatigue curves and control charts. Produces `metrics.sqlite` and charts.
- Auditor (adversary) - hunts for ad-days where Meta-reported purchases and UTM-attributed orders do not reconcile. Produces a mismatch list by ad and date.
- Editor - cuts the Monday note to at most three decisions, each with its evidence. Produces a note of 150 words or fewer.

### Step 4 - outside judge

A separate replay script scores catches and false alarms against the hand-labelled bad weeks. The model that builds the system never sees the labels.

## Final answer

## Original ask
"Pull last week's Meta ad numbers into a table and write a short summary for Monday's meeting."

## Ambition read
- Altitude: rung 0 weekly table -> rung 1 scale/hold/kill calls -> rung 2 standing ad-health system that flags problems within a day
- Depth: reconcile Meta with store orders, separate fatigue from saturation; raw ad-level daily data plus UTM-tagged orders; SQLite, fatigue curves, control charts; 26-week replay against known bad weeks
- Range: "catch money leaking from ads early and get a busy owner to act in five minutes"; analyst (metrics + charts), auditor (mismatch list), editor (150-word note)
- Outside judge: separate replay script scored against hand-labelled bad weeks the builder never sees
- Access: Meta ad-level data - present; UTM-tagged store orders - missing; spend history - present; labelled bad weeks - missing; scheduler - present

## Upgraded ask (paste this)
First, get two things: a store orders export with UTM tags (utm_content = ad ID), and a hand-labelled list of known bad weeks from the owner. Nothing below is valid without them. Then build a standing ad-health system for our Meta account. Load raw ad-level daily Meta data and UTM-attributed store orders into SQLite. Fit a per-ad fatigue curve and a control chart for cost per purchase, and reconcile Meta-reported purchases with UTM-attributed orders per ad per day. Target: flag a broken pixel within 24 hours, and catch all 3 known bad weeks in a 26-week replay with at most 1 false alarm a month. Before trusting Ads Manager's top ad, test that belief against UTM-attributed orders. Run three passes: an analyst builds the metrics, an auditor lists every ad-day where Meta and store numbers do not reconcile, and an editor writes a Monday note of at most three decisions in 150 words or fewer. A separate replay script scores the system against the labelled weeks; do not grade your own results. Stop when the replay target passes or after three tuning rounds, and report which one happened.

## What I cut and why
- "Maximize profit" rung: no outside check can confirm it in one cycle.
- A creative-generation agent: useful later, but it is the work, not the ask.
