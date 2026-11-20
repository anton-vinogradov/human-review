---
name: human-review
description: "Use for code reviews and self-review before handing work over. Check why each change is needed, preserve required behavior, and prepare a change a person can understand."
---

# Human Review

[English](SKILL.md) | [Русский](SKILL.ru.md)

Start by reviewing the task itself. Add the criteria below to your usual review plan. Apply them equally to your own changes and to changes by others. Fix confirmed defects within scope. If the user asks for a read-only review, leave the files as they are and explain what needs fixing.

After making fixes, reread the entire diff from the original version to the current one. Trace the affected behavior, run the relevant checks, and repeat the review while you can still fix defects. Check whether the task, code, and tests explain the result without the chat history. Express the conditions that determine behavior in the code or tests. Explain a non-obvious constraint and its reason where it applies.

If one issue needs a decision from the user, continue with independent fixes. Report what you fixed and checked. For each remaining defect, describe a reachable scenario, the consequence, the evidence, and why it remains. If you could not run a check, say which conclusion is still unverified. A review can finish with no findings.

## Before the review

For an installed skill, run `python3 "<skill-directory>/scripts/update.py"` once before the review. Use the directory containing this file. On Windows, use `py -3` if needed. If the skill was supplied as a standalone attachment, proceed directly to the review.

The user opts in to automatic updates for each installation. The helper checks the source at most once every 24 hours and preserves local edits. If it returns `updated`, reread this file and use the updated instructions for the current review. Otherwise, continue with the installed version, including when tools, network access, or permissions are unavailable. Skip further attempts and permission requests solely for an update. Consent to update applies only to the skill installation. [Installation and automatic updates](README.md)

Use [human-writing](https://github.com/anton-vinogradov/human-writing) for all writing within the task. If the skill is not installed, read `skills/human-writing/SKILL.md` in its repository. If those instructions are unavailable, lead with the main point. State the action, conditions, and result. Distinguish verified facts from assumptions and keep the details needed to decide or act.

## 1. Task scope and simplicity

### The task and the review plan

Determine the intended outcome and how to check that the entire task is complete. Compare the request with the goal of the parent task, the requirements of related tasks, and the project's current state. Check the assumptions, whether the requirements are compatible, and whether the plan can work. Look for missing information in the project and related tasks first. If information is still missing, ask the user a specific question and explain which decision depends on the answer.

If parts of the change are easier to review separately, propose a plan for splitting the task. Give the result of each part, their order and dependencies, and a check for the whole task.

### The choice of solution and the scope of edits

Before judging individual edits, independently outline the most direct way to achieve the goal. Look for existing solutions in the project and its libraries and frameworks. See how other parts of the project use them. Use a suitable mechanism when one exists. Extend it when the required behavior fits its purpose and preserves the guarantees for other callers.

Compare your outline with the proposed solution. For each new operation, branch, setting, and piece of state, find what relies on it and what would be lost without it. Simplify when all required guarantees hold. If the proposed approach cannot work, find another way within scope. When you need additional permission, explain why and request it before taking the action that requires it. If a choice changes the goal, scope, or expected behavior, explain the options and consequences, recommend one, and let the user decide. Propose an incidental improvement separately and include it only if the user expands the task.

Change working code when you can say what will become easier to understand, verify, or modify. Prefer the solution with fewer conditions and relationships to keep in mind. On frequently executed paths, remove unnecessary calculations and temporary objects while preserving behavior and readability. Keep the existing implementation when the alternatives are equivalent.

Preserve the familiar file structure and code order unless the task or a defect requires a change. Aim for a small diff that shows when and how behavior changes. If later edits remove the reason for an earlier change, restore the original fragment. For a substantial redesign, walk through a concrete execution and explain what became simpler.

### Project conventions and dependencies

Follow project conventions, including line breaks. Choose names that convey purpose and match what a value contains. In the absence of explicit rules, follow established names and the way the project integrates components with the same purpose. Depart from that pattern when you can identify a property it lacks.

Consider an external library when it replaces custom code and simplifies the solution. Add a new test-only dependency yourself. Before using it in production code, explain the benefit and get the user's agreement, even if tests already use it. Check which version other parts of the project use. Use a compatible version already in the project, taking it from the shared version declaration where one exists. If an upgrade addresses a specific problem in the task, propose it and identify the affected parts. Verify their compatibility when upgrading.

### Explicit parameter values

Justify each fixed value. For an explicit argument, check the implementation or contract to find out whether it is required, which value the code would receive without it, and what constraints apply. In tests, account for shared setup, base classes, and helpers. Keep a value that matters to behavior, test reliability, or understanding a call, including when defaults might change. Remove an explicit value that merely repeats an effective setting.

## 2. Follow data from its source to the result

Trace affected paths from the data source to its consumers, including alternate branches, external calls, and calls made by a framework. Check data transformations and type casts. When reordering operations or changing collections, preserve the properties consumers rely on, including ordering, comparison rules, and access to state.

Define a value once when all its uses express the same fact. Keep independently meaningful values separate even if they happen to match. In a test, define a shared value where its role is clear. Identify what changes the state and what will see an update. Derive values from a common source. When a copy is necessary, check how it stays current. A snapshot must retain the value at the time of the event.

Remove a parameter that is the same in every supported call if behavior and compatibility are preserved. If a new operation is unreachable, wire it into the path where the task requires it or remove it. For a new check, identify a real input that reaches it. Before removing an existing check, trace the case it protects, including the scenario covered by its test. Remove it only when the required result holds on every supported path. If you cannot verify part of a path, keep the protection and say what still needs checking.

Follow the lifecycle of work that continues after a call returns. Check data lifetimes, ordering across threads, stopping, cancellation, retries, and cleanup on failure. Determine what happens to other items and later runs when one fails. If the contract promises a complete result, check that failure cannot expose a partial result to another operation. Release resources on every path.

## 3. Match the operation to the purpose of the call

Choose an operation whose name expresses the intended action and whose contract guarantees the result under the given conditions. Check the implementation or API documentation, including failure behavior. If the contract is sufficient and the caller passes the wrong data or misuses the result, fix the caller. If the operation violates its contract for valid input, fix its implementation.

If the required outcome occurs only as a side effect, or the call performs extra actions, find or extract an operation for the intended action. Keep distinct actions separate when combining them adds conditions a reader must work through to understand either call.

Before removing an adapter, determine what the caller would have to know about the other component to call it directly. Keep adapters that reconcile data formats, calling conventions, or guarantees. Remove the extra step when calling directly is simpler and preserves behavior.

## 4. Share code that expresses a shared rule

Find out why similar fragments need to change together. Define a shared rule in one place. When only part of the behavior is shared, combine only that part. Keep actions with independent reasons to change separate.

Extract repeated actions into an operation when it reduces code and each call makes its result clear. If understanding a call still requires tracing the internal branches, reconsider which actions belong in the operation.

## 5. Arrange code in reading order

Place fragments that need to be compared near each other. Declare a variable close to its first use in the narrowest scope that needs it. Preserve the conditions, timing, order, and number of evaluations, along with resource lifetimes.

Group related scenarios together in a test file. Choose an order that makes differences between adjacent cases visible and reduces the need to reconstruct the setup.

## 6. One test checks one guarantee

### Guarantee and coverage

Name the behavior the test must establish under its stated conditions. Keep several steps in one test when together they prove that guarantee. Test independent guarantees separately, even when they share setup.

Before adding a test, identify a reachable defect that existing tests would miss, including tests at other levels. If the scenario already exists, add the missing check for the same guarantee. Keep two similar tests when each detects a real defect the other would miss. For each production change, check whether a test would fail if the changed behavior broke.

When simplifying a test, preserve its ability to detect the original defect. For a known failure, reproduce it and verify that the test fails for the expected reason and passes after the fix. If you cannot verify this, retain the part of the test on which the result may depend.

### Scenario and setup

Name the test for the behavior it checks. When the relationship between conditions, actions, and outcomes needs an explanation, describe it using Given/When/Then. Test at the level that most clearly shows how real conditions produce the result. Add a server, network, or other infrastructure only when the guarantee requires it.

Keep every value that determines the result visible in the test. A helper's name and arguments should reveal the conditions it sets up. If a starting state is sufficient, establish that it is reachable. If the guarantee covers the transition into that state, exercise the transition. Make sure the code the test is meant to protect actually runs.

In a failure test, verify that execution reached the intended operation. If the timing or cause of failure matters, distinguish it from a different failure with the same external result. To test the response to another component's answer, you can use a stub. Its answer must be possible under the conditions of the test. If the guarantee covers how the answer is produced, supply real input and run the component's code.

### Outcomes and waiting

Check an outcome observable when using the program or calling its code. Check an intermediate action on its own when the action itself is the promised guarantee.

If the result is already available, check it immediately. Otherwise, wait for the required event or state. A timeout bounds the wait if the event never arrives. Check that the clock used for the timeout advances in the test. Account for execution pauses on a slow machine. When the requirement specifies a deadline, test that deadline.

## 7. A comment is a reason to examine the code

Find out what a reader would have to guess without the comment. If the same meaning can be expressed more clearly within scope, fix the code and remove the redundant comment. Keep an explanation of a non-obvious constraint, its reason, and what happens if it is violated.

If removing the underlying cause requires broader rework, explain the benefit and how it relates to the task. If the task depends on the rework, propose doing it first. If the two can be completed separately, offer to include the rework in the current task or handle it in a separate one.

## 8. The task determines how much documentation is needed

Read the existing instructions and compare behavior before and after the change. Edit the text when the task calls for it or when someone would otherwise have to guess an action, its order, a condition, or the outcome. Include information about existing behavior only when it is needed to use the change.

In API documentation, describe the available operations and any conditions or guarantees that the declaration does not convey. If the declaration makes the contract clear, omit optional prose or use an empty documentation block when the project follows that convention. Leave current call sites and their individual purposes out of the reference.

## 9. Logs show what the system does

Check the logs of affected systems whose failure disrupts the program or other systems, or requires intervention. Add missing records for operations and state changes within scope. For an operation, record the action and outcome. For a transition, record the old and new states. When records from several operations interleave, use existing information to connect each event to its outcome. Add data only when the connection is otherwise unclear.

In a failure message, identify the operation and the cause. State any action required for recovery or say if the system will retry on its own. When a metric or status signals failure, check that it also signals unexpected failures.

After adding a log record, review the component's log as a whole. If events outside the task are missing or similar events are described inconsistently, show what is difficult to trace. Propose filling the gaps and making records consistent in the current task or in a separate ticket.

Thank you in advance for the careful review and fixes!
