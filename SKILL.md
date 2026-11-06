---
name: human-review
description: "Review code changes for necessity, behavioral regressions, lifecycle correctness, and avoidable complexity. Use for a requested code review or pre-delivery self-review of a patch."
---

# Human Review

Ask why each change is needed, what behavior it changes, and whether an existing mechanism already does the work. A sound review may conclude that a suspected problem is absent or that no change is necessary.

## Scope

In review-only requests, report findings without modifying the patch. Apply fixes when requested, within the authorized scope. Do not turn a focused review into an unrelated refactor.

Follow the project's contracts, formatting, and verification requirements. This skill does not prescribe line lengths, test commands, or a commit, push, or CI workflow.

## Review method

### Justify each change

For each added operation, branch, field, parameter, or abstraction, identify the requirement it serves or the regression it prevents. Look for an existing mechanism before introducing another. Use this as an internal check; report material conclusions rather than narrating every line.

For removals, verify which callers and guarantees depended on the deleted code. Do not replace a removed abstraction with the same complexity scattered across its callers.

Investigate apparently redundant code before removing it. Ask which earlier change made it necessary.

### Compare behavior before and after

Trace the relevant path in both versions. Replacing an implementation can remove an implicit guarantee, such as validation at startup, resource cleanup, or reloading configuration. Identify the operation that previously supplied the guarantee and whether the new path still supplies it.

Check when failures occur as well as what fails. Preserve required behavior; distinguish it from incidental behavior before proposing a compatibility change.

### Establish lifecycle and ownership

Distinguish a ready object, a factory or callback, and an actual invocation. Determine who creates and owns each relevant object, how long it lives, and when its state becomes stale or is refreshed.

Follow callers and conditional paths before claiming duplicate creation or loading. A callback reference alone does not prove when it runs: inspect the receiving code and paths that use an existing object instead.

### Fix the cause across layers

Check where state and work move after a simplification. Removing a cached field does not fix stale state if a caller now captures the same snapshot in a lambda.

Verify each affected path separately. A dynamic provider may need fresh values while a fallback intentionally loads once. Do not impose one lifecycle on both without a requirement.

### Simplify without losing meaning

Prefer an existing compatible contract over a wrapper or adapter that adds no semantics. Keep an adapter when it performs a necessary conversion or enforces a contract.

Judge simplicity by the concepts, state, and indirection a reader must track. Fewer lines are not sufficient evidence. Preserve local formatting and avoid unrelated cleanup that obscures the functional change.

### Explain non-obvious reasons

Keep comments that explain why an apparently unnecessary operation preserves a required guarantee. Remove repetition of obvious actions. A useful explanation should prevent a plausible but incorrect simplification.

Keep API documentation precise about the contract. Write operational instructions around what the reader must change, when, and how to verify the result. Read edited documentation as a complete section before checking its technical accuracy and rendering.

### Verify the observable result

Choose focused checks that distinguish the required behavior from the regression. Use explicit inputs and expected outcomes. Exercise repeated operations when freshness or reuse matters, and check the affected fallback paths.

Assert internal call counts only when they establish a meaningful requirement, such as loading once. Do not add tests that merely reproduce the implementation. State which checks ran and which relevant behavior remains unverified.

## Judgment and delivery

Treat the user's diagnosis and your own first impression as hypotheses. Confirm them against the actual code and contract. If evidence contradicts a suspicion, explain the path that disproves it instead of inventing a fix.

For each material finding, identify the affected path, the triggering condition, the concrete consequence, and the supporting evidence. Separate confirmed defects from unresolved questions and optional improvements. Recommend the smallest change that addresses the cause.

Explain a substantial design alternative through one concrete execution: which object does what, what becomes simpler, and what behavior or compatibility changes. Resolve material tradeoffs with the user when they fall outside the agreed scope; do not make routine authorized fixes depend on another approval.

After making requested fixes, review the final diff for necessity and read the resulting code in context. Report the resulting behavior, relevant verification, and any remaining limitation. If there are no supported findings, say so plainly.
