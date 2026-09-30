---
name: seed
description: Turn a codebase into a "seed" — a shareable folder that carries a project's idea, context, commitments, examples and checks, so another person's coding agent can grow their own version without ever seeing the code. Use when the user wants to share what they built without open-sourcing it, says "make a seed", "/seed", "spec this so someone else can build it", or wants a stack-agnostic handover of a project.
---

# Seed

A **seed** is what you share instead of code. It carries everything another builder needs
to grow their own version — the idea, the context, what it must and must never do, concrete
examples, and a way to check the result — and nothing that ties them to your implementation.

> Same seed, different soil, different plant — but you can check it's the right species.

Keep the metaphor in mind; it settles most judgment calls:

- **An idea is a description of the plant.** It's easy to admire and grows into anything.
- **Code is a cutting.** It's an exact clone that only thrives in the parent's soil, and it
  gives everything away.
- **A seed carries the genetics.** It carries what makes it *this* plant (the commitments)
  and leaves out how the parent happened to grow.
- **The receiver's stack is the soil.** Ask about it (profile questions); don't prescribe it.
- **The checks are the species test.** However it grew, is it the right species?

When unsure whether something belongs in the seed, ask: *is this genetics, or is it how my
plant happened to grow?*

You are running on the **author's** machine, reading their code. The code never leaves; only
the `seed/` folder does. Your job is to extract what matters and leave behind what doesn't.

## Principles

- **Describe what, not how.** No copied code, no verbatim prompts, no file-by-file tour. The
  receiver should be free to use any stack. Interface shapes may be written in neutral
  notation (JSON-ish, tables), never as source.
- **Idea and context first.** Most of the value is *why*: the problem, the principles, the
  non-goals, the decisions that were made the hard way. Prose carries this well.
- **Pin what prose can't.** Interfaces, data shapes, invariants and must-nevers go in as
  exact commitments. This is where rebuilds silently diverge.
- **Show, don't only tell.** Concrete examples (input → expected behaviour) carry things a
  spec misses. Derive them from the real code and tests, not from imagination.
- **"Good" is not "correct".** Where quality is a judgment (tone, ranking, difficulty,
  UX feel), give labelled good/bad examples and a rubric, not rules.
- **The author controls disclosure.** Nothing is final until they have reviewed what the
  seed reveals.

## Step 1 — Read the project

Gather everything you can before asking anything. The project already holds most of the
idea, context and history; the author should only be asked what it genuinely can't tell you.

Build understanding from, in rough order of value:

- README, docs, design notes, AGENTS/CLAUDE files — stated intent and principles.
- **Tests** — the author's own statement of what must hold. Mine them hard.
- The core modules — entry points, data model, state transitions, external boundaries
  (APIs, storage, LLM calls, filesystem, network).
- **The why, where it's actually written down.** Dead ends, reversals and "never do X" fixes
  are the most valuable context and never appear in code. Look in this order:
  1. PR descriptions (`gh pr list --state all` and `gh pr view`) and issues.
  2. Design, review and post-mortem docs; TODOs and changelogs.
  3. Commit messages with bodies, and large or reverting commits (`git log --stat`).
  Terse commits ("Update", "fix") carry almost nothing; don't spend effort on them beyond
  the timeline.
- Config and constants — thresholds and defaults often encode hard-won decisions.
- Build and deploy config — reveals the shell around the core (mobile wrapper, server,
  CLI) and what's incidental.

For anything larger than a few thousand lines, if your agent can run subagents or parallel
tasks, fan out read-only ones by area (otherwise work through the areas in turn), and
have them report conclusions with `file:line` evidence, not file dumps. A split that works:

- **Core rules** — the runtime behaviour, state model, invariants, and examples from tests.
- **Data and quality** — the content or data model, pipelines, and what makes output good
  or bad, with labelled real examples.
- **History and decisions** — timeline, key decisions with reasons, open problems, and
  decisions whose reason isn't recorded anywhere.
- **Shell, meta and privacy** — features around the core (core / optional stage / shell),
  what could reasonably differ (profile questions), and sensitive material by location.

### Record where docs and code disagree

Keep an explicit list of every place the docs, tests, code and history disagree (a
threshold, a level number, a rule the docs state but the code doesn't enforce). These are
the most useful findings of the whole run. For each, note what each source says. Default to
the **code** as the truth of current behaviour, unless the project says otherwise, and
carry the rest into Step 2 or the review.

### Look for sensitive material across the whole repo

Not just what might end up in the seed: committed `.env` files, admin endpoints, store and
team identifiers, deployment URLs, and **personal data committed in docs** (user reports,
support exports, logs with real comments or emails). Record locations only; never copy the
values. Anything found here is withheld from the seed and listed in the review.

From this, draft your own answers to the scoping questions:

- **What is the product?** One sentence.
- **What is the core?** Which parts the seed should regrow; which are shell or incidental
  (wrappers, admin tooling, i18n, deployment). Signals: where the tests concentrate, where
  history is densest, what the README leads with.
- **What looks private?** Prompts, model/tuning parameters, datasets, credentials,
  client or internal-system names, anything commercially distinctive.
- **Who is it for?** Default to a capable peer outside the author's team.

Record each answer with its evidence and a confidence level.

## Step 2 — Ask only about the gaps

Show the author a short summary of what you found (product, core vs. shell, proposed
private list) so they can correct it at a glance. Then ask what the project couldn't answer.

**There's no fixed number of questions; there's a bar.** Ask a question only if both hold:
the project can't answer it, and the answer changes what the seed says. Everything that
fails the bar becomes an `(inferred)` default in the review instead. Ask the questions
together in one batch, most consequential first, using a structured question tool if your
agent has one (split into several calls if the tool limits how many fit in one). Typical
candidates:

- Disagreements from Step 1 where the difference changes what the seed says. Skip ones where
  the code is clearly current, and unexplained reversals that don't affect the rebuild.
- Whether a borderline part is core or incidental, when the evidence is split.
- **How much to disclose**, when the project has valuable data, prompts or tuning. Offer
  levels, e.g. principles only / principles + a small real sample, with tuning as ranges /
  the full dataset and exact values. Secrets and personal data are always withheld; don't
  ask about those.
- Intent that exists only in the author's head: what "good" feels like, what they'd change
  if rebuilding, what they'd never compromise on — only if the project is silent on it.

Offer your inferred answer as the first option for each, so confirming is one click. If
the project answered everything, skip the questions and say so. Keep a list of anything
still uncertain; it goes into the review in Step 6 rather than blocking the work.

## Step 3 — Sort every decision

For each meaningful decision you find, put it in exactly one pile:

| Pile | Meaning | Goes to |
|---|---|---|
| **Commitment** | Must hold in any faithful rebuild | `COMMITMENTS.md` |
| **Choice** | Reasonable builders could differ; depends on their setup | Profile question in `SEED.md` |
| **Incidental** | Framework, library, file layout, style | Dropped |
| **Private** | Author asked to withhold it | Dropped, but noted in the review |

The test for commitment: *if a rebuild got this wrong, would the author say "that's not my
thing"?* When the evidence is thin, make your best call, mark it `(inferred)`, and add it to
the review list — don't interrupt the author for each one. Over-committing anchors the
receiver to your stack; under-committing lets the rebuild drift.

## Step 4 — Write the seed

Write to `seed/` at the repo root (ask before overwriting an existing one):

```
seed/
  SEED.md          start here — idea, context, principles, profile questions, how to build
  COMMITMENTS.md   interfaces, data shapes, invariants, must / must-never
  examples/        concrete behaviour, one file per area
  checks/          runnable or gradeable checks against any build
```

### `SEED.md`

- **Open with the framing**, one or two lines for the receiver: this is a seed, not a
  cutting. Grow your own version in your own soil. The commitments are the genetics to
  keep, and the checks confirm it's the right species.
- **What it is** — one paragraph a stranger understands.
- **The problem and who it's for.**
- **The core loop / how it works** — conceptually, as a user or system experiences it.
- **Principles** — what it optimises for, in priority order.
- **Non-goals and when not to build it.**
- **Decisions and why** — the non-obvious ones, especially from history ("we tried X,
  it failed because Y"). This section is the seed's biggest advantage over code.
- **Profile questions** — the Choices pile, phrased as questions the building agent must
  ask its user before building, each with the author's own answer as a stated default.
- **Build instructions for the receiving agent** — e.g.:
  *"Read this whole folder. Ask the profile questions. Build in the stages below, running
  `checks/` after each. Do not search for or copy any existing implementation."*
  Give 3–6 stages, core first, each ending with which checks should pass.

### `COMMITMENTS.md`

- Interfaces and data shapes in neutral notation, with exact names where names matter
  (vocabularies, flags, status values, file formats — the places independent builds
  invent incompatible variants).
- Invariants and state rules, stated precisely.
- **Must-never** list, each with a one-line reason.
- Tag each commitment with its source (`test`, `code`, `docs`, `history`, `author`) and
  mark any you inferred without direct evidence as `(inferred)`.
- Where the author withheld exact tuning, state it as a range or a relationship ("rises
  from about 5% at the easiest to about 30% at the hardest"), not as the exact value.
- No paths, file names or function names from the original: the receiver never sees the
  repo, and they anchor the rebuild to its structure.

### `examples/`

Concrete cases, derived from real behaviour:

- Prefer translating existing tests into neutral input → expected-outcome form.
- Where safe, **run the original** to record outputs (read-only operations, fixtures,
  sandboxed temp dirs — never against the author's real accounts or data without asking).
- Include edge cases and failure cases, not only happy paths.
- For interactive or agentic behaviour, include short traces: situation → what it did →
  why that was right.
- For data- or content-driven projects, include a **small real sample** of the data (within
  the disclosure level the author chose), large enough for the core to be built and checked
  against it. Check that it's sufficient: a quick throwaway implementation should be able
  to produce valid output from it.

### `checks/`

Make the examples checkable against *any* build:

- `checks/vectors.json` — deterministic cases asserting properties, not exact output where
  exact output is incidental (e.g. "contains X", "state is Y", "rejects"), so they survive a
  different implementation.
- `checks/ADAPTER.md` — the one contract a build implements to be checked: a command that
  takes a case as JSON on stdin and writes the result as JSON on stdout.
- `checks/run.py` (or `.sh`) — a small, dependency-light runner: `run.py --adapter "<cmd>"`.
- `checks/quality.md` — for judgment-heavy areas: a rubric plus labelled good and bad
  examples, meant to be graded by a person or a model.
- `checks/CHECKLIST.md` — properties no automated check can see (deployment, security
  boundaries, UX feel), as a manual list.

- Where the project produces output (generated data, a catalog, reports), give the checker a
  mode that checks **existing output** without an adapter, so the author's shipped output
  can be checked directly.

Keep checks honest: *a failure is a real deviation; a pass is necessary, not sufficient.*

### Writing with subagents

If your agent supports subagents, for a large seed split the writing by area (e.g. core rules + their checks; data + quality
+ its checker). Give each subagent exact ownership of specific files, so no two write the
same one; have them put shared sections as fragments in a scratch directory; then assemble
`COMMITMENTS.md` and `ADAPTER.md` yourself. **After each subagent returns, verify that every
file it reports writing actually exists at the expected path** and is non-empty. Give
subagents absolute paths; relative ones can land in the wrong directory, including the
author's repo. If a file is missing, look for it elsewhere (and move it out of any repo),
or write it yourself from the evidence.

## Step 5 — Self-check

1. **The original must pass its own seed.** Write an adapter for the author's code (in a
   scratch location, not inside `seed/` or the author's repo) and run `checks/`. Also run
   any output checks against the author's real shipped output. **Re-run these yourself**;
   don't rely on a subagent's reported result.
2. **Triage every failure**. It is one of three things:
   - **The seed is wrong** (usually a rule taken from stale docs) → fix the seed.
   - **An accepted exception** → record it as an explicit, reasoned waiver, never a silent skip.
   - **A bug in the author's project** (the code breaks its own stated rule) → keep the rule,
     waive the case for the self-check, and report it to the author as a finding.
3. **Prove the checks can fail.** Vectors derived from the original's behaviour will pass
   against it by construction. So mutate something (bump an expected value, break one rule
   in a copy of the output) and confirm the checks catch it. A check that can't fail proves
   nothing.
4. **Optional blind regrow** (offer it; it costs time and tokens; needs subagent support, or
   a separate fresh agent session): start a fresh agent that
   may read *only* `seed/` in an empty directory, has no web access to the original, and
   builds the core stages in a different stack. Run the checks on it. Then look for gaps:
   anything where the rebuild passes the checks but behaves differently from the original
   is a missing commitment or example — add it and note what changed.

## Step 6 — Disclosure review

Before calling the seed done, check it for leakage and show the author what it reveals:

- Scan `seed/` for secrets, tokens, keys, emails, internal hostnames, customer or personal
  data (grep common secret shapes; check examples recorded from real runs especially).
- Flag any passage that is code or a prompt in disguise (long verbatim strings, step-by-step
  mirroring of a specific function).
- Confirm nothing from the Private pile appears.

Write `seed-review.md` **next to** `seed/`, not inside it — it is for the author only.
It maps where sensitive material lives, so it must never be committed: if the project is a
git repo, add `seed-review.md` to `.git/info/exclude` (local-only, so the author's
`.gitignore` stays untouched) and check that `git check-ignore seed-review.md` confirms it.
Tell the author you did this. It covers:

- What the seed discloses, section by section, at a glance.
- What was deliberately left out (incidental and private), so the author can pull more in.
- Commitments marked `(inferred)` that the author should confirm, and the docs-vs-code
  disagreements where the seed followed the code.
- Self-check results, waivers, and known gaps (say plainly if the blind regrow wasn't run:
  the self-check shows the seed is *consistent* with the original, not that it's *enough*
  to build from).
- **Findings about the project**: bugs, stale docs, and targets the project misses. These
  surfaced during extraction, and the author will want them regardless of the seed.
- Sensitive material found in the repo itself (e.g. personal data committed in docs), by
  location.

Clean up before finishing: remove caches (`__pycache__`, build output) from `seed/`, and
leave scratch adapters and conversion scripts out of the author's repo.

Finish by telling the author, in a few lines: what the seed covers, what the self-check
showed, what needs their confirmation, any findings about their project, and that `seed/`
alone is what they share.
