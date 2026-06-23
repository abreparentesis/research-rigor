---
name: convergent-review
description: >-
  Use when hardening a design spec, plan, PR diff, or document to genuine
  convergence, not accepting the first reviewer's "looks good." Runs fresh,
  diverse, non-anchored critics in rounds, curates their findings, applies the
  material ones, and stops only after K consecutive CLEAN rounds. Triggers:
  "harden this", "review until converged", "don't accept the first oracle",
  "keep reviewing until clean", "adversarial convergence", "tighten this
  spec/plan/diff", "loop the reviewer", "stress-test this design".
---

# Convergent Review

Iterative adversarial review that runs until it genuinely converges, instead of
trusting one reviewer's first "CONVERGED", which is unreliable because a single
or repeated reviewer **anchors** and rubber-stamps.

**Announce at start:** "I'm using convergent-review to harden <target> until K consecutive clean rounds."

## When to use
- Hardening a spec, plan, PR diff, doc, prompt, or schema before committing to it.
- Any time quality matters more than speed and you suspect the first review is too generous.

## When NOT to use
- Trivial or already-shipped artifacts (over-refinement degrades good work; see Evidence).
- When there is no editable target to revise between rounds.

## Why rounds + fresh + diverse (the three rules that make it work)
1. **External, not self.** A model reviewing its own work amplifies self-bias. Use separate critic agents.
2. **Fresh + non-anchored.** Each round, the critic must NOT see prior verdicts or prior critics' findings. Anchoring drags reviewers onto brittle "it's basically fine" trajectories.
3. **Diverse, structured lenses.** Rotate the angle each round (and run a few in parallel within a round). Identical critics produce sycophantic false consensus; diverse roles find what repetition can't.

## The algorithm

Two nested levels. The OUTER loop is sequential (each round reviews the *revised* artifact). The INNER round is your orchestrator: parallel critics + a synthesis step.

```
cleanRounds = 0 ; round = 0 ; trajectory = []
while cleanRounds < K and round < MAX_ROUNDS:
    round++
    # INNER (parallel): dispatch 1–3 FRESH critics in ONE message so they run concurrently.
    # Each critic: blind to prior rounds, assigned the next lens(es), told CONVERGED is a valid answer.
    findings = parallel(critics for this round's lenses)
    # SYNTHESIZE + CURATE (you, the main loop; this is the load-bearing step):
    #   dedup; DROP nits and cosmetic preferences; REJECT anything that adds bloat /
    #   feature-creep / Phase-2 scope; keep only MATERIAL findings; rank them.
    material = curate(findings)
    trajectory.push(material.count)
    if material is empty:
        cleanRounds++
    else:
        cleanRounds = 0
        apply the curated fixes to the target ; commit
# Report the trajectory (e.g. 5 → 6 → 1 → 0) and the converged target.
```

## Parameters & defaults (evidence-based)
- **K = 2** consecutive clean rounds (one clean round is false-convergence-prone). Raise to 3 for high-stakes.
- **MAX_ROUNDS = 6** hard cap (iteration helps early, then plateaus or harms, so stop).
- **Critics per round = 2** by default, 1 for small targets, 3 for big ones. Use diverse lenses, never identical.
- **Bias = cut.** Prefer simplification; treat "smart simple beats feature-bloated" as the prime directive unless told otherwise.
- **Convergence is a valid verdict.** Instruct every critic NOT to manufacture findings to seem useful, and to flag when a proposed fix would add bloat.

## Lens library (rotate; pick per artifact)
- **Design / spec:** trust-of-the-signal (no overclaiming), simplicity / anti-bloat, engagement / user-value, build-feasibility, **over-cut** (did simplification break something load-bearing?), internal-consistency / dangling-references.
- **Plan:** spec-coverage (every requirement → a task), placeholder/ambiguity, type/interface consistency, task right-sizing, testability.
- **Code / diff:** correctness & edge cases, security, performance, reuse/duplication, error handling, test coverage.
- **Prose / doc:** claim accuracy, structure/flow, audience fit, cut-the-fluff, internal contradiction.

Always include an **over-cut / "did a prior fix break something?"** lens after the second round: the loop's own cutting bias needs a counterweight.

## Critic dispatch protocol
Dispatch fresh subagents with the Agent tool, one per assigned lens. This package ships two purpose-built critics that already speak this protocol: `oracle` (strategic lens) and `council` (adversarial two-pass lens). Use them as the default pair. If you copied `SKILL.md` on its own without the bundled agents, fall back to `general-purpose`, which exists in every Claude Code install. In each critic prompt:
- Give the target path and the assigned lens(es). Do **not** reveal prior rounds' verdicts or findings.
- Demand the structure: `VERDICT: CONVERGED | IMPROVEMENTS REMAIN`; a ranked list of **material-only** findings, each tagged `[CUT]/[MERGE]/[CHANGE]/[SHARPEN]/[ADD]` + axis + exact location + one-line fix; and a one-line biggest-risk.
- Hard rules in the prompt: "material only; no cosmetic nits; no speculative features; if a fix adds bloat say so; CONVERGED is the correct answer if nothing material remains; do not invent work."
- For breadth in one round, emit multiple Agent calls in a **single message** so they run in parallel; then you synthesize.
- **Search provider (optional, auto-detected):** the `oracle` critic verifies external claims with whatever search is installed, preferring a semantic-search MCP (Exa, Perplexity, or any other web-search MCP) and falling back to built-in `WebSearch`/`WebFetch`. Never required; if nothing is connected, the built-ins are used.

## Synthesis & curation (the part you must own)
This is why this is a Skill, not a hands-off Workflow: the main loop curates with full context and the user in the loop.
- Merge duplicate findings across critics; a finding two lenses independently raise is high-signal.
- **Reject** feature-creep, scope-creep (defer to a later phase), and cosmetic wording. Apply the rest.
- When critics disagree on severity, you decide, and you may overrule a finding you judge wrong (record why).
- Apply fixes as a clean revision; re-verify deterministically (grep/test) that each fix landed without new contradictions; commit per round.

## Final report
End with: the per-round findings trajectory, the converged artifact's location/commit, and a one-line "ready / not ready" with the last critic's verdict quoted.

## Optional: `--autonomous` (scale) mode
For unattended, at-scale hardening (e.g. many targets overnight, or many lenses × rounds with no human), and only if your harness exposes the `Workflow` tool, drive the loop with its loop-until-dry pattern (deterministic K-consecutive-clean control flow + parallel critics + a synthesizer agent + a fixer agent). Trade-off: the fixer agent curates blind to the conversation, so it loses the nuanced bloat-rejection of main-loop curation; use only when no human is available to steer. Requires explicit multi-agent opt-in.

## Evidence (why these defaults)
- Diverse critics > a single repeated reflector: **MAR: Multi-Agent Reflexion** (arXiv 2512.20845); homogeneous unguided debate fails: **The Cost of Consensus** (arXiv 2605.00914).
- External feedback reduces self-bias: **Pride and Prejudice** (ACL 2024). Anchoring on prior drafts: **Revision or Re-Solving?** (arXiv 2604.01029).
- Over-refinement degrades good work; need a stopping rule + verify-first: **MAGICORE** (EMNLP 2025), **Self-Correction as Feedback Control** (arXiv 2604.22273). Iteration helps early then plateaus: **Another Turn, Better Output?** (arXiv 2509.06770).
