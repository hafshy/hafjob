# Workflow

Shared by `manual-apply` and `auto-apply`. Only step 8 differs.

Tools: `mcp__claude-in-chrome__*` — `tabs_context_mcp`, `tabs_create_mcp`, `navigate`, `read_page`, `find`, `form_input`, `file_upload`, `computer` (click / screenshot). If they are deferred, load them in one `ToolSearch` call first. If the extension is not connected, tell the user to run `/chrome` and stop.

## 0. Validate input

- No URL in the arguments → ask for the job application URL and stop until given. It is the one required input.
- Parse `vault=` if present. Everything else in the arguments is ignored.

## 1. Load the vault

Follow vault.md: resolve the vault, ensure `HAFJOB/` exists (first-run flow if not), read `Profile.md` frontmatter + body, `Resume.md` or `resume_note`, and index every `Answers/*.md` by its `question`.

## 2. Log `started`

Append to `Log.md` (log.md). Company/role may be unknown yet — use `?` and fix them in later lines.

## 3. Open the posting

`tabs_create_mcp` → `navigate` to the URL. Then `get_page_text` or `read_page` to learn company, role, and whether this is already the form.

- Posting page with an "Apply" button → click it. If it opens a new tab, switch to it.
- Login / SSO / CAPTCHA → log `blocked`, ask the user to handle it in Chrome, wait for "done", re-read the page.
- Posting closed / 404 → log `failed`, tell the user, stop.

Detect the ATS from the URL host or DOM markers:

| host contains | ats |
|---|---|
| `greenhouse.io` | greenhouse |
| `lever.co` | lever |
| `ashbyhq.com` | ashby |
| `myworkdayjobs.com` / `workday` | workday |
| `linkedin.com` | linkedin |
| otherwise | other |

If `references/ats/<ats>.md` exists, read it before step 4.

## 4. Read the form

`read_page` with `filter: "interactive"`. Build a list of fields: `{ref, label, type, required, options}`. Use `find` for labels that are visually present but not in the tree. Ignore search boxes, cookie banners, and nav.

Multi-page forms: treat each page as its own steps 4–7 and move on with the page's "Next"/"Continue" button. Do the gate (step 8) once, right before the final Submit.

## 5. Map each field to a source

For every field pick exactly one:

| source | when |
|---|---|
| `profile:<key>` | label clearly matches a `Profile.md` key (first/last name split from `name`; phone; email; location; linkedin; github; website; work authorization; sponsorship; notice period) |
| `upload` | file input for resume / CV |
| `answer:<note>` | an `Answers/` note matches the label (vault.md matching rule) |
| `declined` | EEO / demographic field with a decline option (safety.md) |
| `draft` | free-text question answerable from `Profile.md` body + resume (answers.md) |
| `blank` | optional field with nothing to say (e.g. optional cover letter, "how did you hear about us") |
| `hard-stop` | anything listed in safety.md |
| `unknown` | required field with no source above (e.g. "Years of Swift experience", unfamiliar dropdown) |

Don't stretch a match. A wrong confident fill is worse than a question.

## 6. Ask the user — one batched round

Collect everything that needs input and ask in a single message, numbered:

- every `unknown`
- every `hard-stop` (explain which rule triggered)
- every `draft` — show the full draft text and ask approve / edit / skip. Skip this for a `draft` only when an `Answers/` note with `reuse: true` covers it (then it's `answer:`, not `draft`).
- for `answer:` notes with `reuse: false` — show the text, ask confirm / edit.

For dropdowns, list the available options. After the user answers, log `asked N questions` and ask once whether any typed free-text answers should be saved to `Answers/` (vault.md format).

If the user answers nothing (cancels) → log `cancelled`, write the application note with `status: cancelled`, stop.

## 7. Fill

- `form_input` per field with its `ref`. Selects: pass the option text. Checkboxes: boolean. Radio groups: click the matching option with `computer`.
- Uploads: uploads.md.
- Replace `{{company}}` / `{{role}}` in answer text.
- After filling the page, `read_page` again and compare each value. React/Workday forms sometimes drop values — refill anything that didn't stick, once. Still wrong → tell the user which field and continue.
- Log `filled N fields, M uploads`.

## 8. Gate

Print the table:

```
| # | Field | Value | Source |
```

Free-text values truncated to ~80 chars in the table; the user already saw the full text in step 6.

- **manual-apply**: wait for `yes` / `edit <#> <new value>` / `cancel`. Apply edits with `form_input`, reprint the changed rows, ask again. Only `yes` proceeds.
- **auto-apply**: if the table contains no row whose source is `unknown`, `draft`, or `hard-stop` (all resolved in step 6 into `user`/`answer:`), proceed without waiting. Otherwise behave exactly like manual-apply.

Never click Submit on any page before this gate has passed.

## 9. Submit

Click the Submit / Apply / Send application button with `computer`. Wait ~3 s. `get_page_text` and look for confirmation ("submitted", "thank you for applying", "application received"). Screenshot with `computer` `screenshot` and save to `HAFJOB/Applications/<basename>.png`.

- Confirmation visible → `status: applied`.
- Validation errors → read them, fix if the fix is a source you already have, re-gate (step 8) on the changed rows, resubmit once. Otherwise ask the user.
- Nothing recognizable → `status: filled-not-submitted`, do not click Submit again, tell the user to check the tab.

## 10. Write records and report

1. Write `HAFJOB/Applications/<date> <company> - <role>.md` (log.md).
2. Append the final Log.md line (`submitted` / `filled-not-submitted` / `failed` / `cancelled`).
3. Save any answers the user agreed to keep into `Answers/`.
4. In chat: status line, the field table from step 8, and the note path as a wikilink `[[<basename>]]`.

Leave the Chrome tab open.
