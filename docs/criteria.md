# Criteria for a good change

The model uses the same criteria for its own changes and for changes made by others. It fixes confirmed defects within the task. After making fixes, it reviews the whole diff, the difference between the starting and current versions. It traces the affected behavior and checks whether the change delivers the requested result. If this review finds another defect the model can fix, it fixes it and checks the whole result again.

During this review the model checks whether the task, code, and tests explain when behavior changes and what produces the intended result. If understanding it requires remembering the conversation, the model finds the missing connection. It makes the relevant condition explicit in the code or test. If the reason for a constraint needs explaining, the model gives a short explanation where the constraint applies.

While a defect awaits a decision from the user, the model works on the others. At the end it reports what it checked. For each remaining defect it explains the consequences and what prevented a fix. When it could not run a check, it says what remains unverified and why.

The model uses human-writing for everything it writes during the task, including messages to the user and text that becomes part of the project.

## 1. Task scope and simplicity

### The task and the review plan

The model starts every review by examining the task, even when a solution already exists. It establishes the intended outcome and how to verify that the whole task is complete. For a task that belongs to a larger effort, the model checks the goal of the root task. In related tasks it checks requirements and outcomes the work depends on. It compares the task with the current project to find out whether the assumptions hold, the requirements fit together, and the plan is feasible.

When information is missing, the model first looks in the project and related tasks. If a question remains open, it asks the user for the specific information and explains which conclusion or choice depends on it.

Once the goal and scope are clear, the model plans the checks it would normally make for this change. It adds the checks from human-review and carries out the whole plan. When a criterion adds detail to an existing check, the model covers it in that check.

If the final review would require evaluating several decisions at once, the model considers which can be assessed separately. When this makes review easier, it suggests splitting the task. Its plan states the result of each part, the order required by dependencies, and how to verify that the parts together deliver the original outcome.

### Choosing a solution and setting the scope of the change

Before examining individual edits, the model works out the most direct way to meet the goal within its constraints. It looks for the required behavior in the project and the libraries and frameworks already in use. It uses a suitable mechanism when one exists. If a project component already does part of the job, the model extends it when the change fits its purpose and preserves the guarantees for other callers.

The model compares this approach with the one the task proposes or the code implements. If the existing solution is more complex, it identifies the requirement or guarantee that the additional mechanism satisfies. It retains the mechanism when the task would otherwise be only partly fulfilled. It uses the direct approach when it delivers the result and preserves all guarantees.

When a proposed approach fails, the model looks for another way to achieve the same goal within scope and continues once it finds one. When it needs permission or a decision about the goal, scope, or expected behavior, it explains what is blocking progress and asks a specific question. If there is a choice, it describes the options, their consequences, and its recommendation. It raises incidental improvements separately and includes them in the work only if the user expands the scope.

The model keeps the edits the current task needs. When the code already delivers the requested result, it explains which aspect of the change makes the code easier to understand, verify, or modify. Given solutions with the same behavior, it chooses the one that requires keeping fewer conditions and relationships in mind.

Frequently executed code can justify an edit for another reason. The model identifies repeated computations and temporary objects created during the operation. It eliminates unnecessary work when the required behavior and readability are preserved. When the alternatives are equivalent, it keeps the existing code.

The diff should show when behavior changes and what the result will be. The model preserves the existing structure where possible. A small but significant change is easier to see when familiar code keeps its place and shape. The model changes the structure when the task or a defect requires it, aiming for a smaller diff.

### Project conventions and dependencies

The model follows project conventions. It chooses names that show an element’s purpose. Where there is no naming rule, it follows established usage. It connects a new component the same way the project connects others serving the same purpose. It uses a different approach when the existing one cannot deliver the required behavior.

The model also considers libraries and frameworks it knows that are not yet part of the project. It checks what custom code the library would replace and whether that would make the solution clearer. It adds a suitable test-only dependency itself. Before introducing one into production code, it explains what it would simplify and asks the user for approval. This also applies to a library already used only in tests.

When adding a dependency, the model checks where the project already uses it and how it specifies the version. It uses the project’s compatible version and its shared version source when one exists. If a newer version resolves a problem with the task or its implementation, the model proposes an upgrade. It identifies the specific fix and the parts of the project affected. If the version changes, it verifies compatibility there.

### Explicit parameters

For each explicit parameter the model asks why that value was chosen and what would change if it were omitted. It checks whether omission is allowed and what value the code would receive. In tests it accounts for shared setup.

The model keeps an explicit value when the call requires it, it provides the necessary behavior, or it makes clear what the result depends on. This matters when a change to a default could alter program behavior or make a test unreliable. It removes a setting that merely repeats the effective value when omission preserves behavior, test reliability, and clarity.

## 2. Data paths from source to result

The model aims to trace every path the affected data can take. It finds where values originate, where they go, when they change, and which code uses the result. It includes external calls and operations invoked by the framework. When it cannot verify part of a path, it identifies the conclusion that remains unverified.

When a value is repeated and must mean the same thing everywhere it is used, the model defines it once and reuses it. It keeps values separate when they can change independently, even if they happen to be equal today.

In a test the model defines a shared value where its role in the scenario is visible. To test that an operation preserved the supplied identifier, it defines the identifier at the start of the scenario, passes it to the operation, and compares it with the identifier in the result.

For data that changes over time, the model considers the purpose of each record. If a copy of a current value is needed, it checks how that copy is kept up to date. A record of the value at the time of an event preserves that value when the source changes.

The model uses these paths to determine which values and actions are possible. It removes a parameter when every supported call passes the same value and removal preserves behavior and compatibility with external code. For a new operation that cannot be reached in normal use, it checks whether the task requires it. It calls a required operation where the action belongs and removes an unnecessary one.

For a new check the model identifies a real situation where it would change the program’s behavior. Before removing a check it traces the case the check protects against. If a test describes that case, the model checks how it can arise in production.

The model removes the check once it can show that the condition already holds on every supported path or that the program will still deliver the required result. When a path remains uncertain, it keeps the check and explains what it could not establish.

## 3. The operation serves the purpose of the call

The model chooses an operation whose name expresses the purpose of the call and whose contract guarantees the result. The contract specifies the conditions for the call and what the operation promises under those conditions.

The model compares behavior with the contract. If the operation already supports the required action but the caller passes the wrong data or misuses the result, it fixes the caller. If the operation violates its own contract for valid inputs, it fixes the implementation so all callers get the promised result.

If the desired result occurs only as a side effect or the call also does unnecessary work, the model finds an operation for the required action and uses it.

It introduces a separate operation when the new action has a different meaning or when combining actions would require reasoning through extra conditions to understand each call.

An adapter connects components with different calling conventions or data representations. Before removing it, the model checks what a direct call would require the caller to know or do. It keeps an adapter that performs a needed conversion or shields the caller from another component’s details. It removes the extra step when a direct call preserves the required behavior and is easier to read.

## 4. Shared code represents a shared rule

When two fragments look alike, the model identifies the requirement each one serves. If a change to one requirement should change both fragments in the same way, the model defines the common rule in one place. If they share only part of their behavior, it shares only that part. It keeps actions separate when they can change independently.

The model extracts repeated actions into an operation when this reduces code and each call makes its result clear. If understanding that result still requires tracing branches inside the operation, the model reconsiders which actions belong together.

## 5. Reading order

The model places code that needs to be read together close together when execution order permits. It declares a variable as close to its first use as possible, within the part of the code that needs the value. Placement accounts for when, under what conditions, and how often the value must be computed.

In a test file, the model orders scenarios so the next test rarely forces a change of subject or a fresh analysis of the setup. A scenario describes the starting conditions, action, and expected outcome. Placing related scenarios together makes their differences and the purpose of each test easier to see.

## 6. One test verifies one guarantee

### Guarantee and coverage

A guarantee is the behavior the program promises under given conditions. The model keeps steps together when they jointly verify one guarantee. It tests independent guarantees separately, even when they share setup.

Before adding a test the model identifies a defect in the changed behavior that existing tests would miss. If a test already creates the right scenario and only needs an assertion for the same guarantee, the model extends it. It adds a separate test for a distinct scenario or guarantee when that coverage is missing.

It keeps two similar tests when each catches a real defect the other would miss. When differences in data or setup add nothing to the check, one test is enough.

If simplifying a test changes the conditions for a failure or how its consequences are checked, the model verifies that the test still detects the defect. For a known bug it reproduces the bug, checks that the simplified test fails for the expected reason, and checks that it passes with the fix. When it cannot confirm this, it keeps the part of the test that detection may depend on.

### Scenario and starting conditions

The test name tells which behavior is being checked. When the relationship between conditions, actions, and their order needs explanation, the model describes the scenario using Given/When/Then. Given gives the starting conditions, When the action, and Then the expected result.

The model chooses the test level that best shows why the initial conditions lead to the expected result. This can encompass one operation or several components together. Conditions that determine the result remain visible in the test. When a helper handles setup, its name and arguments show what conditions it creates.

If only the initial state matters, the model starts there after verifying it is reachable in normal operation. When the guarantee covers the transition into that state, the test goes through the transition. In either case, it checks that the test executes the code whose failure it should detect.

For a failure scenario the model verifies that execution reaches the operation where the failure is supposed to occur. When the guarantee depends on the cause or timing, it uses an assertion that distinguishes this failure from another one with the same outward result.

When testing a reaction to another component’s response, the model may substitute a stub. The stub gives a response the real component could produce under the test conditions. When the test also covers how the component produces that result, the model supplies the relevant inputs and runs the component’s code.

### Results and waits

The test checks a result observable through normal use or by calling the code. The model checks an intermediate action only when the program promises the action itself.

The model checks an immediately available result without waiting. For a result that arrives later, the test waits for an event or state showing the required action happened. A timeout bounds the wait so the test fails if the event never arrives.

When the guarantee specifies a deadline, the test verifies that deadline.

## 7. A comment calls for a closer look at the code

When a comment explains the code, the model asks what would be hard to infer without it. The cause may lie in new or existing code. If it can express the same logic more clearly within the task, it fixes the code and removes the comment that has become redundant.

If addressing the cause would expand the task, the model explains what the refactoring would improve and how it relates to the current change. It proposes doing the refactoring first when the solution depends on it. If the task can be completed separately, it asks whether to include the refactoring now or keep it for a separate task.

Even clear code can leave the reason for a constraint obscure. The model keeps a comment that explains the reason and what would happen if the constraint were violated.

## 8. Documentation serves a task

Before editing documentation, the model reads the existing text and determines what has changed for the person using it. The instructions should make clear what to do, in what order, and what result to expect.

The model edits the text when the task calls for it, or when the current text would make someone miss a step, guess a condition, or expect a different result. It fixes the passage that causes the difficulty. It adds details about unchanged behavior only when they are needed to carry out the task. If the existing text already suffices, it keeps it.

In API documentation the model describes what a caller needs to know. When the name, parameters, and return type already convey the purpose, conditions, and guarantees, it follows project style. It omits an optional description or leaves an empty documentation block when that is the convention.

If part of the contract is not visible in the declaration, the model explains it. It omits the list of current call sites and their purposes. These can change while the contract stays the same.

## 9. The log shows what the system does

The model checks that the important systems affected by the task log their events. A system is important here when a failure can disrupt use of the program, break another system, or require intervention. The model adds missing entries for operations and state transitions involved in the task. An operation’s entry identifies the action and outcome. A transition’s entry identifies the previous and new state.

When entries for several operations can interleave, the model checks that each entry can be connected to its operation. It uses information already available in the log and adds only the data needed to connect an action to its outcome.

A failure message should show which operation failed and why. If recovery requires an action, the model states it in the message. If the system will retry on its own, it states that as well.

When adding an entry, the model reviews the component’s log as a whole. It checks that important operations and state changes can be traced and that similar events are described consistently. If it finds gaps or inconsistencies outside the task, it shows which events are hard to follow or compare. It proposes filling the gaps and making the entries consistent in the current task or a separate ticket.

Thank you in advance for the careful review and fixes!
