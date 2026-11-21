# Lens: branch

The work is committed and about to merge, so judge the change as a whole.

## Focus
- **Architectural fit:** does the change follow existing patterns, or does it introduce a new one without justification?
- **Atomic commits:** one logical change per commit, with no unrelated bundling.
- **Scope discipline:** nothing beyond the stated intent.
- **Public API impact:** callers and consumers identified, breaking changes flagged.
- **Migration and rollout safety**, where it applies.

## Failure modes
List up to 3 concrete ways this branch could break after merge. Each must come from the visible code, not a generic risk:
- **what:** how it fails;
- **where:** `file:function`;
- **trigger:** the realistic input or state;
- **detected_by:** the test, alert, or signal that would catch it, or `nothing`. Never claim a test ran.
- **covered_in_this_pr:** yes or no.

An empty list is valid.

## Requirements
Restate each requirement from the PR description, the linked story, or the plan doc. Mark each one:
- **met:** cite the code;
- **not met;**
- **needs human check:** it can't be judged from code alone, such as UI or browser behavior.

If no requirements text exists, write "no requirements source found" and skip this section.
