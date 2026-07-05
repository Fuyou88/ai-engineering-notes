---
name: skill-design-guide
description: Use when designing, refactoring, or reviewing an agent skill. Produces concrete artifacts such as trigger definitions, routing tables, capability inventories, noise tables, scorecards, and gate checklists. Helps avoid overloaded skill entries, abstract slogans, ability loss, missing verification, and token-heavy default context.
---

# Skill Design Guide

## When to Use

Use this skill when the user asks to:

- create a new agent skill
- turn prompt experience into a reusable skill
- refactor an existing skill
- review whether a skill is well designed
- improve skill token economy
- check whether a skill is too broad, too abstract, or missing gates

## Do Not Use

Do not use for ordinary feature implementation, debugging, document writing, or prompt editing unless the target artifact is an agent skill.

## Task Modes

### Design New Skill

Must output:

1. Skill Worthiness Check
2. Trigger Definition
3. Do-not-use cases
4. Task Routing
5. File Structure Decision
6. Required Artifacts / Gates
7. Verification Strategy
8. `SKILL.md` Draft

### Refactor Existing Skill

Must output:

1. Existing Capability Inventory
2. Problems & Noise Table
3. Proposed Routing
4. Reference Responsibility Check
5. Migration Plan
6. Ability-loss and token-economy risk check
7. Verification / degraded-mode check

### Review Existing Skill

Must output:

1. Scorecard
2. Blocking Issues
3. Suggested Fixes
4. KISS / Token Economy Assessment
5. Skill worthiness / over-engineering assessment
6. Verdict: Pass / Needs changes / Reject

## Skill Worthiness Check

Before designing a new skill, decide whether it should exist. Not every prompt should become a skill.

| Question | Answer | Decision Impact |
|---|---|---|
| Is the task repeated or likely to repeat? | | |
| Is failure costly, subtle, or hard to detect? | | |
| Does it need tools, references, scripts, or assets? | | |
| Does it need explicit routing, gates, or verification? | | |
| Would a short prompt or normal doc be enough? | | |
| Decision | create skill / keep as prompt-doc / defer | |

Default: if the task is one-off, low-reuse, and low-risk, do not over-engineer it into a skill.

## File Structure Decision

Choose structure by skill type and complexity. Do not copy a fixed template by default.

| Skill Type | Best-fit Structure | Notes |
|---|---|---|
| lightweight | `SKILL.md` only | Use when routing and references are unnecessary. |
| workflow / stage-based | `references/` by phase | Use for multi-step work with gates. |
| tool-based | `scripts/` + small usage reference | Put deterministic operations in scripts. |
| knowledge-based | `references/` by domain | Split APIs, schemas, policies, domain rules. |
| format / asset-based | `scripts/` + `assets/` + format rules | Use for document, image, deck, PDF, or template workflows. |

## Reference Responsibility Check

Each reference must have one responsibility. Split by phase, domain, tool, output artifact, or project rules.

| Reference | Responsibility | Loaded When | Overlap Risk |
|---|---|---|---|

Rules:

- Do not make every skill use `intake/extract/architecture/implementation/verify`.
- Do not place workflow, templates, project rules, and anti-patterns in one reference.
- Do not duplicate the same rule across references unless one is a short router and the other is detailed guidance.

## Abstract-to-Artifact Rewrite

Any important abstract statement must be rewritten into one of:

- step
- artifact
- gate
- route
- check
- boundary
- fallback

Use this table when reviewing or refactoring:

| Abstract Statement | Problem | Rewrite As | Result |
|---|---|---|---|

Examples:

| Abstract Statement | Problem | Rewrite As | Result |
|---|---|---|---|
| ensure high quality | not checkable | gate | completion requires Verify Report |
| preserve coverage | unclear action | artifact | produce Coverage Matrix |
| improve maintainability | vague | artifact | produce Component / Capability Mapping |
| optimize token usage | vague | route/check | load references by task mode; avoid raw payload dumps |

## Capability Preservation

When refactoring a skill, first inventory existing capabilities:

| Existing Capability | Keep / Move / Remove | New Location | Reason |
|---|---|---|---|

Rules:

- Default to Keep.
- Do not remove effective capabilities just to reduce tokens.
- Move large or low-frequency details behind routing or references.
- `Remove` requires an explicit reason and risk assessment.

## Task Routing Table

For Design or Refactor tasks, define how future users should be routed:

| User Intent | Route / Mode | Required Inputs | Required Outputs |
|---|---|---|---|

Rules:

- Simple tasks should not load a full SOP.
- Complex tasks must not skip required artifacts.
- If uncertainty is high, route to the next higher complexity level.

## Problems & Noise Table

Use when reviewing or refactoring a skill:

| Current Text / Rule | Problem Type | Impact | Fix |
|---|---|---|---|

Problem Type examples:

- overloaded entry
- abstract slogan
- duplicated rule
- missing gate
- missing trigger boundary
- token-heavy default path
- ability loss risk
- shared capability pollution
- data source inconsistency
- fixed structure overfit
- reference responsibility overlap
- over-engineered one-off task
- missing degraded mode
- unverifiable completion

## Verification Strategy

Completion must have an explicit verification or self-check artifact.

| Task Type | Verification Artifact |
|---|---|
| UI | DOM diff / screenshot comparison / interaction check |
| API | contract test / request-response check |
| Data | consistency check / migration dry-run |
| Refactor | impact analysis / tests |
| Docs | completeness checklist |
| Logs / search | query replay / sample validation |

## Degraded Mode Check

Fallbacks are allowed, but limitations must be explicit.

| Failure | Fallback | Limitation | Completion Claim Allowed? |
|---|---|---|---|

Rules:

- If a required tool or source is unavailable, record why.
- If using fallback data, mark what remains unverified.
- Do not claim full completion when verification was degraded.

## Scorecard

For reviews, score each dimension from 1 to 5:

| Dimension | Score | Evidence | Fix |
|---|---:|---|---|
| Trigger clarity | | | |
| Do-not-use boundary | | | |
| Main entry thinness | | | |
| Task routing | | | |
| Progressive loading | | | |
| File structure fit | | | |
| Reference responsibility | | | |
| Artifact gates | | | |
| Verification closure | | | |
| Degraded-mode handling | | | |
| Capability preservation | | | |
| Shared capability boundary | | | |
| Data/source consistency | | | |
| KISS | | | |
| Token economy | | | |
| Abstract-to-artifact rewrite | | | |

Verdict:

- 4.5–5: Pass
- 3.5–4.4: Needs minor changes
- 2.5–3.4: Needs major changes
- <2.5: Reject / redesign

## KISS / Token Economy Checks

- [ ] Is this worth becoming a skill rather than a one-off prompt/doc?
- [ ] Is `SKILL.md` a router rather than a knowledge dump?
- [ ] Is the file structure chosen by need, not copied from a fixed template?
- [ ] Are details moved to references only when useful?
- [ ] Are references loaded by task mode, phase, domain, or tool need?
- [ ] Does each reference have one clear responsibility?
- [ ] Are low-frequency edge cases kept out of the default path?
- [ ] Are repeated gates or slogans removed?
- [ ] Are effective capabilities preserved but not loaded by default?
- [ ] Are large payloads summarized first and opened only when needed?
- [ ] Are templates and examples only as detailed as needed for execution?

## Required Gates

Before proposing a new or refactored skill:

- [ ] Trigger and do-not-use cases are explicit.
- [ ] Task modes and routing are explicit.
- [ ] Skill worthiness is checked; one-off / low-reuse tasks are not over-engineered into skills.
- [ ] File structure is chosen by skill type and complexity.
- [ ] Reference responsibilities are explicit and non-overlapping.
- [ ] Important abstract statements are converted to steps, artifacts, gates, checks, boundaries, or fallbacks.
- [ ] Existing effective capabilities are preserved or intentionally removed with rationale.
- [ ] Shared capabilities are protected from business-specific pollution.
- [ ] Count/list/empty/action data source consistency is checked when applicable.
- [ ] Completion has a verification or self-check requirement.
- [ ] Tool failure or data absence has a degraded-mode path.
- [ ] The default path is KISS and token-conscious.

## Final Delivery

Always include:

- Mode: Design / Refactor / Review
- Artifacts produced
- Key decisions
- Skill worthiness decision, if designing a new skill
- File structure decision
- Risks or remaining gaps
- Verification / self-check
- Degraded-mode limitations, if any
- Verdict, if reviewing
