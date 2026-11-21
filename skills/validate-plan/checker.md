# Checker brief

You are fact-checking a plan against the real codebase. **Do not edit, write, stage, or commit anything. Report only.** Don't propose new plan items or a rewrite.

## 1. Claim inventory

Go through each plan step and list every claim it depends on that can be checked:
- file paths, modules, functions, types, and their signatures;
- current behavior ("X already does Y", "Z returns ...");
- data shapes and schemas, config keys, environment variables, CLI flags, commands, test names;
- integration points ("A calls B", "this hooks into C", "nothing else uses D");
- how a library or external API behaves;
- assumptions stated as fact, or implied by a step.

Mark a claim **load-bearing** if the plan's approach fails when it's wrong. Verify the load-bearing claims first and most thoroughly. If you run out of effort, unchecked minor claims go to `unverified`. Never mark a claim confirmed without checking it.

## 2. Verify

Prefer primary sources, in this order:
1. **Code**: read the file and the implementation. For behavior, trace callers and data flow; one read is sampling, not validation.
2. **First-party docs**: the repo's README, AGENTS.md, CLAUDE.md, package manifests, and the library's official docs (a docs tool such as Ref or Exa if available, otherwise fetch the official docs page; official sources only).
3. **Git history** (`git log -p`, `git blame`), for claims about past or recent behavior.

**Absence** claims ("no other callers", "no tests"): search 2+ ways before accepting them, such as alternate names, re-exports, dynamic references, and other test locations.

Never judge a claim from memory. If you can't settle it, it's `unverified`.

## 3. Verdict per claim

- `confirmed`: give the evidence (`path:line`, a doc quote, or the command and its output).
- `wrong`: state what the source actually shows, as a full fact, with the evidence.
- `unverified`: say what you tried and what would settle it.

Severity for `wrong`, and for load-bearing `unverified`:

| Severity | Meaning |
|---|---|
| Blocker | A false premise the approach rests on; the plan can't work as written (the symbol doesn't exist, the API behaves the opposite way, the data model is wrong). |
| High | A wrong signature or integration point, a missing dependency, or a step that conflicts with real behavior; causes major rework. |
| Medium | An assumption presented as fact, an outdated reference, or an underspecified step that is likely wrong. |
| Low | Cosmetic: a stale path that doesn't change the build, or a naming slip. |

## 4. Coverage

Map each numbered request clause to the plan steps that deliver it. Report:
- clauses that no step delivers;
- steps that serve no clause.

## Output

1. **Claims table**: step, claim, load-bearing (yes/no), verdict, severity, evidence.
2. **Plan impact**: for each wrong claim, the plan steps it invalidates.
3. **Coverage**: unmapped clauses and unmapped steps.
