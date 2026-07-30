# catch-before-jobs-vanish

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**Language:** English · [한국어](README.ko-KR.md)

I built this while job-hunting in the EU: fresh postings kept vanishing from
LinkedIn's "last 24 hours" search before I ever saw them. So I turned the
chase into a scheduled agent, with one promise: **if a posting that truly
fits you goes up, you hear about it within 24 hours — before it disappears.**

Every morning it searches LinkedIn + Indeed and the careers pages of the
companies I'm actively gunning for, scores each posting against the evidence
in my CV on a 100-point rubric, verifies that top matches are still open, and emails me only what's
worth my time. The rubric isn't guesswork: it was corrected by a backtest of
**29 of my real applications with known outcomes, including CV screens passed
at Uber and Google**. Built and run with [Claude](https://claude.com) in
Cowork mode. The name is the problem statement: **catch them before they
vanish**.

> **v3 (2026-07).** Rewritten after a month of daily production runs and the
> backtest above. What changed and why: [`CHANGELOG.md`](CHANGELOG.md).

## Demo

*(Coming soon: the example persona from [`your-input/`](your-input) run
through a full pipeline pass — the morning email and the Sheet it writes, as
screenshots.)*

## What is this

A daily job-alert pipeline. Four things it does for you:

| Feature | In one line |
|---|---|
| **Evidence-based fit scoring** | Extracts each JD's actual requirements and grades them against written evidence in your `cv.md` — no impression scores. The score is a priority signal for where to spend application effort ([rubric](skills/job-alert/fit-scoring-rubric.md)). |
| **Skill calibration** | Sorts your skills into **Core / Transferable (with caps) / Gap**, so the scorer knows what you do, what's adjacent, and what you can't claim. An agent drafts it from your CV; you confirm it. |
| **Daily scan** | Searches LinkedIn + Indeed (last 24 hours) and checks your Aiming companies' careers pages on every run; once a week, a deep-scan day sweeps the rest of your watchlist. |
| **One posting = one row** | Three fingerprints (LinkedIn jobId / careers job-ID / company + normalized title) are matched against the whole Sheet with no time window — a duplicate never comes back. |

## Why it's different

Think of it as hiring a headhunter who works only for you. It doesn't spray
applications everywhere — it watches the market every morning and points your
attention at the postings your career evidence actually supports.

Plenty of projects publish agent prompts — that part is commodity. What this
repo adds is the **operating record**: a public rubric that a month of daily
runs and a backtest of past real applications (CV screens passed at Uber and
Google included) actually corrected, with every change and its reason written
down in the [changelog](CHANGELOG.md) and the
[rubric](skills/job-alert/fit-scoring-rubric.md).

## How to start

Not a program you install — a work order an AI agent runs every morning. No
server, no code setup; plan on about 30 minutes the first time. You need:

- An AI agent that can run a daily scheduled task — built and tested with
  Claude in Cowork mode.
- Three connectors enabled in Claude: **Chrome** (reads job sites), **Google
  Drive** (writes the Sheet), **Gmail** (sends the alert).
- A Google account and an email address for the alerts.

Then four steps — [setup.md](setup.md) walks through each one, no git or
coding knowledge assumed:

1. **Get the repo** — `git clone`, or Code → Download ZIP on this page.
2. **Create a 2-tab Google Sheet** and paste its Sheet ID into
   `your-input/config.md` —
   [setup.md, Step 2](setup.md#step-2--create-the-google-sheet).
3. **Fill in [`your-input/`](your-input)** — copy the four `*.example.md`
   files (a fully fictional demo persona) and edit them with your details.
4. **Hand it to your agent.** In Claude Code, two commands install it as the
   `job-alert` skill:

   ```
   /plugin marketplace add Journey-512/catch-before-jobs-vanish
   /plugin install catch-before-jobs-vanish@catch-before-jobs-vanish
   ```

   On Cowork or any other agent, copy the prompt block from
   [`skills/job-alert/SKILL.md`](skills/job-alert/SKILL.md) into a daily
   scheduled task —
   [setup.md, Step 4](setup.md#step-4--hand-the-pipeline-to-your-agent).

Then trigger it once manually (or wait for tomorrow's email) and check the
results against [setup.md, Step 5](setup.md#step-5--test-run).

## How to use

A morning with the example persona — a fictional 5-year mobility & travel PM
based in Berlin, targeting Senior PM roles across the EU:

1. **9:00 — the email arrives.** Only Top (85+) and Strong (70-84) matches;
   say a Senior PM posting at one of their Aiming companies scored 86, with a
   one-line Fit Reason: which requirements matched, the main gap, the angle
   to lead with. Headhunter posts are tagged `[Headhunter]`; auto-added
   companies come with a veto prompt. On a quiet day the mail still arrives:
   "No new matches. System alive."
2. **They open the Sheet** when they want the full picture: every finding is
   a row, including what the email left out as `Excluded (...)` or
   `Closed (date)` — so "why isn't this in the email?" always has an answer.
3. **They apply, and write it down.** `Applied` in the Status column
   (`Applied (referral)` if someone referred them), later updated to
   `Passed - CV` or `Rejected - CV`.
4. **Once a week, the deep-scan email adds a funnel summary** built from
   those Status entries: applications and CV-pass rate per score band, cold
   and referral kept separate, sample sizes included. That's the data they
   use when adjusting `preferences.md` or a calibration line — the run
   reports, they decide.

## How it works

Four stages, run as one scheduled task every morning:

```mermaid
flowchart LR
  A[("1 · Read your files<br/>cv · preferences<br/>companies · config")] --> B["2 · Find candidates<br/>24h search · normalize<br/>hard gates · dedup"] --> C["3 · Grade each one<br/>evidence score 0-100<br/>cutoff 70 · still open?"] --> D[("4 · Record and notify<br/>Sheet rows · morning email<br/>heartbeat")]
```

The first three stages only read — every write happens in stage 4, so a run
that dies mid-way leaves nothing half-done. Under the hood the stages split
into ten single-purpose steps; the prompt itself is the map:
[`skills/job-alert/SKILL.md`](skills/job-alert/SKILL.md), with a Korean guide
at [`docs/skill.ko-KR.md`](docs/skill.ko-KR.md).

The pipeline has exactly **two rule sources**: the pipeline prompt (the
logic, never edited) and [`your-input/`](your-input) (your data, read at
runtime). The Google Sheet holds records, never rules.

A few safeguards a month of production runs promoted into rules:

- **Permalinks, never search URLs** — a 24h-filtered search URL is a moving
  window that dies within hours; per-posting `jobs/view/{id}` links don't.
- **Liveness check** — everything scoring above 80 is verified still open
  before the email goes out. No more applying to ghosts.
- **Heartbeat** — zero matches still sends "No new matches. System alive.",
  so silence always means breakage, never zero results.

**The outcome loop.** The Status column you keep by hand (`Applied` /
`Passed - CV` / `Rejected - CV` / `Lost`, plus a `(referral)` suffix where it
applies) doubles as label data. On the deep-scan day the email appends a
funnel summary — CV-pass rate per score band, cold and referral separated,
always with sample sizes. The pipeline measures and reports; changing the
rules stays your call.

## Project structure

```
catch-before-jobs-vanish/
├── README.md · README.ko-KR.md      this page (English · Korean)
├── CHANGELOG.md                     version history, with the reason next to each change
├── setup.md                         30-minute setup walkthrough (EN + KO)
├── skills/
│   └── job-alert/
│       ├── SKILL.md                 the daily pipeline prompt — all of the logic, zero personal data
│       └── fit-scoring-rubric.md    the scoring rubric + what the backtest changed
├── .claude-plugin/                  manifest files behind the two-command install
├── docs/
│   └── skill.ko-KR.md               Korean guide to the pipeline prompt
├── your-input/                      your personal data (gitignored — only examples are public)
│   ├── README.md                    what goes in each of the four files
│   └── *.example.md                 filled-in examples (fictional persona) — copy and edit
├── LICENSE                          MIT
└── .gitignore                       keeps your real files out of git
```

## Tech stack

| Layer | Choice |
|---|---|
| Runtime | Claude — a scheduled prompt in Cowork mode (how the author runs it), or the `job-alert` skill in Claude Code (two-command install) |
| Collection | LinkedIn + Indeed through the Chrome connector; described at concept level |
| Records | Google Sheets (2 tabs) through the Google Drive connector |
| Alerts | Gmail connector, draft-then-send |
| Configuration | Plain Markdown files in `your-input/` |
| Servers / code | None |

## FAQ

**Does it apply for me?** No — on purpose. Applying is judgment; this tool's
job is to make sure you never miss the window and never waste a morning
re-searching.

**Is this allowed on LinkedIn?** This repo documents collection at the
concept level only and ships no scraping recipes. Run it with your own
account, at a human pace, and comply with each site's terms.

**Will the score predict whether I pass the CV screen?** No, and the backtest
says exactly that. The score is a priority signal for where to spend
application effort; the outcome loop exists to measure the rest.

**Can I use an agent other than Claude?** Yes, if it can browse the web,
read/write a Google Sheet, send email, and run on a daily schedule.

## About

I'm a product manager. I built this for my own EU job search, and it has run
every morning since. The longer build story — the decisions, failures, and
recoveries behind it — is being written up separately; a link will be added
here once it's published.

## Disclaimer

- Not affiliated with LinkedIn, Indeed, Google, or any company mentioned.
- Open source that runs locally, on your own accounts. There is no hosted
  service behind it, and the author never collects, stores, or accesses any
  of your data — postings, Sheet, and email all stay with you.
- Collection is documented at concept level; you are responsible for
  complying with each site's terms of service.
- Fit scores are a prioritization signal, not a prediction of screening
  outcomes.
- An AI agent acts on your behalf here — reviewing what it wrote and sent
  each morning remains your responsibility.
- No guarantee of job-search results.

## License

[MIT](LICENSE). Fork it, adapt it, ship your own.

## Contact

[LinkedIn](https://www.linkedin.com/in/journeymjlee/) ·
[hemegi.lee@gmail.com](mailto:hemegi.lee@gmail.com)
