---
name: auto-apply
description: Fill and submit a job application form in Chrome from the Obsidian vault's HAFJOB/ folder without a confirmation step, as long as every required field maps to Profile.md, a reuse:true Answers/ note, or an existing upload; otherwise asks the user first. Use when the user gives a job application URL and wants it submitted immediately if the data is sufficient. Requires the Claude in Chrome extension.
---

# HafJob — auto-apply

Usage: `/hafjob:auto-apply <job-url> [vault=/abs/path/to/vault]`

Identical to [manual-apply](../manual-apply/SKILL.md) — same references, same workflow steps 0–10 — except the gate.

## Gate rule for this skill

At workflow step 8, submit **without waiting** only when every row's source is one of: `profile:*`, `answer:*` (from a `reuse: true` note), `upload`, `declined`, `blank`.

If step 6 had to ask the user anything (an `unknown`, `draft`, `hard-stop`, or `reuse: false` answer), the run is no longer fully automatic: print the table and wait for `yes` exactly like manual-apply. Drafts never auto-submit on their first use; once the user approves one and saves it with `reuse: true`, the next application with that question is automatic.

Everything under manual-apply's **Non-negotiables** applies unchanged.
