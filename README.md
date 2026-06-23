# research-rigor

A rigor toolkit for researchers and research engineers, packaged as a Claude Code plugin. It covers the full arc of disciplined work, from deciding what to build to hardening how you build it, and ships the critic agents that drive the review. It is self-contained: no dependency on another plugin at runtime.

| Phase | Skill | What it does |
|---|---|---|
| Decide what to build | [`design-spec`](skills/design-spec/SKILL.md) | Brainstorms intent, requirements, and design through one-question-at-a-time dialogue, proposes 2-3 approaches, and writes an approved design spec. The WHAT, before any HOW. |
| Don't rebuild it | [`prior-art-check`](skills/prior-art-check/SKILL.md) | Searches GitHub, npm/PyPI, HuggingFace, MCP catalogs, managed APIs, and academic prior art, then returns a scored Adopt / Extend / Compose / Build recommendation. |
| Literature search | [`research-papers`](skills/research-papers/SKILL.md) | Turns a topic into a defensible literature search across arXiv, OpenAlex, DBLP, Semantic Scholar, PubMed, and SSRN, with full-text fetch, citation-graph ranking, and a non-skip reading protocol. |
| Harden the decisions | [`convergent-review`](skills/convergent-review/SKILL.md) | Runs fresh, diverse, non-anchored critics in rounds until a spec, plan, diff, or document genuinely converges, instead of trusting the first reviewer's "looks good." |
| Plan the build | [`implementation-plan`](skills/implementation-plan/SKILL.md) | Turns a spec into a step-by-step, reviewable plan of bite-sized, independently testable tasks, then hands off to execution. |
| Run the whole arc | [`project-kickoff`](skills/project-kickoff/SKILL.md) | Orchestrates the above: design-spec, prior-art, a Decision Record, convergent-review to harden by argument, cheapest-first de-risking, then implementation-plan. |

The plugin also ships two purpose-built critic agents used by `convergent-review`: [`oracle`](agents/oracle.md) (strategic lens) and [`council`](agents/council.md) (adversarial two-pass lens).

## Install

```
/plugin marketplace add <your-org>/research-rigor
/plugin install research-rigor
```

Or vendor it into a project by copying the `skills/` and `agents/` folders into the project's `.claude/` directory. Each skill also works on its own if you copy just its folder into `~/.claude/skills/`.

Invoke a skill by describing the task ("does this already exist?", "find the top papers on X", "harden this spec until it converges") or by naming it directly.

## Optional capabilities (auto-detected)

Every external provider is optional. The skills inspect what is available at runtime, prefer the strongest option present, and fall back to built-in tools when nothing is connected. Nothing here is required to install or run the plugin, but provisioning these gives better results.

| Capability | Used by | If absent, falls back to | Recommended to provision |
|---|---|---|---|
| `gh` CLI (authenticated) | prior-art-check | `WebSearch site:github.com` | Yes. GitHub is the primary prior-art surface. |
| Exa or Perplexity MCP, or any web-search MCP | oracle, prior-art-check, research-papers | built-in `WebSearch` / `WebFetch` | Optional. Richer, ranked semantic search. |
| OpenAlex polite-pool email (`OPENALEX_EMAIL`) | research-papers, prior-art-check | keyless common pool (slower) | Yes. Free, just an email, roughly doubles full-text recovery. |
| HuggingFace public API (`curl`) | prior-art-check | `WebFetch` of huggingface.co | Optional, keyless. |
| Python deps for full-text parsing (`pymupdf4llm`, `Pillow`) | research-papers | `pdftotext`, then abstract-only | Yes. The script bootstraps its own virtualenv on first use. |
| `SEMANTIC_SCHOLAR_API_KEY`, `NCBI_API_KEY` | research-papers | keyless pools (rate-limited) | Optional, higher rate limits. |
| Managed-API catalogs (RapidAPI, Composio, Pipedream, Zapier) | prior-art-check | skipped with a note | Optional, for the "API for cents" angle. |

No paid inference API is ever called. The optional providers above are free or free-tier by default; paid managed APIs are only ever surfaced as candidates for you to evaluate, never invoked.

To wire a provider whose tool name the agents do not already list (for example a self-hosted or differently-named search MCP), add that tool's name to the `tools:` line in [`agents/oracle.md`](agents/oracle.md).

## The critics

`convergent-review` dispatches the two bundled agents as a diverse pair so each round gets real lens diversity rather than two copies of the same reviewer.

| Agent | Lens | Tools |
|---|---|---|
| `oracle` | Strategic and architectural: overclaiming, simplicity, feasibility, internal consistency, over-cut. | Read, Grep, Glob, Bash, web search (built-in or semantic MCP) |
| `council` | Adversarial two-pass: correctness, regression, edge cases, coverage, security, then a self-challenge of its own findings. | Read, Grep, Glob, Bash |

## convergent-review defaults

| Parameter | Default | Note |
|---|---|---|
| K (consecutive clean rounds) | 2 | Raise to 3 for high-stakes work. |
| MAX_ROUNDS | 6 | Hard cap. Iteration helps early, then plateaus. |
| Critics per round | 2 | 1 for small targets, 3 for large. Always diverse lenses. |

The defaults come from published findings on iterative LLM refinement:

- Diverse critics beat one repeated reflector: [MAR: Multi-Agent Reflexion](https://arxiv.org/abs/2512.20845).
- Unguided homogeneous debate underperforms: [The Cost of Consensus](https://arxiv.org/abs/2605.00914).
- External feedback reduces self-bias: [Pride and Prejudice](https://aclanthology.org/2024.acl-long.826/).
- Anchoring on a prior draft can mislead the reviewer: [Revision or Re-Solving?](https://arxiv.org/abs/2604.01029).
- Over-refinement degrades good work, so a stopping rule matters: [MAgICoRe](https://arxiv.org/abs/2409.12147), [Self-Correction as Feedback Control](https://arxiv.org/abs/2604.22273).
- Gains arrive early then flatten: [Another Turn, Better Output?](https://arxiv.org/abs/2509.06770).

## License

[MIT](LICENSE). Third-party attributions are in [NOTICE](NOTICE).

## Credits

- `design-spec` and `implementation-plan` are adapted from the `brainstorming`, `writing-plans`, and `executing-plans` skills in [obra/superpowers](https://github.com/obra/superpowers) (MIT), decoupled to run self-contained, with spec and task discipline influenced by [github/spec-kit](https://github.com/github/spec-kit) (MIT).
- `prior-art-check` composes workflow and decision-matrix ideas from `affaan-m/ECC`, `mturac/skill-hunter`, `hashicorp/agent-skills`, `ComposioHQ/awesome-claude-skills`, and `VoltAgent/awesome-agent-skills`. See the skill's Provenance section.
