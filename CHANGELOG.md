# Changelog

All notable changes to the `council-review` skill are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.1.0] — Initial public release

### Added
- `SKILL.md` — the skill definition: scoping, lens selection, Workflow-based
  fan-out, structured-output schema, and synthesis rules.
- Four canonical lens packs: content/marketing routine, software/codebase,
  product/strategy/business plan, document/writing/proposal.
- `PRD.md` — full product requirements document describing the problem,
  target users, functional and non-functional requirements, design choices,
  and out-of-scope items.
- `README.md` — install, usage, mechanism overview, and cost notes.
- MIT `LICENSE`.

### Design choices documented
- Report-only by default — the skill never modifies the subject.
- Parallel fan-out via the Workflow tool; sequential per-lens fallback when
  Workflow is unavailable.
- Independence is enforced — no lens prompt references another lens's
  expected findings.
- Synthesis surfaces conflicts; it does not average them away.
