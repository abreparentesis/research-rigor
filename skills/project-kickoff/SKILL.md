---
name: project-kickoff
description: >-
  Entry point for starting a new project or a non-trivial feature. Orchestrates
  the full decision-first kickoff: design-spec (the WHAT), prior-art-check
  (does it already exist), a written Decision Record, convergent-review to harden
  the decisions by argument, and a cheapest-first de-risking pass for the
  assumptions argument can't settle — THEN hands to implementation-plan. Use when the
  user says "new project", "let's build X", "kick off", "start a feature", or
  begins substantial creative work where the consequential decisions matter more
  than the implementation steps. Wraps design-spec as its first stage; do not
  invoke design-spec separately when this is running.
---

# Project Kickoff

The most expensive error in a new project is a decision error: you build the
wrong thing well, and no plan fixes that. This skill front-loads the decisions —
captures them explicitly, hardens them, and resolves the ones that need evidence
— before any "how" work begins.

It is pure orchestration. It invokes three existing skills unchanged and owns two
new artifacts: the **Decision Record** and the **De-risking Queue**.

**Announce at start:** "I'm using project-kickoff to run design-spec → prior-art → decision-hardening → de-risking before we plan."

## When to use
- Starting a new project, service, library, or substantial feature.
- Any creative work where the bet (what / build-vs-adopt / which approach) carries
  more risk than the implementation sequencing.

## When NOT to use
- Trivial or single-file changes, renames, typos, format-only edits.
- Work fully contained in a module the user is already editing.
- A truly simple feature where design-spec's short-design path is enough —
  invoke `design-spec` directly instead.
- Fixing a bug — that's `systematic-debugging`, not a kickoff.

## What this skill owns vs delegates

| Stage | Owned / Delegated |
|---|---|
| 1. Brainstorm the WHAT | delegate → `design-spec` |
| 2. Prior-art (does it exist) | delegate → `prior-art-check` |
| 3. Decision Record | **owned** (template below) |
| 4. Harden decisions by argument | delegate → `convergent-review` |
| 5. De-risking Queue (resolve by evidence) | **owned** (template below) |
| 6. Hand to planning | delegate → `implementation-plan` |

Do not reimplement the delegated skills. Invoke them and let them run.

## Checklist

Create a task per item and complete them in order:

1. **Lightweight prior-art scan** — a 60-second "does this category of thing
   exist?" pass to inform the questions. NOT the full search yet.
2. **Brainstorm** — invoke `design-spec`. It explores context, asks
   questions one at a time, proposes 2-3 approaches, and writes its design spec.
   **Intercept its terminal handoff:** when it would invoke implementation-plan, stop —
   the remaining stages run first.
3. **Full prior-art** — invoke `prior-art-check` now that scope is crisp. This is
   design-spec's "Search Before Building" phase done properly. Its
   Adopt/Extend/Compose/Build verdict becomes one decision in the record.
4. **Write the Decision Record** — enrich design-spec's spec with the explicit
   decision structure (template below). One source of truth.
5. **Harden by argument** — invoke `convergent-review` on the Decision Record with
   design/spec lenses, critics aimed at the *decisions and assumptions*, not the
   prose. It prunes YAGNI decisions and sharpens the rest until K clean rounds.
6. **De-risk by evidence** — for decisions flagged `Resolved by: evidence`, run
   the De-risking Queue cheapest-and-most-fatal first. A killed or changed
   decision loops back to step 4 (re-harden the delta only).
7. **Plan** — once the record is both argued-clean and evidence-resolved, invoke
   `implementation-plan`.

## Flow

```
  lightweight prior-art  →  design-spec (intercept handoff)  →  full prior-art
        ↓
  Decision Record (owned)
        ↓
  convergent-review  ── prune + sharpen by argument ──┐
        ↓                                              │ loop on changed decisions
  De-risking Queue (owned)  ── resolve by evidence ───┘
        ↓
  implementation-plan
```

Two ordering rules that make this correct:
- **Prune before you probe.** convergent-review runs before the evidence probes so
  it can delete decisions argument can settle — you don't spend a spike on a
  decision YAGNI will cut. The most existential probe (does it already exist?) is
  already answered by stage 3, before any hardening.
- **The arrow is not one-way.** An evidence probe that kills or changes a decision
  sends you back to the Decision Record, then re-harden only what moved.

## Decision Record (owned artifact)

Write it as the lead section of design-spec's spec file (the one already at
`docs/specs/YYYY-MM-DD-<topic>-design.md`), so there is a single
artifact that convergent-review hardens and implementation-plan reads. The rest of
design-spec's design sits below it as the supporting detail.

```markdown
## Decision Record

### D<n>: <decision title>
- **Decision:** <what was chosen>
- **Alternatives considered:** <the real options — from design-spec's approaches
  and prior-art's candidates, not strawmen>
- **Why this one:** <reason>
- **Load-bearing assumption:** <the one thing that must be true for this to be right>
- **Falsifier:** <the observation that would prove it wrong>
- **Resolved by:** argument | evidence
- **Cheapest probe** (only if evidence): <spike / benchmark / search / interview> ·
  <rough effort, e.g. 1h / half-day> · fatal to the design? <yes/no>
```

The `Resolved by` flag is the router: `argument` decisions go to convergent-review;
`evidence` decisions go to the De-risking Queue. The build-vs-adopt verdict from
prior-art is itself a decision here — and prior-art was its cheapest probe.

## De-risking Queue (owned artifact)

Only the decisions flagged `Resolved by: evidence`. Append it to the same file.
Order so the probes that could cheaply kill the whole design sit at the top — run
top-down and stop the moment a fatal probe comes back negative.

```markdown
## De-risking Queue

| # | Assumption (Dn) | Cheapest probe | Effort | Fatal? | Result | → next |
|---|---|---|---|---|---|---|
```

Harvested from `idea-to-test-map`'s mechanism (the one-thing-that-must-be-true +
cheapest-resolving-test + cheapest-most-fatal-first ordering), generalized from
market bets to technical assumptions. The venture framing, the Alive/Contingent/Dead
labels, and the HTML render are intentionally dropped — a markdown table is enough.

This stage is conditional: if every decision is `Resolved by: argument` (settled by
reasoning plus prior-art), there is nothing to probe — skip it. Don't manufacture a
test for a decision argument already closed.

## Final report

End with: which path prior-art recommended, the convergent-review trajectory
(e.g. 5 → 1 → 0), any decision a probe changed, and the spec path — then the
implementation-plan handoff.
