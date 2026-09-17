# HafJob

Claude Code plugin that fills job application forms in your own Chrome from an Obsidian vault.

> **v0.1 — untested against live ATS forms.** Expect quirks on the first Greenhouse/Lever/Workday run. Please open an issue with the ATS and the field that broke.

Two skills, same workflow, different gate:

| Skill | What it does |
|---|---|
| `/hafjob:manual-apply <url>` | Fills the form, shows every value with its source, submits only after you say `yes`. |
| `/hafjob:auto-apply <url>` | Same, but submits without asking **if** every field came from your profile, a reusable saved answer, or an upload. Anything it had to ask you about turns the run into manual mode. |

Both stop and ask when data is missing, ambiguous, or sensitive. Neither invents facts.

## Requirements

- [Claude Code](https://code.claude.com/docs) on a direct Anthropic plan (Pro/Max/Team/Enterprise)
- Google Chrome (or Edge/Brave/Arc) with the [Claude in Chrome extension](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn) v1.0.36+
- An Obsidian vault (or any folder — Obsidian doesn't need to be running)

## Install

```bash
git clone https://github.com/hafshy/hafjob.git
claude --chrome --plugin-dir ./hafjob
```

Run `/chrome` inside Claude Code once to confirm the extension shows **Enabled / Installed**.

## Vault setup

HafJob only touches `<vault>/HAFJOB/`. Your existing notes are never read or written, except a note you explicitly wikilink from `Profile.md`.

On first run it creates the folder and asks for name, email, phone, and a resume PDF. Or create it yourself:

```
<vault>/HAFJOB/
├── Profile.md        # frontmatter: name, email, phone, location, linkedin, github,
│                     #   work_authorization, resume: "HAFJOB/Attachments/resume.pdf" …
│                     # body: free-form summary used to ground narrative answers
├── Resume.md         # markdown resume (or point Profile.md `resume_note` at an existing note)
├── Attachments/      # resume.pdf (≤10 MB)
├── Answers/          # reusable Q&A notes, one per question, `reuse: true|false`
├── Applications/     # one note per application — written by HafJob
└── Log.md            # append-only timeline — written by HafJob
```

Full template and matching rules: [references/vault.md](references/vault.md).

## Usage

```
/hafjob:manual-apply https://boards.greenhouse.io/acme/jobs/123
/hafjob:auto-apply https://jobs.lever.co/acme/abc vault=/path/to/other/vault
```

Vault resolution: `vault=` argument → `vault:` in `./.hafjob.md` → current directory.

What happens:

1. Opens the URL in a Chrome tab (uses your existing login).
2. Reads the form, maps each field to a source: profile key, saved answer, upload, draft, or unknown.
3. Asks you everything it can't source, in one batched message. Drafted free-text answers are always shown before use.
4. Fills the form, uploads the resume, re-reads to verify values stuck.
5. Gate: manual waits for `yes`; auto proceeds only if nothing was asked in step 3.
6. Submits, screenshots the confirmation, writes `Applications/<date> <company> - <role>.md` and a `Log.md` line.

## Hard stops (always asks, both modes)

Login/CAPTCHA · legal attestations · salary questions · government IDs · EEO/demographic questions (selects "Prefer not to say" when offered) · any question with no source in `HAFJOB/`.

See [references/safety.md](references/safety.md).

## Layout

```
.claude-plugin/plugin.json
skills/manual-apply/SKILL.md
skills/auto-apply/SKILL.md
references/            safety · vault · workflow · answers · uploads · log · ats/
PLAN.md                design decisions + research
```

## Not in scope

Job search, fit scoring, tailored resume generation, batch applying. See [PLAN.md](PLAN.md) for why.

## License

MIT
