# Setup

**Language:** English (below) · [한국어](#설정-한국어)

A step-by-step guide to running `catch-before-jobs-vanish` for yourself. Plan on
about 30 minutes the first time.

## Prerequisites

- An AI agent with **Chrome** (web browsing), **Google Drive** (your Sheet), and
  **Gmail** (sending alerts) connected, plus the ability to run a daily scheduled
  task. Built and tested with **Claude in Cowork mode**; any agent with the same
  connectors should work.
- A Google account (for the Sheet) and an email account to receive alerts.

## Step 1 — Fork and clone

```bash
git clone https://github.com/<your-username>/catch-before-jobs-vanish.git
cd catch-before-jobs-vanish
```

## Step 2 — Create the Google Sheet

Make a new Google Sheet with three tabs:

**`Jobs`** — columns:

| Date Added | Company | Job Title | Location | Source | Posted Date | Link | Status | Fit Score | Fit Reason |
|---|---|---|---|---|---|---|---|---|---|

**`Companies`** — your watchlist:

| Index | Name | URL | From which sites | HQ | Topics | Match Level | Memo |
|---|---|---|---|---|---|---|---|

**`PRD`** — a free-form tab where you keep the canonical rules (titles, filters,
thresholds). The agent re-reads this each run, so changing a rule is just editing a
cell. This is your live override on top of `your-input/`.

Copy the Sheet ID from its URL (the long string between `/d/` and `/edit`).

## Step 3 — Fill in `your-input/`

The fastest way: the folder ships four filled-in **example files** for a made-up
person. Copy each, drop the `.example` from the name, and edit it with your details:

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

- `cv.md` — your career summary for scoring
- `preferences.md` — titles, locations, industries, recency, threshold
- `companies.md` — target-company whitelist + auto-add rule
- `config.md` — Sheet ID, email recipient, subject, schedule time, repo path

Details and more examples: [`your-input/README.md`](your-input/README.md). Your
`*.md` files are gitignored (only the `*.example.md` versions are public), so your
real data never gets pushed. Put your real data in the un-`.example` files.

## Step 4 — Register the scheduled task

1. Open [`scheduler-prompt-template.md`](scheduler-prompt-template.md).
2. Copy the prompt block.
3. Replace the one repo-path placeholder on the first line with your local clone
   path so it can find `your-input/`. **Make no other edits** — everything else is
   read at runtime.
4. Create a scheduled task in your agent with that prompt and a cron schedule
   matching `config.md` (default `0 9 * * *` — daily at **9:00 AM CEST** in the
   example; change it to your own timezone).

In Cowork mode you can just say: *"Run this prompt every day at 9 AM"* and paste the
block.

## Step 5 — Test run

Trigger the task once manually (or wait for the first scheduled fire). Check that:

- The `Jobs` tab gets new rows with absolute `Posted Date` values (YYYY-MM-DD, never
  "Last 24h").
- Links are `jobs/view/{id}` permalinks, not search URLs.
- You receive an email grouped into Top / Strong / Maybe / New Companies.
- Any non-acceptable-location roles are in the Sheet as `Excluded` but **not** in the
  email.

## Step 6 — Tune

After a few days you'll see a real score distribution. Adjust:

- The email threshold in `preferences.md` (or the PRD tab) if you get too many / too
  few.
- The title allow/exclude lists if noise creeps in.
- The auto-add gate in `companies.md` if the watchlist grows too fast or too slow.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Saved LinkedIn links go dead by afternoon | A search URL got saved instead of a permalink | Confirm the agent extracts per-posting `jobs/view/{id}` permalinks, not the search URL |
| A pasted cell shows unrelated text | Clipboard was set on the wrong origin | Have the agent write the clipboard on the destination page (the Sheet) right before pasting |
| `Posted Date` says "Last 24h" | Relative label stored instead of parsed | Ensure Step 2 of the prompt parses to YYYY-MM-DD at capture |
| Wrong company's job recovered | Trusted LinkedIn's auto-selected `currentJobId` | Parse the full result list and match by company + title |
| Company career page check fails | Career pages are fragile (ATS migrations, JS-only) | Expected — it's a Tier-2 fallback; LinkedIn + Indeed cover most postings |

---

<a name="설정-한국어"></a>

# 설정 (한국어)

**언어:** [English](#setup) · 한국어

`catch-before-jobs-vanish`를 직접 돌리기 위한 단계별 안내입니다. 처음엔 30분 정도 잡으세요.

## 사전 준비

- **Chrome**(웹 탐색), **Google Drive**(구글 시트), **Gmail**(알림 발송)을 연결하고,
  매일 예약 작업을 돌릴 수 있는 AI 에이전트. **Claude Cowork 모드**로 제작·테스트했으며,
  같은 커넥터를 갖춘 에이전트면 됩니다.
- 구글 계정(시트용)과 알림 받을 이메일 계정.

## 1단계 — 포크 후 클론

```bash
git clone https://github.com/<본인-아이디>/catch-before-jobs-vanish.git
cd catch-before-jobs-vanish
```

## 2단계 — 구글 시트 만들기

탭 3개짜리 새 구글 시트를 만듭니다.

**`Jobs`** — 컬럼:

| Date Added | Company | Job Title | Location | Source | Posted Date | Link | Status | Fit Score | Fit Reason |
|---|---|---|---|---|---|---|---|---|---|

**`Companies`** — 관심 회사 목록:

| Index | Name | URL | From which sites | HQ | Topics | Match Level | Memo |
|---|---|---|---|---|---|---|---|

**`PRD`** — 핵심 규칙(직무명, 필터, 점수 기준)을 적어두는 자유 형식 탭. 에이전트가 매번
다시 읽으므로, 규칙을 바꾸려면 셀만 고치면 됩니다. `your-input/` 위에 얹는 실시간 덮어쓰기
역할입니다.

시트 URL에서 `/d/`와 `/edit` 사이의 긴 문자열이 시트 ID입니다. 복사해 두세요.

## 3단계 — `your-input/` 채우기

가장 빠른 방법: 이 폴더엔 가공 인물로 **이미 채워진 예시 파일** 4개가 있습니다. 각 파일을
복사해 이름에서 `.example`만 빼고 본인 내용으로 고치세요:

```bash
cd your-input
cp cv.example.md cv.md
cp preferences.example.md preferences.md
cp companies.example.md companies.md
cp config.example.md config.md
cd ..
```

(윈도우 PowerShell: `Copy-Item cv.example.md cv.md` 처럼)

각 파일의 용도:

- `cv.md` — 점수 매기기에 쓸 경력 요약
- `preferences.md` — 직무명, 지역, 산업, 최신성, 점수 기준선
- `companies.md` — 목표 회사 화이트리스트 + 자동 추가 규칙
- `config.md` — 시트 ID, 이메일 수신자, 제목, 실행 시각, 레포 경로

자세한 설명과 예시: [`your-input/README.md`](your-input/README.md). 본인 `*.md` 파일은
gitignore 처리되어(`*.example.md`만 공개됨) 실제 데이터는 절대 푸시되지 않습니다. `.example`이
없는 파일에 실제 데이터를 넣으세요.

## 4단계 — 예약 작업 등록

1. [`scheduler-prompt-template.md`](scheduler-prompt-template.md)를 엽니다.
2. 프롬프트 블록을 복사합니다.
3. 첫 줄의 레포 경로 자리표시자 한 곳을, `your-input/`을 찾을 수 있도록 본인 클론 경로로
   바꿉니다. **다른 부분은 손대지 마세요** — 나머지는 실행할 때 읽어옵니다.
4. 그 프롬프트와 `config.md`에 맞는 cron 일정(기본 `0 9 * * *` — 예시 기준 매일
   **오전 9시 CEST**, 본인 시간대로 바꾸세요)으로 예약 작업을 만듭니다.

Cowork 모드에선 그냥 *"이 프롬프트를 매일 오전 9시에 실행해줘"*라고 말하고 블록을 붙여넣어도
됩니다.

## 5단계 — 테스트 실행

작업을 한 번 수동으로 돌려보거나 첫 예약 실행을 기다립니다. 확인할 것:

- `Jobs` 탭에 새 행이 들어오고 `Posted Date`가 절대 날짜(YYYY-MM-DD)다 — "Last 24h" 아님.
- 링크가 검색 URL이 아니라 `jobs/view/{id}` 영구 링크다.
- Top / Strong / Maybe / New Companies로 묶인 이메일을 받았다.
- 시간대가 안 맞는 직무는 시트엔 `Excluded`로 남되 이메일엔 **안** 들어왔다.

## 6단계 — 조정

며칠 지나면 실제 점수 분포가 보입니다. 조정하세요:

- 너무 많거나 적으면 `preferences.md`(또는 PRD 탭)의 이메일 기준선.
- 잡음이 섞이면 직무명 허용/제외 목록.
- 목록이 너무 빨리/느리게 늘면 `companies.md`의 자동 추가 게이트.

## 문제 해결

| 증상 | 원인 추정 | 해결 |
|---|---|---|
| 저장한 링크드인 링크가 오후엔 죽음 | 영구 링크 대신 검색 URL을 저장함 | 에이전트가 검색 URL이 아니라 공고별 `jobs/view/{id}` 영구 링크를 추출하는지 확인 |
| 붙여넣은 셀에 엉뚱한 텍스트 | 클립보드를 잘못된 출처에서 설정함 | 붙여넣기 직전에 목적지 페이지(시트)에서 클립보드를 쓰도록 함 |
| `Posted Date`에 "Last 24h" | 상대 표현을 파싱 안 하고 그대로 저장 | 프롬프트 2단계가 캡처 시점에 YYYY-MM-DD로 변환하는지 확인 |
| 엉뚱한 회사 공고가 복구됨 | 링크드인 자동선택 `currentJobId`를 믿음 | 결과 목록 전체를 파싱해 회사+직무명으로 매칭 |
| 회사 채용 페이지 확인 실패 | 채용 페이지는 취약함 (ATS 이전, JS 전용) | 정상 — 2차 보조 수단이며 링크드인+인디드가 대부분 커버 |
