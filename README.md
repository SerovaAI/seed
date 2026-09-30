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
  the decisions that make it this thing and not a lookalike (`SEED.md`, `COMMITMENTS.md`).
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
3. **Writes `seed/`:**
   ```
   seed/
     SEED.md          start here: idea, principles, decisions and why, profile questions, build stages
     COMMITMENTS.md   interfaces, data shapes, invariants, must / must-never
     examples/        concrete behaviour, drawn from your tests and real runs
     checks/          vectors plus a runner that works with any build, a quality rubric, a manual checklist
   ```
4. **Self-checks:** your original code must pass its own seed. Any failure is either a wrong
   rule in the seed, an accepted exception, or a bug in your project, and it tells you which.
5. **Reviews disclosure:** it scans for secrets and personal data and writes
   `seed-review.md` *next to* `seed/`, for your eyes only. The review covers what the seed
   reveals, what was left out, what you need to confirm, and anything it found in your
   project along the way.

**Share only the `seed/` folder.** Read `seed-review.md` before you do.

Expect a real run to take a while and use a fair number of tokens. On a mid-sized project it
fans out several subagents.

## Grow from a seed (the receiver)

You don't need this plugin. Put the `seed/` folder in an empty directory, open any coding
agent there, and say:

```
Build this from seed/SEED.md. Ask me the profile questions first.
```

`SEED.md` tells the agent to:
- read the whole folder;
- ask you the setup questions (platform, backend, and so on), with the author's choices as
  defaults;
- build in stages, running `checks/` after each one;
- never look for the original.

## Status

Early. It has been tried on one real project, a mobile word-puzzle game, where the original
passed its own 30 engine checks and its 658 shipped levels passed the content checks. **A
blind rebuild from a seed has not been tested yet.** That's what this trial is for.

Feedback wanted, especially from both sides:
- **Author:** did it ask the right questions? Is anything in the seed you'd rather not share?
- **Receiver:** what was missing when you built from it? Where did the checks catch you, and
  where did they miss?
