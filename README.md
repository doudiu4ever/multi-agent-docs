# multi-agent-docs

A Claude Code skill that encodes conventions for organizing a repository's docs
when more than one AI coding agent co-develops it — for example Claude Code
reading `CLAUDE.md` and Codex reading `AGENTS.md`.

> 中文版:[README.zh-CN.md](README.zh-CN.md)

## What it solves

When several agents work in the same repo, each reads its own entry file
(`CLAUDE.md`, `AGENTS.md`, `<TOOL>.md`). Without discipline, a shared design doc
ends up pointing one agent at another agent's instructions, or silently assumes
only one tool exists. This skill defines the boundary rules that keep such a repo
coherent.

## The model

Docs split into two tiers, and the tier decides what may reference what:

- **Entry docs** (per-agent, fixed names: `CLAUDE.md`, `AGENTS.md`, `<TOOL>.md`)
  — thin routers; the only place tool-specific behavior belongs.
- **Shared docs** (project-specific names: design, setup, README, architecture)
  — the substance; read identically by every agent.

Five rules govern them:

1. **One-way dependency** — entry docs may point into shared docs; shared docs
   never point back at an entry doc.
2. **Shared docs are tool-neutral** — use role words ("the coding agent", "code
   review"), not "Claude" / "Codex".
3. **Entry docs mirror each other** — identical except title, self-ordering, and
   genuine tool quirks.
4. **Constraints follow capability, not identity** — phrase by what the
   environment can do, not a hardcoded machine or tool name.
5. **Sync discipline** — any rule change updates all affected docs in the same
   edit.

## Usage

The skill lives in [`SKILL.md`](SKILL.md). It triggers when a project has — or is
about to have — multiple per-agent entry docs, or when you are writing, editing,
or reviewing `CLAUDE.md`, `AGENTS.md`, or shared docs. `SKILL.md` includes an
audit procedure, a recommended entry-doc skeleton, and a pre-finish checklist.
