# Seed

**Share what you built without sharing the code.**

A *seed* is a folder you hand to someone else's coding agent in place of your code. It holds
the idea, the context, the decisions and why they were made, what the software must and must
never do, real examples, and checks that prove a rebuild is right. It leaves out your
implementation, your stack and anything private. Their agent grows its own version, in
their own stack, and runs your checks against it.

> Same seed, different soil, different plant, but you can check it's the right species.

## Why a seed

There are three ways to pass on something you've built. Plants have the same three.

| You could share… | Like handing over… | What goes wrong |
|---|---|---|
| **The idea** (a blog post, a demo) | a description of the plant | It's easy to admire and easy to get wrong. Nothing pins down what it must be, so every attempt grows into something different. |
| **The code** | a cutting | It's an exact clone of your plant, but it only thrives in soil like yours: your stack, your conventions, your assumptions. It also gives away everything. |
| **A seed** | a seed | It carries what makes it *this* plant and nothing about how yours happened to grow. |

The parts of a seed map onto the metaphor:

- **The genetics are the commitments.** They cover what it must do, what it must never do, and
  the decisions that make it this thing and not a lookalike (`README.md`, `COMMITMENTS.md`).
- **The soil is the receiver's world.** That means their stack, platform and constraints. The
  seed asks about the soil before growing (profile questions) and doesn't prescribe it.
- **The plant is the code.** Each receiver grows their own. The code will look different
  every time, and that's fine.
- **The species check is the checks.** However it grew, you can test that it's the right
  species (`checks/`). A pass is necessary, not sufficient: a healthy plant still needs
  someone to look at it (`CHECKLIST.md`, `quality.md`).
- **What stays behind is yours.** Your plant, your garden and your trade secrets. A seed
  carries only what it needs to grow.

This repo holds one skill, `seed`, which builds a seed from a codebase. It's a standard
[Agent Skills](https://agentskills.io) `SKILL.md`, so it works in any agent that supports the
format: Claude Code, Codex, Gemini CLI, Cursor, GitHub Copilot, and others.

## Install

This is a private repo, so your git access must be able to read it.

**Any agent** — the skill is just the folder `skills/seed/`. Copy or symlink it into your
agent's skills directory:

| Agent | Personal | Per project |
|---|---|---|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Codex, Gemini CLI, Copilot, Cursor and others (shared path) | `~/.agents/skills/` | `.agents/skills/` |

```sh
git clone git@github.com:SerovaAI/seed.git ~/src/seed
ln -s ~/src/seed/skills/seed ~/.agents/skills/seed     # or ~/.claude/skills/seed
```

Check your agent's docs if it uses a different path. The
[`skills` CLI](https://github.com/vercel-labs/skills) can also install it into most agents:
`npx skills add SerovaAI/seed`.

**Claude Code as a plugin** — instead of copying:

```
/plugin marketplace add SerovaAI/seed
/plugin install seed@serova
```

## Make a seed (the author)

Open your agent in the repo you want to share and say "make a seed of this project" (or
`/seed` in agents that expose skills as commands). The skill:

1. **Reads the project first:** README, docs, tests, code, PR history, design and review
   docs. It works out the idea, the core, the history and what looks private.
2. **Asks only what it can't tell:** only questions whose answers change the seed, e.g. how much to disclose, or
   where your docs and code disagree. Each question offers its own best guess as the default.
3. **Writes `seed/`** in your repo. Everything for you sits at the top; the seed itself is
   `seed/publish/`, releasable as-is.
   ```
   seed/
     REVIEW.md        for you: what the seed reveals, what to confirm, what it found in your project
     adapter/         wraps your code so you can re-run the checks as the project changes
     sources.md       each example and vector, mapped back to the test it came from
     publish/         the seed — this becomes the new repo
       README.md        start here: idea, principles, decisions and why, profile questions, build stages
       COMMITMENTS.md   interfaces, data shapes, invariants, must / must-never
       examples/        concrete behaviour, drawn from your tests and real runs
       checks/          vectors plus a runner that works with any build, a quality rubric, a manual checklist
   ```
4. **Self-checks:** your original code must pass its own seed. Any failure is either a wrong
   rule in the seed, an accepted exception, or a bug in your project, and it tells you which.
5. **Reviews disclosure, then offers to publish:** it scans for secrets and personal data
   and writes `seed/REVIEW.md` — what the seed reveals, what was left out, what you need to
   confirm, and anything it found in your project along the way. Only after you've read that
   does it offer to create the GitHub repo from `seed/publish/` (private by default) and push.

`seed/publish/` is releasable as it stands — nothing in it needs stripping. `seed/` as a
whole is yours to commit or not; nothing in it is unsafe in your own repo.

Expect a real run to take a while and use a fair number of tokens. On a mid-sized project it
fans out several subagents.

## Grow from a seed (the receiver)

You don't need this plugin. Clone the seed repo, open any coding agent in it, and say:

```
Build this from the README. Ask me the profile questions first.
```

The README tells the agent to:
- read the whole folder;
- ask you the setup questions (platform, backend, and so on), with the author's choices as
  defaults;
- build in stages, running `checks/` after each one;
- never look for the original.

Checking a build works the same way for every seed. The agent writes a small **adapter** —
glue around its own code that reads one test case as JSON on stdin and writes the result on
stdout, described in `checks/ADAPTER.md` — and runs
`python3 checks/run.py --adapter "<your command>"`. Ops it hasn't built yet are skipped, not
failed. Two parts of `checks/` aren't automated: `quality.md` is a rubric for a person or a
model to grade real output against, and `CHECKLIST.md` is ticked by hand against the running
build. A pass means the build hasn't deviated on anything the checks can see; it doesn't mean
the build is finished.

## Status

Early. It has been tried on one real project, a mobile word-puzzle game, where the original
passed its own 30 engine checks and its 658 shipped levels passed the content checks. **A
blind rebuild from a seed has not been tested yet.** That's what this trial is for.

Feedback wanted, especially from both sides:
- **Author:** did it ask the right questions? Is anything in the seed you'd rather not share?
- **Receiver:** what was missing when you built from it? Where did the checks catch you, and
  where did they miss?
