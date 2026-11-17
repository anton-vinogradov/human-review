---
name: human-review
description: "Check whether code changes are necessary, preserve required behavior, and keep the solution simple. Use for code reviews and self-review before handing over work."
---

# Human Review

[English](SKILL.md) | [Русский](SKILL.ru.md)

A review strives for a change that a maintainer can accept without regret. Every part of it is needed. It behaves correctly in every situation the system can meet, including failures. It fits the project like the code around it and is as simple as the task allows. The people who run it can see what it does, and its tests prove what it promises.

Each section states one aim and the directions that lead to it. Examples show a direction; they do not limit it. If a case matches no example, apply the aim. Set the review scope from the task requirements and the affected behavior.

## 1. Every part of the change is needed

Aim: each element of the change serves a requirement that someone relies on, and the diff contains nothing else.

Before you judge individual edits, outline the simplest solution of the task, even if a solution already exists. Start from the task's goal and acceptance criteria, not from the implementation steps it prescribes. Compare the change with the outline. Each component, state owner, protocol, public member, or rule that the outline does not need must have a reason. Do not keep a design only because the task text prescribes it. Question rules that the task text prescribes, such as a formula, a retry policy, or a wait, the same way as code.

List what the change adds: mechanisms, public members, fields and collections, rules, checks and special cases, values derived from shared data. For each item, name the requirement it serves and who relies on it: a caller, an operator, or another guarantee. Remove an item that nothing relies on. A member that only the change itself calls does not need to be public.

For each edit, identify the requirement it meets and how it changes behavior, and check whether an existing tool or solution can replace it. Ask: why is it needed, and why can't it be simpler and clearer? Determine what changes if it is removed or simplified.

If the simpler outline weakens a guarantee, describe the trade-off and let the task owner choose.

Keep the diff minimal without losing required behavior. Do not add preparation for hypothetical future tasks or unrelated formatting changes.

## 2. The change fits the project like its peers

Aim: new code does what the project already does in the way the project does it, so a reader meets nothing new without a reason.

For each new mechanism, such as a thread, a scheduler, a cache, a registry, or a protocol between components, search the project for an existing one with the same purpose. In the report, name what you found or say that you found nothing. Before you reject an existing mechanism, read how its callers use it: they often show how to supply the property it lacks. Name the missing property and the callers you checked.

When a part gets a lifecycle, threads, metrics, configuration, or logging, connect it through the same path that its peer components use. If peers differ, follow the path that most parts of the same kind use; an exception does not set the pattern. A direct call from central startup or shutdown code, or a one-off way of doing what peers do in a common way, needs a reason.

Do it the right way in the change that creates the need. If the right way requires restructuring existing code outside the task, name the debt, its reason, and its cost in the review, and let the task owner decide whether to do it now or in a separate task.

Fix a problem in the layer responsible for it. First learn what the component that receives the data or call can already do. If that is enough, fix the calling code. Do not give a component extra responsibilities just to make calls look alike. Keep adapters that perform a necessary conversion or uphold a contract between components.

Place each decision where its reason lives. The code that owns a loop or a lifecycle decides when to stop and what to do after a failure. A unit of work stays a plain operation that throws.

Use the project's terminology, naming conventions, formatting rules, and test framework features. Read nearby code and tests for solutions.

## 3. Behavior is correct in every reachable situation

Aim: for each situation the system can meet, you can say what happens, and that statement rests on code you have read.

Review the entire affected flow: calling code, conditions, bypass paths, and code that consumes the result. Some work continues after the call returns. Find out what context it takes from the caller and how long it keeps it.

Find out who owns each piece of state, when it is loaded, and which operations see an update. If other components or nodes can change the state, derive any status about it, such as a failure count, from what you observe now, not only from your own actions.

Read the implementation or contract of each helper, library, or framework call that your conclusion depends on, including what it does on failure. Do not infer its behavior from its name or parameters. If you did not read it, mark the conclusion as unverified.

For each synchronization or hand-off between threads, name the state or order it protects. Use the lightest primitive that protects it, and remove it when that state is gone. A stop or cancel signal must reach every unit of work, not only the current blocking call.

When you change the order of operations, or replace or merge collections, tasks, or mechanisms, list the properties that their users relied on and keep them or state the new limit. Examples: iteration order, how keys are compared, whether callers can change returned data, timing precision, and isolation of failures between units.

## 4. Failures are contained and visible

Aim: no failure, expected or not, passes silently, stops unrelated work, or leaves state that nobody can see.

For each operation, list how it can fail: expected errors, unchecked exceptions, errors such as a missing class or a failed assertion, interruption, and timeouts. Follow each failure to where it ends up, find out what keeps running after it, and how its cause reaches the user or operator. Decide the scope of each failure: whether one bad element of the input, such as an entry or a record, fails the whole operation or is skipped with a warning. Follow what the platform does with the same input.

Give background and asynchronous work the most attention: scheduled tasks, callbacks, futures, worker loops. Read what the executor does with an exception that escapes a task. It may cancel all later runs and keep the exception where nobody reads it. A failure of one unit must not stop the other units or the schedule.

Every failure must reach the same signals as the expected ones: a log record with the cause and the affected objects, and the counter, metric, or status meant to warn about failures. A signal meant to warn operators must cover every way the mechanism can stop.

Publish the results of an operation only after the whole operation succeeds. Do not change state midway and then restore it on failure. Release resources on every path.

## 5. Operators can see what the system does

Aim: from logs and metrics alone, a person on call can tell what an important subsystem did, when, and why it failed.

Each action of an important subsystem leaves a short, specific log record: the start of periodic work at debug level, outcomes at info level, failures at warn or error level with the cause.

A log record, metric, or status states what its reader needs to act, such as what failed and when the next attempt is. It covers every case it claims to cover.

## 6. The design is as simple as the task allows

Aim: the reader needs the fewest concepts, state owners, and relationships to understand the change. A small diff alone does not mean a simple design.

Keep state that changes together in one owner, next to the operation that changes it. This owner also controls access to the state. Each fact lives in one place. Derive a value where you need it instead of keeping a field that copies it. Two collections over the same elements, or one that holds a filtered copy of another, split the owner: keep one collection of elements that each hold all their state, and filter when you iterate. Compute a value derived from shared data at the owner of that data and reuse it; look for the same computation in other readers, including code outside the diff. If every reader shapes the data the same way, store it in that shape. If a dependency becomes available later, start with a harmless default instead of a special path for early calls.

If other code must follow a protocol with the internals of a component or report what it did, look for a design where one owner orders the operations.

Choose the simplest mechanism and rule that give the required precision and guarantees. Add precision, caching, or notifications only when the task needs them. Prefer a rule that its user can predict from one sentence. Keep each refinement only if you can name the problem it solves in this system. When a simplification lowers precision, find every setting and guarantee that depended on it, and state the new limit in the documentation or enforce it.

Compute cheap, harmless data always. Data computed only when a switch is on adds states that other code must handle, such as a value computed earlier with the switch off. Let a switch control behavior, not data.

For each check, branch, or special case that the change adds, name a realistic input that reaches it. Then look for what already guarantees the same outcome: the data structure, an earlier step, a called API whose implementations in use honor its contract, or a library that consumes the data. The check is redundant if such a guarantee exists or only contrived inputs reach it. Do not repeat a validation that the next step or a library performs anyway, unless the task needs an earlier or clearer error. If the platform's own code makes the same check, keep it. Do not use this reasoning to drop a guarantee that existing code gives: changing existing behavior needs a requirement.

Before you recommend removing any check, describe the worst scenario it protects against, including cases with several participants. If the author covered the check with a test, take the scenario from that test. If you cannot show that the scenario is impossible or harmless, ask the author.

For each explicitly set parameter, determine what changes if it is removed. Check the API implementation for defaults, required arguments, and constraints. Account for all sources of the effective value, including base classes, shared setup, and helpers in tests. Keep it if removing it breaks the code, changes required behavior, or keeps a test from reliably checking its scenario. If a suitable value is already set or the setting does not affect the scenario, it is redundant. Similar settings in nearby code do not prove that it is needed.

Make the result of each operation explicit instead of relying on a side effect. Check whether each data transformation and cast is necessary. Declare variables in the narrowest scope near their first use, and preserve conditions, evaluation order, and resource lifetimes when you move them.

## 7. Code that runs often is cheap

Aim: code costs no more than its job needs, especially where it runs often or where others call it at their own pace, such as metrics, monitoring, and per-message or per-request paths.

Find out how often each piece of code runs and who calls it.

Write with plain constructs: loops, arrays, and primitive values. A stream, a chain of lambdas, boxing, or a temporary collection needs a reason each time. On a frequently run path, extra work or allocation per call is a finding.

## 8. Values and names say what the reader needs

Aim: each value and name tells the reader exactly what they need, and nothing else.

Justify the choice of every fixed value. Extract a value used repeatedly with the same meaning into a variable or constant.

A name shows the element's purpose in the project's terms and matches what the element holds or returns, including the keys and values of a map. Keep it as short as that allows. Leave caveats, bounds, and implementation precision out of names; if a caveat matters, state it once in the documentation.

## 9. Tests prove the guarantees in real situations

Aim: each test proves one guarantee on a situation that users, operators, or the environment can create, and fails when that guarantee breaks.

For each test, name the single guarantee it checks. Its name, inputs, actions, and result must explain the idea without relying on the discussion history.

Give each test a short description in the project's documentation format, even if its name looks clear. Write it in three parts: Given — the state that the test creates, When — the action, Then — the result that the test checks. When and Then together state the guarantee. Describe setup methods and helpers that prepare the test state too: first what they give the tests, then how. Write for a reader who sees the test for the first time. Then compare the test with the description. Remove setup, actions, and checks that the description does not need. If the test needs a step that the description does not explain, add it to the description or simplify the test.

Make a separate pass for overloaded tests. For each test, list the required behaviors behind its assertions, including checks hidden in helpers. Split independent behaviors into focused tests and keep only the setup each one needs. A shared failure or setup does not make independent requirements one scenario. Keep stages together when a later check needs the earlier stage, such as recovery after a failure. Different inputs or failure points handled by the same code need not become separate tests.

Choose the smallest set of scenarios that covers the changed behavior and possible regressions. Derive them from what users, operators, and the environment can produce, not from the branches of the code. Include failures the code does not expect, such as an unchecked exception or an error from a pluggable component, especially in background work.

For each test, name a supported action or external event that creates its starting state. What would someone observe if the code broke? Assert that observable effect itself, not a sign of it such as a new object or a changed field. Trace the path to that point. Check what has already succeeded and which settings can still change. A direct API test can also protect a documented contract.

Create a scenario through the inputs that users, operators, or the environment produce: data, files, configuration, and calls. If the scenario depends on another component's result, run that component on such inputs. Do not stub its result. Use test doubles only for time, concurrency, unavailable external systems, failures that real inputs cannot cause, and components that the scenario does not depend on.

In the report, list the test doubles of the reviewed tests. For each one, name what it replaces and the real situation it stands for. Say whether real inputs could create that situation. Go through every test that uses the double, not only the first one you notice. For each test whose scenario comes from a stubbed result that real inputs could produce, name the level where real components create it.

Before you call a test double or a part of a test unnecessary, break the production code that it guards and run the test. If you cannot run the test, ask the author instead of stating that it checks nothing.

Compare tests that promise the same outcome, including tests in other classes or at other layers. Name a reachable regression that one catches while the others pass. If everything it checks is already covered, remove the duplicate.

Test each requirement at the lowest level where real inputs can create its scenario. Add infrastructure, external dependencies, or waits only when the required behavior cannot be checked without them.

Keep tests as simple, clear, and small as possible. Justify each nontrivial part, and check whether a simpler approach works. Do not hide the scenario's meaning behind helper functions, flags, and complex setup. Order tests so that moving to the next one needs as little context switching as possible.

Confirm that the test actually reached the condition or event being tested. For a failure test, check that the intended failure occurred.

Break the code that each test protects and run the test: the test must fail. Also go the other way: for each production change, name the test that fails without it. A change that no test notices, such as wiring into startup or shutdown, needs a test at the level where it takes effect. Before removing a test, show which remaining test catches each reachable regression. If only the fixture permits a failure, identify the production step that prevents it. Breaking existing lines shows only that they are tested. It cannot show handling that is missing, so look for missing handling separately, as described in section 4.

Wait for the required event instead of an arbitrary delay. Justify every wait that uses a timeout. Where possible, replace it with an action that does not need that wait. For each wait with a timeout, find the clock that the timeout reads, and check that this clock moves in the test environment. Otherwise the test hangs instead of failing when the event never happens.

A slow or paused machine must not change the test result. Assume a long pause between any two steps of the test and of the code it runs. Use a timeout only to limit the wait for an event that must happen, and make it long enough that only a missing event exceeds it. Do not expect that an event has not happened yet after some delay, that one thread runs before another without synchronization, or that the clock does not move between two calls. When a check compares times, leave a margin larger than such a pause, or compute the expected value from the same time values that the code uses.

Release resources even when an error occurs.

## 10. Documentation and explanations are plain

Aim: a reader who did not take part in the discussion understands the change and can act on it without guessing.

Use the [human-writing](https://github.com/anton-vinogradov/human-writing) skill for all work texts within the task. If the skill is not installed, read `skills/human-writing/SKILL.md` in its repository. If those instructions are unavailable, apply the rules in this section.

Start with the main point. Be specific: who does what, under which conditions, and with what result. Use verified facts. Do not invent system behavior or present assumptions as established causes. Keep technical details that help the reader understand the solution or carry out an action.

If code needs a comment, check its readability and the overall approach. First consider clear names and a simpler solution that needs no explanation. Keep a comment about a non-obvious reason or constraint only if the reason cannot be changed within the task and no solution without the explanation has been found. Express obvious actions through code and names.

Before editing documentation, compare the behavior being described before and after the change, and read the existing description. Justify each edit by a need in the current task: what would the reader misunderstand or be unable to do without it? Do not describe unchanged behavior just because you studied it, unless it is necessary context or the task requires it. Do not duplicate information. If the need for an edit is not established, leave the documentation unchanged. Keep documentation diffs minimal.

If the behavior of a code element other than a test is clear from its name, do not add a description. Use an empty documentation block only if the project's style requires one. When documenting a code element, describe its contract and capabilities, not where, how, by whom, or why it is used.

## 11. Findings rest on evidence

Aim: each finding can be checked by someone else and leads to the smallest sufficient fix.

Treat the user's initial diagnosis and your own first impression as hypotheses to check. This includes a forecast that a proposed design will be more complex: sketch it and compare both versions by concepts, state, branches, and where each element's data and behavior live.

For each finding, show the scenario that triggers it, its consequence, the evidence, and the smallest sufficient fix. Distinguish bugs from open questions and optional improvements. A review with no findings is a valid result.

## 12. The final solution is checked again

Aim: the final diff as a whole meets every aim above, not only each fix in isolation.

After making fixes, reread the entire affected flow and the final diff. When a finding or a fix changes a requirement, repeat the outline from section 1 for the affected part: mechanisms, checks, and tests kept for the old requirement may no longer be needed.

Compare each changed fragment of existing code, tests, and documentation with the original. If a later change removed the reason for it, restore the original and keep only the lines the task needs. Check that every remaining edit is necessary and that existing tools and simpler solutions have been considered.

For a substantial rework, walk through a concrete execution example. Explain what became simpler and what changes to behavior or compatibility this required.

Report your conclusions, completed checks, and remaining gaps. Explicitly identify and visually highlight each unresolved problem, for example with `⚠️ Unresolved problem`, or ask the user a specific question.

## 13. Update the installed skill when using it

Before a review, run `python3 "<skill-directory>/scripts/update.py"` once. Use the directory containing this `SKILL.md`, regardless of the project being reviewed. On Windows, use `py -3` if needed.

The user enables automatic updates once for each installation. Do not enable them on the user's behalf during a review. Installation and the command to enable updates are described in [README.md](README.md).

The script contacts the upstream source at most once a day and preserves user edits.

If the script reports `updated`, reread this `SKILL.md` and use the updated instructions in the current review. You do not need to run the script again for this invocation of the skill.

Otherwise, continue with the installed version. If tools, network access, or permissions are unavailable, do not retry or request broader permissions just to update the skill.

Permission to update the skill does not grant permission to change the project being reviewed.

## Thank you

Thank you for the careful work. A thorough review saves the author and the users from real problems, and it is appreciated.
