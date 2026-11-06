# Human Review

A Codex skill for reviewing whether a code change is necessary, preserves the required behavior, and handles object lifetimes correctly.

Example request:

> Use $human-review to review this diff. Explain which additions are necessary, verify lifecycle assumptions, and identify simpler existing mechanisms. Keep the review read-only.

See [SKILL.md](SKILL.md) for the complete instructions and [agents/openai.yaml](agents/openai.yaml) for Codex interface metadata.
