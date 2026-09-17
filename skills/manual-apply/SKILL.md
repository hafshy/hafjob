---
name: manual-apply
description: Fill a job application form in Chrome from the Obsidian vault's HAFJOB/ folder, show every value and its source, and submit only after the user says yes. Use when the user gives a job application URL and wants to review before applying. Requires the Claude in Chrome extension.
---

# HafJob — manual-apply

Usage: `/hafjob:manual-apply <job-url> [vault=/abs/path/to/vault]`

The URL is required. Without it, ask for it and stop.

Read these before starting, in order:

1. [references/safety.md](../../references/safety.md) — hard stops, grounding, untrusted page, filled ≠ applied.
2. [references/vault.md](../../references/vault.md) — where data lives, first-run setup, `Answers/` matching.
3. [references/workflow.md](../../references/workflow.md) — steps 0–10. Run them in order.

Read when the step needs them: [references/answers.md](../../references/answers.md) (drafting free text), [references/uploads.md](../../references/uploads.md) (resume upload), [references/log.md](../../references/log.md) (Log.md line + application note), `references/ats/<ats>.md` if one exists for the detected ATS.

## Gate rule for this skill

At workflow step 8, **always** print the field table and wait for the user's explicit `yes`. `edit <#> <value>` changes a row and re-asks; `cancel` stops and records `status: cancelled`. Nothing else proceeds to Submit. This applies even if every field came from `Profile.md` or a `reuse: true` answer.

## Non-negotiables

- One application per invocation; never batch, never browse to other postings.
- Ask, don't guess: any `unknown`, `draft`, or `hard-stop` field goes to the user in one batched question round before filling.
- Never fill EEO/demographic, government ID, salary, or legal-attestation fields without the user's explicit answer this session (EEO: select the decline option when it exists).
- Write only inside `<vault>/HAFJOB/`. Read outside it only for a note wikilinked from `Profile.md`.
- `Log.md` gets `started` at the beginning and a terminal event at the end, every run, including failures.
