# `council-review`

> A Claude Code skill that convenes a council of independent expert reviewers
> to pressure-test any project, system, document, or plan — in parallel —
> then synthesizes one prioritized report.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A single reviewer produces a coherent but narrow read of any non-trivial
artifact. Running reviewers sequentially does not fix this — each one
anchors on the previous one's framing. `council-review` runs 3–6 lenses
**in parallel and blind**, then a chair synthesizes the findings,
surfacing where lenses agreed (high-confidence) and where they
conflicted (interesting tensions).

It is **report-only**. It never modifies the subject under review.

---

## Install

The skill is a single `SKILL.md` file. Drop the repo into your skills
directory:

```bash
git clone https://github.com/nemock/council-review.git ~/.claude/skills/council-review
```

Then restart Claude Code (or start a new session). The skill will appear
under its name in the available-skills list.

To update later: `git -C ~/.claude/skills/council-review pull`.

To uninstall: `rm -rf ~/.claude/skills/council-review`.

---

## Use

Trigger the skill by asking for a multi-perspective review:

```
/council-review the routine in scheduled_routines/founder_tip_tuesday/
```

```
Run a council review on my landing page draft at /tmp/landing.md — be thorough.
```

```
Pressure-test the architecture in PRD.md from 4 angles: market, tech, ops, risk.
```

You can also let Claude pick the skill from natural-language phrasing:

```
Get me multiple perspectives on this PR before I merge it.
Red-team my pricing strategy.
Review this from different angles.
```

The skill will scope the subject, select lenses, fan out reviewers, and
return a single synthesis report.

---

## What you get

```
# Council Review — <your subject>

## Verdict
<overall health, 2-4 sentences>

## Council
<one line per lens with its headline take>

## Strengths to preserve
- ...

## Cross-cutting findings (multiple lenses agreed)
- ...

## Conflicts & tensions
- <where lenses disagreed and the chair's call>

## Prioritized recommendations
1. **<change>** (severity, lens) — what to change and why it matters.

## Per-lens detail
<per-lens summaries preserved for the depth reader>
```

The verdict + prioritized recommendations are designed to be skimmed and
acted on. The per-lens detail is there when you want the raw read.

---

## How it works

The skill drives Claude Code's `Workflow` tool to fan out one subagent
per lens — each in its own context, each blind to the others — and
returns the findings as structured JSON. The invoking session then acts
as the **chair**: it reads all findings and writes one report following
fixed synthesis rules (agreement is signal, conflicts are not averaged,
severity is re-ranked holistically, strengths are named).

The skill ships with four canonical lens packs (content/marketing
routine, software/codebase, product/strategy, document/writing) and
adapts them to the subject. You can also name your own lenses.

See [`SKILL.md`](SKILL.md) for the full mechanism and
[`PRD.md`](PRD.md) for design rationale.

---

## Cost note

Each lens is a full subagent context. Five lenses on a moderate artifact
is a non-trivial token spend (and a 60–180s wall clock). Scale lens
count to the decision's importance:

- **3 lenses** — quick pressure test.
- **4 lenses** — standard review.
- **5–6 lenses** — "thorough" or "red-team."

The skill won't go above 6 lenses without an explicit ask.

---

## What this is NOT

- **Not a code-fixer.** The skill recommends; it never edits. If you
  want changes applied, that's a separate explicit step.
- **Not a single-lens review.** For a sharp single-perspective read,
  use the corresponding focused skill (e.g. `/code-review`,
  `/security-review`).
- **Not deep research.** Reviewers evaluate the provided subject only —
  they don't go fetch external sources. For research, use
  `/deep-research`.

---

## Attribution

Inspired by Andrej Karpathy's
[`llm-council`](https://github.com/karpathy/llm-council), which
implements the parallel-review-then-chair pattern as a multi-vendor
LLM web app. This skill ports the *idea* into the Claude Code skill
format — the parallel reviewers are subagents of one model, and the
chair is the invoking session itself.

---

## License

[MIT](LICENSE) — use it, fork it, ship it.
