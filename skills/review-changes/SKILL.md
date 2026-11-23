---
name: review-changes
description: Review uncommitted changes or the current branch. Use when asked to review changes, the diff, uncommitted work, or the branch. Picks scope automatically, runs a fresh reviewer subagent, then validates its findings before reporting.
argument-hint: "[code|staged|branch|<base-ref>] [instructions]"
---

Two passes: a fresh `general-purpose` subagent finds issues, then this session validates them. The reviewer must not share this session's context, so it isn't anchored on whoever wrote the code.

## 1. Scope

Arguments: `$ARGUMENTS`

Always take a baseline first, whatever the arguments, so step 4 can detect reviewer edits:
- `git status --porcelain`
- `{ git diff HEAD; git ls-files -o --exclude-standard -z | xargs -0 cat; } | git hash-object --stdin` (status alone misses further edits to an already-modified file)

If the first word is `code`, `staged`, `branch`, or a ref that resolves (`git rev-parse --verify <ref>`), use that scope and skip the table. A ref means branch scope against that base. `branch` still resolves the base as below. Treat any remaining words as extra instructions for the reviewer.

Otherwise, run these in parallel:
- `gh pr view --json baseRefName -q .baseRefName` (fails when there is no PR)
- `git symbolic-ref --short refs/remotes/origin/HEAD`

Base: the PR's `baseRefName` first, then `origin/HEAD` without the `origin/` prefix, then whichever of `main` or `master` exists. Use `origin/<base>` if it resolves, since a stale local base pulls already-merged commits into the diff. Then run `git rev-list --count <base>..HEAD`.

| Tracked changes | Commits ahead | Scope |
|---|---|---|
| base unresolved | – | ask for the base |
| yes | 0 | code: staged + unstaged (`git diff HEAD`) |
| no | >0 | branch: `git diff <base>...HEAD`, `git log <base>..HEAD` |
| yes | >0 | ask: uncommitted changes only, or the whole branch including them? |
| no (untracked only) | 0 | ask: which untracked files to review? |
| no | 0 | say there is nothing to review and stop |

`git diff` never shows untracked files. In code scope, if `git ls-files -o --exclude-standard` lists any, ask whether to include them. Included untracked files go to the reviewer as files to read in full.

Ask with AskUserQuestion. Never guess a scope.

## 2. Gather

- Changed files: `git diff --name-only` for the scope (`--cached` for staged; `<base>...HEAD` for branch).
- Branch scope also needs: commit list, PR title and body (`gh pr view --json title,body`), and any story or plan doc the user names or the PR links to. These feed the Requirements section of `lens-branch.md`.
- Find project rules: `CLAUDE.md`, `AGENTS.md`, `.cursor/rules`, and lint configs in the repo root and in the changed files' directories.

## 3. Review (subagent)

Read `checklist.md` and the lens for the scope: `lens-code.md` or `lens-branch.md`. Spawn one `general-purpose` agent. Its prompt contains:
- the scope's exact diff command and the list of changed files, plus any included untracked files;
- the requirements text, for branch scope;
- the paths of the project rule files;
- the full text of `checklist.md` and the lens;
- the user's extra instructions, if any.

Don't pass your own opinion of the change. The reviewer works from the code alone.

## 4. Validate

Read this skill's `rigor.md` and apply it to every finding the reviewer returns.

Then read the branch's ledger, `review-changes/<branch>.md` under the directory `git rev-parse --git-common-dir` prints, and apply the repeat rule in `rigor.md`. On the base branch or a detached HEAD, skip this and say so in one line. If there is no ledger, say "no earlier review record".

Re-run both baseline commands from step 1. If either output changed, the reviewer edited files. Report that first, at the top of the output.

## 5. Output

State the scope in one line: code, staged, or branch vs `<base>`, with the file count. Then give the sections in the order `rigor.md` defines. If there are no findings, say "none found".

After the output, unless the ledger was skipped, append one line per validated finding to it: date, `path`, enclosing function (or heading; else the file), severity, and a one-line issue. Create its directory first (`mkdir -p`), since a branch name with a slash nests.
