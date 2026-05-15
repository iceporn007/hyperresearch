# Agent Instructions For The Hermes Adapter

This directory is additive integration material only. Do not edit HyperResearch
core files, Claude integration files, `README.md`, `pyproject.toml`, or `src/`
when working on the v0 Hermes adapter.

## Goals

- Keep one high-quality Hermes orchestrator skill first, named `hyperresearch`.
- Make the adapter compatible with Hermes external skill directories.
- Reuse HyperResearch CLI and MCP capabilities.
- Keep all adapter-specific files under `integrations/hermes/`.

## Constraints

- Do not duplicate the full 16-step Claude skill tree as separate Hermes skills in v0.
- Do keep the 16-step V8 workflow explicit inside the Hermes `hyperresearch` skill.
- Do not introduce install-time writes outside the Hermes config instructions.
- Do not assume Hermes has access to Claude tools such as `Skill`, `Task`, or
  `TodoWrite`.
- Do not preserve Claude syntax in Hermes-facing instructions.
- Use official Hermes documentation and repository examples as source of truth.

## Review Checklist

- `integrations/hermes/skills/hyperresearch/SKILL.md` has valid YAML
  frontmatter.
- `integrations/hermes/skills/hyperresearch/references/runtime-mapping.md`
  maps Claude constructs to Hermes behavior.
- Manual install instructions use `skills.external_dirs`.
- Optional MCP instructions use `mcp_servers`.
- Report templates preserve citations, provenance, uncertainty, contradictions,
  and claim/evidence tracking.
- The adapter is labeled unofficial.
