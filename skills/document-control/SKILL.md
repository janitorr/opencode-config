---
name: document-control
description: Use when deciding how project docs should persist across opencode agent sessions instead of chat history - e.g. structuring a docs/ folder with an index, wiring AGENTS.md to load it at session start, tracking an open-questions doc, or fixing a stale/duplicated docs setup. NOT for user-facing docs, API docs, docstrings, or formal QMS-style document control.
---

# Document Control

A pattern for keeping project documentation as the source of truth across
sessions, instead of relying on chat/session history (which agents don't
retain between sessions).

## The three doc categories

Every doc in the project should fall into exactly one category. State the
category at the top of the file if it's not obvious from context.

| Category | Update rule | Example |
|---|---|---|
| **Living** | Append/update freely as facts change. Resolved items stay in place, struck through with a date + source, never deleted — preserves the reasoning trail of what used to be assumed. | an open-questions/tracker doc |
| **Stable** | Edited in place when genuinely new *confirmed* info arrives. Not append-only — superseded facts are replaced, not accumulated. | a confirmed brief/spec doc |
| **Historical** | Frozen once superseded. Never edited again except a one-time pointer note redirecting to what replaced it. | old prep notes, early drafts |

## The index

One doc (conventionally `docs/README.md`) is the entry point: a short table
of contents listing every other doc under its category, one line each. Keep
it as a plain index, not a narrative summary — a "current state" paragraph
is tempting but goes stale fast and duplicates what the living doc already
says accurately.

## Wiring an entry point with AGENTS.md (opencode-specific)

opencode auto-loads a project-root `AGENTS.md` into **every** session's
context, regardless of which agent is active (custom agent, or built-ins
like `explore`/`general`/`build`) — just by walking up from cwd. No
`opencode.json` config is needed for this (that's only required for
*additional* instruction files via the `instructions` field).

Use this to make the doc-reading habit repo-wide instead of baking it into
one custom agent's prompt (which would only help when that specific agent
is active):

```markdown
# Project Rules

## Start of session
Before doing new work, read `docs/README.md` first — it's the index for
this project's living documents. Treat the docs as source of truth for
project state; don't rely on chat history alone.

## Working conventions
- `docs/<living-doc>.md` is a living doc: resolved items stay in place,
  struck through with a date and source, not deleted.
- `docs/<stable-doc>.md` is stable: edited in place, not appended to.
- Historical docs are frozen once superseded — one-time pointer note only.
- Don't create new docs speculatively. Create them when there's real
  content to put in them.
```

## Gotchas

- **Don't build a parallel session/changelog log.** Git commit history
  already records what changed and when, with a message. A separate
  narrative log duplicates this and drifts out of sync. Use `git log
  --oneline` / `git log -p` for "what happened when," and keep doc content
  organized by topic, not chronology.
- **Don't scaffold empty placeholder docs** for topics you expect to need
  later. Create a doc when there's real content, not speculatively — an
  index entry pointing at nothing is worse than no entry.
- **Resolving an open question is an edit, not a deletion.** Strike through
  the question text and add the answer + date/source inline, so a future
  reader can see both what was asked and what changed.
