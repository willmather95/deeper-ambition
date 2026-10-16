# deeper-ambition

Go deeper, go further, push the boundaries.

`be-more-ambitious` is an agent skill for Claude Code, Codex and any agent that reads `SKILL.md` files. Hand it a task or prompt that feels small. It hands back a rewritten version that aims higher and can still be checked.

It is built on Felix Rieseberg's essay [How to be more ambitious: A tutorial for humans with models](https://felixrieseberg.com/how-to-be-more-ambitious-a-tutorial-for-humans-with-models/). The skill is my own interpretation of his method, so read the essay for the original.

## What it does

It runs your ask through three lenses:

- Altitude means aiming at the outcome the task serves, then at a standing system that keeps producing it. "Fix this crash" becomes "keep the app crash-free".
- Depth means working the way the best expert in the field would, with raw data, their real tools and a hard number to hit.
- Range means restating the deliverable as the job it does, then borrowing from fields that already do that job, with a small crew where every role produces something.

It also does two things most rewrites skip. It checks what the model can actually access before promising anything. And it names an outside check, because a model should never grade its own work.

You get back one block you can paste straight into an agent, plus a list of what was cut and why.

## Install

Claude Code:

```bash
git clone https://github.com/willmather95/deeper-ambition.git
```

```bash
cp -r deeper-ambition/skills/be-more-ambitious ~/.claude/skills/
```

Codex: copy the same folder into `~/.codex/skills/`.

With the skills CLI:

```bash
npx skills add willmather95/deeper-ambition
```

## Use it

Say things like "be more ambitious with this", "am I thinking too small?" or "what would a top expert do here?" and paste your prompt. A full before-and-after lives in `skills/be-more-ambitious/references/worked-example.md`.

## What's inside

```
skills/be-more-ambitious/
  SKILL.md                      the workflow, checklist and output format
  references/worked-example.md  weekly ads report -> standing ad-health system
  evals/routing.md              prompts that should and should not trigger it
```
