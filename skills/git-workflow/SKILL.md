---
name: git-workflow
description: Use when creating git commits, writing or amending commit messages, staging changes, or recovering from a bad commit (wrong files staged, bad message, commit needs undoing). Covers message style, detecting a repo's existing convention, commit granularity, and safe recovery. Not for read-only inspection like git status/diff/log.
---

# Git Commit Workflow

## 1. When to commit

Commit on explicit request, or under a checkpoint the user pre-approved
("commit as you go", "commit after each fix"). A pre-approval scopes to that
task only - it does not carry into the next request.

Finishing an edit is not a reason to commit.

## 2. Determine the convention before writing anything

Work down this list and stop at the first hit:

1. **`commitlint.config.*` or `.commitlintrc*` exists** -> Conventional
   Commits, mandatory. The commit-msg hook rejects anything else. Stop here.
2. **Read the log** - `git log --oneline -30` and `git log -5 --format=%B`.
   Match the dominant pattern: prefix scheme, subject style, whether bodies
   are used, whether trailers appear. The repo wins over any preference of
   yours.
3. **No clear pattern** -> check for release tooling: `.releaserc*`,
   `release.config.*`, `.changeset/`, `cliff.toml`,
   `release-please-config.json`, or `semantic-release` /
   `standard-version` / `@changesets/cli` in `package.json`.
   - Found -> Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`).
   - Not found -> area prefix: `area: imperative subject`.

Type prefixes with no tooling parsing them are mostly ritual - the type is
usually already obvious from the verb. Area prefixes answer the question a
reader actually has when scanning a log ("does this touch my code?").

## 3. Deriving the area prefix

Take it from the touched paths - the package, module, or top-level source
directory. `src/auth/**` -> `auth:`. `packages/cli/**` -> `cli:`.

- Two unrelated areas is a signal to **split the commit**, not to write
  `auth, api: ...`.
- A change genuinely spanning the whole repo drops the prefix.

## 4. Message rules

- Imperative mood. Subject <= 72 chars, no trailing period.
- Body explains **why**; the diff already shows what. Omit the body entirely
  when the change is self-evident.
- No attribution or generated-by trailers unless the existing log already
  carries them.

```
❌ Added comprehensive error handling to improve robustness
✅ stream: handle EPIPE in the writer

❌ fix: fix bug
✅ auth: reject refresh tokens past their exp claim

❌ Updated the config file and also fixed a typo in the readme
✅ (two commits)
```

Pass multi-line messages as repeated `-m` flags rather than embedding
newlines in one quoted string:

```sh
git commit -m "auth: reject expired refresh tokens" -m "Tokens past exp were accepted because the clock-skew allowance was applied twice."
```

## 5. Staging

- Never `git add -A` or `git add .` when the worktree contains changes you
  did not make. Stage explicit paths.
- Read `git diff --staged` before **every** commit. No exceptions.
- If the staged diff spans unrelated concerns, split it into separate
  commits.

## 6. Recovery

| Situation | Do this |
| --- | --- |
| Bad message, commit not pushed | `git commit --amend -m "..."` |
| Wrong files in last commit, not pushed | `git reset --soft HEAD~1`, restage, recommit |
| Wrong file staged | `git restore --staged <path>` |
| Bad message, already pushed | Leave it. Fix forward. |
| Hook rejected the commit | Fix the cause, make a new commit |

**Ask the user first** before anything that discards work or rewrites shared
history: `reset --hard`, `restore`/`checkout` over modified files, `clean`,
force-push, rebasing pushed commits, `--no-verify`.

Never run commands that open an editor or prompt (`git commit` with no `-m`,
`rebase -i`, `add -p`) - they hang the session.

## Checklist

- [ ] User asked for this commit, or pre-approved committing on this task
- [ ] Convention determined from commitlint / log / release tooling - in that order
- [ ] `git diff --staged` read in full
- [ ] Only intended paths staged; no secrets, no build artifacts
- [ ] Subject imperative, <= 72 chars, no trailing period
- [ ] Diff is one logical concern, or split into several commits
- [ ] No uninvited trailers
