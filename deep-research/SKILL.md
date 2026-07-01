---
name: deep-research
description: Use when the user asks for a deep research pass, source discovery, literature scan, market scan, research dossier, or wants to understand a topic beyond a single quick answer. Runs a depth-tiered external research loop with multiple query angles, gap analysis, source ranking, and a cited final dossier.
---

# Deep Research

Run a depth-tiered external research loop, then produce one ranked, cited research dossier. This is for research that needs breadth, source quality, and follow-up rounds, not a single-answer lookup.

**Core principle:** discovery is a loop with rounds. Do not search one phrase, summarize the first few links, and stop. Break the topic into angles, research them separately, identify gaps, run follow-up queries, deduplicate, rank, and synthesize with citations.

## When to use

Use this skill when the user asks for:

- A deep research pass on a topic
- A research dossier for an article, video, memo, project, or product decision
- Source discovery across web pages, papers, docs, GitHub repos, videos, or reports
- A comparison of companies, tools, technical approaches, markets, or trends
- A research-backed answer where citations and confidence matter

Do **not** use this for a quick factual lookup. If the user only needs one fact, answer directly with a normal search.

## Step 0 — Pick a depth

Research can get expensive, so choose a depth. If the user clearly asks for a specific depth, proceed with that. Otherwise default to `light`.

| Preset | Rounds | Queries per round | Total queries | Use when |
|---|---:|---:|---:|---|
| `fast` | 1 | `[3]` | 3 | Quick source scan or familiar topic |
| `light` | 2 | `[3, 2]` | 5 | Most research requests |
| `deep` | 3 | `[3, 3, 3]` | 9 | Broad, unfamiliar, high-stakes, or source-heavy topics |

If scope is unclear, ask one short question:

> Research **<topic>** at which depth: **fast** (3 queries), **light** (5), or **deep** (9)?

## Step 1 — Frame the topic

Read the request and extract:

- Core topic
- Goal or intended use
- 3–5 key themes
- Named entities: people, companies, tools, papers, projects, laws, dates, competitors
- Constraints: time range, geography, industry, technical level, source types to prefer or avoid
- Any seed links the user provided

Then generate round-1 queries from different research angles. Do not just vary the wording of the same query.

Good angles include:

- **Direct terms:** the obvious phrase or topic name
- **Related concepts:** adjacent ideas, enabling technologies, or parent categories
- **Named entities:** companies, authors, projects, products, competitors, papers
- **Alternate framings:** how critics, users, vendors, academics, or operators would describe it
- **Evidence queries:** benchmarks, case studies, lawsuits, incidents, adoption data, pricing, limitations
- **Counterpoint queries:** failures, criticism, risks, skepticism, negative results

Write the queries down before dispatching research.

## Step 2 — Run discovery rounds

For each round, run one research thread per query. If the environment supports subagents or parallel tool calls, run the threads in parallel. If not, run them sequentially.

Each research thread should:

1. Search the web for its query.
2. Prefer primary sources when available: official docs, papers, standards, filings, changelogs, source repos, interviews, talks, company blogs with concrete details.
3. Use secondary sources for context: credible journalism, analyst reports, explainers, tutorials, user discussions.
4. Avoid low-signal SEO pages unless they point to better sources.
5. Extract or read the strongest sources, not just titles and snippets.
6. Return no more than 15 findings.

Each finding should use this shape:

```json
{
  "title": "Source title",
  "source_type": "paper|docs|github|news|blog|report|video|forum|dataset|other",
  "url": "https://...",
  "relevance": "high|medium|low",
  "summary": "2-3 sentences focused on what this source contributes.",
  "key_concepts": ["concept 1", "concept 2"],
  "evidence_quality": "primary|secondary|anecdotal|unknown"
}
```

### Gap analysis between rounds

After every round except the last:

1. Deduplicate findings collected so far.
2. Ask what is still thin, missing, contradictory, outdated, or over-reliant on one source type.
3. Generate exactly the next round's query count from the preset.
4. Make follow-up queries sharper than round 1.

Follow-up queries should target gaps like:

- Missing primary sources
- Missing criticism or limitations
- Missing implementation detail
- Missing adoption, market, pricing, or usage data
- Claims that need verification
- New entities discovered in earlier rounds

## Step 3 — Deduplicate and rank sources

Deduplicate by URL and by obvious duplicates, such as the same paper on arXiv and a mirror, or the same announcement syndicated across multiple sites.

Rank sources using:

- User-provided seed links: 1.0
- High relevance primary source: 0.9
- High relevance secondary source: 0.8
- Medium relevance primary source: 0.7
- Medium relevance secondary source: 0.5
- Low relevance or anecdotal source: 0.2

Keep the top 8–12 sources at full detail. Demote the rest to one-line summaries so the dossier stays useful instead of becoming a link dump.

## Step 4 — Synthesize the dossier

Produce one research dossier in the final answer or save it to a file if the user asked for an artifact.

Use this structure:

```markdown
# <Topic> — Research Dossier

> One-line summary of what this research covers and why it matters.

## Executive summary
A concise synthesis of the main answer. State what the evidence says, what is uncertain, and what changed your view.

## Key findings
- <Finding> — source: <title/link>
- <Finding> — source: <title/link>

## What is proven vs. still speculative
- **Validated / well-supported:** <claims backed by primary or multiple credible sources>
- **Promising but not settled:** <claims with partial evidence>
- **Hype / weak evidence:** <claims that are vendor-led, anecdotal, or unsupported>

## Practical implications
Translate the research into concrete advice for the user's stated goal. Include what to try, avoid, compare, build, buy, monitor, or investigate next.

## Sources ranked
1. <Title> — score <0.0-1.0> — <source_type> — <url> — why it matters
2. <Title> — score <0.0-1.0> — <source_type> — <url> — why it matters

## Open questions
- <Question the research did not fully answer> — what would answer it
- <Question the research did not fully answer> — what would answer it
```

## Quality rules

- Attribute claims. Do not present borrowed claims as your own conclusions.
- Separate evidence from inference.
- Prefer primary sources over summaries.
- Include dissenting or negative evidence when it exists.
- Do not bury uncertainty. Say what the research could not resolve.
- Do not overfit to the first search results.
- Do not dump every source at full length. Rank first, then summarize.
- Do not use fake citations or inaccessible sources you did not actually inspect.

## Common mistakes

- **One-pass research.** The point is rounds plus gap analysis.
- **Same query three ways.** Research angles should be meaningfully different.
- **No primary sources.** If the topic has docs, papers, filings, repos, or official data, use them.
- **No skepticism pass.** Every serious topic needs limitations, criticism, failures, or counter-evidence.
- **Link dump output.** The user needs synthesis and ranked sources, not a pile of tabs.
- **Unsupported confidence.** Strong language requires strong evidence.
