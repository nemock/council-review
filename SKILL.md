---
name: council-review
description: >-
  Pressure-test any project, system, document, or plan from multiple independent
  expert perspectives at once, then synthesize one prioritized report. Convenes a
  "council" of reviewer agents — each assigned a distinct lens (e.g. brand/voice,
  content/editorial, technical reliability, distribution/growth, operations/
  maintainability) — that review in parallel, then a chair synthesizes their
  findings, surfacing agreements and conflicts. Use when the user asks for a
  "council review," "multi-perspective review," "pressure test," "review this
  from different angles," "panel review," "get multiple perspectives on,"
  "red-team this," or wants a project analyzed from several points of view.
  Report-only by default — it recommends, it does not change anything.
---

# Council Review

Convene a council of independent expert reviewers to pressure-test a subject from
several distinct points of view, then synthesize their findings into one
prioritized report. The value is **independent perspectives that don't contaminate
each other** plus a synthesis that surfaces where they agree (high-confidence) and
where they conflict (the interesting tensions).

This skill is report-only. It recommends; it does not modify the subject. If the
user wants changes applied afterward, that's a separate, explicit follow-up.

## When to use

- "Run a council review on X" / "pressure-test this" / "review this from different angles"
- Evaluating a routine, codebase, feature, document, plan, strategy, PR, or design
- Any time multiple independent lenses would catch more than one reviewer would

## Mechanism

This skill drives the **Workflow tool** to fan out one agent per lens (parallel,
independent), each returning structured findings, then the chair (you) synthesizes.
Invoking this skill is explicit opt-in to multi-agent orchestration — go ahead and
call Workflow. If Workflow is unavailable, fall back to reviewing each lens yourself
in sequence and synthesizing, but say so.

## Step 1 — Scope the subject

Identify exactly what is under review and gather the inputs the reviewers need:

- A path / directory / file set / PR / pasted artifact. Resolve it concretely.
- Read enough yourself to know the subject's *type* (content routine? service
  codebase? strategy doc? product spec?) — this drives lens selection in Step 2.
- Decide how reviewers will access the subject:
  - **File-readable** (default): pass each agent the key paths and let it read them.
  - **Not file-readable** (e.g. an isolated subprocess, a pasted blob, a remote
    artifact): embed the full subject text into each agent's prompt. Reviewers
    can't review what they can't see.

## Step 2 — Choose the lenses

Pick **3–6 lenses tailored to the subject**. Each lens must be genuinely distinct —
overlapping lenses waste agents. The user may name lenses or a focus; honor that.
Otherwise select from a pack below or compose your own.

**Canonical pack — content / marketing routine** (the proven default mix):
1. **Brand, positioning & voice** — does it reinforce the intended brand and audience?
2. **Content sourcing & editorial quality** — is the input selection and output quality good and sustainable?
3. **Technical reliability & failure modes** — where does it break, especially unattended?
4. **Distribution & growth strategy** — are the channels, formats, timing, and reach choices right?
5. **Operations & maintainability** — is it robust, observable, and cheap to maintain over time?

**Pack — software / codebase:**
correctness & bugs · security · performance & scalability · architecture &
maintainability · testing & coverage · developer experience / API design.

**Pack — product / strategy / business plan:**
market & positioning · customer & demand evidence · business model & unit economics ·
competitive & differentiation · execution & operational feasibility · risk & failure modes.

**Pack — document / writing / proposal:**
argument & logic · evidence & sourcing · clarity & structure · audience fit & voice ·
completeness & gaps · counterarguments / red-team.

Adapt freely: drop lenses that don't apply, add subject-specific ones (regulatory,
legal, accessibility, cost, etc.). Scale lens count to the user's ask — a quick
pressure test is 3 lenses; "be thorough" is 5–6.

## Step 3 — Run the council (Workflow)

Fan out one agent per lens in parallel, each with the shared structured schema below,
then return the raw findings. Template — adapt the lens list, subject context, and
how reviewers access the subject:

```javascript
export const meta = {
  name: 'council-review',
  description: 'Multi-perspective council review of <subject>',
  phases: [{ title: 'Council review' }],
}

const SUBJECT = `<one-paragraph description of what's under review + how to access it:
either "read these paths: ..." or the embedded artifact text itself>`

const SCHEMA = {
  type: 'object', additionalProperties: false,
  required: ['lens', 'overall_take', 'strengths', 'issues', 'top_recommendations'],
  properties: {
    lens: { type: 'string' },
    overall_take: { type: 'string', description: '2-4 sentence verdict from this lens' },
    strengths: { type: 'array', items: { type: 'string' } },
    issues: { type: 'array', items: {
      type: 'object', additionalProperties: false,
      required: ['severity', 'title', 'detail', 'recommendation'],
      properties: {
        severity: { type: 'string', enum: ['high','medium','low'] },
        title: { type: 'string' },
        detail: { type: 'string' },
        recommendation: { type: 'string' },
      } } },
    top_recommendations: { type: 'array', items: { type: 'string' },
      description: 'ranked: the 2-4 changes this lens would prioritize' },
  },
}

const base = `You are one member of an expert review council evaluating the following subject.
This is a REVIEW ONLY — recommend, do not change anything. Be specific: cite the exact
mechanism/line/section, distinguish real problems from nitpicks, and make recommendations
actionable. Disagreement with current design choices is welcome if justified.

SUBJECT:
${SUBJECT}`

const LENSES = [
  { label: 'review:lens-1', prompt: `${base}\n\nYOUR LENS — <name>: <2-4 sentences of exactly what to evaluate from this angle, with pointed questions>` },
  // ... one entry per lens
]

const findings = await parallel(LENSES.map(l => () =>
  agent(l.prompt, { label: l.label, phase: 'Council review', schema: SCHEMA })
))
return findings.filter(Boolean)
```

Each lens prompt should: (a) carry the shared `base` context, (b) state the lens
name, and (c) give 2–4 sentences of pointed, lens-specific things to evaluate plus
guiding questions. Sharper lens prompts → sharper findings.

## Step 4 — Synthesize (the chair)

You are the chair. Read all returned findings and write ONE report. Do not just
concatenate the agents:

- **Cross-cutting themes** — issues ≥2 lenses raised independently are high-confidence; lead with these.
- **Conflicts** — where lenses disagree, surface the tension and take a position (more recent / better-evidenced wins); do NOT average them away.
- **Prioritized recommendations** — a single ranked list, highest-leverage first, each with what-to-change and why. Pull severity from the agents but re-rank holistically.
- **Keep what's working** — name the strengths worth preserving so recommendations don't read as "rewrite everything."

## Output format

```
# Council Review — <subject>

## Verdict
<2-4 sentences: is this well-designed? overall health.>

## Council
<one line per lens: name + its headline take>

## Strengths to preserve
- ...

## Cross-cutting findings (multiple lenses agreed)
- ...

## Conflicts & tensions
- <where lenses disagreed and your call>

## Prioritized recommendations
1. **<change>** (severity, lens) — what to change and why it matters.
2. ...

## Per-lens detail
<collapsible/short per-lens summaries for the reader who wants depth>
```

## Notes

- **Report-only.** Never edit the subject as part of this skill. Offer a follow-up if the user wants fixes applied.
- **Scale to the ask.** Quick pressure test = 3 lenses; "thorough"/"red-team" = 5–6 plus sharper, more adversarial lens prompts.
- **Independence is the point.** Don't let one lens's prompt reference another's expected findings; they review blind and the chair reconciles.
- **Token awareness.** Each lens is a full agent; 5–6 lenses on a large subject is a meaningful spend. Match lens count to how much the decision warrants.
