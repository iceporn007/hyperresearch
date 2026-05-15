# HyperResearch Adapter Context For Hermes

This directory contains an unofficial Hermes Agent adapter for HyperResearch.

## Scope

Use the existing HyperResearch CLI and MCP server as capabilities. Do not
reimplement vault, search, fetch, sync, lint, graph, or report logic.

Important paths:

- Skill root for external Hermes discovery: `integrations/hermes/skills`
- Main skill: `integrations/hermes/skills/hyperresearch/SKILL.md`
- Runtime mapping: `integrations/hermes/skills/hyperresearch/references/runtime-mapping.md`
- Templates: `integrations/hermes/templates`
- Manual install guide: `integrations/hermes/install.md`
- Optional MCP config reference: `integrations/hermes/mcp.json`

## Operating Rules

- Treat this as an unofficial adapter unless accepted upstream.
- Preserve the HyperResearch name and V8 workflow.
- Preserve source provenance for every factual claim.
- Distinguish fact, inference, and opinion in outputs.
- Maintain a claim/evidence table during research.
- Record uncertainty, contradictions, and unresolved source tensions.
- After a final draft exists, patch it surgically instead of regenerating it.
- Do not use hidden authenticated crawling.
- Do not automatically use browser profiles, cookies, credentials, or sessions.
- Ask the user before using authenticated sources or saved browser profiles.

## Preferred Capability Order

1. Use HyperResearch MCP read tools when configured in Hermes.
2. Use `hyperresearch ... --json` CLI commands through Hermes terminal tools.
3. Use Hermes web search/extract only to discover URLs, then persist selected
   source pages with `hyperresearch fetch`.

## Porting Notes

The Hermes skill is `/hyperresearch`, not `/deep-research`. Its runtime mapping
is stored at `skills/hyperresearch/references/runtime-mapping.md`. Do not
preserve Claude syntax; preserve HyperResearch behavior.

## Attribution

HyperResearch is by Jordan Gibbs. This adapter is an unofficial Hermes Agent
compatibility pack.
