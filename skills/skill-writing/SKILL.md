---
name: skill-writing
description: Use when authoring or reviewing a SKILL.md - writing an effective description that actually triggers, structuring the body, deciding what goes in the main file vs a references/ subfolder, and avoiding skills that bloat context or never get invoked. For exact frontmatter schema, file placement, and permission config, see the customize-opencode skill instead.
---

# Writing Good Skills

A skill only helps if (a) the agent decides to load it, and (b) once loaded,
it changes behavior for the better without wasting context. Most bad skills
fail at one of these two points.

## 1. The description is the whole game

Before the body is ever read, the agent sees only `name` + `description` in
`<available_skills>`. If the description doesn't trigger correctly, nothing
else in the file matters.

Rules:
- Write in **third person** ("Use when...", not "I help with...").
- State **both** what the skill does and **when** to use it.
- **Front-load literal keywords** the user would actually type: file
  extensions, CLI verb names, error message fragments, product/tool names.
  Don't paraphrase them into generic language.
- If the skill should stay quiet on adjacent topics, gate it explicitly:
  "Use ONLY when...".
- Test it mentally: could a model, given *only* the description, correctly
  decide to invoke this for 5 differently-phrased requests, and correctly
  skip it for similar-but-different requests?

Bad: `description: Helps with diagrams.`
Good: `description: Use when working with .c4/.likec4 files or LikeC4
CLI/config questions where exact DSL/CLI syntax is required...`
(real example: `.agents/skills/likec4-dsl/SKILL.md`)

## 2. Keep the body lean - progressive disclosure

SKILL.md is read in full every time the skill loads. Only put in it what's
needed for the *common* case:

- Core rules, conventions, and a short workflow/checklist.
- Concrete ✅/❌ examples for the specific mistakes models actually make with
  this tool/format - these are worth far more than abstract prose.
- If you're countering a known failure mode (model substitutes a
  similar-but-wrong command, drops a required flag, picks the wrong
  overload), name the anti-pattern explicitly and show the wrong output next
  to the right one.

Push everything else into a `references/` subfolder, one topic per file,
and just point at it by path from SKILL.md - the model reads a reference
file only when the task actually needs that depth. Don't inline 400 lines
of API detail that's only relevant 5% of the time.

Do **not** restate things the model already knows (general language syntax,
common tool usage). Only include what's non-obvious, project/tool-specific,
versioned, or easy to hallucinate.

## 3. When to add subfolders

- `references/*.md` - deep syntax/config/API detail, split by topic, linked
  from SKILL.md but not inlined.
- `scripts/*` - helper scripts the skill tells the model to *run* rather
  than reimplement inline.
- `evals/*` - test cases for checking the skill actually changes behavior
  (advanced; see `.agents/skills/likec4-dsl/evals/` for a real example).

Small skill -> single SKILL.md is enough. Growing past ~200 lines or
covering multiple large sub-topics -> split into references/.

## 4. Test after writing

- Skills aren't hot-reloaded - restart opencode after adding/editing one.
- Try triggering it with a few different natural phrasings to confirm the
  description catches them.
- If the model loads the skill but ignores its guardrails, the instruction
  is probably buried too deep or too verbose - move it earlier, make it
  shorter, or add an explicit ✅/❌ pair.

## Checklist

- [ ] `name` matches folder name, lowercase-hyphenated, unique across all
  skill locations (project + global, opencode/claude/agents-compatible)
- [ ] `description` states what + when, front-loads real trigger keywords,
  under the 1024-character hard limit
- [ ] Body covers the common case only; rare/deep detail moved to references/
- [ ] Concrete examples (ideally ✅/❌) for known failure modes, not just prose
- [ ] No restating of general knowledge the model already has
- [ ] Restarted opencode and confirmed the skill triggers on realistic phrasing
