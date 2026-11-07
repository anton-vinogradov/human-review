# Human Review

[English](README.md) | [Русский](README.ru.md)

Code review instructions for checking whether each change is necessary, preserves required behavior, and uses a simpler solution where possible.

Installation depends on the application running the model. The commands below are for Codex. Other agents may use different skill directories and invocation methods.

## Quick start

If your agent supports skills, file access, and command execution, paste this request into it:

```text
Install human-review from https://github.com/anton-vinogradov/human-review as a skill for the current agent.
Use the installation method and skill directory supported by this agent.
If installation as a standalone Git clone is supported, use it. Leave automatic updates disabled.
Check that the skill is available. If skill installation is unsupported, say so.
```

After installation in Codex, you can send this request:

> Use $human-review to review this diff. Explain which additions are necessary, verify lifecycle assumptions, and identify simpler existing mechanisms. Keep the review read-only.

For a chat without skill support, attach [SKILL.md](SKILL.md), the diff, and the source files needed to understand the changes. Then send:

```text
Review the attached diff using the attached human-review instructions.
Skip the automatic-update section. If human-writing is unavailable, use the writing rules in section 7.
```

This applies the instructions in the current chat. It does not install the skill or enable automatic updates.

## Requirements

- Reviewing with the attached instructions requires neither Git nor Python.
- Installation with `git clone` requires Git.
- Automatic updates require Git, Python 3.9 or later, network access, and permission to modify the installed skill.
- Documentation and explanations use [human-writing](https://github.com/anton-vinogradov/human-writing). If it is not installed, the agent reads its instructions from the repository. If those instructions are unavailable, section 7 of human-review includes the core writing rules.

The complete skill is available in [English](SKILL.md) and [Russian](SKILL.ru.md). `SKILL.md` is the installation entry point; `SKILL.ru.md` is its Russian translation. Codex interface metadata is in [agents/openai.yaml](agents/openai.yaml).

## Manual installation in Codex

To support automatic updates, install a standalone Git clone. In macOS, Linux, Windows Git Bash, or PowerShell:

```sh
git clone --branch main https://github.com/anton-vinogradov/human-review.git "$HOME/.agents/skills/human-review"
```

If the destination already exists, Git stops without overwriting it. See the migration instructions below. A copied skill still works for review, but automatic updates require the Git installation.

## Enable automatic updates

Automatic updates are off until you explicitly enable them for this installation:

```sh
python3 "$HOME/.agents/skills/human-review/scripts/update.py" --enable
```

On Windows, replace `python3` with `py -3` if needed. If you installed the skill elsewhere, use that path in these commands.

Enabling updates authorizes downloading and applying future commits from this repository's **main branch**, including changes to the updater itself. A commit hash identifies the installed version. The helper only changes the skill installation; it does not grant permission to edit projects being reviewed.

When the agent uses the skill, it runs the helper before reviewing. The helper checks at most once per 24 hours, including failed network attempts, and applies only fast-forward updates. After an update, the agent rereads `SKILL.md` for the current review.

- Local edits, staged changes, extra files (including ignored files), and divergent local commits are preserved; the update is skipped.
- A different origin, a branch other than `main`, or a detached checkout also skips updates.
- Missing tools, unavailable network, timeouts, or insufficient permissions do not block review. The installed instructions remain available. A Git operation has a 15-second timeout.
- Checks happen when the skill is used. There is no background service; execution still depends on the agent and the environment's permissions.

To check immediately, bypassing only the daily interval:

```sh
python3 "$HOME/.agents/skills/human-review/scripts/update.py" --force
```

To turn automatic updates off:

```sh
python3 "$HOME/.agents/skills/human-review/scripts/update.py" --disable
```

The preference, last attempted check, and update lock are stored in the installation's `.git` directory. `--force` does not bypass the preference or the protection for local changes. The helper prints a JSON result (`enabled`, `disabled`, `current`, `updated`, or `skipped`); check that result to confirm a preference was saved. Update failures are best-effort and do not signal that a review must stop.

## Migrate an existing installation

Existing copies cannot acquire this updater merely because the GitHub repository changes.

- **Clean Git clone without local commits:** update the `main` checkout with `git pull --ff-only` from this repository. Run the `--enable` command above once.
- **Copied files, a skill-installer download, or a Git clone with local edits or commits:** move the old skill folder to a backup outside all skill discovery directories. Clone into the original location, then run `--enable`. Keep the backup until you have checked for personal changes. Reapplying those changes to the installation will suspend automatic updates.

Keep one discoverable `human-review` installation to avoid selecting an older duplicate. No reset, automatic merge of local changes, or deletion of an existing installation is needed.

## Development

Run the updater tests without network access:

```sh
python3 -B -m unittest discover -s tests -v
```
