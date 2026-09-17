# Logging

Two files, both under `HAFJOB/`. Write them at the points marked in workflow.md.

## Log.md — append-only timeline

One line per event, appended to the end. Never edit or delete earlier lines. Create the file with `# HafJob Log` as the first line if it does not exist.

```
- YYYY-MM-DD HH:MM  <event>  <mode>  <company> / <role>  <detail>
```

Events, in the order they can occur:

| event | detail |
|---|---|
| `started` | the URL |
| `blocked` | `login` / `captcha` / `attestation` — what the user must do |
| `asked` | `N questions` |
| `filled` | `N fields, M uploads` |
| `submitted` | `→ [[<application note>]]` |
| `filled-not-submitted` | reason (`no confirmation`, `user cancelled at gate`) |
| `failed` | one-line reason |
| `cancelled` | one-line reason |

Example:

```
- 2026-09-17 14:02  started    manual  Acme / Senior iOS Engineer  https://boards.greenhouse.io/acme/jobs/123
- 2026-09-17 14:05  asked      manual  Acme / Senior iOS Engineer  3 questions
- 2026-09-17 14:08  filled     manual  Acme / Senior iOS Engineer  14 fields, 1 upload
- 2026-09-17 14:09  submitted  manual  Acme / Senior iOS Engineer  → [[2026-09-17 Acme - Senior iOS Engineer]]
```

Append with a single shell command, e.g. `printf '%s\n' "- $line" >> "$VAULT/HAFJOB/Log.md"`.

## Applications/<date> <company> - <role>.md

One note per application, written at the end (step 10) but drafted mentally from step 5 onward. If the same filename exists, append ` (2)`.

```markdown
---
company: Acme
role: Senior iOS Engineer
url: https://boards.greenhouse.io/acme/jobs/123
ats: greenhouse          # greenhouse | lever | ashby | workday | linkedin | other
mode: manual             # manual | auto
status: applied          # applied | filled-not-submitted | needs-input | failed | cancelled
applied: 2026-09-17      # date of submission, omit if not submitted
---

## Fields submitted

| Field | Value | Source |
|---|---|---|
| First name | Hafshy | profile:name |
| Resume | resume.pdf | upload |
| Why do you want to work here? | (first 80 chars…) | answer:Why do you want to work here |
| Years of Swift experience | 6 | user |

## Questions asked to user

| Question | Answer | Saved to Answers/ |
|---|---|---|
| Years of Swift experience | 6 | no |

## Notes

- Hard stops hit, ATS quirks worth remembering, confirmation screenshot path.
```

Source values: `profile:<key>`, `resume`, `answer:<note name>`, `draft` (approved narrative), `user` (typed this session), `upload`, `declined` (EEO prefer-not-to-say), `blank`.

Full free-text answers go in the note only if the user said to save them; otherwise truncate to ~80 chars in the table so the note stays readable.

## Screenshot

Save the confirmation screenshot as `HAFJOB/Applications/<same basename>.png` and link it from Notes.
