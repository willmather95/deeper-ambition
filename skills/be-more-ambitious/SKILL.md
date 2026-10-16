---
name: be-more-ambitious
description: 'Rewrite a task, prompt or recurring chore into a deeper, higher-reaching version that can still be verified, using the Altitude, Depth and Range lenses from Felix Rieseberg''s essay "How to be more ambitious: A tutorial for humans with models". Use when the user says "be more ambitious", "aim higher", "am I thinking too small", "level this up", "what would a top expert do here", "10x this ask", or hands over a prompt or repeating task that feels small for what a model can do. Differentiator - upgrades the ask before work starts; it does not do the work, beat a named reference, or design a long-run loss function.'
---

# Be More Ambitious

Turn a small ask into an ambitious one that a model can still finish and an outside check can still verify. Method credit: Felix Rieseberg, "How to be more ambitious: A tutorial for humans with models", https://felixrieseberg.com/how-to-be-more-ambitious-a-tutorial-for-humans-with-models/. This skill is an interpretation of the essay, not its text. Lines marked *(skill addition)* are not from the essay.

Speed (same work, faster) is assumed. This skill works the other three lenses:

| Lens | Question | Upgrade looks like |
|---|---|---|
| Altitude | What do I want to happen after this is done? | Fix a crash -> keep the app crash-free |
| Depth | How would the best expert in the field do it? | Markdown table -> database, model, held-out test |
| Range | Which other fields have solved this verb? | One analyst -> analyst + reporter + editor |

Aim for a deeper version of the ask, not a bigger one. More scope with the same method is not ambition.

## Workflow

Run the steps in order. Write the answers down; do not keep them in your head.

### 0. State check - what access exists now

List what the model can actually touch today: raw data, logs, tools, a machine it can install software on, connectors, test devices. Mark each item `present`, `missing`, or `unknown - ask user`. If you cannot confirm an item with a tool, mark it `unknown - ask user`. Never assume access.

### 1. Altitude - climb from task to outcome

Ask "and then what?" two or three times. Each answer is a rung.

- Rung 0: the task as asked.
- Rung 1: the outcome the task serves.
- Rung 2: a standing system that keeps producing that outcome with fewer human touches.

For each rung, write the access it needs, using the step 0 list. Pick the highest rung that an outside check can still confirm and whose access is present or can be granted. If a rung's access is missing, the first deliverable is "grant this access", not the ambitious task.

Stop climbing when no outside check could confirm the goal ("grow the business" fails this test). Rung 0 is the right answer for a true one-off; say so instead of inflating it.

Signals that a task wants more altitude: it recurs, it lives in a recurring meeting, or a human re-prompts the same fix each week.

Creative work (taste-led writing, art, brand voice) gains less from Altitude. *(skill addition)* Lean on Depth and Range there and keep a human as the judge.

### 2. Depth - work like the top expert

Answer these four before rewriting anything:

1. What would a top professional do here that the ask never mentions?
2. What is the rawest material they would start from? (logs, event tables, source files, not a summary)
3. Which instruments and methods would they use? (stats models, profilers, simulations, SQL - the model knows the tools even if the user does not)
4. What independent evidence would prove the result?

Then apply:

- **Hard, specific target.** Swap "faster" for a number that sounds a bit unreasonable (200 ms, not 5 s). If it looks impossible, push before you lower it.
- **Test the assumption.** Hand over raw data and ask the model to try to break the belief the task rests on ("is churn really an onboarding problem?").
- **Make new knowledge.** Where it fits, ask for something nobody has yet: a simulation, a leading signal, a model, validated on held-out data.

### 3. Range - borrow other fields

- **Nouns to verbs.** Restate the deliverable as the job it does. A "deck" is really a way to make a causal story credible to busy people and get a decision fast. The verb shows which fields apply. (Technique from Kelley and Littman, *The Art of Innovation*, as cited in the essay.)
- **Assemble a crew of 2-4 roles** picked from the verb. Add one adversary (a short seller, a forensic accountant, a hostile reviewer) when the result will move money, touch a production system, or make a public claim.
- **Each role does the work, not the voice.** "Research the short case the way a short seller would and list the evidence" is right. "Write like a short seller" is role-play and makes output worse.

### 4. Outside judge

Name the check that runs outside the model that did the work: a test suite, a held-out dataset, a benchmark script, a fresh-context reviewer, a real user metric. The model never grades its own output. Prefer a check the model can run itself for fast feedback.

### 5. Write the upgraded ask

Use the output format below, exactly. Keep the upgraded ask as one block the user can paste straight into an agent.

### 6. Verify the rewrite, then fix and re-check

Check the rewrite against each item. Fix any failure and check again before returning.

- [ ] The chosen rung is named, it has an outside check, and rung 0 was kept if the task is a one-off.
- [ ] Every rung's access is listed as present, missing, or unknown. Nothing unknown is treated as present.
- [ ] The target is a specific number or a pass/fail condition, not "better".
- [ ] The method changed, not just the vocabulary. Fail if the rewrite only adds expert-sounding words.
- [ ] Crew roles each produce an artifact; none is a persona voice.
- [ ] An outside judge exists and is not the model that did the work.
- [ ] Scope is something a model can finish. If not, cut a rung and say so.

## Output format

```markdown
## Original ask
(verbatim)

## Ambition read
- Altitude: rung 0 -> rung N (one line per rung)
- Depth: expert delta; raw material; instruments; evidence
- Range: verb rewrite; crew roles and what each produces
- Outside judge: the check, and who or what runs it
- Access: item - present | missing | unknown - ask user

## Upgraded ask (paste this)
(one block: outcome, raw inputs, hard target, methods, crew tasks, judge, stop condition)

## What I cut and why
(rungs or ideas dropped because no outside check could confirm them or access was missing)
```

Before your first run, or when the user asks for an example, read `references/worked-example.md` (weekly ads report upgraded to a standing ad-health system).

## Failure modes

| Looks like | Fix |
|---|---|
| Rewrite sounds expert, same steps | Redo Depth question 3; the instruments must change |
| "Act as a world-class X" | Turn it into a task the role performs, with an artifact |
| Model scores its own result | Add an external check before returning |
| Goal so high nothing can check it | Drop one rung |
| Ambitious ask, no data access | Make access the first deliverable |
| Bigger scope, same method | Go back to Depth; deeper, not bigger |

## Pairs with

When the upgraded ask has a real bar to beat (a competitor page, a benchmark), hand it to a reference-beating loop. When it turns into a long autonomous run, hand it to a loss-function or harness designer. When the premise itself looks wrong, run first-principles analysis before upgrading.
