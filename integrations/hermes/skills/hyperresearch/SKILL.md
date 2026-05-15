---
name: hyperresearch
description: "Run the HyperResearch deep-research workflow through Hermes Agent."
version: 0.2.0
author: "Unofficial Hermes adapter for HyperResearch by Jordan Gibbs"
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [hyperresearch, research, deep-research, evidence, provenance, mcp]
    category: research
    requires_toolsets: [terminal]
    config:
      - key: hyperresearch.cli_path
        description: "Path to the HyperResearch CLI executable."
        default: "hyperresearch"
        prompt: "HyperResearch CLI path"
      - key: hyperresearch.vault_root
        description: "Workspace path containing, or intended to contain, the .hyperresearch vault."
        default: "."
        prompt: "HyperResearch vault root"
---
# HyperResearch

Run the HyperResearch V8 deep-research workflow through Hermes Agent.

This is an unofficial Hermes adapter. Preserve HyperResearch behavior, names,
artifacts, and invariants. Do not call Claude Code syntax or install `.claude`
assets from this skill.

## Source Of Truth

Use the existing HyperResearch CLI and optional MCP server as capabilities:

```bash
hyperresearch init . --json
hyperresearch archive-run --json
hyperresearch vault-tag <slug> --json
hyperresearch search "<query>" --include-body --json
hyperresearch fetch "<url>" --tag "<vault_tag>" --json
hyperresearch note show <id1> <id2> --json
hyperresearch lint --json
hyperresearch status --json
```

If the HyperResearch MCP server is configured in Hermes, prefer MCP for
read-only vault operations: search, read, read_many, list, backlinks, hubs,
status, and lint. Use terminal CLI commands for writes, fetches, run archiving,
vault-tag minting, and report files.

Read `references/runtime-mapping.md` before adapting Claude-specific behavior.

## Core Invariants

1. Canonical research query is gospel. Persist it once at
   `research/query-<vault_tag>.md` and reuse it verbatim.
2. Wrapper requirements are separate from the canonical query.
3. Use `hyperresearch vault-tag` to mint a collision-safe `vault_tag`.
4. Run `hyperresearch archive-run --json` before a new run.
5. Keep the 16-step V8 pipeline explicit, even though Hermes uses one skill.
6. Steps are sequential at the outer level; parallelize only inside a step.
7. Full tier requires the triple-draft ensemble and synthesis path.
8. After the final report exists, patch surgically. Do not regenerate.
9. Preserve source provenance. Final factual claims need citations.
10. Record contradictions, uncertainty, and source tensions.
11. Do not use hidden authenticated crawling.
12. Do not automatically use browser profiles, cookies, credentials, or saved
    sessions. Ask the user before using authenticated sources.

## Tier Routing

Step 1 writes `pipeline_tier` to `research/prompt-decomposition.json`.

| Tier | Steps |
|---|---|
| light | 1 -> 2 -> 10 single draft -> 15 -> 16 |
| full | 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8 -> 9 -> 10 -> 11 -> 12 -> 13 -> 14 -> 15 -> 16 |

Default to `full` when the prompt asks for deep analysis, contested evidence,
literature review, forecasting, or a defended thesis. Use `light` for bounded
lookup, simple comparison, cataloging, or short structured survey.

## Bootstrap

1. Resolve `hyperresearch.cli_path` and `hyperresearch.vault_root` from Hermes
   skill config when available. Default to `hyperresearch` and `.`.
2. Check or create the vault:

   ```bash
   hyperresearch status --json
   hyperresearch init . --json
   ```

3. Archive prior run scratch:

   ```bash
   hyperresearch archive-run --json
   ```

4. Resolve the canonical query:
   - If `research/prompt.txt` exists, use its contents.
   - Otherwise use the user's verbatim research prompt.
   - Keep save-path, citation-format, and wrapper instructions separate.
   - If `research/wrapper_contract.json` exists, read it.

5. Mint a unique `vault_tag`:

   ```bash
   hyperresearch vault-tag <short-topic-slug> --json
   ```

6. Write `research/query-<vault_tag>.md` with YAML frontmatter and the verbatim
   query.
7. Write `research/scaffold.md` with run config, modality, wrapper
   requirements, tier rationale placeholder, coverage checklist, and artifact
   ledger.
8. Create or update `research/temp/orchestrator-checkpoints.md` with the
   16-step checklist below.

## Artifact Ledger

Use these artifacts for checkpoints and recovery:

| Step | Artifact |
|---|---|
| 1 | `research/scaffold.md`, `research/prompt-decomposition.json`, `research/temp/coverage-matrix.md` |
| 2 | vault notes tagged with `vault_tag`, `research/temp/search-plan.md`, `research/temp/coverage-gaps.md`, optional `research/temp/redundancy-audit.md` |
| 3 | `research/temp/contradiction-graph.json`, `research/temp/consensus-claims.json` |
| 4 | `research/loci.json` |
| 5 | interim vault notes with `type: interim` and `## Committed position` |
| 6 | `research/comparisons.md` |
| 7 | `research/temp/source-tensions.json` |
| 8 | `research/corpus-critic-gaps.json`, `research/temp/corpus-critic-results.md` |
| 9 | `research/temp/evidence-digest.md` |
| 10 | `research/temp/draft-{a,b,c}.md` for full, or `research/notes/final_report_<vault_tag>.md` for light |
| 11 | `research/temp/synthesis-plan.md`, `research/temp/synthesis-outline.md`, `research/temp/synthesis-conflicts.md`, `research/temp/synthesis-pass1.md`, `research/notes/final_report_<vault_tag>.md` |
| 12 | `research/critic-findings-{dialectic,depth,width,instruction}.json` |
| 13 | `research/temp/post-critic-fetch-log.md` |
| 14 | `research/patch-log.json` and edited final report |
| 15 | `research/polish-log.json` and edited final report |
| 16 | `research/readability-recommendations.json`, `research/readability-decisions.json`, final report |

## Subagent Contract In Hermes

When delegating work, every delegate prompt must include:

1. `research_query`, copied verbatim from `research/query-<vault_tag>.md`.
2. Pipeline position: step number, what came before, what comes next.
3. Specific inputs: `vault_tag`, file paths, source IDs, locus, output path.
4. Allowed operations and forbidden operations.
5. Required output schema or artifact path.

If Hermes cannot enforce a permission lock for a delegate, enforce it by
explicit prompt instructions, pre-created output stubs, post-run artifact
validation, and one stricter retry when output is missing.

## The 16-Step Workflow

### Step 1 - Prompt Decomposition

Runs for all tiers. Extract atomic prompt items, required H2 headings,
time horizons, period-pinned historical periods, scope conditions, tier,
response format, and citation style. Write `research/prompt-decomposition.json`
and `research/temp/coverage-matrix.md`. Fix any coverage gap before Step 2.

### Step 2 - Width Sweep

Runs for all tiers. Build `research/temp/search-plan.md` from multiple lenses,
search existing vault first, discover sources, fetch selected URLs through
`hyperresearch fetch`, preserve provenance, write `coverage-gaps.md`, and for
full tier write redundancy audit when claims files exist. Light targets 12-20
sources. Full targets 40-80 sources.

### Step 3 - Contradiction Graph

Full tier only. Read `research/temp/claims-*.json`, pair opposing claims,
write `research/temp/contradiction-graph.json`, and write
`research/temp/consensus-claims.json`.

### Step 4 - Loci Analysis

Full tier only. Run two independent loci-analyst delegates if possible, merge
and score 1-6 depth loci, require at least one dialectical locus unless
justified, and write `research/loci.json`.

### Step 5 - Depth Investigation

Full tier only. Delegate one depth investigator per budgeted locus, capped at
6. Each writes an interim note ending with `## Committed position`.

### Step 6 - Cross-Locus Reconcile

Full tier only. Read committed positions, identify 3-5 cross-locus dynamics,
and write `research/comparisons.md` with draft engagement guidance.

### Step 7 - Source Tensions

Full tier only. Extract expert disagreements and orphan tensions into
`research/temp/source-tensions.json`. Each tension needs both sides, evidence,
a committed resolution, and decision relevance.

### Step 8 - Corpus Critic

Full tier only. Check period-pinned primary-source coverage, run a corpus
critic delegate, fill critical/high gaps with targeted fetchers, update
`comparisons.md`, and write `research/temp/corpus-critic-results.md`.

### Step 9 - Evidence Digest

Full tier only. Rank load-bearing claims and quoted support, group by atomic
item, include consensus and contested claims, and write
`research/temp/evidence-digest.md`.

### Step 10 - Triple Draft

Runs for all tiers. Light tier reads the vault and writes one final report
directly to `research/notes/final_report_<vault_tag>.md`, then skips Steps
11-14. Full tier defines three analytical angles, curates different 20-50
source-ID lists, delegates three draft orchestrators, and validates
`research/temp/draft-a.md`, `draft-b.md`, and `draft-c.md`.

### Step 11 - Synthesize

Full tier only. Read all three drafts fully, resolve factual conflicts, write
`synthesis-plan.md`, `synthesis-outline.md`, and `synthesis-conflicts.md`, then
produce exactly one final report at
`research/notes/final_report_<vault_tag>.md`. Do not re-synthesize after this.

### Step 12 - Critics

Full tier only. Run independent dialectic, depth, width, and instruction
critics. Each writes its own `research/critic-findings-*.json`. Critics do not
edit the draft.

### Step 13 - Gap Fetch

Full tier only. Fetch missing sources required by critic findings, append new
evidence to `research/temp/evidence-digest.md`, and write
`research/temp/post-critic-fetch-log.md`.

### Step 14 - Patcher

Full tier only. Respect `research/skip-patcher.txt` if present. Pre-create
`research/patch-log.json`, run an edit-only patcher delegate when possible,
apply critic findings as surgical hunks, and never regenerate the draft.

### Step 15 - Polish

Runs for all tiers. Pre-create `research/polish-log.json`, run an edit-only
polish auditor when possible, strip scaffold leaks and hygiene failures, then
run wrapper-report, locus-coverage, scaffold-prompt, and patch-surgery lint
rules.

### Step 16 - Readability Audit

Runs for all tiers. Run one readability recommender, write
`research/readability-recommendations.json`, selectively apply safe structural
improvements, and write `research/readability-decisions.json`.

## Recovery

If context is compressed or the run resumes later:

1. Read `research/temp/orchestrator-checkpoints.md`.
2. Inspect the artifact ledger.
3. Find the highest completed step whose exit criterion is satisfied.
4. Resume from the next required step for the tier.

## Final Response

Tell the user the final report path, `vault_tag`, tier used, provenance status,
major unresolved uncertainties or contradictions, and any behavior that could
not be ported exactly because Hermes lacks an equivalent.
