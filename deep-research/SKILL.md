---
name: deep-research
description: Use when the user asks for a deep research pass, source discovery, literature scan, market scan, comparison, or cited dossier that needs more than a quick lookup. Runs an external research loop: query angles, discovery rounds, gap queries, source ranking, and synthesis.
---

# Deep Research

Run an external research loop that turns a broad question into a cited dossier.

**Loop:** frame -> angle queries -> research threads -> gap queries -> rank -> synthesize.

Use this for research that needs source quality, disagreement, and follow-up rounds. Do not use it for one factual lookup.

## 0. Choose depth

Default to `light` unless the user asks otherwise.

| Depth | Rounds | Queries | Use when |
|---|---:|---:|---|
| `fast` | 1 | 3 | Quick scan |
| `light` | 2 | 3, then 2 | Normal dossier |
| `deep` | 3 | 3, then 3, then 3 | Broad, unfamiliar, or high-stakes topic |

If depth changes the cost or runtime materially, ask once:

> Research **<topic>** at **fast**, **light**, or **deep** depth?

Completion: depth is chosen and the query count for each round is known.

## 1. Frame the request

Extract:

- Topic
- User's goal
- Constraints: time range, geography, industry, source types, exclusions
- Seed links or named sources
- 3–5 themes
- Named entities: people, companies, products, papers, projects, laws, competitors

Completion: you can say what decision, article, comparison, or understanding the research should support.

## 2. Create angle queries

Do not search the same phrase three ways. Create distinct angles.

Use these angle types:

- **Direct:** the obvious topic phrase
- **Adjacent:** parent categories, enabling technologies, related concepts
- **Entity:** named companies, authors, tools, papers, competitors
- **Evidence:** benchmarks, case studies, filings, datasets, pricing, adoption, incidents
- **Counterpoint:** criticism, failures, limitations, risks, skepticism
- **Operator:** implementation details, workflows, how people actually use it

For each query, write:

```json
{"query":"...","angle":"direct|adjacent|entity|evidence|counterpoint|operator","why":"what this angle should uncover"}
```

Completion: round 1 has exactly the required number of non-overlapping queries.

## 3. Run discovery threads

For each query, run one research thread. Use parallel agents/tool calls if available; otherwise run them sequentially.

Each thread:

1. Searches the web for its query.
2. Opens the strongest sources, not just snippets.
3. Prefers primary sources: docs, papers, standards, filings, repos, changelogs, datasets, talks, interviews.
4. Uses credible secondary sources for context: journalism, analyst reports, explainers, technical blogs, user discussions.
5. Avoids SEO filler unless it points to a better source.
6. Returns at most 10 findings.

Finding shape:

```json
{
  "title": "...",
  "url": "https://...",
  "source_type": "paper|docs|github|news|blog|report|video|forum|dataset|other",
  "evidence_quality": "primary|secondary|anecdotal|unknown",
  "relevance": "high|medium|low",
  "summary": "2-3 sentences on what this source contributes",
  "key_concepts": ["..."],
  "claim_to_verify": "optional"
}
```

Completion: every query produced findings, or you can state which query failed and why.

## 4. Gap-check between rounds

After each round except the last, deduplicate the findings so far and identify what is still missing.

Look for gaps:

- No primary source
- No counter-evidence
- Too much vendor language
- Missing implementation detail
- Missing adoption, market, pricing, or benchmark data
- Contradictory claims
- Outdated sources
- Newly discovered entity worth chasing

Generate exactly the next round's query count. Follow-up queries must target named gaps, not repeat round 1.

Completion: each follow-up query states the gap it is meant to close.

## 5. Rank sources

Deduplicate by canonical URL and obvious mirrors.

Score:

- 1.0 seed source from the user
- 0.9 high-relevance primary source
- 0.8 high-relevance secondary source
- 0.7 medium-relevance primary source
- 0.5 medium-relevance secondary source
- 0.2 low-relevance or anecdotal source

Keep the top 8–12 sources with detail. Demote the rest to one-line notes.

Completion: sources are ranked, duplicates are collapsed, and weak sources are not treated like strong evidence.

## 6. Write the dossier

Use this structure:

```markdown
# <Topic> — Research Dossier

> One-line answer: what the research found and why it matters.

## Executive summary
The short synthesis. Include the main conclusion, strongest evidence, and biggest uncertainty.

## Research angles
- <query> — <angle> — <why it was included>

## Key findings
- <finding> — source: <title/link>

## Evidence quality
- **Well-supported:** <claims backed by primary or multiple credible sources>
- **Promising but unsettled:** <claims with partial evidence>
- **Weak or hype:** <claims that are vendor-led, anecdotal, or unsupported>

## Practical implications
What the user should do, compare, build, buy, avoid, monitor, or research next.

## Sources ranked
1. <Title> — <score> — <source_type> — <url> — why it matters

## Open questions
- <question> — what would answer it
```

Completion: every major claim links back to a source, uncertainty is explicit, and the output is synthesis rather than a link dump.

## Failure modes

- **One-pass summary:** stop and run gap queries.
- **Query clones:** replace with distinct angle queries.
- **No primary sources:** add a docs/paper/filing/repo/dataset query.
- **No skepticism:** add a counterpoint query.
- **Vendor fog:** separate shipped facts from marketing claims.
- **Link dump:** rank and synthesize before answering.
- **Fake confidence:** label uncertainty and open questions.
