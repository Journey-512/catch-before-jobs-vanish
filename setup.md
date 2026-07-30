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

Make a new Google Sheet with **two tabs**. The Sheet only stores records —
all rules live in the prompt template and `your-input/`, so there is nothing
else to configure here.

**`Jobs`** — 10 columns:

| Date Added | Company | Job Title | Location | Source | Posted Date | Link | Status | Fit Score | Fit Reason |
|---|---|---|---|---|---|---|---|---|---|

The `Status` column is shared: the system writes `Excluded (...)` / `Closed
(date)`, and **you** write your application outcomes (`Applied`, `Passed - CV`,
`Rejected - CV`, `Lost` — add a `(referral)` suffix for referral applications).
Those hand-written statuses feed the outcome loop; the system never overwrites
them.

**`Companies`** — your watchlist, 9 columns:

| Index | Company | Aiming | Match Level | URL | Source site | HQ | Topics | Memo |
|---|---|---|---|---|---|---|---|---|

`Aiming` is a manual flag for the 1-3 companies you're actively gunning for.
An Aiming company gets a wider search every run: a 7-day window instead of
24h, your extended locations, and a direct careers-page check. Set it by hand
only — auto-added rows always leave it blank.

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

The fastest way: the folder ships four filled-in **example files** for a
made-up person (a fictional 3-year travel & mobility PM). Copy each, drop the
`.example` from the name, and edit it with your details:

```bash
cd your-input
cp cv.example.md cv.md
cp preferences.example.md preferences.md
cp companies.example.md companies.md
cp config.example.md config.md
cd ..
```

(Windows PowerShell: `Copy-Item cv.example.md cv.md`, etc.)

What each file is for:

- `cv.md` — your career evidence for scoring, plus a **Skill calibration**
  section
- `preferences.md` — titles, locations, industries, recency, cutoff, home
  timezone, output language, plus an **Eligibility** section (knockouts)
- `companies.md` — target-company whitelist with **Aiming** flags + auto-add
  rule
- `config.md` — Sheet ID, email recipient, subject, schedule, deep-scan
  weekday (one day a week the run sweeps your whole watchlist, not just the
  fresh postings), per-tier cap

**About the Skill calibration section in `cv.md` — read this once.** It's a
list where each line states how far you've *actually* gone with a skill. It
exists because AI scorers over-credit by default: the word "SQL" anywhere in
a CV tends to be read as "SQL practitioner". Write `SQL: I read and tweak
dashboard queries` and the scorer can no longer give you full credit on a
hands-on data-engineering requirement. The more honest this list, the closer
your daily scores track reality.

How to build it: ask your agent to *draft* the section from your CV, then go
line by line and correct it yourself before keeping it. The final say stays
with you on purpose — if the same AI that scores you also sets its own
limits, the check means nothing.

You can also leave it empty: the system still runs, the scorer just falls
back to one default rule — only what's written in the CV counts, nothing gets
assumed on top. Add a line whenever a score looks off.

Details and skeletons: [`your-input/README.md`](your-input/README.md). Your
real `cv.md` / `preferences.md` / `companies.md` / `config.md` are on git's
ignore list (only the `*.example.md` versions are public), so your personal
data never gets pushed.

## Step 4 — Hand the pipeline to your agent

Two routes to the same place — pick the one that matches your agent. Either
way, the schedule itself lives in your agent and should match `config.md`:
the default `0 9 * * *` is cron notation for "every day at 9:00 AM" — use
your own timezone and preferred hour.

**Skill install — if your agent is Claude Code.** Run these two commands
inside Claude Code:

```
/plugin marketplace add Journey-512/catch-before-jobs-vanish
/plugin install catch-before-jobs-vanish@catch-before-jobs-vanish
```

That registers the pipeline as the `job-alert` skill. Then create your
scheduled task with a one-line prompt — ask it to run the `job-alert` skill
and say where your clone is, e.g. *"Run the job-alert skill; my repo is at
`C:\Users\me\catch-before-jobs-vanish`"*. The path is how the skill finds
your `your-input/`.

**Copy-paste — Cowork and any other agent** (the route the author's own
daily runs use).

1. Open [`skills/job-alert/SKILL.md`](skills/job-alert/SKILL.md).
2. Copy the prompt block (on GitHub, the copy button in the block's corner
   grabs the whole thing).
3. Replace the one repo-path placeholder in Step 0 with your local clone path
   so it can find `your-input/`. **Make no other edits** — everything else is
   read at runtime.
4. Create a scheduled task in your agent with that prompt.

In Cowork mode you can just say: *"Run this prompt every day at 9 AM"* and
paste the block.

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
- Watchlist growing too fast or slow → the auto-add gate in `companies.md`.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Saved LinkedIn links go dead by afternoon | A search URL got saved instead of a permalink | Confirm the agent extracts per-posting `jobs/view/{id}` permalinks, not the search URL |
| `Posted Date` says "Last 24h" | Relative label stored instead of parsed | Ensure Step 2 of the prompt parses to YYYY-MM-DD at capture |
| Wrong company's job recovered | Trusted LinkedIn's auto-selected `currentJobId` | Parse the full result list and match by company + title |
| The same posting keeps reappearing | Dedup can't see history (Sheet unreadable) or careers URLs not canonicalized | Check the email for the "history not checked" notice; verify one canonical URL format per careers site |
| A whitelisted company never shows results | Company career page changed, or a broken company filter silently returns empty | Expected for careers pages (they change without notice) — the run falls back to job boards and discloses the gap; never trust an empty filter result without a keyword cross-check |
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

**탭 2개**짜리 새 구글 시트를 만듭니다. 시트에는 기록만 쌓입니다. 규칙은 전부
프롬프트 template과 `your-input/`에 있어서 시트 쪽에는 더 설정할 것이 없습니다.

**`Jobs`** — 10열:

| Date Added | Company | Job Title | Location | Source | Posted Date | Link | Status | Fit Score | Fit Reason |
|---|---|---|---|---|---|---|---|---|---|

`Status` 열은 시스템과 내가 같이 씁니다. 시스템은 `Excluded (...)` / `Closed
(날짜)`를 적고 **나는** 지원 결과를 적습니다 (`Applied`, `Passed - CV`,
`Rejected - CV`, `Lost` — referral로 지원했다면 `(referral)`을 뒤에 붙입니다).
내가 적은 값은 나중에 점수 검증 자료가 됩니다. 시스템이 덮어쓰는 일은 없습니다.

**`Companies`** — 관심 회사 목록, 9열:

| Index | Company | Aiming | Match Level | URL | Source site | HQ | Topics | Memo |
|---|---|---|---|---|---|---|---|---|

`Aiming`은 지금 집중해서 노리는 회사 1-3곳에 직접 적어 두는 표시입니다. Aiming
표시가 있는 회사는 매일 실행 때 검색 범위가 넓어집니다: 최근 24시간 대신 7일치를
보고, `preferences.md`에 적어 둔 확장 지역까지 찾고, 그 회사 채용 페이지도 직접
확인합니다. 이 표시는 반드시 직접 적으세요 — 시스템이 자동으로 추가한 회사에는
절대 붙지 않습니다.

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

가장 빠른 방법: 이 폴더에는 가공 인물(창작한 여행·모빌리티 3년차 PM)로 **미리
채워 둔 예시 파일** 4개가 있습니다. 각 파일을 복사해 이름에서 `.example`만 빼고
본인 내용으로 고치세요:

```bash
cd your-input
cp cv.example.md cv.md
cp preferences.example.md preferences.md
cp companies.example.md companies.md
cp config.example.md config.md
cd ..
```

(윈도우 PowerShell에서는 `Copy-Item cv.example.md cv.md`처럼 쓰면 됩니다.)

각 파일의 용도:

- `cv.md` — 채점에 쓸 경력 정리 + **Skill calibration** 섹션
- `preferences.md` — 직무명, 지역, 산업, 최신성, 이메일 컷, 홈 타임존, 출력
  언어 + **Eligibility** 섹션 (지원이 무의미해지는 조건)
- `companies.md` — 관심 회사 목록 (**Aiming** 표시 포함) + 자동 추가 규칙
- `config.md` — 시트 ID, 이메일 수신자, 제목, 실행 일정, 딥스캔 요일
  (일주일에 하루, 새 공고만이 아니라 관심 회사 전체를 훑는 날), tier당
  회사 수 제한

**`cv.md`의 Skill calibration 섹션은 한 번 짚고 넘어갈게요.** 스킬마다 "내가
실제로 어디까지 해봤는지"를 한 줄씩 적어 두는 목록입니다. 왜 필요하냐면, AI
채점기는 CV에 SQL이라는 단어가 있기만 해도 SQL 실무자로 후하게 점수를 주는
경향이 있기 때문입니다. 예를 들어 `SQL: 대시보드 쿼리를 읽고 고치는 수준`이라고
적어 두면, 채점기는 데이터 엔지니어링을 요구하는 공고에서 그 이상으로는 점수를
못 줍니다. 이 목록이 정직할수록 매일 받는 점수가 내 실제 수준에 가까워집니다.

만드는 방법: 에이전트에게 "내 CV를 보고 Skill calibration 초안을 만들어 줘"라고
시킨 다음, 나온 초안을 한 줄씩 읽으면서 실제 내 수준에 맞게 본인이 고쳐서
확정하세요. 마지막 확인을 사람이 하는 데에는 이유가 있습니다 — 채점하는 AI가
자기 채점 기준까지 스스로 정하면, 견제 장치가 있으나 마나가 되기 때문입니다.

이 섹션은 비워 둬도 됩니다. 그래도 시스템은 정상으로 돌아갑니다. 채점기는 "CV에
적힌 내용까지만 인정하고, 적혀 있지 않은 능력은 없는 것으로 본다"는 기본
규칙만으로 채점합니다. 쓰다가 점수가 이상하다 싶은 날, 그때 한 줄씩 추가해도
됩니다.

자세한 설명과 양식: [`your-input/README.md`](your-input/README.md). 본인이 만든
`cv.md` / `preferences.md` / `companies.md` / `config.md`는 git 제외
목록(.gitignore)에 들어 있습니다(`*.example.md`만 공개). 개인 데이터가 GitHub에
올라갈 일은 없습니다.

## 4단계 — 에이전트에 파이프라인 넘기기

방법은 두 가지입니다. 본인 에이전트에 맞는 쪽을 고르세요. 어느 쪽이든 실행
일정은 에이전트에서 정하고 `config.md`와 맞춥니다. 기본값 `0 9 * * *`는
"매일 오전 9시"라는 뜻의 cron 표기입니다. 본인 시간대와 원하는 시각으로
바꾸면 됩니다.

**스킬 설치 — 에이전트가 Claude Code라면.** Claude Code 안에서 명령 두 줄을
실행합니다:

```
/plugin marketplace add Journey-512/catch-before-jobs-vanish
/plugin install catch-before-jobs-vanish@catch-before-jobs-vanish
```

그러면 파이프라인이 `job-alert` 스킬로 등록됩니다. 예약 작업은 한 줄이면
됩니다. "job-alert 스킬을 실행해줘. 내 repo는
`C:\Users\<본인>\catch-before-jobs-vanish`"처럼 스킬 실행과 본인 repo 경로를
함께 말해 주세요. 스킬은 그 경로로 `your-input/`을 찾습니다.

**복사해 붙여넣기 — Cowork 등 다른 에이전트라면.** (만든 사람도 매일 이
방식으로 돌립니다.)

1. [`skills/job-alert/SKILL.md`](skills/job-alert/SKILL.md)를 엽니다.
2. 프롬프트 블록을 복사합니다 (GitHub에서는 블록 모서리의 복사 버튼을 누르면
   한 번에 됩니다).
3. `your-input/`을 찾을 수 있도록, Step 0의 repo 경로 자리표시자 한 곳만 본인
   폴더 경로로 바꿉니다. **다른 부분은 손대지 마세요** — 나머지는 실행할 때
   읽어옵니다.
4. 그 프롬프트로 에이전트에 예약 작업을 만듭니다.

Cowork 모드에서는 그냥 *"이 프롬프트를 매일 오전 9시에 실행해줘"*라고 말하고
블록을 붙여넣어도 됩니다.

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
- 회사 목록이 너무 빨리 (또는 너무 안) 는다 → `companies.md`의 자동 추가 규칙.

## 문제 해결

| 증상                             | 원인 추정                                             | 해결                                                                                                |
| ------------------------------ | ------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| 저장한 링크드인 링크가 오후에 죽어 있음         | 영구 링크 대신 검색 URL을 저장함                              | 에이전트가 검색 URL이 아니라 공고별 `jobs/view/{id}` 영구 링크를 추출하는지 확인                                            |
| `Posted Date`에 "Last 24h"라고 적힘 | 상대 표현을 날짜로 바꾸지 않고 그대로 저장함                         | 프롬프트 2단계가 캡처 시점에 YYYY-MM-DD로 변환하는지 확인                                                             |
| 엉뚱한 회사의 공고가 저장됨                | 링크드인이 자동 선택한 `currentJobId`를 그대로 믿음               | 결과 목록 전체를 읽어 회사+직무명으로 맞는 공고를 찾도록 확인                                                               |
| 같은 공고가 자꾸 다시 들어옴               | 중복 제거가 과거 기록을 못 봤거나(시트 읽기 실패), 채용 페이지 URL 표기가 제각각 | 이메일에 "과거 대조 못 함" 공지가 있는지 확인, 채용 사이트별로 URL이 한 가지 형식으로 통일되는지 확인                                     |
| 관심 회사 공고가 안 잡힘                 | 채용 페이지가 개편됐거나, 깨진 회사 필터가 조용히 빈 결과를 돌려줌            | 채용 페이지는 원래 예고 없이 바뀝니다 — 잡보드로 대체 검색하고 공백을 이메일에 알리는 게 정상 동작. 빈 검색 결과는 키워드 검색으로 한 번 더 확인해야 믿을 수 있습니다 |
| 이메일이 아예 안 옴                    | 실행이 죽었거나 발송에 실패함                                  | 하트비트 규칙 덕에 침묵 = 고장입니다: 예약 작업이 실행됐는지부터 보고, 그 다음 Gmail 초안함을 확인하세요 — 발송에 실패하면 초안이 남아 있습니다            |
