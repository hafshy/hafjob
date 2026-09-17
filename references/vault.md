# Vault

HafJob reads and writes only inside `<vault>/HAFJOB/`. Everything else in the vault is off-limits — never create, edit, or read notes outside that folder, except read-only access to a note the user explicitly wikilinks from `Profile.md`.

## Resolve the vault

In order, first hit wins:

1. `vault=/abs/path` argument on the command.
2. `vault:` key in `./.hafjob.md` (frontmatter) in the current working directory.
3. The current working directory.

A vault is recognized by a `.obsidian/` directory. If it's missing, say so once ("no `.obsidian/` found, treating the folder as plain markdown") and continue.

## Layout

```
HAFJOB/
├── Profile.md          # facts + resume path; body = free-form summary for narrative answers
├── Resume.md           # markdown resume, or Profile.md may point to an existing note
├── Attachments/        # resume.pdf and any other uploads (≤10 MB each)
├── Answers/            # one note per reusable Q&A
├── Applications/       # one note per application (written by HafJob)
└── Log.md              # append-only timeline (written by HafJob)
```

## First run

If `HAFJOB/` does not exist:

1. Create `HAFJOB/`, `HAFJOB/Attachments/`, `HAFJOB/Answers/`, `HAFJOB/Applications/`, and an empty `HAFJOB/Log.md` (`# HafJob Log\n`).
2. Write `HAFJOB/Profile.md` from the template below with empty values.
3. Ask the user, in one message, for the minimum: `name`, `email`, `phone`, and the path to a resume PDF. Copy the PDF into `Attachments/` only if the user agrees; otherwise store the absolute path they gave.
4. Fill those in and continue with the application. Do not block on optional fields.

## Profile.md

```markdown
---
name: ""
email: ""
phone: ""
location: ""            # "City, Country"
linkedin: ""
github: ""
website: ""
work_authorization: ""  # e.g. "Indonesian citizen, no sponsorship needed in ID"
needs_sponsorship: ""   # yes | no | depends
notice_period: ""
resume: "HAFJOB/Attachments/resume.pdf"   # vault-relative or absolute path
resume_note: ""         # optional wikilink to an existing note, e.g. "[[My Resume]]"
---

Free-form summary: who you are, what you're looking for, notable achievements.
HafJob uses this body plus Resume.md to ground narrative answers.
```

Read frontmatter with plain text tools (`sed -n '/^---$/,/^---$/p'`), no YAML library needed for flat `key: value` pairs. Quote-strip values.

## Resume

Use `HAFJOB/Resume.md` if present. If `resume_note` is set, read that note instead (read-only). If neither exists, narrative answers may only use the `Profile.md` body; say so when a draft would be thin.

## Answers/

One note per question. Filename = the question, sanitized for the filesystem.

```markdown
---
question: "Why do you want to work here?"
reuse: true            # true = may be used without asking; false = always re-confirm
---
Answer text. `{{company}}` and `{{role}}` are replaced at fill time.
```

Match a form label to an answer note by exact match first, then by obvious paraphrase (same intent, e.g. "Why are you interested in this role?" ≈ "Why do you want to work here?"). When unsure it's the same question, treat it as no match.

Only write a new answer note when the user approves the text **and** says to reuse it. Default `reuse: true` in that case; the user can flip it.
