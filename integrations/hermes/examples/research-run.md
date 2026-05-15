# Example Hermes Research Run

This is an illustrative runbook for the unofficial HyperResearch Hermes
adapter. Replace the query and paths with your actual workspace.

## User Prompt

```text
/hyperresearch Compare the evidence for approach A and approach B in the last
three years. Give a recommendation and explain uncertainty.
```

## Step 1: Check HyperResearch

```bash
hyperresearch status --json
```

If there is no vault:

```bash
hyperresearch init . --json
```

## Step 2: Archive Prior Scratch Artifacts

```bash
hyperresearch archive-run --json
```

## Step 3: Mint A Vault Tag

```bash
hyperresearch vault-tag approach-a-vs-b --json
```

Example result:

```json
{
  "ok": true,
  "data": {
    "vault_tag": "approach-a-vs-b-a1b2c3",
    "slug": "approach-a-vs-b",
    "suffix": "a1b2c3"
  }
}
```

## Step 4: Search Existing Vault

```bash
hyperresearch search "approach A approach B last three years" --include-body --json
```

Record candidate note IDs in `research/temp/claim-evidence-table.md`.

## Step 5: Fetch New Sources

Use Hermes web tools only to discover candidate URLs. Persist selected sources:

```bash
hyperresearch fetch "https://example.org/source" --tag "approach-a-vs-b-a1b2c3" --json
```

Do not rely on search snippets in the final report.

## Step 6: Maintain Evidence Tables

Create:

```text
research/temp/claim-evidence-table.md
research/temp/source-tensions.md
```

Use the templates under `integrations/hermes/templates/`.

## Step 7: Continue The V8 Pipeline

For a light-tier run, Step 10 writes one report directly:

Write:

```text
research/notes/final_report_approach-a-vs-b-a1b2c3.md
```

For a full-tier run, preserve the HyperResearch V8 sequence:

```text
Step 3  contradiction graph
Step 4  loci analysis
Step 5  depth investigation
Step 6  cross-locus reconciliation
Step 7  source tensions
Step 8  corpus critic and gap fill
Step 9  evidence digest
Step 10 triple draft: research/temp/draft-{a,b,c}.md
Step 11 synthesis: research/notes/final_report_approach-a-vs-b-a1b2c3.md
Step 12 four critics
Step 13 post-critic gap fetch
Step 14 patcher
Step 15 polish
Step 16 readability audit
```

## Step 8: Patch Only After Final Draft

After the final report exists, apply only surgical edits and log them:

```text
research/patch-log.json
```

## Step 9: Final Checks

```bash
hyperresearch lint --json
hyperresearch status --json
```

Final response to the user should include the report path, confidence level,
major contradictions, and any unresolved gaps.
