---
name: prior-art-check
description: Use AFTER scoping is clear (or as the structured-search phase inside `superpowers:brainstorming`) and BEFORE writing any custom code for a non-trivial feature, module, integration, library, service, or utility. Searches GitHub (PRIMARY), npm/PyPI, HuggingFace Hub (pretrained models, for ML/AI features), MCP server catalogs (ComposioHQ + VoltAgent awesome lists), managed-API providers (RapidAPI, Composio, Pipedream, Zapier), and — for algorithm- or research-shaped features only — academic prior art (EXA fast pass, escalating to the `research-papers` lit-review skill, with a keyless OpenAlex fallback), then returns a structured Adopt / Extend / Compose / Build recommendation with maturity signals. Complements brainstorming — brainstorming clarifies WHAT to build, prior-art-check determines IF it already exists.
---

# prior-art-check — Don't Build What Already Exists

Systematizes the "search for existing solutions before implementing" discipline.
Inherits the 5-phase workflow from `affaan-m/ECC` `search-first` and the explicit
decision matrix from `mturac/skill-hunter`, re-weighted so **GitHub is the primary
search surface** (OSS-first reuse as the default weighting).

## Relationship to `superpowers:brainstorming`

These skills CHAIN, they don't compete:

1. **`superpowers:brainstorming`** fires FIRST when build intent is vague. It
   clarifies *what* the user actually wants (intent, requirements, constraints,
   acceptance criteria). Its "Search Before Building" phase is a placeholder.
2. **`prior-art-check`** (this skill) fires NEXT, once scope is clear enough to
   query a registry. It IS the structured implementation of brainstorming's
   "Search Before Building" phase: real `gh search`, `npm search`, MCP catalog
   sweeps, managed-API checks, and a scored Adopt / Extend / Compose / Build
   recommendation.
3. **Code** comes after — informed by both phases.

If scope is already crisp (the user gave a specific feature with constraints),
skip straight to this skill. If scope is vague ("build me a chat app"),
brainstorming first.

## When to fire

**Auto-trigger** when the user expresses a build/integrate intent AND scope is
crisp enough to query a registry. Examples:

- "let's add real-time chat with auth" (after brainstorming has clarified scale, auth provider, persistence)
- "build a PDF parser for invoice extraction, must run offline"
- "implement rate limiting per-user, sliding window"
- "integrate Stripe billing — recurring subscriptions only"
- "set up a Redis-backed job queue for image resizing"

**Manual invocation:** `/prior-art <topic>` (skips brainstorming when scope is
already clear)

**Skip** if any of these are true:
- Single-line / typo / rename / format-only change
- Work is fully contained inside an existing module the user is already editing
- User explicitly said "skip the search, just build it"
- Task is exploratory question-answering, not implementation
- Scope is still vague — defer to `superpowers:brainstorming` to scope first

## Workflow

```
┌─────────────────────────────────────────────────────────┐
│  0. PREFLIGHT — what channels can I actually search?    │
│     Report missing tools honestly, don't silent-skip.    │
├─────────────────────────────────────────────────────────┤
│  1. SCOPE — what is actually needed?                    │
│     One-sentence problem statement.                      │
│     Language/runtime/framework constraints.              │
│     Hard constraints (license, offline, compliance).     │
├─────────────────────────────────────────────────────────┤
│  2. PARALLEL SEARCH (run ALL channels in one turn)      │
│     ┌────────────┐ ┌────────────┐ ┌────────────┐        │
│     │  GITHUB    │ │ npm / PyPI │ │   MCP /    │        │
│     │ (PRIMARY)  │ │            │ │   Skill    │        │
│     │            │ │            │ │  catalogs  │        │
│     └────────────┘ └────────────┘ └────────────┘        │
│                     ┌────────────┐                       │
│                     │  Managed   │                       │
│                     │ APIs (paid │                       │
│                     │  services) │                       │
│                     └────────────┘                       │
├─────────────────────────────────────────────────────────┤
│  3. RANK by maturity signals                            │
│     stars, last-push recency, contributor count,         │
│     license, fit-to-need, integration cost               │
├─────────────────────────────────────────────────────────┤
│  4. DECIDE — present matrix, name a recommendation      │
├─────────────────────────────────────────────────────────┤
│  5. IMPLEMENT — adopt / extend / compose / build        │
└─────────────────────────────────────────────────────────┘
```

## Phase 0 — Preflight

Before searching, check tool availability. Report each channel's status; do not
silently skip a channel that is unavailable.

| Channel | Check | If missing |
|---|---|---|
| GitHub | `gh auth status` | fall back to `WebSearch` site:github.com |
| npm | `npm --version` | use `WebFetch` of npmjs.com/search |
| PyPI | `python3 -m pip --version` | use `WebFetch` of pypi.org/search |
| HuggingFace | `curl -s 'https://huggingface.co/api/models?limit=1'` (public, no auth) | fall back to `WebFetch` of huggingface.co/models?search= |
| EXA (web) | `EXA_API_KEY` set or `mcp__exa__web_search_exa` loaded | fall back to `WebSearch` |
| Academic (gated) | EXA (`mcp__exa__web_search_exa`/`exa:search`) → the bundled `research-papers` skill → OpenAlex `curl -s 'https://api.openalex.org/works?search=test'` (keyless) | none available → `WebSearch` arxiv.org |
| MCP catalogs | `gh` to list contents | skip with note |

## Phase 1 — Scope

State, in one sentence each:
1. What functionality is needed
2. Hard constraints (language, license, runtime, offline, compliance, budget)
3. What "good enough" looks like (acceptance criteria)

Do NOT proceed past this phase if the scope is ambiguous. Ask the user.

## Phase 2 — Parallel search

Issue **all of these in a single tool-call turn** so they run concurrently. Time
budget: 30-90 seconds for the whole phase.

### a) GitHub (PRIMARY — heaviest weight)

Run at least two of these in parallel:

```bash
# Topic + maturity filter — prefer maintained, real projects
gh search repos "<topic>" --sort=stars --limit=15
gh search repos "<topic>" --language=<lang> --sort=updated --limit=10

# Code search — finds projects that EXPOSE the API you want
gh search code "<distinctive function or concept>" --limit=20
```

For each candidate, fetch stars, last-push date, license, open-issue count via
`gh api repos/{owner}/{repo}`. Drop anything not pushed in the last 12 months
unless its star count alone justifies it.

### b) Package registries (npm + PyPI)

```bash
# npm — keyword search, sorted by quality
npm search "<topic>" --searchlimit 15

# PyPI — uses pip's search-via-pypi.org fallback or WebFetch
python3 -m pip index versions <best-guess-package-name> 2>&1 | head
# Or:
# WebFetch  https://pypi.org/search/?q=<topic>
```

Cross-check with GitHub: the npm/PyPI package often points to a GitHub repo —
use that as a signal of maintenance and adoption.

### c) HuggingFace Hub (pretrained models)

If the feature involves ML/AI — NER, classification, embeddings, transcription,
translation, summarization, vision, OCR, etc. — a pretrained model very likely
already exists, and adopting one beats training from scratch. Search the public
Hub API (returns JSON, no auth needed):

```bash
# Model search, ranked by downloads (adoption proxy)
curl -s "https://huggingface.co/api/models?search=<topic>&sort=downloads&direction=-1&limit=15" \
  | jq -r '.[] | "\(.id)\tdownloads=\(.downloads)\tlikes=\(.likes)\ttask=\(.pipeline_tag)"'

# Narrow by task when you know it (token-classification, text-classification,
# automatic-speech-recognition, translation, summarization, image-classification…)
curl -s "https://huggingface.co/api/models?pipeline_tag=token-classification&search=<topic>&sort=downloads&direction=-1&limit=15" \
  | jq -r '.[] | "\(.id)\tdownloads=\(.downloads)\tlikes=\(.likes)"'

# Drill into one model's metadata: license, recency, task, size
curl -s "https://huggingface.co/api/models/<org>/<model>" \
  | jq '{id, downloads, likes, lastModified, pipeline_tag, license: .cardData.license, library: .library_name}'
```

Fallback if `curl`/`jq` unavailable: `WebFetch https://huggingface.co/models?search=<topic>&sort=downloads`.

Capture per candidate: **downloads** (adoption), **likes**, **license**
(`cardData.license`), **lastModified** (maintenance), **pipeline_tag** (task fit),
and model **size in params** where deployment cost matters. These map directly
onto the Phase 3 axes — downloads→Adoption, lastModified→Maintenance,
license→License, pipeline_tag/size→Fit. Beware permissive-looking models with
non-commercial or gated licenses (e.g. some Llama/CC-BY-NC weights); check
`cardData.license` before recommending Adopt.

### d) MCP servers + Claude skills (catalog search)

The two large catalogs (verified May 2026): ComposioHQ's awesome list (~61k⭐)
and VoltAgent's (~22k⭐). Search both:

```bash
# Search Composio's curated catalog
gh search code "<topic>" --repo=ComposioHQ/awesome-claude-skills --limit=10

# Search VoltAgent's
gh search code "<topic>" --repo=VoltAgent/awesome-agent-skills --limit=10

# Also check your own installed skills/MCP first
ls ~/.claude/skills/ ~/.codex/skills/ 2>/dev/null
cat ~/.claude/.mcp.json .mcp.json 2>/dev/null
```

### e) Managed APIs (the "API for cents" angle)

If the feature is the kind of thing a SaaS provider offers (auth, email, OCR,
geocoding, payments, search, translation, transcription, moderation):

```bash
# RapidAPI hub
# WebFetch  https://rapidapi.com/search/<topic>

# Composio catalog
# WebFetch  https://composio.dev/integrations  (search the page)

# Pipedream / Zapier app catalogs for integration-shaped problems
```

Capture: pricing per call/month, free tier, SLA, compliance (SOC2/HIPAA if user
mentioned).

### f) Academic / algorithmic prior art (GATED — EXA first, escalate as needed)

Fire this channel ONLY when the feature is **algorithm- or research-shaped** —
a novel method, a non-obvious data structure, an ML/AI approach, or anything
where the right "existing solution" is a published *method* rather than an
installable package. Skip it for plumbing (CRUD, integrations, infra glue,
known-SaaS-shaped problems) — there, papers are noise.

This channel does NOT return an Adopt target. It informs the **Build** path:
when nothing installable fits, the seminal method tells you *what* to implement
(and often links a reference implementation) instead of reinventing it.

Three tiers, cheapest first — stop as soon as you have the answer:

**1. Fast pass — EXA** (`mcp__exa__web_search_exa`, or the `exa:search` skill).
Already available, no key, no scholarly rate limits. One semantic query usually
surfaces the key papers, the canonical repo, AND the surrounding discussion in a
single shot. For "is there a known approach to <X>?" this is normally enough.
EXA's blind spot: it has **no citation graph** — it cannot tell you which paper
is the *most influential* or trace what cites what. When that matters, escalate.

**2. Rigorous pass — the bundled `research-papers` skill** (ships in this
plugin; invoke it by name). Runs as an
isolated Opus subagent across arXiv, OpenAlex, DBLP, Semantic Scholar, PubMed,
SSRN with **influence/recency ranking, citation-graph traversal, dedup, and
full-text fetch**. Use it when you need to rank candidate methods by influence
or pin down *the* canonical reference, not just relevant hits. Trigger with the
topic as a method search: *"find and compare published methods for <topic>"*.
(It already leans on OpenAlex, so it needs no Semantic Scholar key.)

**3. Keyless API fallback — OpenAlex** (no EXA, no skill). OpenAlex (OurResearch;
240M+ works) is genuinely keyless — just add your email for the faster "polite
pool", no form, no approval — and it carries the same citation-graph data as
Semantic Scholar across a broader corpus:

```bash
# Influence-ranked search, keyless polite pool
curl -s "https://api.openalex.org/works?search=<topic>&sort=cited_by_count:desc&per_page=15&mailto=you@example.com" \
  | jq -r '.results[] | "\(.cited_by_count)cites\t\(.publication_year)\t\(.title)\tdoi=\(.doi // "-")\toa=\(.open_access.oa_url // "-")"'
```

(Semantic Scholar's `/graph/v1/paper/search` also works key-less but shares a
rate pool that 429s; its key is request-by-form and dev-oriented, not strictly
academic — OpenAlex sidesteps the question, so prefer it.)

**Second hop (all tiers) — bridge papers to code.** Papers aren't adoptable, so
take the top result's title / arXiv ID / DOI and run a GitHub pass (`gh search
repos "<title or method> implementation"`) to find the reference or community
implementation. Papers With Code, the old paper→code linker, was sunset in 2025,
so this hop is manual.

Map onto the Phase 3 axes: `cited_by_count` → Adoption, `publication_year` →
Maintenance/recency, title/abstract → fast Fit triage, `doi` / `open_access` →
the route to a usable implementation.

## Phase 3 — Rank

For each candidate, score on these axes (1-5):

| Axis | What it means |
|---|---|
| Fit | Does it actually do what we need, with our language/runtime? |
| Maintenance | Pushed in last 6 months? Active issues? Contributor count? |
| Adoption | Stars / downloads / dependents — proxies for "battle-tested" |
| License | MIT / Apache / BSD = green. GPL / AGPL = think before adopting. No license = avoid. |
| Integration cost | How much glue code? Does it pull in 50 transitive deps? |
| Risk | Single-maintainer? Last commit 3 years ago? Known CVEs? |

Drop anything scoring ≤2 on Fit or Maintenance. Keep at most the top 5.

## Phase 4 — Decide

Present a compact table to the user:

```
Candidates:
| Repo / Package | Stars | Pushed | License | Score | Fit notes |
|---|---|---|---|---|---|
| ...
```

Pick a path:

| Path | When |
|---|---|
| **Adopt as-is** | Exact match, well-maintained, permissive license |
| **Extend / wrap** | Strong foundation, needs a thin layer for project specifics |
| **Compose** | Two or three small packages each cover part of the problem |
| **Build custom** | Nothing suitable, OR fit < 3 across all candidates, OR all licenses block |

State the recommendation in one sentence with the reason. If recommending Build,
explicitly say *why* every candidate failed — this is the case the user is most
worried about.

## Phase 5 — Implement

- **Adopt:** `npm install` / `pip install` / `gh repo clone` and use directly
- **Extend:** install + write a thin wrapper module in the project; commit the
  wrapper, not vendored source
- **Compose:** install each, write a small orchestrator
- **Build:** proceed to the original implementation request, but with the
  research informing design choices

In all cases, drop a one-line comment at the top of any new file pointing to the
research finding (e.g., `# chose httpx over requests because: native async + same API`).

## Anti-patterns to refuse

- **Jumping to code** without searching — defeats the whole skill
- **Silent skipping** — if a channel was unavailable, say so in the report
- **Stars-only ranking** — a 50k⭐ abandoned repo loses to a 200⭐ actively-maintained one
- **Recommending the user's first guess** without listing alternatives
- **Over-wrapping** — adopting a library and then writing a 500-line abstraction over it
- **Ignoring managed APIs** — sometimes $0.001/call beats any OSS option

## Output format

Always end with this exact block so the user can scan it:

```
PRIOR-ART REPORT
================
Scope:        <one sentence>
Searched:     GitHub ✓ | npm ✓ | PyPI ✓ | HuggingFace ✓ | MCP catalogs ✓ | RapidAPI ✓ | Academic (EXA/research-papers/OpenAlex) ✓/n-a
Top candidates:
  1. <name> (<stars>⭐, pushed <date>) — score X/5 — <fit notes>
  2. ...
Recommendation: ADOPT | EXTEND | COMPOSE | BUILD
Reason:       <one or two sentences>
Next step:    <exact install command OR "proceed with custom build">
```

## Provenance

Composed from:
- `affaan-m/ECC` `skills/search-first/SKILL.md` (5-phase workflow, preflight)
- `mturac/skill-hunter` (Adopt/Extend/Compose/Build matrix, scoring axes)
- `hashicorp/agent-skills` (reuse-first design pattern at production scale)
- `ComposioHQ/awesome-claude-skills` + `VoltAgent/awesome-agent-skills` (catalog targets)

Author: Sebastian, 2026.
