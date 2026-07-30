# Setup

**Language:** English (below) · [한국어](#설정-한국어)

A step-by-step guide to running `catch-before-jobs-vanish` for yourself. Plan
on about 30 minutes the first time.

## Prerequisites

This system is not a program you install — it's a **work order that an AI
agent runs every morning**. So there's no server to rent and no code to set
up. What you need:

- **An AI agent that can run scheduled tasks.** Built and tested with
  **Claude in Cowork mode**, which has scheduling built in.
- **Three connectors enabled in Claude.** A connector is Claude's official
  way of plugging into another service on your account — you switch them on
  in Claude's settings:
  - **Chrome** — opens job sites and reads postings.
  - **Google Drive** — reads and writes the Google Sheet that stores results.
  - **Gmail** — sends you the morning alert email.
- **A Google account** (the Sheet lives there) and an email address for the
  alerts.

Using a different agent? It needs the same four abilities: browse the web,
read/write a Google Sheet, send email, and run on a daily schedule.

## Step 1 — Get the repo

```bash
git clone https://github.com/<your-username>/catch-before-jobs-vanish.git
cd catch-before-jobs-vanish
```

New to git? On the GitHub page, **Code → Download ZIP** works too — unzip it
anywhere and everything below is the same.

## Step 2 — Create the Google Sheet

Make a new Google Sheet with **two tabs**. The two tabs hold different
kinds of data: **`Jobs`** is posting history — every find, the dedup
evidence, and the application outcomes you type. **`Companies`** is your
company registry — the single source of truth for which companies the
pipeline watches and how.

**`Jobs`** — 10 columns:

| Date Added | Company | Job Title | Location | Source | Posted Date | Link | Status | Fit Score | Fit Reason |
|---|---|---|---|---|---|---|---|---|---|

The `Status` column is shared: the system writes `Excluded (...)` / `Closed
(date)`, and **you** write your application outcomes (`Applied`, `Passed - CV`,
`Rejected - CV`, `Lost` — add a `(referral)` suffix for referral applications).
Those hand-written statuses feed the outcome loop; the system never overwrites
them.

**`Companies`** — your company registry, 9 columns:

| Index | Company | Aiming | Match Level | URL | Source site | HQ | Topics | Memo |
|---|---|---|---|---|---|---|---|---|
| 1 | `<Company name>` | `<Aiming or blank>` | `<Strong or Soft>` | `<careers URL or blank>` | `<source or blank>` | `<HQ or blank>` | `<topics or blank>` | `<note or blank>` |

This tab is the **single source of truth for your company list** — there is
no local companies file. Before the first run, enter your target companies
directly into this tab, one row per company
([`your-input/companies.example.md`](your-input/companies.example.md) is a
format reference). The pipeline reads the tab at the start of every run: it
drives the company-scoped searches, the Aiming checks, and the weekly deep
scan. The pipeline also appends rows for new companies it discovers (if
enabled in `config.md`) — auto-added rows join those searches from the next
run, and values you typed in existing rows are never overwritten.

`Aiming` is a manual flag for the 1-3 companies you're actively gunning for.
An Aiming company gets a wider search every run: a 7-day window instead of
24h, your extended locations, and a direct careers-page check. Set it by hand
only — auto-added rows always leave it blank.

**Upgrading from an earlier version?** Your old `your-input/companies.md` is
not read anymore — and nothing deletes it; it's your personal file. Move its
rows into this tab: Name → Company, careers URL → URL, Match level → Match
Level, Aiming → Aiming, note → Memo; fill the remaining columns only where
you actually know the value. Check the tab against the old file to confirm
everything moved, then decide yourself whether to archive or delete the
file.

**Where does the sheet go?** Anywhere in your Google Drive — any folder, any
file name. The system finds it by **Sheet ID**, not by name or location. Copy
the ID from the sheet's address bar:

```
https://docs.google.com/spreadsheets/d/THIS-LONG-STRING-IS-THE-ID/edit
```

The long string between `/d/` and `/edit` is the Sheet ID. In the next step
you'll paste it into `your-input/config.md` (the `Google Sheet ID:` line) —
that's the only place it goes. The sheet just needs to sit in the same Google
account your Drive connector uses.

## Step 3 — Fill in `your-input/`

Two of the folder's **example files** are filled in for a made-up person (a
fictional 3-year travel & mobility PM) and meant for copying — drop the
`.example` from the name and edit with your details:

```bash
cd your-input
cp preferences.example.md preferences.md
cp config.example.md config.md
cd ..
```

(Windows PowerShell: `Copy-Item preferences.example.md preferences.md`, etc.)

The other two examples are references, not copy targets:
`companies.example.md` shows the format of the `Companies` tab you filled in
Step 2, and `cv.example.md` shows what a converted `cv.md` looks like.

What each runtime file is for:

- `cv.md` — your career evidence for scoring, plus a **Skill calibration**
  section; created by the one-time conversion below, not by copying
- `preferences.md` — titles, locations, industries, recency, cutoff, output
  language, an **Eligibility** section (knockouts), plus **Remote-work
  compatibility**: the work-hours timezone you'll actually work in, and the
  maximum timezone difference (default 4 hours) a remote role may sit from
  it before it's excluded
- `config.md` — Sheet ID, email recipient, subject, deep-scan weekday (one
  day a week the run sweeps your whole watchlist, not just the fresh
  postings), per-tier cap, the **Alert delivery** section (alert schedule
  in plain language + alert timezone), plus the **Company discovery**
  settings (whether new companies get auto-added to the `Companies` tab,
  and the maturity bar they must clear)

Your local input is these three files. Your company list is not one of
them — it lives in the Sheet's `Companies` tab (Step 2).

Timezones are written in Region/City form — `Europe/Amsterdam`,
`Europe/Berlin`, `Asia/Seoul`, `America/New_York` — never a bare city name,
a GMT offset, or an
abbreviation like CET or KST: Region/City keeps your local clock right
when daylight saving shifts the offset. The two timezone fields do
different jobs and are never merged: the alert timezone (`config.md`) sets
the run's dates, the deep-scan weekday, `Date Added`, and email times,
while the work-hours timezone (`preferences.md`) only judges remote roles.
They may hold the same value — the demo persona gets alerts in
`Asia/Seoul` and plans to work in `Europe/Berlin`, so they differ.

**Your `cv.md`: paste, convert, review.** Create
`your-input/cv-original.md` and paste the complete text of your existing CV
under this heading — plain text copied out of your CV is fine, no Markdown
cleanup needed:

```markdown
# Original CV

<Paste the complete text of your existing CV here>
```

Only have a PDF or DOCX? Ask your agent to extract the full text into
`cv-original.md` first — the daily pipeline never reads PDF or DOCX files.
Then give your agent this conversion request:

```
Read `your-input/cv-original.md` and create `your-input/cv.md`.

Use `your-input/cv.example.md` only as a reference for the output structure.
Never copy facts from the fictional example.

Structure the result as:

# CV summary (for fit scoring)
## Positioning
## Experience
## Strengths
## Domains I know well
## Skill calibration

Preserve every company, role, date, number, achievement, and skill exactly as
supported by `cv-original.md`. Do not invent, strengthen, or infer experience
that is not written in the source.

You may read `preferences.md` to understand the target role and industries,
but preferences are context, never career evidence.

Draft Skill calibration conservatively from explicit CV evidence:
- Core: direct, repeated experience
- Transferable: partial or adjacent experience, with an honest cap
- Gap: do not infer a gap merely because the CV is silent

If evidence is missing or ambiguous, omit the claim instead of guessing.
Do not anonymize company names unless I explicitly ask.
After creating the file, show me what I need to review before it is used.
```

Before you register the scheduled task, review the generated `cv.md`:

1. Company names, titles, dates, and numbers match the original.
2. No new experience or achievements were invented.
3. `Strengths` and `Domains I know well` stay within the actual evidence.
4. The `Skill calibration` Core / Transferable / Gap split matches your
   real level of experience.

`cv-original.md` stays yours and private; the daily run reads only
`cv.md`. Nothing regenerates automatically — when your CV changes, rerun
the conversion and review the new `cv.md`.

**Upgrading from an earlier version?** A good, already-reviewed `cv.md`
stays valid — no `cv-original.md` needed. Also rename two settings if your
files predate this version: `Schedule:` (cron) becomes `Alert schedule:`
plus `Alert timezone:` in `config.md`, and `Home timezone` plus `Remote
anchor rule` become `Work-hours timezone` plus `Maximum timezone
difference` in `preferences.md`. The run stops and names the field when
`Alert timezone` or `Work-hours timezone` is missing (a missing `Alert
schedule` alone never blocks a manual run, and a missing `Maximum timezone
difference` falls back to 4 hours).

**About the Skill calibration section in `cv.md` — read this once.** It's a
list where each line states how far you've *actually* gone with a skill. It
exists because AI scorers over-credit by default: the word "SQL" anywhere in
a CV tends to be read as "SQL practitioner". Write `SQL: I read and tweak
dashboard queries` and the scorer can no longer give you full credit on a
hands-on data-engineering requirement. The more honest this list, the closer
your daily scores track reality.

How to build it: the conversion request above *drafts* it from your CV —
then go line by line and correct it yourself before keeping it. The final
say stays with you on purpose — if the same AI that scores you also sets
its own limits, the check means nothing.

You can also leave it empty: the system still runs, the scorer just falls
back to one default rule — only what's written in the CV counts, nothing gets
assumed on top. Add a line whenever a score looks off.

Details and skeletons: [`your-input/README.md`](your-input/README.md). Your
real `cv-original.md` / `cv.md` / `preferences.md` / `config.md` are on
git's ignore list (only the `*.example.md` versions are public), so your
personal data never gets pushed.

## Step 4 — Hand the pipeline to your agent

Two routes to the same place — pick the one that matches your agent. Either
way, the actual trigger lives in your agent's scheduled task, not in
`config.md`: the pipeline never reads the `Alert schedule` line to create
or change a schedule. Register the task to match `config.md`'s Alert
delivery section — in plain language, no cron needed, e.g. "daily at 09:00
in Asia/Seoul". Two things follow: if you later change the schedule or
timezone in `config.md`, change the scheduler's registration too (editing
the file alone updates nothing), and after registering, check that the
scheduler's next-run time matches the local time you intended.

**Skill install — if your agent is Claude Code.** Run these two commands
inside Claude Code:

```
/plugin marketplace add Journey-512/catch-before-jobs-vanish
/plugin install catch-before-jobs-vanish@catch-before-jobs-vanish
```

That registers the pipeline as the `job-alert` skill. Then create your
scheduled task with a one-line prompt — say the schedule, the timezone, and
where your clone is, e.g. *"Run the job-alert skill daily at 09:00 in
Asia/Seoul; my repo is at `C:\Users\me\catch-before-jobs-vanish`"*. The
path is how the skill finds your `your-input/`.

**Copy-paste — Cowork and any other agent** (the route the author's own
daily runs use).

1. Open [`skills/job-alert/SKILL.md`](skills/job-alert/SKILL.md).
2. Copy the prompt block (on GitHub, the copy button in the block's corner
   grabs the whole thing).
3. Replace the one repo-path placeholder in Step 0 with your local clone path
   so it can find `your-input/`. **Make no other edits** — everything else is
   read at runtime.
4. Create a scheduled task in your agent with that prompt.

In Cowork mode you can just say: *"Run this prompt daily at 09:00 in
Asia/Seoul"* and paste the block.

## Step 5 — Test run

Trigger the task once manually (or wait for the first scheduled fire). Check:

- The `Jobs` tab gets new rows with absolute `Posted Date` values (YYYY-MM-DD,
  never "Last 24h").
- Links are `jobs/view/{id}` permalinks, not search URLs.
- The email groups matches into **Top / Strong** (nothing below the cutoff),
  lists any auto-added companies with a veto prompt, and links the Sheet.
- Any location violations found inside a JD are in the Sheet as
  `Excluded (...)` but **not** in the email.
- Re-run on a quiet day (or with a deliberately narrow filter) and confirm the
  **heartbeat**: zero matches still sends "No new matches. System alive."

## Step 6 — Tune (two feedback loops — don't mix them)

1. **Calibration loop — is the scoring right?** A score looks off → read that
   row's Fit Reason → add or tighten **one line** in `cv.md`'s Skill
   calibration → the next run scores better. The prompt and rubric stay
   frozen; only your data evolves.
2. **Outcome loop — does the score predict anything?** Keep the Status column
   current as you apply. The deep-scan day's report summarizes CV-pass rate
   per score band (cold and referral separated). Only adjust preferences or
   calibration on patterns with real sample sizes — check n first.

Routine knobs, anytime:

- Too many / too few emails → the cutoff in `preferences.md` (70 is a default,
  not a law).
- Noise creeping in → the title allow/exclude lists.
- Watchlist growing too fast or slow → the Company discovery settings in
  `config.md`.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Saved LinkedIn links go dead by afternoon | A search URL got saved instead of a permalink | Confirm the agent extracts per-posting `jobs/view/{id}` permalinks, not the search URL |
| `Posted Date` says "Last 24h" | Relative label stored instead of parsed | Ensure Step 2 of the prompt parses to YYYY-MM-DD at capture |
| Wrong company's job recovered | Trusted LinkedIn's auto-selected `currentJobId` | Parse the full result list and match by company + title |
| The same posting keeps reappearing | Dedup can't see history (Sheet unreadable) or careers URLs not canonicalized | Check the email for the "history not checked" notice; verify one canonical URL format per careers site |
| A company in your `Companies` tab never shows results | Company career page changed, or a broken company filter silently returns empty | Expected for careers pages (they change without notice) — the run falls back to job boards and discloses the gap; never trust an empty filter result without a keyword cross-check |
| Aiming / company / deep-scan searches all skipped | The `Companies` tab couldn't be read, or its header drifted from the 9-column contract | The email names the skipped scope and reason; restore the 9-column header shown in Step 2, confirm the Sheet ID and sharing, then rerun |
| No email at all | The run died, or the send failed | The heartbeat rule means silence = breakage: check the task ran, then check Drafts — draft-then-send leaves a draft behind on a failed send |

---

<a name="설정-한국어"></a>

# 설정 (한국어)

**언어:** [English](#setup) · 한국어

`catch-before-jobs-vanish`를 직접 돌리는 단계별 안내입니다. 처음이면 30분
정도 잡으세요.

## 사전 준비

이 시스템은 설치하는 프로그램이 아닙니다. **AI 에이전트가 매일 아침 실행하는
작업 지시문**입니다. 그래서 서버를 빌릴 필요도, 코드를 만질 일도 없습니다.
필요한 것은 세 가지입니다.

- **예약 작업(scheduled task)을 돌릴 수 있는 AI 에이전트.** 이 시스템은 Claude
  **Cowork 모드**로 만들고 테스트했습니다. Cowork에는 예약 실행 기능이 들어
  있습니다.
- **Claude에 커넥터(connector) 3개 연결.** 커넥터는 Claude가 내 계정의 다른
  서비스에 접근하도록 이어 주는 공식 연결 기능입니다. Claude 설정에서 켭니다:
  - **Chrome** — 채용 사이트를 열어 공고를 읽을 때 씁니다.
  - **Google Drive** — 결과를 기록하는 구글 시트를 읽고 쓸 때 씁니다.
  - **Gmail** — 아침 알림 이메일을 보낼 때 씁니다.
- **구글 계정 1개**(시트는 이 계정에 둡니다)와 알림 받을 이메일 주소.

다른 AI 에이전트를 쓴다면, 같은 능력 4가지를 갖췄는지만 확인하면 됩니다: 웹
탐색, 구글 시트 읽고 쓰기, 이메일 발송, 매일 예약 실행.

## 1단계 — 저장소 받기

```bash
git clone https://github.com/<본인-아이디>/catch-before-jobs-vanish.git
cd catch-before-jobs-vanish
```

git이 낯설면 GitHub 페이지에서 **Code → Download ZIP**으로 내려받아 아무
폴더에나 풀어도 됩니다. 아래 과정은 똑같습니다.

## 2단계 — 구글 시트 만들기

**탭 2개**짜리 새 구글 시트를 만듭니다. 두 탭은 맡는 데이터가 다릅니다.
**`Jobs`**는 공고 이력입니다 — 발견한 공고, 중복 확인 근거, 그리고 내가 적는
지원 결과가 쌓입니다. **`Companies`**는 회사 registry입니다 — 파이프라인이
어떤 회사를 어떻게 지켜볼지를 정하는 유일한 원본입니다.

**`Jobs`** — 10열:

| Date Added | Company | Job Title | Location | Source | Posted Date | Link | Status | Fit Score | Fit Reason |
|---|---|---|---|---|---|---|---|---|---|

`Status` 열은 시스템과 내가 같이 씁니다. 시스템은 `Excluded (...)` / `Closed
(날짜)`를 적고 **나는** 지원 결과를 적습니다 (`Applied`, `Passed - CV`,
`Rejected - CV`, `Lost` — referral로 지원했다면 `(referral)`을 뒤에 붙입니다).
내가 적은 값은 나중에 점수 검증 자료가 됩니다. 시스템이 덮어쓰는 일은 없습니다.

**`Companies`** — 회사 registry, 9열:

| Index | Company | Aiming | Match Level | URL | Source site | HQ | Topics | Memo |
|---|---|---|---|---|---|---|---|---|
| 1 | `<Company name>` | `<Aiming or blank>` | `<Strong or Soft>` | `<careers URL or blank>` | `<source or blank>` | `<HQ or blank>` | `<topics or blank>` | `<note or blank>` |

이 탭이 **회사 목록의 유일한 원본**입니다 — 로컬에 회사 파일은 없습니다. 첫
실행 전에 관심 회사를 이 탭에 한 줄에 한 회사씩 직접 입력하세요 (형식 참고:
[`your-input/companies.example.md`](your-input/companies.example.md)).
파이프라인은 매 실행 시작 시 이 탭을 읽어 회사별 검색, Aiming 검색, 주간
딥스캔의 대상을 정합니다. 새로 발견한 회사를 이 탭에 추가하기도 합니다
(`config.md`에서 켜 둔 경우). 자동 추가된 행은 다음 실행부터 그 검색들에
포함되고, 기존 행에 내가 적은 값을 파이프라인이 덮어쓰는 일은 없습니다.

`Aiming`은 지금 집중해서 노리는 회사 1-3곳에 직접 적어 두는 표시입니다. Aiming
표시가 있는 회사는 매일 실행 때 검색 범위가 넓어집니다: 최근 24시간 대신 7일치를
보고, `preferences.md`에 적어 둔 확장 지역까지 찾고, 그 회사 채용 페이지도 직접
확인합니다. 이 표시는 반드시 직접 적으세요 — 시스템이 자동으로 추가한 회사에는
절대 붙지 않습니다.

**이전 버전에서 업그레이드한다면?** 예전 `your-input/companies.md`는 더 이상
읽히지 않습니다 — 그렇다고 시스템이 지우지도 않습니다. 개인 파일이니까요.
행을 이 탭으로 옮기세요: Name → Company, careers URL → URL, Match level →
Match Level, Aiming → Aiming, note → Memo. 나머지 열은 확실히 아는 값만
채웁니다. 옮긴 내용이 빠짐없는지 시트에서 원래 파일과 대조해 확인한 뒤, 원래
파일을 보관할지 지울지는 직접 결정하세요.

**시트는 어디에 두나요?** 구글 드라이브 안이면 어디든, 이름이 뭐든 상관없습니다.
시스템은 시트를 이름이나 위치가 아니라 **시트 ID**로 찾습니다. 시트를 열고
주소창을 보면:

```
https://docs.google.com/spreadsheets/d/이-긴-문자열이-시트-ID/edit
```

`/d/`와 `/edit` 사이의 긴 문자열이 시트 ID입니다. 이걸 복사해 두었다가 다음
단계에서 `your-input/config.md`의 `Google Sheet ID:` 줄에 붙여넣으면 됩니다 —
넣는 곳은 거기 한 군데뿐입니다. 시트는 Google Drive 커넥터에 연결한 그 구글
계정에 있기만 하면 됩니다.

## 3단계 — `your-input/` 채우기

이 폴더의 예시 파일 중 2개는 가공 인물(창작한 여행·모빌리티 3년차 PM)로
**미리 채워 둔 복사용**입니다 — 이름에서 `.example`만 빼고 본인 내용으로
고치세요:

```bash
cd your-input
cp preferences.example.md preferences.md
cp config.example.md config.md
cd ..
```

(윈도우 PowerShell에서는 `Copy-Item preferences.example.md preferences.md`처럼
쓰면 됩니다.)

나머지 예시 2개는 복사하지 않는 참고 자료입니다. `companies.example.md`는
2단계에서 만든 `Companies` 탭의 입력 형식을, `cv.example.md`는 변환된
`cv.md`가 어떤 모습인지 보여줍니다.

각 런타임 파일의 용도:

- `cv.md` — 채점에 쓸 경력 정리 + **Skill calibration** 섹션. 복사가 아니라
  아래의 1회성 변환으로 만듭니다
- `preferences.md` — 직무명, 지역, 산업, 최신성, 이메일 컷, 출력 언어,
  **Eligibility** 섹션 (지원이 무의미해지는 조건) + **Remote-work
  compatibility**: 실제로 근무하려는 시간대(`Work-hours timezone`)와 remote
  공고가 그 시간대에서 벗어나도 되는 최대 차이(`Maximum timezone
  difference`, 기본 4시간)
- `config.md` — 시트 ID, 이메일 수신자, 제목, 딥스캔 요일 (일주일에 하루,
  새 공고만이 아니라 관심 회사 전체를 훑는 날), tier당 회사 수 제한 +
  **Alert delivery** 섹션 (자연어로 적는 알림 일정 + 알림 시간대) +
  **Company discovery** 설정 (새 회사를 `Companies` 탭에 자동 추가할지,
  그리고 통과해야 하는 회사 성숙도 기준)

로컬 입력은 이 세 파일이 전부입니다. 회사 목록은 여기 없습니다 — 2단계의
`Companies` 탭에서 관리합니다.

시간대는 지역/도시 형식으로 적습니다 — `Europe/Amsterdam`, `Europe/Berlin`,
`Asia/Seoul`, `America/New_York`. 도시 이름만 적거나 GMT+9 같은 오프셋, CET·KST 같은
약어는 쓰지 않습니다. 지역/도시 형식이어야 서머타임으로 오프셋이 바뀌어도
현지 시각이 유지됩니다. 두 시간대 필드는 역할이 달라 합치지 않습니다:
`config.md`의 알림 시간대(Alert timezone)는 실행 날짜, 딥스캔 요일, `Date
Added`, 이메일 시각의 기준이고, `preferences.md`의 근무 시간대(Work-hours
timezone)는 remote 공고 판정에만 씁니다. 값이 같아도 되지만, 데모의 가공
인물은 알림은 `Asia/Seoul`에서 받고 근무는 `Europe/Berlin`을 계획해서 서로
다릅니다.

**`cv.md` 만들기: 붙여넣고, 변환하고, 검토합니다.**
`your-input/cv-original.md`를 만들고 아래 제목 밑에 기존 이력서의 전체
텍스트를 그대로 붙여넣으세요 — 이력서에서 복사한 일반 텍스트면 충분하고,
Markdown으로 다시 정리할 필요 없습니다:

```markdown
# Original CV

<여기에 기존 이력서의 전체 텍스트를 붙여넣으세요>
```

PDF나 DOCX만 있다면, 에이전트에게 내용을 빠짐없이 추출해 `cv-original.md`로
저장해 달라고 먼저 요청하세요. 매일 실행되는 파이프라인이 PDF·DOCX를 직접
읽는 일은 없습니다. 그다음 에이전트에게 이 변환 요청문을 주세요:

```
`your-input/cv-original.md`를 읽고 `your-input/cv.md`를 만들어줘.

`your-input/cv.example.md`는 출력 구조를 참고하는 용도로만 쓰고,
가공 인물의 사실은 절대 가져오지 마.

결과 구조:

# CV summary (for fit scoring)
## Positioning
## Experience
## Strengths
## Domains I know well
## Skill calibration

회사·직책·날짜·수치·성과·skill은 `cv-original.md`가 뒷받침하는 그대로
보존해. 원문에 없는 경험을 지어내거나 부풀리거나 추론하지 마.

타겟 직무와 산업을 이해하려고 `preferences.md`를 읽는 건 되지만,
선호는 맥락일 뿐 경력 증거가 아니야.

Skill calibration은 CV의 명시적 증거에서 보수적으로 초안을 잡아줘:
- Core: 직접적이고 반복된 경험
- Transferable: 부분적이거나 인접한 경험, 정직한 상한과 함께
- Gap: CV에 안 적혀 있다는 이유만으로 Gap으로 추정하지 말 것

증거가 없거나 모호하면 추측하지 말고 그 주장은 빼.
내가 명시적으로 요청하지 않는 한 회사명을 익명화하지 마.
파일을 만든 뒤, 쓰기 전에 내가 검토해야 할 부분을 보여줘.
```

예약 작업을 등록하기 전에, 만들어진 `cv.md`를 검토하세요:

1. 회사명·직책·날짜·수치가 원본과 일치하는가
2. 새로운 경력이나 성과가 만들어지지 않았는가
3. `Strengths`와 `Domains I know well`이 실제 증거를 벗어나지 않는가
4. `Skill calibration`의 Core·Transferable·Gap 분류가 실제 경험 수준과
   맞는가

`cv-original.md`는 본인이 보관하는 비공개 파일이고, 매일 실행은 `cv.md`만
읽습니다. 자동으로 다시 만들어지지 않으니, 이력서가 바뀌면 변환을 다시
실행하고 새 `cv.md`를 검토하세요.

**이전 버전에서 업그레이드한다면?** 이미 잘 검토된 `cv.md`는 그대로
유효합니다 — `cv-original.md`를 새로 만들 필요 없습니다. 다만 설정 필드
2가지는 이름을 바꿔야 합니다: `config.md`의 `Schedule:`(cron)은 `Alert
schedule:` + `Alert timezone:`으로, `preferences.md`의 `Home timezone`과
`Remote anchor rule`은 `Work-hours timezone`과 `Maximum timezone
difference`로. `Alert timezone`이나 `Work-hours timezone`이 없으면 실행이
멈추고 어떤 필드가 문제인지 알려줍니다 (`Alert schedule`만 없는 경우는 수동
실행을 막지 않고, `Maximum timezone difference`가 없으면 기본 4시간으로
동작합니다).

**`cv.md`의 Skill calibration 섹션은 한 번 짚고 넘어갈게요.** 스킬마다 "내가
실제로 어디까지 해봤는지"를 한 줄씩 적어 두는 목록입니다. 왜 필요하냐면, AI
채점기는 CV에 SQL이라는 단어가 있기만 해도 SQL 실무자로 후하게 점수를 주는
경향이 있기 때문입니다. 예를 들어 `SQL: 대시보드 쿼리를 읽고 고치는 수준`이라고
적어 두면, 채점기는 데이터 엔지니어링을 요구하는 공고에서 그 이상으로는 점수를
못 줍니다. 이 목록이 정직할수록 매일 받는 점수가 내 실제 수준에 가까워집니다.

만드는 방법: 위의 변환 요청문이 CV에서 초안을 잡아 줍니다. 나온 초안을 한
줄씩 읽으면서 실제 내 수준에 맞게 본인이 고쳐서 확정하세요. 마지막 확인을
사람이 하는 데에는 이유가 있습니다 — 채점하는 AI가 자기 채점 기준까지 스스로
정하면, 견제 장치가 있으나 마나가 되기 때문입니다.

이 섹션은 비워 둬도 됩니다. 그래도 시스템은 정상으로 돌아갑니다. 채점기는 "CV에
적힌 내용까지만 인정하고, 적혀 있지 않은 능력은 없는 것으로 본다"는 기본
규칙만으로 채점합니다. 쓰다가 점수가 이상하다 싶은 날, 그때 한 줄씩 추가해도
됩니다.

자세한 설명과 양식: [`your-input/README.md`](your-input/README.md). 본인이 만든
`cv-original.md` / `cv.md` / `preferences.md` / `config.md`는 git 제외
목록(.gitignore)에 들어 있습니다(`*.example.md`만 공개). 개인 데이터가 GitHub에
올라갈 일은 없습니다.

## 4단계 — 에이전트에 파이프라인 넘기기

방법은 두 가지입니다. 본인 에이전트에 맞는 쪽을 고르세요. 어느 쪽이든 실제
실행 트리거는 `config.md`가 아니라 에이전트에 등록한 예약 작업입니다.
파이프라인이 `Alert schedule` 줄을 읽어 예약을 만들거나 바꾸는 일은
없습니다. 예약은 `config.md`의 Alert delivery 섹션과 맞춰서, cron 없이
자연어로 등록하면 됩니다 (예: "Asia/Seoul 기준 매일 오전 9시"). 여기서 두
가지가 따라옵니다: 나중에 `config.md`의 일정이나 시간대를 바꾸면 스케줄러의
예약도 같이 수정해야 하고(파일만 고치면 아무것도 안 바뀝니다), 등록한 뒤에는
표시되는 다음 실행 시각이 원하는 현지 시각과 맞는지 확인하세요.

**스킬 설치 — 에이전트가 Claude Code라면.** Claude Code 안에서 명령 두 줄을
실행합니다:

```
/plugin marketplace add Journey-512/catch-before-jobs-vanish
/plugin install catch-before-jobs-vanish@catch-before-jobs-vanish
```

그러면 파이프라인이 `job-alert` 스킬로 등록됩니다. 예약 작업은 한 줄이면
됩니다. "Asia/Seoul 기준으로 매일 오전 9시에 job-alert skill을 실행해줘. 내
repo는 `C:\Users\<본인>\catch-before-jobs-vanish`"처럼 일정·시간대와 본인
repo 경로를 함께 말해 주세요. 스킬은 그 경로로 `your-input/`을 찾습니다.

**복사해 붙여넣기 — Cowork 등 다른 에이전트라면.** (만든 사람도 매일 이
방식으로 돌립니다.)

1. [`skills/job-alert/SKILL.md`](skills/job-alert/SKILL.md)를 엽니다.
2. 프롬프트 블록을 복사합니다 (GitHub에서는 블록 모서리의 복사 버튼을 누르면
   한 번에 됩니다).
3. `your-input/`을 찾을 수 있도록, Step 0의 repo 경로 자리표시자 한 곳만 본인
   폴더 경로로 바꿉니다. **다른 부분은 손대지 마세요** — 나머지는 실행할 때
   읽어옵니다.
4. 그 프롬프트로 에이전트에 예약 작업을 만듭니다.

Cowork 모드에서는 그냥 *"이 프롬프트를 Asia/Seoul 기준 매일 오전 9시에
실행해줘"*라고 말하고 블록을 붙여넣어도 됩니다.

## 5단계 — 테스트 실행

작업을 한 번 수동으로 실행해 보거나 첫 예약 실행을 기다립니다. 확인할 것:

- `Jobs` 탭에 새 행이 들어오고 `Posted Date`가 절대 날짜(YYYY-MM-DD)로 적혀
  있다 — "Last 24h" 같은 상대 표현이 아니라.
- 링크가 검색 URL이 아니라 공고별 `jobs/view/{id}` 영구 링크다.
- 이메일이 **Top / Strong** 두 묶음으로 오고(컷 아래 공고는 없음), 자동 추가된
  회사가 거부 안내와 함께 표시되고, 시트 링크가 붙어 있다.
- JD 본문에서 발견된 지역 문제는 시트에 `Excluded (...)`로 남아 있지만 이메일엔
  **안** 들어왔다.
- 공고가 없는 조용한 날(또는 일부러 조건을 좁혀서) 한 번 더 돌려 보고
  **하트비트(heartbeat)**를 확인한다: 매칭이 0건이어도 "No new matches. System
  alive."라는 메일이 온다. 메일이 안 오면 결과가 없는 게 아니라 시스템이 죽었다
  — 이걸 구분하려고 있는 장치입니다.

## 6단계 — 조정 (피드백 루프 두 개 — 섞지 마세요)

1. **Calibration loop — 채점이 맞나?** 점수가 이상한 날 → 그 행의 Fit Reason을
   읽고 → `cv.md`의 Skill calibration에 **한 줄**을 추가하거나 고칩니다 → 다음
   실행부터 반영됩니다. 프롬프트와 기준표는 그대로 두고 내 데이터만 다듬는
   방식입니다.
2. **Outcome loop — 점수가 예측을 하나?** 지원할 때마다 Status 열을 채워
   두세요. 딥스캔 날 리포트가 점수 구간별 서류 통과율(cold와 referral은 따로)을
   정리해 줍니다. preferences나 calibration을 고칠 땐 건수가 충분히 쌓인
   패턴인지부터 확인하세요 — 두세 건으로 성급하게 바꾸면 오히려 나빠집니다.

다음 설정은 이유가 생기면 언제든 바꾸면 됩니다:

- 이메일이 너무 많거나 적다 → `preferences.md`의 컷 (70은 기본값이지 정답이
  아닙니다).
- 엉뚱한 직무가 섞인다 → 직무명 허용/제외 목록.
- 회사 목록이 너무 빨리 (또는 너무 안) 는다 → `config.md`의 Company
  discovery 설정.

## 문제 해결

| 증상                             | 원인 추정                                             | 해결                                                                                                |
| ------------------------------ | ------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| 저장한 링크드인 링크가 오후에 죽어 있음         | 영구 링크 대신 검색 URL을 저장함                              | 에이전트가 검색 URL이 아니라 공고별 `jobs/view/{id}` 영구 링크를 추출하는지 확인                                            |
| `Posted Date`에 "Last 24h"라고 적힘 | 상대 표현을 날짜로 바꾸지 않고 그대로 저장함                         | 프롬프트 2단계가 캡처 시점에 YYYY-MM-DD로 변환하는지 확인                                                             |
| 엉뚱한 회사의 공고가 저장됨                | 링크드인이 자동 선택한 `currentJobId`를 그대로 믿음               | 결과 목록 전체를 읽어 회사+직무명으로 맞는 공고를 찾도록 확인                                                               |
| 같은 공고가 자꾸 다시 들어옴               | 중복 제거가 과거 기록을 못 봤거나(시트 읽기 실패), 채용 페이지 URL 표기가 제각각 | 이메일에 "과거 대조 못 함" 공지가 있는지 확인, 채용 사이트별로 URL이 한 가지 형식으로 통일되는지 확인                                     |
| `Companies` 탭의 회사 공고가 안 잡힘        | 채용 페이지가 개편됐거나, 깨진 회사 필터가 조용히 빈 결과를 돌려줌            | 채용 페이지는 원래 예고 없이 바뀝니다 — 잡보드로 대체 검색하고 공백을 이메일에 알리는 게 정상 동작. 빈 검색 결과는 키워드 검색으로 한 번 더 확인해야 믿을 수 있습니다 |
| Aiming·회사별·딥스캔 검색이 통째로 건너뛰어짐     | `Companies` 탭을 읽지 못했거나 헤더가 9열 계약과 달라짐              | 이메일에 건너뛴 범위와 사유가 표시됩니다. 2단계의 9열 헤더로 복구하고 시트 ID·공유 설정을 확인한 뒤 다시 실행하세요                                |
| 이메일이 아예 안 옴                    | 실행이 죽었거나 발송에 실패함                                  | 하트비트 규칙 덕에 침묵 = 고장입니다: 예약 작업이 실행됐는지부터 보고, 그 다음 Gmail 초안함을 확인하세요 — 발송에 실패하면 초안이 남아 있습니다            |
