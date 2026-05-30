---
name: multi-agent-docs
description: >-
  Conventions for organizing a repository's docs when more than one AI coding
  agent co-develops it (e.g. Claude Code reading CLAUDE.md and Codex reading
  AGENTS.md). Use this whenever a project has — or is about to have — multiple
  per-agent entry docs, or when you are writing/editing/reviewing CLAUDE.md,
  AGENTS.md, or shared docs like DESIGN.md / DEV_ENVIRONMENT.md / README. Also
  use it when a doc says "see CLAUDE.md" / "see AGENTS.md" from a shared file,
  when an agent-entry doc and its sibling have drifted out of sync, or when
  shared docs name one specific tool (Claude, Codex, "Claude review") where they
  should be tool-neutral. Trigger this even if the user just says "tidy up the
  docs", "two agents will work on this", or "make the docs agent-agnostic".
---

# Multi-Agent Doc Boundaries

When two or more AI coding agents work on the same repository, each reads its own
entry file: Claude Code reads `CLAUDE.md`, Codex reads `AGENTS.md`, and others
follow the same pattern. That creates a layout problem: where does guidance live,
and who is allowed to point at whom? Getting this wrong silently sends one agent
to read another agent's instructions, or makes a shared design doc quietly assume
only one tool exists.

This skill encodes the boundary rules that keep such a repo coherent.

## The two doc tiers

Split every project doc into exactly one of two tiers, and treat the tier as the
thing that decides what may reference what.

- **Entry docs (per-agent):** `CLAUDE.md`, `AGENTS.md`, and any future
  `<TOOL>.md`. One per agent. Short. They are the *only* place tool-specific
  behavior belongs. Each agent reads exactly one of these and treats it as the
  front door.
- **Shared docs (project detail):** every other doc in the repo — design specs,
  environment/setup notes, README, architecture notes, backlog: the actual
  project material. **Their exact set and filenames are project-specific** —
  one repo's `DESIGN.md` + `DEV_ENVIRONMENT.md` is another's
  `docs/architecture.md` + `CONTRIBUTING.md`. Don't assume any particular name;
  the rule is about the *tier*, not the filename. Every agent reads these, and
  they must read identically no matter which agent opened them.

Note on filenames: the **entry docs** are fixed by the tools themselves
(`CLAUDE.md` for Claude Code, `AGENTS.md` for Codex, `<TOOL>.md` for others), so
this skill names them directly. The **shared docs** are whatever this project
happens to call them — identify them per repo rather than expecting a fixed list.

The entry docs are thin routers; the shared docs are the substance. Keep design
and progress detail out of entry docs and in shared docs, so there is a single
source of truth instead of two copies that drift.

## Rule 1 — Dependency is one-way: entry → shared only

Entry docs may point into shared docs ("gameplay design lives in `DESIGN.md`").
**Shared docs must never point back at an entry doc.**

Why this matters, concretely: if `DESIGN.md` says "see `CLAUDE.md` for the build
constraint", then Codex — which reads `AGENTS.md` — gets sent to a file that is
not its own. Best case it's confused; worst case it follows guidance written for
a different agent. Naming *both* ("see CLAUDE.md / AGENTS.md") is not a fix
either: it bloats the shared doc and re-couples it to the entry layer. The clean
shape is a directed edge from entry to shared, never the reverse.

If a shared doc currently back-references an entry doc, the content it points at
either (a) belongs in the shared doc itself, or (b) is genuinely tool-specific
and belongs *only* in each entry doc. Move it accordingly and drop the reference.

## Rule 2 — Shared docs are tool-neutral

A shared doc is read by every agent, so it must not assume a particular one.
Avoid naming a single tool where a role word would do:

- "the remote agent writes the script" — not "Claude writes the script"
- "code review flagged this (P2)" — not "Codex review flagged this (P2)"
- "run the agent CLI" — not "run Claude Code"

Why: a single tool name in a shared doc silently encodes "this project is a
Claude project" (or a Codex project), which stops being true the moment a second
agent joins — and it reads slightly wrong to whichever agent isn't named. Use
"agent", "the coding agent", "code review", etc. Capability-based phrasing beats
identity-based phrasing for the same reason (see Rule 4).

## Rule 3 — Keep entry docs mirrored

The per-agent entry docs should carry the *same* project-facing rules, so an
agent's behavior doesn't depend on which front door it came through. Treat them
as mirror images: when you change a rule in one, make the matching change in the
others in the same edit.

The only legitimate differences between two entry docs are:

1. The title / first line that identifies the tool ("Claude Code entry
   guidance" vs "Codex entry guidance").
2. The self-vs-other ordering on any line that lists the entry docs (each file
   may name itself first).
3. Genuinely tool-specific operational notes (e.g. one tool's CLI quirk).

Everything else — non-negotiables, architecture boundaries, verification
commands — should be byte-for-byte identical. A quick check: strip the lines that
mention the entry-doc filenames, then diff the two files; what's left should match
except for the title/first-line.

## Rule 4 — Constraints follow capability, not machine/tool identity

When an entry doc states an environment constraint, phrase it by *capability*,
not by a hardcoded machine or tool name, because agents may later run in places
the original author didn't anticipate.

- Good: "only an environment that actually has the GUI editor + a display may
  claim visual verification; without it, mark it as needing verification
  elsewhere."
- Fragile: "this machine can't verify visuals" — wrong the day an agent runs on
  the machine that *can*.

This keeps the rule self-consistent across every agent and every host.

## Rule 5 — Sync discipline on every rule change

When you change a project rule, workflow constraint, verification command,
architecture convention, or long-lived decision, update the affected shared docs
in the *same* change, and keep the entry docs mirrored. Don't leave one doc
describing the old world. If you rename or remove a section that another doc
pointed into, fix the pointer in the same edit (and recall Rule 1 — that pointer
should be coming *from* an entry doc, never from a shared doc).

## How to audit an existing repo

First identify the two tiers for *this* repo, then run the checks. The entry
docs are the tool files that exist (`CLAUDE.md`, `AGENTS.md`, …); the shared docs
are every other doc this project keeps.

```bash
# Name the tiers for this repo. ENTRY is a regex; SHARED is the file list.
ENTRY='CLAUDE\.md|AGENTS\.md'                 # add more <TOOL>.md as they appear
SHARED=$(git ls-files '*.md' | grep -vE "$ENTRY")   # tune the glob to the repo

# Rule 1: shared docs must not name entry docs. Expect no hits.
grep -nE "$ENTRY" $SHARED 2>/dev/null

# Rule 2: shared docs should be tool-neutral. Inspect any hits.
grep -nE 'Claude|Codex|Copilot|Cursor|Gemini' $SHARED 2>/dev/null

# Rule 3: entry docs mirror each other. After stripping the lines that name the
# entry files, the remainder should differ only in the title / first line.
diff \
  <(grep -vE "$ENTRY" CLAUDE.md) \
  <(grep -vE "$ENTRY" AGENTS.md)
```

The entry docs might be `CLAUDE.md` + `AGENTS.md` today and grow a third
tomorrow; the shared-doc set is whatever the project actually has, so derive it
(as above) instead of hardcoding names.

## Recommended entry-doc skeleton

Keep each entry doc short and uniform. A workable shape:

```markdown
# CLAUDE.md            (or AGENTS.md, etc.)

<one line: "Claude Code entry guidance. Keep this short; detail lives in shared docs.">

## Read Order
- `<your-design-doc>.md`: <what's in it>
- `<your-setup-doc>.md`: <what's in it>
- `AGENTS.md`: <sibling entry doc; keep rules aligned with this file>

## Project Snapshot
<2–3 lines: stack + current state>

## Non-Negotiables
- <rules that must hold; phrase constraints by capability, not machine name>

## Documentation Sync
- Mirror agent-facing guidance between the entry docs.
- Put detailed design/progress in shared docs, not here.
- One-way dependency: entry docs may point into shared docs; shared docs must
  not back-reference an entry doc. Tool-specific differences live only here.

## Verification
<commands every agent can run; note what needs out-of-environment verification>
```

The sibling entry doc (`AGENTS.md`) is the same file with the title, first line,
and self-ordering flipped.

## Quick checklist

Before finishing any doc edit in a multi-agent repo:

- [ ] No shared doc names an entry doc (Rule 1).
- [ ] No shared doc names a single tool where a role word fits (Rule 2).
- [ ] Entry docs still mirror each other apart from title / self-ordering (Rule 3).
- [ ] Environment constraints are phrased by capability, not machine/tool (Rule 4).
- [ ] Any rule change touched all affected docs in this same edit (Rule 5).
