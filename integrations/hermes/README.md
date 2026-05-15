# HyperResearch Hermes Adapter

Unofficial Hermes Agent adapter for HyperResearch.

This adapter ports the HyperResearch V8 workflow to Hermes while preserving the
HyperResearch name, 16-step pipeline, artifact ledger, `vault_tag`, canonical
query, source provenance, critics, patching, polish, and readability audit. It
does not modify HyperResearch core code or the existing Claude Code integration.

## Status

- Adapter status: unofficial, v0
- Upstream status: not accepted by Nous Research or HyperResearch upstream
- Primary integration mode: Hermes external skill directory
- Optional capability mode: HyperResearch MCP server

## What This Provides

- One Hermes skill: `hyperresearch`
- A runtime mapping reference for Claude-to-Hermes behavior
- Manual Hermes install instructions
- Agent context files for Hermes and other agents
- Report and evidence templates
- Optional MCP config reference for `hyperresearch mcp`

## What This Does Not Do

- It does not replace or modify the Claude Code integration.
- It does not install, call, or preserve Claude syntax such as `Skill(...)`.
- It does not create `.claude/*` files.
- It does not reimplement HyperResearch CLI, vault, sync, fetch, lint, or MCP logic.
- It does not use hidden authenticated crawling or automatic credential/session use.

## Source Of Truth

This adapter follows official Hermes references:

- Hermes skills use a `SKILL.md` file with YAML metadata/frontmatter and optional supporting directories such as `templates/` and `references/`.
- Hermes can scan external skill directories configured under `skills.external_dirs` in `~/.hermes/config.yaml`.
- External skill directories are read-only for discovery; Hermes-created or edited skills are written to `~/.hermes/skills/`.
- Hermes MCP servers are configured under `mcp_servers` in `~/.hermes/config.yaml`.

References:

- https://hermes-agent.nousresearch.com/docs/user-guide/features/skills
- https://hermes-agent.nousresearch.com/docs/user-guide/configuration
- https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp
- https://github.com/NousResearch/hermes-agent
- https://github.com/NousResearch/hermes-agent/tree/main/skills
- https://github.com/NousResearch/hermes-agent/blob/main/skills/software-development/requesting-code-review/SKILL.md

## Skill Layout

```text
integrations/hermes/skills/
└── hyperresearch/
    ├── SKILL.md
    └── references/
        └── runtime-mapping.md
```

The skill appears in Hermes as:

```text
/hyperresearch
```

## Install

Add this repository's Hermes skill directory to `~/.hermes/config.yaml`:

```yaml
skills:
  external_dirs:
    - /absolute/path/to/hyperresearch/integrations/hermes/skills
```

Restart Hermes or reload skills if your Hermes session supports live reload.
See [install.md](install.md) for the full procedure.

## Optional MCP

HyperResearch already ships a read-only MCP server:

```bash
hyperresearch mcp
```

If Hermes is configured to connect to that server, the `hyperresearch` skill can
prefer MCP tools for vault search/read/list/status/lint and use terminal CLI
commands for write operations, fetches, and final report artifacts.

See [mcp.json](mcp.json) for a machine-readable reference config and
[install.md](install.md) for the Hermes `config.yaml` form.

## Runtime Mapping

Read
[runtime-mapping.md](skills/hyperresearch/references/runtime-mapping.md)
for the porting contract. The short version:

- Claude `Skill(...)` continuation becomes Hermes artifact/checkpoint recovery.
- Claude `Task` subagents become Hermes delegation when available.
- Claude `TodoWrite` becomes `research/temp/orchestrator-checkpoints.md`.
- Claude tool locks become prompt contracts plus validation unless Hermes can
  enforce equivalent toolset restrictions.
- Claude `PreToolUse` reminders become explicit skill rules to search the vault
  first and persist selected sources with `hyperresearch fetch`.

## Attribution

HyperResearch is by Jordan Gibbs. This adapter is an unofficial compatibility
pack for Hermes Agent and does not imply endorsement by Jordan Gibbs,
HyperResearch upstream, Nous Research, or Hermes Agent upstream.
