---
name: human-review
description: "Check whether code changes are necessary, preserve required behavior, and keep the solution simple. Use for code reviews and self-review before handing over work."
---

# Human Review

[English](SKILL.md) | [Русский](SKILL.ru.md)

Set the review scope from the task requirements and affected behavior. Examples in the rules do not limit where those rules apply.

## 1. Justify every change and keep the diff minimal

For each edit, identify the requirement it meets and how it changes behavior.

Justify why it is needed. Check whether an existing tool or solution can replace it.

Before you judge individual edits, outline the simplest solution of the task, even if a solution already exists. Start from the task's goal and acceptance criteria, not from the implementation steps it prescribes.

List what the change adds:

- mechanisms, such as threads, schedulers, caches, registries, and protocols between components;
- public members;
- fields and collections;
- rules, such as formulas, limits, retries, waits, and ordering;
- checks and special cases;
- values derived from shared data.

For each item, name the requirement it serves and who relies on it: a caller, an operator, or another guarantee. Remove an item that nothing relies on. A public member that only the change itself calls does not need to be public.

For each new mechanism, search the project for an existing mechanism with the same purpose. In the report, name what you found or say that you found nothing. Before you reject an existing mechanism, read how other code in the project uses it: its callers often show how to supply the property it lacks. In the report, name the missing property and the callers you checked.

Compare the change with your outline. Each component, state owner, protocol, or public API that the outline does not need must have a reason. Do not keep a design only because the task text prescribes it. If the outline is simpler but weakens a guarantee, describe the trade-off and let the task owner choose.

Before you recommend removing a check, describe the worst scenario that it protects against. Include cases with several elements or participants if they are possible. Recommend the removal only if that scenario cannot happen or does no harm. If the author covered the check with a test, take the scenario from that test. If you cannot show that it is harmless, ask the author instead of recommending the removal.

Check whether each part of the change is necessary. Determine what changes if it is removed or simplified.

For every change, ask yourself: “Why is this needed? Why can’t it be simpler and clearer?”

Keep the diff minimal without losing required functionality. Do not add preparation for hypothetical future tasks or unrelated formatting changes.

## 2. Trace the full behavior

Review the entire affected flow: calling code, conditions, bypass paths, and code that consumes the result.

Find out who owns the state, when it is loaded, and which operations will see the updated value. Other components or nodes may change the state too. Then derive any status about it, such as a failure count, from what you observe now, not only from your own actions. Some work continues after the call returns. Find out what context it takes from the caller and how long it keeps it.

Read the implementation of each helper or library call that your conclusion depends on. Do not infer its behavior from its name or parameters. If you did not read it, do not state its behavior as a fact: mark the conclusion as unverified.

Account for when errors occur, retries, resource cleanup, and how the cause of an error reaches the user.

## 3. Fix the problem in the responsible layer

First, learn what the component receiving the data or call can already do.

If its capabilities are sufficient, fix the calling code. Do not give the component extra responsibilities just to make calls look alike.

Keep adapters that perform a necessary conversion or uphold a contract between components.

## 4. Keep the implementation simple and explicit

Judge simplicity by how many concepts, state owners, and relationships the reader must understand. A small diff alone does not mean a simple solution.

Keep state that changes together in one owner, next to the operation that changes it. This owner also controls access to the state. Two collections over the same elements, or one collection that holds a filtered copy of another, split that owner. Keep one collection of objects that each hold all the state of their element, and filter when you iterate. Do not keep a field that copies what another field already gives, such as a typed reference or a flag read from it: derive it where you need it. If a dependency becomes available only later, start with a harmless default, such as a no-op object, instead of a flag or a special path for the early calls.

Compute a value derived from shared data in one place, at the owner of that data, and reuse it. Before writing such a computation, look for the same one in other code that reads the data, including code outside the diff. If every reader converts or groups the data the same way, store it in that shape when it is written.

Sometimes other code has to follow a protocol with the internals of another component or report what it did. In that case, look for a design where one owner orders the operations. When you change the order of operations, check every operation that relied on the old order. When you replace a collection, keep the properties its readers rely on: iteration order, how keys are compared, and whether callers can change what it returns.

Choose the simplest mechanism that gives the required precision and guarantees. Add precision, caching, or notifications only when the task needs them. The same applies to the rules the code enforces: a formula, a retry or backoff policy, randomization, a wait, an ordering promise. Prefer a rule that its user can predict from one sentence. Keep each refinement only if you can name the problem it solves in this system. Question rules that the task text prescribes in the same way as code.

When a simplification makes a mechanism less precise, such as polling instead of exact timing, find every setting and guarantee that depended on the old precision. State the new limit in the documentation or enforce it.

Compute a cheap and harmless value always, rather than only when a switch is on. Each such condition adds states that other code must handle, such as a value computed earlier with the switch off. Let the switch control behavior, not data.

Publish the results of an operation only after the whole operation succeeds. Do not change state midway and then restore it on failure.

For each check, branch, or special case, name a realistic input that reaches it. Then look for what already guarantees the same outcome: the data structure, an earlier step, the contract of a called API, or the library that consumes the data. The check is redundant if such a guarantee exists or only contrived inputs reach it. Do not repeat a validation that the next step or a library performs anyway, unless the task needs an earlier or clearer error. A documented contract counts only if the implementations in use honor it. If the platform's own code makes the same check, keep it too. Judge reachability this way only for checks that the change adds. Do not use it to drop a guarantee that existing code already gives: changing existing behavior needs a requirement.

For each synchronization or hand-off between threads, such as an executor, a future, a lock, a wait, or a context passed to another thread, name the state or order it protects. Use the lightest primitive that protects it. Remove it when that state is gone. A stop or cancel signal interrupts only the current blocking call, so a loop over several units of work must check it before each unit.

For each explicitly set parameter, determine what changes if it is removed. Check the API implementation for defaults, required arguments, and constraints. Account for all sources of effective values. In tests, these may include the base class, shared setup, and helper methods.

Keep the parameter if removing it breaks the code, changes required behavior, or prevents the test from reliably checking its scenario. If a suitable value is already set or the setting does not affect the scenario, it is redundant. Similar settings in nearby code do not prove that the parameter is necessary.

Check whether each data transformation, including type casts, is necessary and look for a simpler solution.

Declare variables as close as possible to their first use, in the narrowest necessary scope. When moving declarations, preserve the required conditions and order of evaluation, as well as resource lifetimes.

The result of each operation must be clear. If code relies on a side effect, consider an explicit action that directly expresses the required change.

## 5. Choose values, names, and formatting deliberately

Justify the choice of every fixed value. Each value must have a clear meaning.

Extract values used repeatedly with the same meaning into variables or constants.

Choose names for code elements that clearly show their purpose. A name must match what the element holds or returns, including the keys and values of a map. Use the project’s terminology and naming conventions.

Follow the project’s formatting rules, including line lengths and line breaks.

## 6. Check requirements with simple, clear tests

For each test, name the single guarantee it checks. Its name, inputs, actions, and result must explain the idea without relying on the discussion history.

Give each test a short description in the project’s documentation format, even if its name looks clear. Write it in three parts: Given — the state that the test creates, When — the action, Then — the result that the test checks. When and Then together state the guarantee. Describe setup methods and helpers that prepare the test state too: first what they give the tests, then how. Write for a reader who sees the test for the first time. Then compare the test with the description. Remove setup, actions, and checks that the description does not need. If the test needs a step that the description does not explain, add it to the description or simplify the test.

Make a separate pass for overloaded tests. For each test, list the required behaviors behind its assertions, including checks hidden in helpers. Split independent behaviors into focused tests and keep only the setup each one needs. A shared failure or setup does not make independent requirements one scenario. Keep stages together when a later check needs the earlier stage, such as recovery after a failure. Different inputs or failure points handled by the same code need not become separate tests.

Choose the smallest set of scenarios that covers the changed behavior and possible regressions.

For each test, name a supported action or external event that creates its starting state. What would someone observe if the code broke? Assert that observable effect itself, not a sign of it such as a new object or a changed field. Trace the path to that point. Check what has already succeeded and which settings can still change. A direct API test can also protect a documented contract.

Create a scenario through the inputs that users, operators or the environment produce: data, files, configuration and calls. If the scenario depends on another component’s result, run that component on such inputs. Do not stub its result. Use test doubles only for time, concurrency, unavailable external systems, and components that the scenario does not depend on.

In the report, list the test doubles of the reviewed tests. For each one, name what it replaces and the real situation it stands for. Say whether real inputs could create that situation. Go through every test that uses the double, not only the first one you notice. For each test whose scenario comes from a stubbed result that real inputs could produce, name the level where real components create it.

Before you call a test double or a part of a test unnecessary, break the production code that it guards and run the test. If you cannot run the test, ask the author instead of stating that it checks nothing.

Compare tests that promise the same outcome, including tests in other classes or at other layers. Name a reachable regression that one catches while the others pass. If everything it checks is already covered, remove the duplicate.

Test each requirement at the lowest level where real inputs can create its scenario. Add infrastructure, external dependencies, or waits only when the required behavior cannot be checked without them.

Keep tests as simple, clear, and small as possible. Keep only setup, operations, and checks needed for the guarantee being tested.

Justify each nontrivial part of the test. Check whether a simpler approach will work.

Use the test framework’s features. Read the project’s tests and look for suitable solutions there.

Do not hide the scenario’s meaning behind test implementation details, including helper functions, flags, and complex setup.

Order tests to make them easy to read. Moving to the next test should require as little context switching as possible.

Confirm that the test actually reached the condition or event being tested. For a failure test, check that the intended failure occurred.

Break the code that each test protects and run the test: the test must fail. Before removing it, show which remaining test catches each reachable regression. If only the fixture permits a failure, identify the production step that prevents it.

Wait for the required event instead of an arbitrary delay. Justify every wait that uses a timeout. Where possible, replace it with an action that does not need that wait.

For each wait with a timeout, find the clock that the timeout reads. Check that this clock moves in the test environment. Otherwise the test hangs instead of failing when the event never happens.

A slow or paused machine must not change the test result. Assume a long pause between any two steps of the test and of the code it runs. Use a timeout only to limit the wait for an event that must happen, and make it long enough that only a missing event exceeds it. Do not expect that an event has not happened yet after some delay, that one thread runs before another without synchronization, or that the clock does not move between two calls. When a check compares times, leave a margin larger than such a pause, or compute the expected value from the same time values that the code uses.

Release resources even when an error occurs.

## 7. Write documentation and explanations in plain language

Use the [human-writing](https://github.com/anton-vinogradov/human-writing) skill for all work texts within the task.

If the skill is not installed, read `skills/human-writing/SKILL.md` in its repository. If those instructions are unavailable, apply the rules in this section.

Start with the main point. Be specific: who does what, under which conditions, and with what result.

Use verified facts. Do not invent system behavior or present assumptions as established causes.

Keep technical details that help the reader understand the solution or carry out an action.

If code needs a comment, check its readability and the overall approach. First consider clear names and a simpler solution that needs no explanation.

Keep a comment about a non-obvious reason or constraint if both conditions hold:

- The reason or constraint cannot be changed within the scope of the task.
- No suitable solution has been found that removes the need for an explanation.

Express obvious actions through code and names.

Write documentation for someone who did not take part in the discussion. Instructions must let the reader complete the task without guessing.

Before editing documentation, compare the behavior being described before and after the change. Read the existing description. Justify each edit by a need in the current task: what would the reader misunderstand or be unable to do without it?

Do not add descriptions of unchanged behavior just because you studied it during the work. Add information about it only as necessary context for the change or when the task explicitly requires it. Do not duplicate information already provided. If the need for an edit is not established, leave the documentation unchanged.

Keep documentation diffs minimal.

If the behavior of a code element other than a test is clear from its name, do not add a description. Use an empty documentation block only if the project’s style requires one.

When documenting a code element, describe its contract and capabilities. Do not list where, how, by whom, or why it is used.

## 8. Support findings with evidence

Treat the user’s initial diagnosis and your own first impression as hypotheses to check. This includes a forecast that a proposed design will be more complex. Sketch it and compare both versions by collections, fields, branches, casts, and where each element's data and behavior live.

For each finding, show the scenario that triggers it, its consequence, the evidence, and the smallest sufficient fix.

Distinguish bugs from open questions and optional improvements.

A review with no findings is a valid result.

## 9. Check and explain the final solution

After making fixes, reread the entire affected flow and the final diff.

When a finding or a fix changes a requirement, repeat the outline from section 1 for the affected part. Mechanisms, checks, and tests kept for the old requirement may no longer be needed.

Compare each changed fragment of existing code, tests, and documentation with the original. If a later change removed the reason for it, restore the original and keep only the lines the task needs.

Check that every remaining edit is necessary and that existing tools and simpler solutions have been considered.

For a substantial rework, walk through a concrete execution example. Explain what became simpler and what changes to behavior or compatibility this required.

Report your conclusions, completed checks, and remaining gaps.

For each unresolved problem, explicitly identify and visually highlight it in the report, for example with `⚠️ Unresolved problem`, or ask the user a specific question.

## 10. Update the installed skill when using it

Before a review, run `python3 "<skill-directory>/scripts/update.py"` once. Use the directory containing this `SKILL.md`, regardless of the project being reviewed. On Windows, use `py -3` if needed.

The user enables automatic updates once for each installation. Do not enable them on the user’s behalf during a review. Installation and the command to enable updates are described in [README.md](README.md).

The script contacts the upstream source at most once a day and preserves user edits.

If the script reports `updated`, reread this `SKILL.md` and use the updated instructions in the current review. You do not need to run the script again for this invocation of the skill.

Otherwise, continue with the installed version. If tools, network access, or permissions are unavailable, do not retry or request broader permissions just to update the skill.

Permission to update the skill does not grant permission to change the project being reviewed.

## Thank you

Thank you for the careful work. A thorough review saves the author and the users from real problems, and it is appreciated.
