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

Check whether each part of the change is necessary. Determine what changes if it is removed or simplified.

For every change, ask yourself: “Why is this needed? Why can’t it be simpler and clearer?”

Keep the diff minimal without losing required functionality. Do not add preparation for hypothetical future tasks or unrelated formatting changes.

## 2. Trace the full behavior

Review the entire affected flow: calling code, conditions, bypass paths, and code that consumes the result.

Find out who owns the state, when it is loaded, and which operations will see the updated value.

Account for when errors occur, retries, resource cleanup, and how the cause of an error reaches the user.

## 3. Fix the problem in the responsible layer

First, learn what the component receiving the data or call can already do.

If its capabilities are sufficient, fix the calling code. Do not give the component extra responsibilities just to make calls look alike.

Keep adapters that perform a necessary conversion or uphold a contract between components.

## 4. Keep the implementation simple and explicit

Judge simplicity by how many concepts, state owners, and relationships the reader must understand. A small diff alone does not mean a simple solution.

For each explicitly set parameter, determine what changes if it is removed. Check the API implementation for defaults, required arguments, and constraints. Account for all sources of effective values. In tests, these may include the base class, shared setup, and helper methods.

Keep the parameter if removing it breaks the code, changes required behavior, or prevents the test from reliably checking its scenario. If a suitable value is already set or the setting does not affect the scenario, it is redundant. Similar settings in nearby code do not prove that the parameter is necessary.

Check whether each data transformation, including type casts, is necessary and look for a simpler solution.

The result of each operation must be clear. If code relies on a side effect, consider an explicit action that directly expresses the required change.

## 5. Choose values, names, and formatting deliberately

Justify the choice of every fixed value. Each value must have a clear meaning.

Extract values used repeatedly with the same meaning into variables or constants.

Choose names for code elements that clearly show their purpose. Use the project’s terminology and naming conventions.

Follow the project’s formatting rules, including line lengths and line breaks.

## 6. Check requirements with simple, clear tests

For each test, name the single guarantee it checks. Its name, inputs, actions, and result must explain the idea without relying on the discussion history.

A shared failure or shared setup does not make independent requirements one scenario. Keep several stages together when they establish that guarantee, such as recovery after a failure.

Choose the smallest set of scenarios that covers the changed behavior and possible regressions.

Model only scenarios that can occur in real use. Check that the combination of conditions created by test setup and test doubles can occur in production. Do not artificially create states that cannot occur in practice.

Before adding a test, trace the call path and preceding checks in production code. Account for differences in conditions, execution stages, and implementations when they change reachability or the expected result. Do not carry conclusions between such cases without checking them. For example, failure to start does not establish behavior after a successful startup. Check existing tests and add only missing coverage.

Test each requirement at the lowest level that can adequately verify it. Add infrastructure, external dependencies, or waits only when the required behavior cannot be checked without them.

Keep tests as simple, clear, and small as possible. Keep only setup, operations, and checks needed for the guarantee being tested.

Justify each nontrivial part of the test. Check whether a simpler approach will work.

Use the test framework’s features. Read the project’s tests and look for suitable solutions there.

Do not hide the scenario’s meaning behind test implementation details, including helper functions, flags, and complex setup.

Order tests to make them easy to read. Moving to the next test should require as little context switching as possible.

Confirm that the test actually reached the condition or event being tested. For a failure test, check that the intended failure occurred.

Wait for the required event instead of an arbitrary delay. Justify every wait that uses a timeout. Where possible, replace it with an action that does not need that wait.

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

Keep documentation diffs minimal.

If the behavior is clear from the name, do not add a description. Use an empty documentation block only if the project’s style requires one.

When documenting a code element, describe its contract and capabilities. Do not list where, how, by whom, or why it is used.

## 8. Support findings with evidence

Treat the user’s initial diagnosis and your own first impression as hypotheses to check.

For each finding, show the scenario that triggers it, its consequence, the evidence, and the smallest sufficient fix.

Distinguish bugs from open questions and optional improvements.

A review with no findings is a valid result.

## 9. Check and explain the final solution

After making fixes, reread the entire affected flow and the final diff.

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
