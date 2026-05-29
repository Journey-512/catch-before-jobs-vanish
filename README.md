# catch-before-jobs-vanish

**Language:** English (below) · [한국어](#한국어)

A daily, self-hosted job-alert agent. Every morning it searches LinkedIn + Indeed
for roles posted in the **last 24 hours**, scores each one against your profile on a
100-point rubric, auto-grows a company watchlist, and emails you only the matches —
before the postings rotate out of the search window and disappear.

Built and run with [Claude](https://claude.com) in Cowork mode. The name comes from
the core problem it solves: fresh postings vanish from "last 24h" search URLs within
hours, so you have to **catch them before they vanish**.

## How it works

```
                       runs daily (e.g. 9 AM, via a scheduled task)
                                       |
                                       v
  your-input/                          ┌───────────────────────────────────┐
  ├── cv.md                       │  thin scheduler prompt                                                                                                             │
  ├── preferences.md   │  reads your-input/ at runtime,                                                                                               │
  ├── companies.md     │  searches, filters, scores, emails                                                                                           │
  └── config.md ───┤                                                                                                                                                             │
                      └───────────────────────────────────┘
                                       |
         
            ┌─────┼──────────────┐
            v                          v                                                                 v
      Tier 1: LinkedIn          Google Sheet                                           Email
      + Indeed (24h)            (Jobs / Companies / PRD)                  matches >= 60,
      Tier 2: company           append + sort,                                      grouped by tier
      career pages              auto-add new companies
      (fallback)
```

The key design choice: your personal data is the single source of truth in
[`your-input/`](your-input), and the scheduler prompt is thin — it reads those files
at runtime. You configure the system by editing four files, never by
editing the prompt. That's why the prompt in this repo contains zero personal data.

## What makes it more than a search alert

- **Permalink capture, not search URLs.** LinkedIn's 24h-filtered search URL is a
  moving window; a posting found at 9 AM is gone by afternoon. The agent extracts
  per-posting `jobs/view/{id}` permalinks so saved links keep working.
- **Fit scoring grounded in *your* criteria.** The 100-point rubric is derived from
  your own CV-tailoring methodology, not arbitrary defaults — so "why did this score
  85?" traces back to a document you wrote.
- **Hard filters for categorical constraints.** A timezone-incompatible remote role
  is excluded outright, not just penalized — because some preferences are binary,
  not continuous.
- **Self-growing watchlist with a quality gate.** New companies that clear a
  stage/industry gate get added automatically and flagged for your veto.

## Quick start

1. Fork/clone this repo.
2. Create the four files in [`your-input/`](your-input) (templates in its README).
3. Set up a Google Sheet with `Jobs`, `Companies`, `PRD` tabs (see
   [`setup.md`](setup.md)).
4. Register a scheduled task using
   [`scheduler-prompt-template.md`](scheduler-prompt-template.md).
5. Wait for tomorrow morning's email.

Full walkthrough: **[setup.md](setup.md)**.

## Repo layout

| Path | What it is |
|---|---|
| [`your-input/`](your-input) | Your personal data (gitignored). The single source of truth. |
| [`scheduler-prompt-template.md`](scheduler-prompt-template.md) | The thin daily prompt. No personal data. |
| [`setup.md`](setup.md) | Step-by-step usage guide. |
| [`docs/architecture.md`](docs/architecture.md) | How the tiered search + scoring + email pipeline fits together. |
| [`docs/fit-scoring-rubric.md`](docs/fit-scoring-rubric.md) | The 100-point rubric, dimension by dimension. |

> **The build story** (the decision log and the failures-and-recoveries from building
> this in a day) lives separately as long-form writing, not in this repo. A link will
> be added here once it's published.

## Requirements

- An AI agent with these connectors: **Chrome** (web browsing), **Google Drive**
  (your Sheet), and **Gmail** (sending the alert) — plus the ability to run a daily
  scheduled task. Built and tested with Claude in Cowork mode.
- A Google account (for the Sheet) and an email account.

## License

[MIT](LICENSE). Fork it, adapt it, ship your own.

---

<a name="한국어"></a>

# catch-before-jobs-vanish (한국어)

**언어:** [English](#catch-before-jobs-vanish) · 한국어

매일 아침 자동으로 도는 채용 알림 에이전트입니다. 링크드인과 인디드에서 **최근 24시간**
안에 올라온 채용공고를 찾아, 100점 만점 기준으로 내 프로필과 얼마나 맞는지 점수를 매기고,
관심 회사 목록을 자동으로 늘려가며, 매칭된 공고만 이메일로 보내줍니다 — 공고가 검색 창에서
밀려나 사라지기 전에 잡아내는 거죠.

[Claude](https://claude.com) Cowork 모드로 만들고 운영합니다. 이름의 의미는, "최근 24시간"
검색 URL에서 새 공고가 몇 시간 만에 사라지기 때문에 **사라지기 전에 잡아야 한다(catch
before they vanish)**는 핵심 문제에서 왔습니다.

## 작동 방식

```
                       매일 실행 (예: 오전 9시, 예약 작업으로)
                                       |
                                       v
  your-input/                                   ┌───────────────────────────────────┐
  ├── cv.md                                │  얇은 스케줄러 프롬프트                                                                                                                │
  ├── preferences.md            │  실행할 때 your-input/을 읽어서                                                                                                │
  ├── companies.md              │  검색·필터·점수·이메일 수행                                                                                                     │
  └── config.md ─────┤                                                                                                                                                             │
		                 └───────────────────────────────────┘
                                       |
            ┌───── ┼────────────┐
            v                          v                                                        v
      1차: 링크드인              구글 시트                                              이메일
      + 인디드 (24h)            (Jobs / Companies / PRD)          60점 이상만,
      2차: 회사 채용             추가 + 정렬,                                         등급별로 묶어서
      페이지 (보조)             새 회사 자동 추가
```

핵심 설계: 개인 데이터는 전부 [`your-input/`](your-input)에 두고 그것이 유일한
기준점입니다. 스케줄러 프롬프트는 얇게 두고, 실행할 때 그 파일들을 읽어옵니다. 즉
설정은 4개 파일만 고치면 되고 프롬프트는 손대지 않습니다. 그래서 이 레포의 프롬프트에는
개인정보가 전혀 없습니다.

## 단순 검색 알림과 다른 점

- **검색 URL이 아니라 영구 링크(permalink)를 저장.** 링크드인의 24시간 필터 검색 URL은
  계속 움직이는 창이라, 오전 9시에 본 공고가 오후엔 사라집니다. 에이전트는 각 공고의
  `jobs/view/{id}` 영구 링크를 추출해 저장하므로 링크가 계속 살아 있습니다.
- **내 기준에 근거한 적합도 점수.** 100점 기준표는 임의 기본값이 아니라 내가 쓰던 이력서
  맞춤화 방법론에서 가져왔습니다 — "이게 왜 85점이지?"가 내가 쓴 문서로 거슬러 올라갑니다.
- **범주형 조건엔 하드 필터.** 시간대가 안 맞는 원격 직무는 감점이 아니라 아예 제외됩니다 —
  어떤 조건은 연속적이 아니라 이분법적이니까요.
- **품질 게이트가 있는 자동 확장 목록.** 단계/산업 기준을 통과한 새 회사는 자동으로
  추가되고, 거부할 수 있도록 이메일에 표시됩니다.

## 빠른 시작

1. 이 레포를 포크/클론합니다.
2. [`your-input/`](your-input)에 4개 파일을 만듭니다 (템플릿은 해당 폴더 README에).
3. `Jobs`, `Companies`, `PRD` 탭이 있는 구글 시트를 준비합니다 ([`setup.md`](setup.md) 참고).
4. [`scheduler-prompt-template.md`](scheduler-prompt-template.md)로 예약 작업을 등록합니다.
5. 내일 아침 이메일을 기다립니다.

전체 안내: **[setup.md](setup.md)**.

## 레포 구성

| 경로 | 설명 |
|---|---|
| [`your-input/`](your-input) | 내 개인 데이터 (gitignore 처리). 유일한 기준점. |
| [`scheduler-prompt-template.md`](scheduler-prompt-template.md) | 얇은 매일 프롬프트. 개인정보 없음. |
| [`setup.md`](setup.md) | 단계별 사용 안내. |
| [`docs/architecture.md`](docs/architecture.md) | 단계별 검색 + 점수 + 이메일 파이프라인 구조. |
| [`docs/fit-scoring-rubric.md`](docs/fit-scoring-rubric.md) | 100점 기준표를 항목별로 설명. |

> **제작 스토리** (하루 만에 만들면서 내린 결정 기록과 실패·복구 사례)는 이 레포가 아니라
> 별도의 장문 글로 정리됩니다. 공개되면 여기에 링크를 추가할게요.

## 필요한 것

- **Chrome**(웹 탐색), **Google Drive**(구글 시트), **Gmail**(알림 발송) 커넥터를
  연결할 수 있고, 매일 예약 작업을 돌릴 수 있는 AI 에이전트. Claude Cowork 모드에서
  제작·테스트했습니다.
- 구글 계정(시트용)과 이메일 계정.

## 라이선스

[MIT](LICENSE). 포크하고, 고쳐서, 직접 운영하세요.
