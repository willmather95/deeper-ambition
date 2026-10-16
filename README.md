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
cp -r deeper-ambition/skills/* ~/.claude/skills/
```

Codex: copy the same folders into `~/.codex/skills/`. To take just one skill, copy only its folder.

With the skills CLI:

```bash
npx skills add willmather95/deeper-ambition
```

## Use it

Say things like "be more ambitious with this", "am I thinking too small?" or "what would a top expert do here?" and paste your prompt. A full before-and-after lives in `skills/be-more-ambitious/references/worked-example.md`.

## More skills in this repo

These are the skills I use every day, cleaned up for anyone to run.

| Skill | What it's for | Credit |
|---|---|---|
| `human-copy` | Finds and strips the patterns that make writing read as AI-generated, with a detect-only mode | Wikipedia's "Signs of AI writing" plus the studies listed in the file |
| `reason` | Settles a judgment call with no clear metric by running blind judges over competing answers until they converge | Inspired by [Udit Goenka's autoresearch](https://github.com/uditgoenka/autoresearch) |
| `predict` | Pre-mortem before you ship. Code gets a five-expert panel; plans get Gary Klein's "it already failed" exercise | [Udit Goenka](https://github.com/uditgoenka/autoresearch), [Gary Klein](https://hbr.org/2007/09/performing-a-project-premortem) |
| `contract-first-delegation` | Hands an agent an outcome with clear scope, authority, a verifier and a stop condition, then lets it pick the route | - |
| `autoresearch` | Sets a goal, a metric and a verify command, then iterates, keeping wins and reverting losses | [Karpathy](https://github.com/karpathy/autoresearch), [Udit Goenka](https://github.com/uditgoenka/autoresearch), [pi-autoresearch](https://github.com/davebcn87/pi-autoresearch) |

`reason`, `predict` and `autoresearch` start helper agents, so run them in a tool that can do that, like Claude Code. `reason` and `predict` work as a pair.

## What's inside

```
skills/
  be-more-ambitious/          workflow, worked example, routing evals
  human-copy/SKILL.md
  reason/SKILL.md
  predict/SKILL.md
  contract-first-delegation/SKILL.md
  autoresearch/SKILL.md
```
