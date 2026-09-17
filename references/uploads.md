# Uploads (Claude in Chrome)

Use `mcp__claude-in-chrome__file_upload`. Do **not** click the upload button or the file input — that opens a native picker you cannot drive.

1. Resolve the resume path from `Profile.md` → `resume`. Vault-relative paths resolve against `<vault>/`. Verify the file exists and is < 10 MB. Missing → hard stop, ask for the path.
2. `read_page` with `filter: "interactive"` (or `find "resume"` / `find "upload"`) to get the `ref` of the `input[type=file]`. On some ATSs the input is hidden behind a styled button; `read_page` with `filter: "all"` still lists it.
3. `file_upload` with `tabId`, `ref`, `paths: ["<absolute path>"]`.
4. Wait 2–3 s, then `read_page` / `find` for the filename. Confirm the ATS shows it attached (filename text, "Remove" link, or parsed-resume banner).
5. Greenhouse, Ashby, and Workday parse the resume and may overwrite name/email/phone/experience fields. Re-read those fields after upload and restore any value that changed from what `Profile.md` says.
6. Cover letter: only upload or paste one if the user provided it or approved a draft this session. "Optional" cover letter fields stay empty unless the user says otherwise.

If `file_upload` fails twice, stop and ask the user to attach the file manually in the Chrome window, then continue.
