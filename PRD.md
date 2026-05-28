# PRD — `council-review` Skill

**Status:** v0.1.0 (initial public release)
**Owner:** Dave Saunders ([@nemock](https://github.com/nemock))
**Last updated:** 2026-05-28

---

## 1. Overview

`council-review` is a Claude Code skill that pressure-tests a subject — a
project, codebase, document, plan, or strategy — from multiple independent
expert perspectives in parallel, then synthesizes the findings into one
prioritized report.

It does not modify the subject. It is report-only. Any follow-up changes
are an explicit, separate action initiated by the user.

The skill is invoked when the user asks for a "council review," a
"pressure test," a "panel review," a "red-team," or any variant that
implies wanting multiple lenses on the same artifact at once.

---

## 2. Problem statement

A single reviewer — human or model — produces a coherent but narrow read
of any non-trivial artifact. The blind spots are systematic, not random:

- A reviewer drawn toward correctness will under-weight distribution.
- A reviewer drawn toward brand will under-weight failure modes.
- A reviewer drawn toward the audience will under-weight maintainability.

Running several reviewers sequentially against the same artifact does not
fix this — each subsequent reviewer is contaminated by the previous one's
framing, anchoring on the same issues and missing the orthogonal ones.

The fix is **independent parallel review** followed by **explicit
synthesis**. Independence catches the orthogonal blind spots. Synthesis
turns N raw reviews into one prioritized action list and, critically,
separates high-confidence findings (≥2 lenses agreed) from contested
ones (lenses disagreed).

The existing single-shot review experience does not produce this. Manual
multi-reviewer workflows do, but they are tedious to set up by hand. This
skill exists to make the parallel-and-synthesize pattern a one-line
invocation.

---

## 3. Target users

| User                                    | Typical artifact                          | Why they need this                                                         |
| --------------------------------------- | ----------------------------------------- | -------------------------------------------------------------------------- |
| Solo operator running content routines  | A scheduled posting routine               | Catches brand drift + reliability bugs + distribution gaps in one pass     |
| Engineer shipping a feature             | A PR or design doc                        | One pass covers correctness, security, perf, maintainability, DX           |
| Product / strategy lead                 | A business plan or product spec           | Surfaces unit-economic, competitive, and execution risks together          |
| Writer / marketer                       | An essay, landing page, or pitch          | Argument, evidence, clarity, audience-fit, and counterargument in one shot |
| Reviewer auditing a third-party project | Any codebase or document handed over cold | Provides scaffolding for a thorough, structured review                     |

The common thread: the user has *one artifact* and wants several distinct
expert reads on it, fast, with a synthesis they can act on.

---

## 4. Goals

- **G1.** Produce a single prioritized report that is more useful than
  any one lens's review.
- **G2.** Surface cross-cutting findings (issues ≥2 lenses raised
  independently) with high confidence.
- **G3.** Surface lens-to-lens conflicts as conflicts, not as averages.
- **G4.** Keep the invocation one-line for the user — no scaffolding
  burden — by carrying lens selection, fan-out, and synthesis inside the
  skill.
- **G5.** Stay report-only: never modify the subject under review.

## 4a. Non-goals

- **N1.** Replacing single-lens review (`/code-review`, `/security-review`,
  etc.). When the user wants one sharp lens, those skills are correct.
- **N2.** Producing diffs, edits, or patches. If the user wants the
  recommendations applied, that is a separate explicit step.
- **N3.** Drawing on outside data (web search, retrieval, etc.). Reviewers
  evaluate the provided subject only. Deep research is `/deep-research`.
- **N4.** Acting as a benchmark or scoring rubric. The synthesis is
  ranked, but the skill does not output a numerical score.

---

## 5. Functional requirements

| ID  | Requirement                                                                                                                                  |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| F1  | The skill MUST scope the subject by resolving paths, files, or pasted artifacts before fan-out.                                              |
| F2  | The skill MUST decide whether reviewers can read files directly or whether the subject text must be embedded in each prompt.                 |
| F3  | The skill MUST select 3–6 lenses, each genuinely distinct, scaled to the user's ask (3 = quick pressure test; 5–6 = "thorough" / red-team).  |
| F4  | The skill MUST honor user-provided lens names or focus.                                                                                      |
| F5  | The skill MUST fan out one agent per lens, in parallel, each independent of the others.                                                      |
| F6  | Each agent MUST return findings in a shared structured schema (overall take, strengths, issues with severity, top recommendations).          |
| F7  | The chair (the invoking model) MUST synthesize the findings into one report — not concatenate them.                                          |
| F8  | The synthesis MUST identify cross-cutting findings (≥2 lenses agreed) and lead with them.                                                    |
| F9  | The synthesis MUST surface conflicts between lenses and take a position rather than average them away.                                       |
| F10 | The synthesis MUST emit a single prioritized recommendation list, ranked holistically (severity from agents informs but does not determine). |
| F11 | The synthesis MUST name strengths to preserve, so the report does not read as "rewrite everything."                                          |
| F12 | The skill MUST NOT modify the subject as part of its operation.                                                                              |
| F13 | If the Workflow tool is unavailable, the skill MUST fall back to per-lens sequential review and disclose the fallback in the output.         |

## 6. Non-functional requirements

| ID  | Requirement                                                                                                                                                                                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| NF1 | **Latency.** Wall-clock time is bounded by the slowest lens, not the sum of all lenses (parallel fan-out).                                                                                             |
| NF2 | **Token cost.** Each lens is a full agent context. The skill MUST scale lens count to the decision's importance and warn implicitly via lens-count discipline.                                         |
| NF3 | **Independence.** No lens prompt may reference another lens's expected findings. Reviewers review blind.                                                                                               |
| NF4 | **Auditability.** The structured per-lens findings MUST be preserved verbatim (in a collapsible "per-lens detail" section), so the user can re-read any lens's raw output and not just the synthesis. |
| NF5 | **Determinism of structure.** The output format is fixed; only the content varies by subject.                                                                                                          |
| NF6 | **Portability.** The skill is a single `SKILL.md` file with no runtime dependencies beyond the Claude Code Workflow tool.                                                                              |

---

## 7. Key design choice — lens taxonomy

Lenses are not invented per invocation. The skill ships with four
canonical packs, each tuned to a subject type, plus the explicit
instruction to compose or adapt as needed:

- **Content / marketing routine** — brand & voice · sourcing & editorial
  quality · technical reliability · distribution & growth · operations &
  maintainability.
- **Software / codebase** — correctness · security · performance ·
  architecture · testing · DX.
- **Product / strategy / business plan** — market · demand evidence ·
  business model · differentiation · execution · risk.
- **Document / writing / proposal** — argument · evidence · clarity ·
  audience fit · completeness · counterargument.

**Why packs and not free composition every time:** Lens overlap is the
failure mode. Two lenses that are 70% the same waste an agent and produce
correlated findings, which then look falsely high-confidence in synthesis.
Packs are pre-tuned for distinctness. Free composition is allowed but
explicitly opt-in.

## 8. Key design choice — synthesis rules

The chair (the invoking model) follows fixed synthesis rules so the
report does not become "five agents stapled together":

1. **Agreement is signal.** Issues raised independently by ≥2 lenses are
   high-confidence and lead the report.
2. **Conflict is signal.** Where lenses disagree, the chair takes a
   position (recency, evidence quality, or first-principles reasoning
   decides) and explains the call. Conflicts are NEVER averaged away.
3. **Severity is re-ranked holistically.** Agents tag severity per lens;
   the chair re-ranks across lenses because a "low" from two lenses can
   be a "high" cross-cuttingly.
4. **Strengths are named.** Without naming what's working, the report
   reads as a demand to start over. The chair lists preserve-this items.

---

## 9. Workflow integration

The skill drives the Claude Code Workflow tool. Invoking the skill is
itself the user's explicit opt-in to multi-agent orchestration.

The reference template in `SKILL.md` defines:

- A shared `base` prompt carrying the subject context.
- A `LENSES` array — one entry per lens, each with a sharply-worded
  `prompt` that names the lens and gives 2–4 sentences of pointed
  evaluation criteria.
- A `SCHEMA` enforcing structured output (`overall_take`, `strengths`,
  `issues[]` with severity, `top_recommendations[]`).
- A single `parallel(LENSES.map(...))` fan-out.

Each lens runs in its own agent context. Independence is enforced by the
schema — no lens sees the others' findings until synthesis.

---

## 10. Output format

```
# Council Review — <subject>

## Verdict
<2-4 sentences: overall health.>

## Council
<one line per lens: name + headline take>

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

This shape is mandatory, not advisory. The user should be able to skim
the Verdict and Prioritized Recommendations and act, then drop into
Per-lens Detail when they want the raw read.

---

## 11. Cost and latency model

- **Wall-clock latency** ≈ time for the slowest lens to complete + a
  small synthesis pass. With 5 lenses on a moderate artifact, expect
  60–180 seconds end-to-end on the standard models.
- **Token cost** ≈ N × (full subject context + lens prompt + lens
  response). For a large artifact, this is non-trivial. Discipline
  around lens count (3 for a quick pressure test, 5–6 for a thorough
  review) is the main cost lever.

The skill does not enforce a hard token budget; the user's session
budget controls remain authoritative.

---

## 12. Risks and failure modes

| Risk                                                                                                       | Mitigation                                                                                                                          |
| ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Lens overlap produces false high-confidence findings.**                                                  | Packs are pre-tuned for distinctness; the skill instructs the chair to consider whether two lenses agreeing is genuine or echoing.  |
| **One lens dominates the synthesis because it wrote the most.**                                            | The chair is instructed to re-rank holistically and name conflicts; verbosity is not weight.                                        |
| **Reviewers cannot see the subject (subprocess, pasted blob, remote).**                                    | The skill explicitly distinguishes file-readable vs. embed-required modes in Step 1 of `SKILL.md`.                                  |
| **Workflow tool unavailable in this environment.**                                                         | Fallback path: per-lens sequential review with explicit disclosure.                                                                 |
| **The user expected edits, not a report.**                                                                 | The skill is loud about being report-only and offers an explicit follow-up. "Review" verbs are interpreted as review, not rewrite.  |
| **Lens count too high for the artifact, burning tokens for diminishing returns.**                          | Lens-count discipline is part of the skill: scale to the ask, default to 3, only go to 5–6 when "thorough"/"red-team" is requested. |

---

## 13. Success metrics

This skill produces qualitative output, not numbers. Success is judged
on the report itself:

- **The user takes action on at least one recommendation.** If a report
  produces nothing actionable, the lens selection or the synthesis
  failed.
- **At least one recommendation surprises the user.** If every finding
  is something the user already knew, the parallel fan-out earned
  nothing over a single-lens review.
- **Conflicts are visible.** A report with zero conflicts on a
  non-trivial artifact almost certainly averaged disagreements away.
- **The user does not need to re-read individual lens outputs to
  understand the synthesis.** The synthesis is self-sufficient; the
  per-lens detail is for depth, not for missing context.

---

## 14. Out of scope

- Numerical scoring or benchmarking.
- Automated remediation (edits, diffs, PRs).
- External-source consultation (web, retrieval, MCP). Reviewers evaluate
  the provided subject only.
- Multi-round debate between lenses. Reviewers go once, blind; the chair
  synthesizes once. No back-and-forth.
- Persistent state between invocations. Each council review is one-shot.

---

## 15. Future considerations

These are not committed; they are noted so contributors don't relitigate
them from scratch:

- **Custom lens packs as files.** Today lens packs live in `SKILL.md`.
  Moving them to `lenses/*.json` would make community contribution
  cleaner.
- **Optional second pass.** A "rebuttal" pass where each lens sees the
  others' findings could surface deeper conflicts but doubles cost and
  loses independence. Worth experimenting with as an opt-in mode, not
  the default.
- **Severity calibration.** Today severity is per-lens; a calibration
  prompt could normalize severities across lenses before synthesis.
- **Output formats.** Markdown is the default; a JSON-only mode could
  feed downstream automation.

---

## 16. Attribution

The "council" pattern (multiple models reviewing in parallel, then a
chair synthesizing) is inspired by Andrej Karpathy's
[`llm-council`](https://github.com/karpathy/llm-council). That project
implements the pattern as a multi-vendor LLM web app. This skill ports
the *idea* — parallel independent review then synthesis — into the
Claude Code skill format, where the parallel agents are subagents of a
single model and the synthesis is performed by the invoking session
itself.
