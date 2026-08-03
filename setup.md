# Setup

**Language:** English · [한국어](#설정-한국어)

A step-by-step guide to running `catch-before-jobs-vanish` for yourself. Set
it up once in about 10 minutes; after that, a scheduled AI agent runs it
automatically every day with no manual work.

## Prerequisites

This is not an installed application. It is a **work order that an AI agent
runs every day**. Prepare these three things:

- **An AI agent that can run scheduled tasks.** This system was built and
  tested with **Claude in Cowork mode**, which includes scheduling.
- **Three Claude connectors.** Enable them in Claude's settings:
  - **Chrome** — checks LinkedIn, Indeed, and company careers pages.
  - **Google Drive** — reads and writes the Google Sheet that stores results.
  - **Gmail** — sends the results email.
- **A Google account and an email address for alerts.** Create the Sheet in
  the account connected through the Google Drive connector.

> **Why Chrome?** The browser-based route is deliberate: non-developers can
> start without configuring API keys or authentication code. If you have
> development experience and official API access to the job sites, you can
> replace the collection stage with an API-based route. This repository does
> not include API integration code, so preserve the same link, date, and
> dedup contracts.

You can use another AI agent as long as it can browse the web, read and write
a Google Sheet, send email, and run on a daily schedule.

## Step 1 — Get the repo

Run these commands in a **terminal**, not in a repository file.

- On Windows, open **PowerShell** or **Windows Terminal** from the Start menu.
- On macOS or Linux, open **Terminal**.

Paste the following lines one at a time and press Enter:

```bash
git clone https://github.com/Journey-512/catch-before-jobs-vanish.git
cd catch-before-jobs-vanish
```

If the `git` command is unavailable or Git is unfamiliar, select **Code →
Download ZIP** on the GitHub page and extract it. GitHub normally names the
extracted folder `catch-before-jobs-vanish-main`.

Before continuing, move the terminal into that extracted folder. Type `cd `
(including the space), drag the extracted folder from your file manager (such
as File Explorer or Finder) into the terminal, and press Enter. This is
equivalent to:

```text
cd "<path-to-your-extracted-catch-before-jobs-vanish-main-folder>"
```

Check that you landed in the right folder: run `dir` (Windows) or `ls`
(macOS) and confirm `setup.md` and a `your-input` folder are listed. If they
are not, you are one level too high — Windows Explorer often places the repo
folder inside another folder of the same name, so open that folder and drag
the inner one in instead. Keep that terminal open; the `cd your-input`
command in Step 3 starts from this folder.

## Step 2 — Create the Google Sheet

Create a new Google Sheet and name its tabs exactly **`Jobs`** and
**`Companies`**. The pipeline finds the tabs by these names.

### `Jobs` tab

Enter these 10 columns in the first row, in this order:

| Date Added | Company | Job Title | Location | Source | Posted Date | Link | Status | Fit Score | Fit Reason |
|---|---|---|---|---|---|---|---|---|---|

The system writes `Excluded (...)` or `Closed (date)` in the `Status`
column. As your application progresses, enter `Applied`, `Passed - CV`,
`Rejected - CV`, or `Lost`. For a referral application, append
`(referral)`, as in `Applied (referral)`. The system never overwrites a
value you entered.

### `Companies` tab

Enter these 9 columns in the first row, in this order:

| Index | Company | Aiming | Match Level | URL | Source site | HQ | Topics | Memo |
|---|---|---|---|---|---|---|---|---|
| 1 | `<Company name>` | `<Aiming or blank>` | `<Strong or Soft>` | `<careers URL or blank>` | `<source or blank>` | `<HQ or blank>` | `<topics or blank>` | `<note or blank>` |

The company list is filled in two ways:

1. **Manual entry:** Before the first run, enter one target company per row.
   For the 1-3 companies you are actively targeting, enter `Aiming` in the
   `Aiming` column; leave it blank for the others. See
   [`your-input/companies.example.md`](your-input/companies.example.md) for
   the exact format and column definitions.
2. **Automatic addition:** If auto-add is enabled under `Company discovery`
   in `config.md`, the pipeline appends newly discovered companies that meet
   your criteria. An auto-added row keeps `Aiming` blank and records only
   confirmed values. The company joins company-scoped searches from the next
   run.

The `Companies` tab is the single source of truth for the company list.
There is no local company-list file, and the pipeline never overwrites values
you entered in an existing row.

**Upgrading from an earlier version?** Your old `your-input/companies.md` is
no longer read, and the system does not delete it. Move its rows into this
tab: Name → Company, careers URL → URL, Match level → Match Level, Aiming →
Aiming, note → Memo. Compare the tab with the original file, then decide
whether to keep or delete that file.

The Sheet can live in any Google Drive folder and use any file name. The
system finds it by **Sheet ID**. Open the Sheet and check the address bar:

```
https://docs.google.com/spreadsheets/d/THIS-LONG-STRING-IS-THE-ID/edit
```

Copy the string between `/d/` and `/edit`. In the next step, paste it into
`Google Sheet ID:` in `your-input/config.md`.

## Step 3 — Fill in `your-input/`

Copy the two template files and replace the `< >` placeholders with your
own values.

macOS or Linux:

```bash
cd your-input
cp preferences.template.md preferences.md
cp config.template.md config.md
cd ..
```

Windows PowerShell:

```powershell
cd your-input
Copy-Item preferences.template.md preferences.md
Copy-Item config.template.md config.md
cd ..
```

The [`your-input` guide](your-input/README.md) contains the one-time CV
conversion request and the field explanations.

Continue after `cv.md`, `preferences.md`, and `config.md` are ready.
If you are upgrading, follow the [3.2 upgrade note](CHANGELOG.md#upgrade-note).

## Step 4 — Hand the pipeline to your agent

Choose the route that matches your agent. The actual trigger is the
**scheduled task registered in your agent**, not `config.md`.

Register the schedule and timezone in plain language to match `Alert
delivery` in `config.md`, for example: `daily at 09:00 in Asia/Seoul`.
If you later change the schedule or timezone in `config.md`, update the
scheduled task separately. After registration, confirm that the next run
time matches the local time you intended.

### Install the skill in Claude Code

Run these two commands inside Claude Code:

```
/plugin marketplace add Journey-512/catch-before-jobs-vanish
/plugin install catch-before-jobs-vanish@catch-before-jobs-vanish
```

After installation, request a scheduled task that includes the schedule,
timezone, and local repository path:

> Run the `job-alert` skill daily at 09:00 in Asia/Seoul. My repository is
> at `C:\Users\me\catch-before-jobs-vanish`.

Replace the example path with the folder where you downloaded the repository.

### Register the prompt in Cowork or another agent

1. Open [`skills/job-alert/SKILL.md`](skills/job-alert/SKILL.md).
2. Copy the entire prompt block.
3. Replace the repository-path placeholder in `Step 0` with your local path.
4. Register it as a scheduled task **without changing anything else**.

In Cowork, paste the prompt and say: `Run this prompt daily at 09:00 in
Asia/Seoul.`

## Step 5 — Test run

Trigger the task once manually or wait for the first scheduled run, then
check the following:

1. The `Jobs` tab has a new row and `Date Added` uses
   `YYYY-MM-DD HH:MM` in the Alert timezone.
2. `Posted Date` is an absolute `YYYY-MM-DD` date and carries an
   `estimated` or `unknown` label when needed. It must never store a relative
   value such as `Last 24h`.
3. Each link points to one specific posting, not a search results page.
   LinkedIn rows use the `jobs/view/{id}` permalink form; Indeed rows and
   careers-page rows carry their own posting URLs.
4. The email includes **Top / Strong** postings, a prompt to review any
   auto-added companies, and a link to the Sheet.
5. Location violations found in the full job description are recorded as
   `Excluded (...)` in the Sheet and excluded from the email.
6. Zero matches still sends the heartbeat message
   `No new matches. System alive.` No email at all means the run or the send
   failed.

## Step 6 — Tune (two feedback loops — don't mix them)

1. **Calibrate the score:** If a score looks wrong, check that row's
   `Fit Reason` and adjust one line in `Skill calibration` in `cv.md`.
2. **Validate against outcomes:** Keep `Status` current as you apply. In the
   weekly report, review CV-pass rates by score band, with cold and referral
   applications kept separate.

Adjust email volume with `Cutoff` in `preferences.md`, title accuracy with
the allow and exclude lists, and company auto-add scope with `Company
discovery` in `config.md`. For scoring details, see the
[fit-scoring rubric](skills/job-alert/fit-scoring-rubric.md).

## Troubleshooting

### A LinkedIn link stops working after a few hours

A search URL may have been saved. Confirm that the agent extracts the
posting-specific `jobs/view/{id}` permalink.

### `Posted Date` contains `Last 24h`

The relative date was stored without conversion. Confirm that Step 2 of the
prompt converts it to `YYYY-MM-DD` at capture time.

### A posting from the wrong company was saved

The run may have trusted LinkedIn's auto-selected `currentJobId`. Confirm
that it parses the full result list and matches both company and title.

### The same posting keeps reappearing

The run may have failed to read Sheet history, or the same careers URL may
use multiple formats. Check the email for a notice that history could not be
checked, then standardize each careers site to one URL format.

### A company in the `Companies` tab returns no postings

The careers page or company filter may have changed. Check the email to
confirm that the pipeline fell back to job boards, and cross-check an empty
result with a keyword search.

### All `Aiming`, company-scoped, and weekly deep-scan searches were skipped

The `Companies` tab may be unreadable or its header may differ from the
required 9-column structure. Restore the Step 2 header, verify the Sheet ID
and sharing, then rerun. The generic LinkedIn and Indeed searches driven by
`preferences.md` still run in this case.

### No email arrives

First confirm that the scheduled task ran, then check Gmail Drafts. A failed
send may leave a draft behind.

---

<a name="설정-한국어"></a>

# 설정 (한국어)

**언어:** [English](#setup) · 한국어

`catch-before-jobs-vanish`를 직접 실행하기 위한 단계별 안내입니다. 처음 한
번만 약 10분 정도 설정하면, 이후에는 별도의 수동 작업 없이 예약된 AI
에이전트가 매일 자동으로 실행합니다.

## 사전 준비

이 시스템은 설치형 프로그램이 아니라 **AI 에이전트가 매일 실행하는 작업
지시문**입니다. 다음 세 가지를 준비하세요.

- **예약 작업을 실행할 수 있는 AI 에이전트.** 이 시스템은 예약 실행 기능이
  있는 Claude **Cowork 모드**를 기준으로 만들고 테스트했습니다.
- **Claude 커넥터 3개.** Claude 설정에서 다음 커넥터를 연결하세요.
  - **Chrome** — LinkedIn, Indeed와 회사 채용 페이지를 확인합니다.
  - **Google Drive** — 결과를 저장할 구글 시트를 읽고 씁니다.
  - **Gmail** — 결과 이메일을 보냅니다.
- **구글 계정과 알림을 받을 이메일 주소.** 시트는 Google Drive 커넥터에
  연결한 계정에 만듭니다.

> **왜 Chrome을 사용하나요?** API 키나 인증 코드를 설정하지 않아도
> 비개발자와 초보자가 바로 시작할 수 있도록 일부러 브라우저 방식으로
> 만들었습니다. 개발 경험이 있고 채용 사이트의 공식 API 접근 권한도 있다면
> 수집 단계를 API 방식으로 바꿀 수 있습니다. 다만 이 저장소에는 API 연동
> 코드가 포함되어 있지 않으며, 링크·날짜·중복 제거 규칙은 기존 스펙과 같게
> 유지해야 합니다.

다른 AI 에이전트를 사용해도 됩니다. 웹 탐색, 구글 시트 읽기·쓰기, 이메일
발송, 매일 예약 실행 기능을 모두 지원하는지 확인하세요.

## 1단계 — 저장소 받기

명령은 저장소 파일이 아니라 **터미널**에 입력합니다.

- Windows에서는 시작 메뉴에서 **PowerShell** 또는 **Windows Terminal**을
  엽니다.
- macOS나 Linux에서는 **Terminal**을 엽니다.

아래 두 줄을 한 줄씩 붙여넣고 Enter를 누르세요.

```bash
git clone https://github.com/Journey-512/catch-before-jobs-vanish.git
cd catch-before-jobs-vanish
```

`git` 명령을 사용할 수 없거나 Git이 낯설다면 GitHub 페이지에서 **Code →
Download ZIP**을 선택해 압축을 푸세요. GitHub에서 받은 압축 파일은 보통
`catch-before-jobs-vanish-main` 폴더로 풀립니다.

계속하기 전에 터미널의 현재 위치를 이 압축 해제 폴더로 옮기세요. 터미널에
`cd `를 입력하고(뒤의 공백 포함), 파일 관리자(파일 탐색기나 Finder 등)에서
압축 해제 폴더를 터미널로 끌어온 뒤 Enter를 누릅니다. 다음 명령과 같은
동작입니다.

```text
cd "<압축을-푼-catch-before-jobs-vanish-main-폴더의-전체-경로>"
```

제대로 된 폴더에 들어왔는지 확인하세요. `dir`(Windows) 또는 `ls`(macOS)를
입력했을 때 `setup.md`와 `your-input` 폴더가 보이면 맞습니다. 보이지 않으면 한
단계 위에 있는 것입니다. Windows 탐색기는 같은 이름의 폴더 안에 저장소 폴더를
한 번 더 만드는 경우가 많으니, 그 폴더를 열어 안쪽 폴더를 다시 끌어오세요. 이
터미널은 계속 사용합니다. 3단계의 `cd your-input`은 이 폴더에서 시작합니다.

## 2단계 — 구글 시트 만들기

새 구글 시트를 만들고 탭 이름을 정확히 **`Jobs`**와 **`Companies`**로
지정하세요. 파이프라인은 이 이름으로 탭을 찾습니다.

### `Jobs` 탭

첫 번째 행에 다음 10개 열을 순서대로 입력하세요.

| Date Added | Company | Job Title | Location | Source | Posted Date | Link | Status | Fit Score | Fit Reason |
|---|---|---|---|---|---|---|---|---|---|

`Status` 열에는 시스템이 `Excluded (...)` 또는 `Closed (날짜)`를 기록합니다.
지원 결과는 사용자가 `Applied`, `Passed - CV`, `Rejected - CV`, `Lost` 중
하나로 입력하세요. 추천을 받아 지원했다면 `Applied (referral)`처럼
`(referral)`을 뒤에 붙입니다. 사용자가 입력한 값은 시스템이 덮어쓰지
않습니다.

### `Companies` 탭

첫 번째 행에 다음 9개 열을 순서대로 입력하세요.

| Index | Company | Aiming | Match Level | URL | Source site | HQ | Topics | Memo |
|---|---|---|---|---|---|---|---|---|
| 1 | `<회사명>` | `Aiming` 또는 빈칸 | `Strong` 또는 `Soft` | `<채용 페이지 URL 또는 빈칸>` | `<출처 또는 빈칸>` | `<본사 또는 빈칸>` | `<관련 분야 또는 빈칸>` | `<메모 또는 빈칸>` |

회사 목록은 다음 두 방식으로 채워집니다.

1. **수동 입력:** 첫 실행 전에 관심 회사를 한 행에 하나씩 직접 입력하세요.
   집중해서 지원할 회사 1~3곳은 `Aiming` 열에 `Aiming`이라고 적고, 나머지는
   빈칸으로 둡니다. 정확한 입력 형식과 각 열의 의미는
   [`your-input/companies.example.md`](your-input/companies.example.md)를
   참고하세요.
2. **자동 추가:** `config.md`의 `Company discovery`에서 자동 추가를 켜면,
   파이프라인이 조건을 통과한 새 회사를 이 탭에 추가합니다. 자동 추가된 행의
   `Aiming`은 빈칸으로 남고, 확인된 값만 기록됩니다. 해당 회사는 다음
   실행부터 회사별 검색 대상에 포함됩니다.

`Companies` 탭이 회사 목록의 유일한 기준입니다. 로컬 회사 목록 파일은
사용하지 않으며, 파이프라인은 기존 행에 사용자가 입력한 값을 덮어쓰지
않습니다.

**이전 버전에서 업그레이드한다면?** 예전 `your-input/companies.md`는 더 이상
읽히지 않으며 시스템이 자동으로 삭제하지도 않습니다. 행을 이 탭으로
옮기세요: Name → Company, careers URL → URL, Match level → Match Level,
Aiming → Aiming, note → Memo. 옮긴 내용을 원래 파일과 대조한 뒤, 기존
파일의 보관 또는 삭제 여부는 직접 결정하세요.

시트는 Google Drive 안의 어느 폴더에 두어도 되고 파일 이름도 자유롭게 정할
수 있습니다. 시스템은 **시트 ID**로 시트를 찾습니다. 시트를 열고 주소창을
확인하세요.

```
https://docs.google.com/spreadsheets/d/THIS-LONG-STRING-IS-THE-ID/edit
```

`/d/`와 `/edit` 사이의 문자열을 복사해 다음 단계에서
`your-input/config.md`의 `Google Sheet ID:`에 붙여넣으세요.

## 3단계 — `your-input/` 채우기

템플릿 파일 두 개를 복사하고 `< >`로 표시된 자리표시자를 본인의 값으로
바꾸세요.

macOS 또는 Linux:

```bash
cd your-input
cp preferences.template.md preferences.md
cp config.template.md config.md
cd ..
```

Windows PowerShell:

```powershell
cd your-input
Copy-Item preferences.template.md preferences.md
Copy-Item config.template.md config.md
cd ..
```

최초 한 번만 실행하면 되는 CV 변환 요청문과 각 필드의 설명은
[`your-input` 한국어 안내](your-input/README.md#cvmd-만들기-한국어-안내)에
있습니다.

`cv.md`, `preferences.md`, `config.md`가 모두 준비되면 다음 단계로
넘어가세요. 기존 버전에서 업그레이드한다면
[3.2 업그레이드 안내](CHANGELOG.md#업그레이드-안내)를 따르세요.

## 4단계 — 에이전트에 파이프라인 넘기기

사용하는 에이전트에 맞는 방법을 선택하세요. 실제 실행을 시작하는 것은
`config.md`가 아니라 **에이전트에 등록한 예약 작업**입니다.

예약 작업의 일정과 시간대는 `config.md`의 `Alert delivery`와 맞춰 자연어로
등록하세요. 예: `Asia/Seoul 기준 매일 오전 9시`. `config.md`의 일정이나
시간대를 나중에 바꾸면 예약 작업도 별도로 수정해야 합니다. 등록 후에는 다음
실행 시각이 원하는 현지 시각과 일치하는지 확인하세요.

### Claude Code에 스킬 설치

Claude Code 안에서 다음 명령 두 줄을 실행하세요.

```
/plugin marketplace add Journey-512/catch-before-jobs-vanish
/plugin install catch-before-jobs-vanish@catch-before-jobs-vanish
```

설치 후 일정, 시간대와 로컬 저장소 경로를 포함해 예약 작업을 요청하세요.

> Asia/Seoul 기준 매일 오전 9시에 `job-alert` 스킬을 실행해 줘. 내 저장소는
> `C:\Users\me\catch-before-jobs-vanish`에 있어.

예시 경로는 실제로 저장소를 받은 경로로 바꾸세요.

### Cowork 등 다른 에이전트에 프롬프트 등록

1. [`skills/job-alert/SKILL.md`](skills/job-alert/SKILL.md)를 엽니다.
2. 프롬프트 블록 전체를 복사합니다.
3. `Step 0`의 저장소 경로 자리표시자 한 곳만 실제 로컬 경로로 바꿉니다.
4. **다른 부분은 수정하지 않은 채** 예약 작업으로 등록합니다.

Cowork에서는 프롬프트를 붙여넣고 `이 프롬프트를 Asia/Seoul 기준 매일 오전
9시에 실행해 줘`라고 요청하면 됩니다.

## 5단계 — 테스트 실행

작업을 한 번 수동으로 실행하거나 첫 예약 실행을 기다린 뒤 확인하세요.

1. `Jobs` 탭에 새 행이 추가되고 `Date Added`가 알림 시간대 기준
   `YYYY-MM-DD HH:MM` 형식인지 확인합니다.
2. `Posted Date`가 `YYYY-MM-DD` 절대 날짜로 저장되고, 필요한 경우
   `estimated` 또는 `unknown` 표시가 붙는지 확인합니다. `Last 24h` 같은 상대
   표현은 저장되면 안 됩니다.
3. 링크가 검색 결과 페이지가 아니라 공고 하나를 가리키는지 확인합니다.
   LinkedIn 공고는 `jobs/view/{id}` 형태의 영구 링크이고, Indeed와 회사 채용
   페이지 공고는 각자의 공고 URL을 사용합니다.
4. 이메일에 **Top / Strong** 공고, 자동 추가된 회사를 검토할 수 있는 안내와
   시트 링크가 포함되어 있는지 확인합니다.
5. 채용공고 본문에서 확인된 지역 조건 위반 공고가 시트에는
   `Excluded (...)`로 기록되고 이메일에서는 제외되는지 확인합니다.
6. 매칭이 0건일 때도 `No new matches. System alive.` 이메일이 오는지
   확인합니다. 이메일이 전혀 오지 않으면 실행 또는 발송 문제입니다.

## 6단계 — 조정 (피드백 루프 두 개 — 섞지 마세요)

1. **점수 보정:** 점수가 이상해 보이면 해당 행의 `Fit Reason`을 확인하고
   `cv.md`의 `Skill calibration` 한 줄을 조정합니다.
2. **결과 검증:** 지원 결과에 맞춰 `Status`를 업데이트합니다. 주간 리포트의
   점수 구간별 서류 통과율은 일반 지원(cold)과 추천 지원(referral)을 나눠
   확인하세요.

이메일 수는 `preferences.md`의 `Cutoff`, 직무 정확도는 허용·제외 직무명,
회사 자동 추가 범위는 `config.md`의 `Company discovery`에서 조정합니다. 채점
방식의 자세한 설명은 [한국어 채점 기준표](docs/fit-scoring-rubric.ko-KR.md)를
참고하세요.

## 문제 해결

### LinkedIn 링크가 몇 시간 뒤 열리지 않음

검색 URL이 저장되었을 수 있습니다. 에이전트가 공고별 `jobs/view/{id}` 영구
링크를 추출하는지 확인하세요.

### `Posted Date`에 `Last 24h`가 저장됨

상대 날짜를 그대로 저장한 경우입니다. 프롬프트 2단계에서 수집 시점에
`YYYY-MM-DD`로 변환하는지 확인하세요.

### 다른 회사의 공고가 저장됨

LinkedIn이 자동 선택한 `currentJobId`를 사용했을 수 있습니다. 결과 목록 전체를
읽고 회사명과 직무명이 모두 일치하는 공고를 선택하는지 확인하세요.

### 같은 공고가 반복해서 들어옴

과거 시트를 읽지 못했거나 같은 채용 페이지 URL이 여러 형식으로 저장되었을 수
있습니다. 이메일에서 과거 이력을 확인하지 못했다는 안내를 확인하고, 같은 채용
사이트의 URL을 한 가지 형식으로 통일하세요.

### `Companies` 탭에 있는 회사의 공고가 검색되지 않음

회사 채용 페이지나 회사 필터가 바뀌었을 수 있습니다. 파이프라인이 채용 정보
사이트로 대체 검색했는지 이메일에서 확인하고, 빈 결과는 키워드 검색으로 한 번
더 확인하세요.

### `Aiming`·회사별·주간 심층 검색이 모두 건너뛰어짐

`Companies` 탭을 읽지 못했거나 헤더가 정해진 9열 구조와 다를 수 있습니다.
2단계의 헤더로 복구하고 시트 ID와 공유 설정을 확인한 뒤 다시 실행하세요. 이
경우에도 `preferences.md`를 사용하는 일반 LinkedIn·Indeed 검색은 계속
실행됩니다.

### 이메일이 전혀 오지 않음

예약 작업이 실행됐는지 먼저 확인한 다음 Gmail 초안함을 확인하세요. 이메일
발송에 실패하면 초안이 남아 있을 수 있습니다.
