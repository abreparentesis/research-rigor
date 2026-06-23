# convergent-review

A Claude Code plugin that hardens a design spec, plan, PR diff, or document by running adversarial review in rounds until it genuinely converges, instead of trusting one reviewer's first "looks good."

A single reviewer, or the same reviewer run twice, tends to anchor on its earlier read and rubber-stamp the work. This plugin counters that with three rules: review with separate critic agents rather than self-review, keep each round's critic blind to prior verdicts, and rotate the analytical lens between rounds. It stops only after K consecutive clean rounds and caps the total at MAX_ROUNDS, so it does not over-refine work that is already good.

The package ships the [skill](skills/convergent-review/SKILL.md) plus two purpose-built critic agents, [`oracle`](agents/oracle.md) (strategic lens) and [`council`](agents/council.md) (adversarial two-pass lens), that already speak the review protocol.

## What it does

1. Dispatch fresh critic subagents for the round, each assigned a distinct lens and told that "converged" is a valid answer.
2. Curate their findings: dedupe, drop cosmetic nits, reject scope creep, keep only the material ones.
3. If anything material remains, apply the fixes to the target and start a new round on the revised version.
4. Repeat until K rounds in a row come back clean, then report the per-round trajectory and the converged result.

## Install

As a plugin (registers the skill and both agents):

```
/plugin marketplace add <your-org>/convergent-review
/plugin install convergent-review
```

Or vendor it into a project by copying the `skills/` and `agents/` folders into the project's `.claude/` directory.

Skill only, without the bundled critics: copy `skills/convergent-review/` into `~/.claude/skills/` (personal) or `.claude/skills/` (project). The skill then falls back to the built-in `general-purpose` agent for its critics.

Invoke it by asking to "harden this," "review until converged," "stress-test this design," or by naming the skill directly.

## The critics

The two bundled agents default to Opus and need no configuration. The skill dispatches them as a diverse pair so each round gets real lens diversity rather than two copies of the same reviewer.

| Agent | Lens | Tools |
|---|---|---|
| `oracle` | Strategic and architectural: overclaiming, simplicity, feasibility, internal consistency, over-cut. | Read, Grep, Glob, Bash, WebSearch, WebFetch |
| `council` | Adversarial two-pass: correctness, regression, edge cases, coverage, security, then a self-challenge of its own findings. | Read, Grep, Glob, Bash |

Both emit the same protocol the skill curates: a `VERDICT`, a ranked list of material-only findings with exact locations and one-line fixes, and a one-line biggest risk.

## Defaults

| Parameter | Default | Note |
|---|---|---|
| K (consecutive clean rounds) | 2 | Raise to 3 for high-stakes work. |
| MAX_ROUNDS | 6 | Hard cap. Iteration helps early, then plateaus. |
| Critics per round | 2 | 1 for small targets, 3 for large. Always diverse lenses. |

## Evidence

The defaults come from published findings on iterative LLM refinement:

- Diverse critics beat one repeated reflector: [MAR: Multi-Agent Reflexion](https://arxiv.org/abs/2512.20845).
- Unguided homogeneous debate underperforms: [The Cost of Consensus](https://arxiv.org/abs/2605.00914).
- External feedback reduces self-bias: [Pride and Prejudice](https://aclanthology.org/2024.acl-long.826/).
- Anchoring on a prior draft can mislead the reviewer: [Revision or Re-Solving?](https://arxiv.org/abs/2604.01029).
- Over-refinement degrades good work, so a stopping rule matters: [MAgICoRe](https://arxiv.org/abs/2409.12147), [Self-Correction as Feedback Control](https://arxiv.org/abs/2604.22273).
- Gains arrive early then flatten: [Another Turn, Better Output?](https://arxiv.org/abs/2509.06770).

## License

[MIT](LICENSE).
