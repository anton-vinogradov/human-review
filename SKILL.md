---
name: human-review
description: "Review patches for necessary scope, preserved behavior, and simpler design. Use for code review or pre-delivery self-review."
---

# Human Review

For each change, ask: what requirement does it serve, what behavior does it alter, and can an existing mechanism do the work?

Review-only requests do not authorize edits. Follow the project's conventions and the user's scope, including test and publication limits.

## Trace the behavior

Compare the old and new execution paths, including callers and bypass branches. Distinguish passing an object, passing a factory, and invoking it. Establish who owns the state, when it is loaded, and which operations see a replacement.

Investigate apparent duplication before removing it. Which earlier change made this operation necessary? A new implementation may have lost an implicit startup check or cleanup. Verify branches before claiming work happens twice.

Check failure timing, retries, resource cleanup, and whether the original cause reaches the user. An error moving from startup to connection setup can change behavior even when both paths eventually fail. Check consumers before treating an internal exception type as a compatibility requirement.

## Simplify the responsible layer

First inspect what the receiving component already supports. If its contract is sufficient, correct what the caller supplies instead of adding another responsibility to the receiver. Do not move work into a shared constructor solely to make callers look alike.

Justify each added operation, field, branch, or abstraction by the current task. For removals, check dependent callers and guarantees. Prefer an existing compatible contract; retain adapters that perform a meaningful conversion or enforce a contract.

Follow state and work across the whole affected path. Removing a cache does not fix stale state if a caller captures the same value in a closure. Conversely, a dynamic provider and a file-based fallback may intentionally load at different times. Unifying them can introduce a regression.

Judge simplicity by the concepts, state owners, and indirection a reader must track. A smaller diff is not always a clearer design. Avoid unrelated formatting and preparation for future tasks.

## Test the requirement

Use explicit inputs and observable outcomes that distinguish the intended behavior from the regression. Keep each test focused; remove settings and helpers unrelated to its execution path or prerequisites. Internal call counts are useful only when the count is a requirement, such as loading once.

Repeat operations when freshness or reuse matters. Check affected alternatives and error paths. Reuse relevant coverage; add tests for missing behavior, not to mirror each implementation detail.

## Explain and deliver

Keep comments that explain a non-obvious reason and prevent a plausible wrong simplification. Remove narration of obvious operations. Keep API contracts precise and user instructions actionable; read edited sections without the diff to check that they stand alone.

Treat both the user's diagnosis and your first impression as hypotheses. Report supported findings with a triggering scenario, consequence, evidence, and the smallest useful correction. Separate defects from open questions and optional improvements. No findings is a valid result.

For a substantial redesign, explain one concrete execution, what becomes simpler, and the behavioral or compatibility cost. Resolve choices outside the agreed scope with the user; continue routine authorized fixes without another approval.

After fixes, read the complete affected flow and the final diff. Report conclusions, checks run, and remaining gaps, rather than narrating a question for every line.
