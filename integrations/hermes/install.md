# Installing The Unofficial HyperResearch Hermes Adapter

This adapter is designed for Hermes external skill directories.

Official Hermes references:

- Skills system: https://hermes-agent.nousresearch.com/docs/user-guide/features/skills
- Configuration: https://hermes-agent.nousresearch.com/docs/user-guide/configuration
- MCP integration: https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp

## Prerequisites

1. Hermes Agent is installed and working.
2. HyperResearch is installed and available as `hyperresearch`, or you know the
   absolute path to the executable.
3. You have a workspace where a HyperResearch vault can exist.

Check:

```bash
hermes --help
hyperresearch --help
```

## Add The External Skill Directory

Edit `~/.hermes/config.yaml`:

```yaml
skills:
  external_dirs:
    - /absolute/path/to/hyperresearch/integrations/hermes/skills
```

Notes:

- Use an absolute path for reliability.
- Hermes expands `~` and environment variables in external skill paths.
- External skill directories are scanned read-only. Hermes-created or edited
  skills are written to `~/.hermes/skills/`.
- If a local skill has the same name, the local skill wins.

Restart Hermes after editing the config. The skill should be available as:

```text
/hyperresearch
```

## Optional: Configure Skill Settings

The skill declares non-secret config settings for:

- `hyperresearch.cli_path`
- `hyperresearch.vault_root`

If your Hermes version supports skill config migration, run:

```bash
hermes config migrate
```

Otherwise, keep the defaults and make sure `hyperresearch` is on `PATH`.

## Optional: Configure HyperResearch MCP

HyperResearch exposes read-only MCP tools with:

```bash
hyperresearch mcp
```

Add this to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  hyperresearch:
    command: "hyperresearch"
    args: ["mcp"]
    enabled: true
    timeout: 120
    connect_timeout: 30
    tools:
      include:
        - search_notes
        - read_note
        - read_many
        - list_notes
        - get_backlinks
        - get_hubs
        - vault_status
        - lint_vault
      resources: false
      prompts: false
```

Restart Hermes or run `/reload-mcp` if available.

Hermes registers MCP tool names with an `mcp_<server>_<tool>` prefix, so these
tools may appear as names like `mcp_hyperresearch_search_notes`.

## Security And Authenticated Sources

This adapter intentionally does not enable hidden authenticated crawling.

Do not use saved browser profiles, cookies, credentials, authenticated sessions,
private repositories, paywalled accounts, or user-specific logged-in pages
unless the user explicitly approves that source access for the run.

## Smoke Test

From a workspace:

```bash
hyperresearch status --json
```

If needed:

```bash
hyperresearch init . --json
```

Then in Hermes:

```text
/hyperresearch Produce a light research brief on a topic with 3 cited sources.
```

Expected artifacts:

- `research/query-<vault_tag>.md`
- `research/scaffold.md`
- `research/prompt-decomposition.json`
- `research/temp/claim-evidence-table.md`
- `research/notes/final_report_<vault_tag>.md`
- `research/readability-decisions.json`

## Attribution

HyperResearch is by Jordan Gibbs. This adapter is an unofficial compatibility
pack for Hermes Agent.
