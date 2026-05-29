# Push instructions

How to get this `public-repo/` onto GitHub as the public repository
`catch-before-jobs-vanish`. This file is for you — you can delete it before or after
pushing (it won't hurt anything if it stays).

## 0. One-time cleanup (do this first)

A leftover `.git` folder from the prep session may exist inside `public-repo/`. Remove
it so you start from a clean slate. In this folder, from **PowerShell**:

```powershell
Remove-Item -Recurse -Force .git
```

or in **Command Prompt**:

```cmd
rmdir /s /q .git
```

(If there's no `.git` folder, skip this — nothing to do.)

## 1. Final PII sweep (verify before every push)

From inside `public-repo/`, search for your own private values and confirm none of
them appear (except your author name, which intentionally lives in the LICENSE).

Build the search from **your own** sensitive strings — don't hard-code real values
into this public file. Things to check for:

- your real name(s) / login handle
- your current and past employer names
- your personal email address
- your Google Sheet ID (or a unique fragment of it)
- your local folder names (e.g. a personal "memory" or "career" folder)
- any real job-posting IDs you pasted while testing

```bash
# Replace the CAPS placeholders with your own values before running:
grep -ri "YOUR_NAME\|YOUR_EMPLOYER\|YOUR_OTHER_EMPLOYER" .
grep -ri "your-email@example.com\|YOUR_SHEET_ID_FRAGMENT" .
grep -ri "your-personal-folder-name" .
# Your author name should appear ONLY in ./LICENSE — verify:
grep -rin "YOUR_AUTHOR_NAME" .
```

(On Windows without grep: `findstr /s /i "YOUR_NAME YOUR_EMPLOYER your-email" *`)

This sweep was run during prep with the real values and came back clean. Re-run it
any time you add files. Note: this file deliberately contains **no** real private
strings, so it's safe even if you keep it in the public repo.

## 2. Initialize and commit

```bash
git init
git add -A
git status            # confirm your-input/cv.md etc. are NOT listed (they're gitignored)
git commit -m "Initial public release: catch-before-jobs-vanish"
```

Before committing, eyeball `git status` and make sure none of these appear:
`your-input/cv.md`, `your-input/preferences.md`, `your-input/companies.md`,
`your-input/config.md`, `scheduler-prompt.md`, `SKILL.md`. Only `your-input/.gitkeep`
and `your-input/README.md` should be tracked from that folder.

## 3. Create the GitHub repo and push

On GitHub, create a new **public** repo named `catch-before-jobs-vanish` (no README,
no .gitignore, no license — this repo already has them). Then:

```bash
git branch -M main
git remote add origin https://github.com/<your-username>/catch-before-jobs-vanish.git
git push -u origin main
```

## 4. After the push — sanity check on github.com

Open the repo in a browser and confirm:

- `your-input/` shows only `README.md` (and the `.gitkeep`), no personal files.
- `scheduler-prompt-template.md` is present (it's the public template — not the
  gitignored `scheduler-prompt.md`).
- There is **no** `case-study/` folder (the build story is kept separately as
  long-form writing — see note below).
- `LICENSE` is the only place your author name appears.

## What gets pushed (14 files)

```
.gitignore
LICENSE
README.md
PUSH_INSTRUCTIONS.md        (optional — delete if you don't want it public)
setup.md
scheduler-prompt-template.md
your-input/.gitkeep
your-input/README.md
your-input/cv.example.md
your-input/preferences.example.md
your-input/companies.example.md
your-input/config.example.md
docs/architecture.md
docs/fit-scoring-rubric.md
```

(Your real `your-input/cv.md` / `preferences.md` / `companies.md` / `config.md` are
gitignored — only the `*.example.md` versions are public.)

> **About the build story / case study:** the sanitized decision-log and
> failures-and-recoveries write-ups were moved out of this repo to
> `../blog-drafts/` (not pushed). They're intended for a blog/Substack post; once
> published, add a link in `README.md`. An empty `case-study/` folder may remain
> locally — Git does not track empty folders, so it won't be pushed. Delete it from
> your file explorer if you like.

## Note on `.gitignore`

The `*scheduler*.md` wildcard was removed during prep specifically so that
`scheduler-prompt-template.md` (the public template) is NOT ignored, while the
personal `scheduler-prompt.md` still is. Keep that distinction if you edit the
ignore file.
