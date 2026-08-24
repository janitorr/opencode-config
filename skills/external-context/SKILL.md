---
name: external-context
description: Use when work needs files outside the current repo - reading code or docs from a sibling directory (../), an absolute or ~/ path, another git repo, or when an external-directory permission prompt appears. Covers choosing between opencode `references`, `instructions`, and `external_directory` permissions instead of passing paths ad hoc every session. NOT for files inside the current project.
---

# External Context

Reading the path is only half the job. If the same outside-the-repo source
will be needed again, say so — don't silently re-read it every session and
leave the user to keep supplying paths.

## Which mechanism

| Situation | Mechanism |
|---|---|
| Docs, specs, or another repo consulted **across sessions** | `references` |
| Rules/standards that must apply to **every** session | `instructions` |
| Genuine one-off read | Nothing. Read it, approve the prompt, move on. |
| Broad directory access for bash/tooling, not a curated source | `permission.external_directory` |

Default to `references`. It is the only option that both advertises the
directory to agents and pre-authorizes access.

Pick `instructions` only for small, always-relevant rule files — they are
loaded in full into **every** session. A large doc tree belongs in
`references`, where it is read on demand.

Git-backed references clone and refresh asynchronously; a freshly added
repo may not be readable on the first attempt.

## Writing the config

Do not guess the schema. Load the **customize-opencode** skill first — it
has the exact `references` shape, the permission syntax, where the config
file lives, and the restart requirement.

The one thing worth knowing up front, because it fails silently: a
reference with no `description` is never advertised to agents. Always set
one.

## Anti-patterns

❌ Reading `~/work/specs/api.md` for the third session running, without ever
mentioning it could be configured.
✅ Read it, then: "This is the third time we've pulled from `../specs` —
want me to add it as a reference so it's available automatically?"

❌ Dumping a 40-file doc directory into `instructions`.
✅ One `references` entry with a description; the agent reads what it needs.

❌ Proposing config, then moving on.
✅ Config is not hot-reloaded — tell the user to restart.
