# HyperResearch Claude-To-Hermes Runtime Mapping

This document maps Claude Code-specific constructs from the original
HyperResearch integration into Hermes Agent behavior. Preserve the behavior,
not the Claude syntax.

## Scope

Source behavior comes from:

- `src/hyperresearch/skills/hyperresearch.md`
- `src/hyperresearch/skills/hyperresearch-*.md`
- `src/hyperresearch/core/agent_docs.py`
- `src/hyperresearch/core/hooks.py`

The Hermes adapter must not edit or depend on `.claude` runtime assets. It uses
Hermes skills, Hermes terminal/delegation capabilities, and the existing
HyperResearch CLI/MCP server.

## Construct Mapping

| Claude construct | Intended HyperResearch behavior | Hermes mapping |
|---|---|---|
| `Skill(skill: "hyperresearch-N-step")` continuation | Load each step procedure fresh at the moment it is needed, preventing context drift and forgotten steps. | Use one Hermes `/hyperresearch` skill with an explicit 16-step procedure, artifact ledger, and `research/temp/orchestrator-checkpoints.md`. When context is lost, recover from artifacts and continue at the next unsatisfied step. Do not write Claude `Skill(...)` syntax. |
| Separate `.claude/skills/hyperresearch-N-*/SKILL.md` files | Keep step instructions modular and loaded just-in-time. | Keep step sections explicit in the Hermes `hyperresearch` skill. If v1 needs progressive disclosure, add Hermes `references/step-*.md` files, but v0 keeps one orchestrator skill plus this mapping reference. |
| `Task` subagent usage | Parallelize bounded work: fetchers, loci analysts, depth investigators, draft orchestrators, critics, patcher, polish auditor, readability recommender. Each subagent receives the verbatim query, pipeline position, and explicit inputs. | Use Hermes delegation when available. Delegate prompts must include the same three-piece contract plus output path/schema and allowed/forbidden operations. If delegation is unavailable, run the work sequentially but preserve artifacts and note reduced parallelism in the final response. |
| `subagent_type: hyperresearch-*` | Select a specialized prompt and permission profile. | Name the delegate role in the prompt, e.g. "You are the hyperresearch-fetcher delegate." Since Hermes may not have pre-registered role files in v0, specialization is prompt-level. Future v1 can add Hermes subskills or reference prompt files. |
| `TodoWrite` | Durable checklist survives compaction and tracks step completion. | Write `research/temp/orchestrator-checkpoints.md` with all 16 steps, tier routing, status, artifact paths, and next action. Update after each step. Recovery uses this plus disk artifacts. |
| Tool permissions such as `tools: Bash, Read, Write`, `Read, Edit`, `Read, Write` | Constrain subagents so they cannot perform out-of-scope actions; most importantly, patcher and polish auditor cannot create or regenerate drafts. | Hermes v0 uses prompt-level permission contracts plus artifact validation. For edit-only phases, pre-create log files, instruct delegates to patch only existing files, and audit diffs/logs after completion. If Hermes supports toolset-restricted delegation in the active runtime, request only the required toolsets. |
| Tool-locked patcher `[Read, Edit]` | Enforce patch-never-regenerate after critics; patcher can only edit final report and pre-created patch log. | Pre-create `research/patch-log.json`. Delegate a patcher with explicit "edit existing files only" instructions. After completion, verify no wholesale rewrite occurred and patch log is populated. If exact tool lock is unavailable, mark this as not exactly ported. |
| Tool-locked polish auditor `[Read, Edit]` | Enforce final hygiene edits without regeneration. | Pre-create `research/polish-log.json`. Delegate a polish auditor with edit-only instructions. Verify report still preserves citations and source content. |
| Synthesizer `[Read, Write]` without Bash/Task | Force one focused synthesis pass from prepared drafts and plans; no new fetching during synthesis. | Delegate a synthesizer with explicit no-fetch/no-search/no-delegation instructions. If Hermes cannot enforce this, validate by checking no new sources were fetched during Step 11 and record any deviation. |
| Non-interactive continuation rule | In Claude `-p`, a bare text response while tasks run can end the process, so the orchestrator must keep making tool calls while subagents run. | Hermes does not have the same documented `end_turn` behavior. Preserve the intent by writing wait-time analysis to `research/temp/orchestrator-notes.md`, avoiding idle polling, and checkpointing progress while delegates run. Exact `end_turn` semantics are not ported. |
| `end_turn` behavior | Avoid accidental premature termination during long subagent waves. | No direct Hermes equivalent in v0. Use explicit checkpoints, artifact writes, and final recovery instructions. |
| `.claude/skills` install target | Claude discovers slash-command skills and internal step skills here. | Hermes discovers skills from `~/.hermes/skills` and `skills.external_dirs`. Configure `integrations/hermes/skills` as an external skill directory. Do not create `.claude/skills`. |
| `CLAUDE.md` generated context | Auto-load project research instructions into Claude sessions, including CLI path, vault conventions, source fetching rules, and curation. | Use `integrations/hermes/HERMES.md`, `AGENTS.md`, and the `/hyperresearch` skill content. Manual install instructions explain external skill config. No automatic project context injection is performed in v0. |
| Claude `PreToolUse` hook | Before web/file search/fetch, remind the agent to check the vault and use `hyperresearch fetch` for source pages. | Hermes v0 has no adapter-installed pre-tool hook. Preserve behavior as skill rules: "search existing vault first" and "persist selected URLs with `hyperresearch fetch` before relying on them." Optional future work: Hermes-native hook/plugin if supported. |
| Hook matcher `Glob|Grep|WebSearch|WebFetch` | Trigger reminders before searches and raw web fetches. | No exact v0 mapping. The `/hyperresearch` skill includes pre-search and pre-fetch rules. |
| Global Claude install | Make `/hyperresearch` available in all Claude Code sessions while lazily installing step skills per project. | Use Hermes `skills.external_dirs` pointing at this repo. External dirs are read-only and discovered as slash commands. No lazy project install is needed. |
| Claude model labels Opus/Sonnet/Haiku | Route different cognitive loads to different model tiers. | Hermes model routing is runtime/user configured. The adapter describes roles by task difficulty, not provider-specific model names. |
| Claude WebFetch avoidance | Avoid transient, lossy source reads; persist source pages into the vault. | Same behavior: Hermes web tools may discover URLs, but selected evidence must be fetched into HyperResearch before use. |
| Authenticated crawl setup through HyperResearch config | Let users explicitly set up browser profiles for login-gated sources. | Do not use authenticated crawling automatically. Ask the user before using browser profiles, cookies, credentials, saved sessions, private sources, or paywalled accounts. |

## Exact Port Gaps

These behaviors cannot be guaranteed exactly by this v0 Hermes adapter unless
the active Hermes runtime exposes equivalent controls:

1. Just-in-time step loading equivalent to Claude `Skill(...)`.
2. Hard per-subagent tool allowlists equivalent to Claude agent `tools:`.
3. Claude non-interactive `end_turn` prevention semantics.
4. Claude `PreToolUse` hook matcher behavior.
5. Pre-registered Claude subagent prompt roster under `.claude/agents`.

The adapter compensates with explicit workflow sections, artifact checkpoints,
delegate prompt contracts, and validation gates.
