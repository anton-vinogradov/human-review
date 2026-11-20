# How to author Human Review

These rules describe how the model turns user feedback into criteria for reviewing code. It draws on the discussion and on changes the user has already accepted. It proposes the wording to the user and adds the criterion to the document if the user accepts it. The accepted criteria guide the skill instructions. The model uses human-writing for every text, including the criteria, the skill and messages to the user.

## Which criteria belong here

A criterion explains why the user prefers one of two valid solutions. If another criterion already explains the choice, or an ordinary review consistently produces the same result, a separate rule adds nothing.

## How the model identifies the goal

The model starts with user feedback or an existing human-review rule. It works out what outcome the feedback calls for and why it matters. If an accepted change has several possible explanations and the choice would change the criterion, the model explains the difference and asks the user.

## How the model expresses a criterion

The model describes the property that matters, how to recognize it and why it affects the choice. It identifies the conditions under which the decision would change. The wording should also apply to a different implementation that achieves the same goal.

## Where a criterion belongs

The model looks for the closest existing criterion. If it already explains the same choice, it is sufficient. If a new case clarifies when that criterion applies, the model proposes an amendment. It places related ideas together and combines them when the wording still explains why two similar changes deserve different treatment.

## How the document reads

The user and the model are the two participants. Each criterion states what the model checks and what it does with the result. When the criterion settles the choice, the text names the action. When the choice depends on the user, the model explains why and asks a question. The user decides whether to answer or accept a proposal. For example, the model independently fixes a confirmed defect within the task. It includes an incidental improvement only if the user chooses to expand the task.

A heading names the property the model uses to assess a change. Each criterion reads as connected prose in the present tense. Each paragraph answers one question and starts with its main point. The model describes what it checks and how the result changes its action. When a choice needs an explanation, it states what depends on that choice. It describes properties of code, tests, documentation and logs directly.

The model explains terms when they first appear. Sometimes the action and its result make the meaning clear. When several unfamiliar terms must be used together, the model briefly defines them first. Ordinary words also need to be precise. If something is called “agreed,” the text should make clear what was agreed and how the model knows.

After revising the text, the model compares it with the accepted version to check the requirements, their reasons and their exceptions. It then rereads the whole document with human-writing as someone encountering the skill for the first time. If the decision only makes sense when reading an example, the model puts the decisive condition into the criterion itself.

Each language version of the skill ends by thanking the model for the work ahead. The last line of the Russian version is “Заранее спасибо за внимательную проверку и исправления!” The English version ends with “Thank you in advance for the careful review and fixes!”

## How to package the skill for installation

The model publishes the English `SKILL.md` as the entry point and the Russian `SKILL.ru.md` with equivalent instructions. The skill name in the metadata stays `human-review` so existing installations and `$human-review` invocations continue to work. Both READMEs show how to install the skill in the user's agent, verify that it is available and start a review. Commands specific to Codex are labeled accordingly.

Automatic updates require a standalone Git clone of the official repository on `main`. The instructions state that updates require Git, Python 3.9 or later, network access and permission to write to the installation directory. For a chat without an installation, attaching the skill file, the change and the relevant source is sufficient. If the user already has a copy with local edits, the instructions explain how to keep it outside any skill directories before migrating to a Git clone. [Installation and migration](https://github.com/anton-vinogradov/human-review/blob/main/README.md)

## How the installed version gets updates

Automatic updates start disabled. The user can opt in once for that installation with `--enable` and opt out later with `--disable`. The README explains that consent covers future changes published to `main`, including changes to the updater itself. This consent applies only to the skill installation. The user's review request determines whether the model may change the project under review.

Both versions of the skill retain the instruction to run the installed `scripts/update.py` before reviewing. The script checks for consent and contacts the source at most once every 24 hours, including failed attempts. For a separate manual check, `--force` bypasses only the time limit. After an `updated` result, the model rereads the instructions and applies them to the current review.

Updates apply only to a clean clone of the expected repository on `main`, and only when the new version is a fast-forward from the installed one. Local edits, extra files or divergent history leave the installation intact. If the check is skipped or tools, permissions or network access are unavailable, the model uses the installed instructions to review the change. [Updater implementation](https://github.com/anton-vinogradov/human-review/blob/main/scripts/update.py)

## What to check before publishing

The model compares both language versions with the accepted criteria, including exceptions, recovery and decisions that stay with the user. It checks that each skill file ends with the exact thank-you line for its language. It also checks the skill metadata and the links, paths and commands in both READMEs. The instructions should explain how to install the skill for the first time and how an existing installation will receive this revision once it is published to `main`.

From the repository root the model runs `python3 -B -m unittest discover -s tests -v`. The tests use disposable Git repositories without network access. They check opting in and out, updating the skill files, preserving local edits, handling network failures and protecting the project under review. When the installation or update procedure changes, the model also follows the changed steps in a separate test installation. [Updater tests](https://github.com/anton-vinogradov/human-review/blob/main/tests/test_update.py)
