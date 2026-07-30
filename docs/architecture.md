# Architecture

**Language:** English (below) · [한국어](#아키텍처-한국어)

How the daily run fits together, and the reasoning behind each design choice.
The prompt that implements all of this is
[`scheduler-prompt-template.md`](../scheduler-prompt-template.md); this
document explains *why* it is shaped the way it is.

## The pipeline: 10 steps in 4 layers

| Layer | Steps | Writes? | Job |
|---|---|---|---|
| **[Input]** | 0 Load config | no | read every run-time input once (`your-input/` 4 files, deep-scan flag) |
| **[Retrieval]** | 1 Collect · 2 Normalize · 3 Hard gates · 4 Dedup | no | *who are today's candidates* — identity and eligibility |
| **[Decision]** | 5 Score · 6 Post-filter · 7 Liveness | no | *how good is each candidate* — grading |
| **[Reporting]** | 8 Sheet write · 9 Email + report | **yes** | record and notify — consume upstream markings only |

Three reasons the layering pays for itself:

1. **Rollback boundary** — nothing before Step 8 leaves a trace, so a run that
   dies mid-way is harmless.
2. **Cost order** — free judgments (card text, sheet lookups) run before
   expensive ones (opening full JDs, liveness requests), which protects the
   daily token budget.
3. **Judgment vs. execution** — Decision steps only *mark* outcomes (duplicate
   copies to clean, Excluded, Closed); Reporting transcribes those markings in
   one place. So "why isn't this posting in the email?" always has a recorded
   answer, and a crash mid-decision never leaves a half-written sheet.

Each rule lives in exactly one step. That sounds like documentation hygiene,
but it's operational: v1 kept its dedup rule in two places, one copy got fixed
and the other didn't, and the stale copy shipped duplicate rows for weeks.

## Two rule sources — and only two

The scheduler prompt carries the **logic** and zero personal data. The
[`your-input/`](../your-input) folder carries the **data**: career evidence
(`cv.md` + Skill calibration), gates and thresholds (`preferences.md` +
Eligibility), the company watchlist (`companies.md` + Aiming flags), and wiring
(`config.md`). At runtime the prompt reads the folder and acts on what it
finds.

The Google Sheet deliberately holds **records, never rules**. v1 kept a third
rule surface in the Sheet (a "PRD" tab that won on conflict); two editable rule
homes meant half-updated rules, which is exactly how the dedup bug survived.
v3 retired it.

Why this split matters:

- **Inspectable** — every rule is in one public template; every personal value
  is in one gitignored folder.
- **Updatable** — change a target title by editing `preferences.md`; when
  scoring feels off, add one Skill-calibration line to `cv.md`. The prompt
  stays frozen.
- **Shareable** — the prompt contains zero personal data, so it lives in a
  public repo unchanged.

## [Retrieval] — who are today's candidates

### Collect: tiered, with a weekly deep scan

Sources are organized by cost and reliability, not treated equally.

| Tier | Source | When it runs | Cost (rough) |
|---|---|---|---|
| 1 | Indeed + LinkedIn (24h filter) | every run | ~1-5K tokens per query |
| Aiming | your flagged 1-3 companies: 7-day window, extended locations, direct careers check | every run | small (few companies) |
| 2 | Strong-match companies | deep-scan day (mandatory); other days budget-permitting | ~5-15K per company |
| 3 | Soft-match companies | deep-scan day (mandatory); other days budget-permitting | ~5-15K per company |

**Tier 1 catches ~90% of postings** because most companies cross-post to
LinkedIn and Indeed. Sweeping every whitelisted company's careers page daily
would cost 300-700K tokens — unviable and unnecessary. So the full sweep runs
**once a week** (the deep-scan day in `config.md`, capped per tier), and only
the few **Aiming** companies get the expensive treatment every day.

Collection is defensive, because job sites are a hostile surface:

- **Verified company filters only.** A wrong company-filter ID fails
  *silently* — an empty result, not an error. An empty filter result is never
  taken as "no jobs"; it's cross-checked with a keyword search.
- **Careers-page recipes rot.** Companies redesign careers sites without
  notice, killing saved URLs and selectors. A dead recipe falls back to the
  job boards for that company and the coverage gap is disclosed — never
  silently swallowed.
- **Promoted cards carry no timestamp.** They're accepted only from companies
  already on your list, with the date marked as an estimate.

### Capturing durable links

LinkedIn's 24h-filtered search URL (`f_TPR=r86400`) is a **rolling window**:
the result set behind an identical URL changes hour by hour. So the agent never
saves a search URL. Instead it extracts per-posting `jobs/view/{id}` permalinks
from the results page and saves those — they don't rotate.

It also does **not** trust LinkedIn's auto-selected `currentJobId`. That ID is
LinkedIn's recommendation for *you*, not a match for your query. The agent
parses the whole result list and matches by company + title.

(Site-specific extraction recipes are deliberately out of scope for this repo —
collection is described at the concept level.)

### Normalize: at capture time, not later

Relative labels ("3 hours ago", "Yesterday") are converted to absolute
`YYYY-MM-DD` **the moment they're captured** — stored data is never relative,
because "Last 24h" is 48h old the next day and silently wrong. Every Posted
Date carries a label: computed-from-relative or defaulted dates are
"estimated", dateless careers pages are "unknown" — and only the truly
dateless get flagged "freshness unconfirmed" in the email. Careers URLs are
canonicalized to one format, which the next step depends on.

### Hard gates: certain failures are dropped with no record

Five gates, judged on card text alone: title allowlist/exclude, recency
window, industry exclusions, an explicitly-required language you don't work
in, and a location the card already rules out. Failures are dropped without a
Sheet row — recording every miss would bury the signal.

Two refinements earned in production:

- The local language of the posting's country is **not assumed** to be
  required — that assumption over-drops postings from non-English-speaking
  countries. Only an explicit fluency requirement triggers the gate.
- If a knockout only surfaces later inside the full JD, it still applies —
  with one exception: location violations found only in the JD are *recorded*
  as Excluded instead (see the boundary rule below).

### Dedup: one posting = one row, forever

Three fingerprints — LinkedIn jobId, numeric careers-site job ID (extracted
after URL canonicalization), and company + normalized title — matched against
**all** existing rows, with no time window. First come, first served: the
earliest row survives and is never re-scored.

v1 deduplicated within a 48-hour window; in production the same posting
re-entered after multi-day gaps and one role accumulated three rows. The
window is gone. If the Sheet can't be read at all this run, the run continues
with within-run dedup and the email discloses "history not checked —
duplicates possible" (missing a fresh posting costs more than a duplicate).

## [Decision] — how good is each candidate

### Scoring: evidence, not impressions

Step 5 is the **only step that opens the full JD**. Each surviving posting is
scored 0-100 by extracting the JD's actual requirements and grading each one
against the written evidence in `cv.md` — with credit grades (Direct /
Adjacent / Weak / Missing), weights that favor differentiating requirements,
and caps that override the total when a must-have is missing or weak. Every
score ships with a one-line **Fit Reason**, so "why did this get 84?" is
answerable from the Sheet row alone. JD text is treated as untrusted data —
instructions embedded in a posting are ignored.

The full rubric — including what a backtest against past real application
outcomes changed — lives in [`fit-scoring-rubric.md`](fit-scoring-rubric.md).

### Post-filter, and the drop-vs-record boundary

The email cutoff (default 70, yours to change in `preferences.md`) and the
location rules mark postings; nothing is deleted. The boundary that keeps the
Sheet meaningful:

- **Certain from the card alone** → dropped at the hard gates, no record.
- **Visible only inside the JD** (the card said "Remote", the body says "US
  time zones required") → scored, then recorded with `Status: Excluded` and
  kept out of the email — so "why isn't this in the email?" always has an
  answer in the Sheet.

Location checks appear in several steps (search query, Aiming extension, hard
gate, post-filter, the rubric's Location points) — that's not duplication;
each occurrence does a different job (narrowing cost, widening the net,
dropping, recording, ranking). Merging them breaks one of the five.

### Liveness: don't email ghosts

Before the email, every posting scoring above 80 — new this run *and* already
in the Sheet — is checked via its link. Closed ones are marked "Closed (date)"
and excluded from the email. "Closed" requires positive evidence (an explicit
message, a 404); a blocked or ambiguous response is marked unverified instead.
Requests are politely spaced — it's a courtesy check, not a crawl. A Status a
human wrote is never overwritten.

## [Reporting] — the only writes

### Writing to the Sheet

All writes happen in Step 8: appending every post-dedup finding (including
Excluded ones) and executing the upstream markings (blanking duplicates,
writing Closed statuses). Write safety rules learned in production: find the
last row dynamically using a column that's never blank, re-verify the target
row immediately before writing, and never trust an earlier snapshot — the
owner edits the Sheet by hand between runs.

- **`Jobs` tab, 10 columns:** Date Added | Company | Job Title | Location |
  Source | Posted Date | Link | Status | Fit Score | Fit Reason.
- **`Companies` tab, 9 columns:** Index | Company | Aiming | Match Level |
  URL | Source site | HQ | Topics | Memo.

Everything written to the Sheet stays in **one language** (set in
`preferences.md`) — mixed-language cells are what you get when the
conversation's language leaks into the data.

The Status column doubles as the **outcome loop**: you type `Applied` /
`Passed - CV` / `Rejected - CV` / `Lost` (with a `(referral)` suffix for
referral applications) as your applications progress, and the deep-scan day's
report summarizes the funnel — CV-pass rate per score band, cold and referral
kept separate. Details in the rubric's "Two feedback loops".

Rows sort by Date Added descending — the first question on opening the sheet
is "what happened today?", and dates are unambiguous while scores can be
retuned. (Skippable if you curate row order by hand.) When a posting shows up
on multiple sources, the Source field is concatenated — more sources, more
confidence it's real and current.

### Auto-adding companies

A posting from a company not on your whitelist joins the watchlist only if it
clears a quality gate (a maturity bar for its market — thresholds in
`companies.md`) **and** an industry match against your Strong/Soft topics.
Auto-adds never get the Aiming flag, and every one is flagged in the email for
veto. No manual review queue, no junk.

### Email + run report

One email per run: Top (≥85) and Strong (70-84) groups, headhunter postings
tagged, freshness-unconfirmed postings flagged, auto-adds listed with a veto
prompt, and a link to the Sheet. **Zero matches still sends** "No new matches.
System alive." — so silence always means breakage, never an empty day. If the
mail tool only creates drafts, it drafts first and then sends; a failed send
leaves the draft as a safety net.

The run ends with a report checklist — found / emailed / auto-added / drop
reasons / liveness results / coverage gaps / duplicates cleaned — because an
unattended system must be auditable the morning something looks off.

## Cost

A normal run is ~25-30K tokens (hard cap 50K; raised on the deep-scan day).
Twice-daily runs were rejected: 24h windows overlap heavily, so a second run
roughly doubles cost for little new signal. A single morning run lands the
email at the start of the workday.

---

<a name="아키텍처-한국어"></a>

# 아키텍처 (한국어)

**언어:** [English](#architecture) · 한국어

매일 실행이 어떻게 맞물리는지, 그리고 각 설계 결정의 근거. 이 구조를 구현한
프롬프트는 [`scheduler-prompt-template.md`](../scheduler-prompt-template.md)이고,
이 문서는 *왜* 그런 모양인지를 설명합니다.

## 파이프라인: 4층 10단계

| 층 | 단계 | 쓰기 | 하는 일 |
|---|---|---|---|
| **[Input] 입력** | 0 설정 읽기 | 없음 | 실행 시점 입력 전부 읽기 (`your-input/` 4파일, 딥스캔 플래그) |
| **[Retrieval] 수집·선별** | 1 수집 · 2 정규화 · 3 하드 게이트 · 4 중복 제거 | 없음 | *오늘의 후보가 누구인가* — 신원과 자격 |
| **[Decision] 판단** | 5 채점 · 6 후처리 · 7 열림 확인 | 없음 | *각 후보가 얼마나 좋은가* — 등급 |
| **[Reporting] 기록·알림** | 8 시트 기록 · 9 이메일 + 리포트 | **있음** | 기록하고 알리기 — 위층의 마킹을 받아 적기만 |

층을 나눈 이유 세 가지:

1. **되돌림 경계** — 8단계 전에는 아무 흔적도 안 남아서, 실행이 도중에 죽어도
   무해합니다.
2. **비용 순서** — 공짜 판정(카드 텍스트·시트 대조)이 먼저, 비싼 판정(JD 열기·
   열림 확인)은 생존자만. 하루 토큰 예산을 지키는 구조입니다.
3. **판단과 실행 분리** — 판단 단계는 결과를 *마킹*만 하고(정리할 중복, Excluded,
   Closed), 기록 층이 그 마킹을 한 곳에서 받아 적습니다. "왜 이 공고가 이메일에
   없지?"의 답이 항상 남고, 판단 도중에 죽어도 반쯤 쓴 시트가 생기지 않습니다.

그리고 어떤 규칙이든 딱 한 단계에만 적습니다. 문서를 깔끔하게 쓰자는 얘기처럼
들리지만 실제 운영 문제입니다: v1은 중복 제거 규칙을 두 곳에 적어 뒀고, 한쪽만
고쳐진 낡은 사본이 몇 주간 중복 행을 만들었습니다.

## 규칙 소스는 둘 — 딱 둘

스케줄러 프롬프트는 **로직**만 들고 있고 개인 데이터는 0입니다.
[`your-input/`](../your-input) 폴더가 **데이터**를 듭니다: 경력 증거(`cv.md` +
Skill calibration), 게이트와 기준값(`preferences.md` + Eligibility), 회사
목록(`companies.md` + Aiming 플래그), 계정 연결(`config.md`). 실행할 때
프롬프트가 이 폴더를 읽고 그대로 동작합니다.

구글 시트는 의도적으로 **기록만, 규칙은 없음**입니다. v1은 시트 안에도 규칙을
적는 자리("충돌 시 우선"인 PRD 탭)를 하나 더 뒀는데, 고칠 수 있는 규칙이 두 곳에
적혀 있으면 한쪽만 고쳐지기 마련입니다 — 중복 버그가 살아남은 경로가 정확히
그것이라 v3에서 없앴습니다.

이 분리가 주는 것:

- **들여다보기 쉬움** — 규칙 전부가 공개 템플릿 한 곳에, 개인 값 전부가
  gitignore 폴더 한 곳에.
- **고치기 쉬움** — 직무명은 `preferences.md`를, 채점이 이상하면 `cv.md`의
  calibration 한 줄을 고칩니다. 프롬프트는 동결.
- **공유하기 쉬움** — 프롬프트에 개인정보가 없어 공개 레포에 그대로 둡니다.

## [Retrieval] — 오늘의 후보가 누구인가

### 수집: 단계별 + 주 1회 딥스캔

소스를 비용과 신뢰도에 따라 차등 취급합니다.

| 단계 | 소스 | 실행 시점 | 비용(대략) |
|---|---|---|---|
| 1 | 인디드 + 링크드인 (24h 필터) | 매 실행 | 쿼리당 ~1-5K 토큰 |
| Aiming | 직접 표시한 회사 1-3곳: 7일치 검색 + 확장 지역 + 채용 페이지 직접 확인 | 매 실행 | 작음 (회사 수가 적음) |
| 2 | Strong 매칭 회사 | 딥스캔 요일 의무, 다른 날은 예산 되는 만큼 | 회사당 ~5-15K |
| 3 | Soft 매칭 회사 | 딥스캔 요일 의무, 다른 날은 예산 되는 만큼 | 회사당 ~5-15K |

대부분 회사가 링크드인·인디드에 교차 게시하므로 **1차가 공고의 약 90%를
잡습니다.** 화이트리스트 전체의 채용 페이지를 매일 도는 건 30만~70만 토큰이라
불가능하고 불필요합니다. 그래서 Strong/Soft 회사 전체 확인은 **주 1회**(딥스캔
요일, tier당 회사 수 제한)만 하고, 비용이 드는 매일 확인은 **Aiming** 회사 몇
곳에만 씁니다.

수집 단계는 일부러 조심스럽게 짰습니다. 채용 사이트는 구조가 예고 없이 바뀌고
접근이 막히기도 하기 때문입니다:

- **검증된 회사 필터만.** 틀린 필터 ID는 에러가 아니라 *빈 결과*로 조용히
  실패합니다. 빈 결과를 "공고 없음"으로 단정하지 않고 키워드 검색으로 교차
  확인합니다.
- **채용 페이지는 예고 없이 바뀝니다.** 회사가 페이지를 개편하면 저장해 둔
  주소와 추출 방법이 한순간에 안 통하게 됩니다. 그런 회사는 잡보드 검색으로
  대체하고, 확인하지 못한 범위를 이메일에 알립니다 — 조용히 넘어가지 않습니다.
- **Promoted(광고) 카드에는 날짜가 없습니다.** 목록에 있는 회사만 수용하고,
  날짜를 추정으로 표기합니다.

### 영구 링크 확보

링크드인의 24시간 필터 검색 URL(`f_TPR=r86400`)은 **움직이는 창**입니다 — 같은
URL인데 뒤의 결과 집합이 시간마다 바뀝니다. 그래서 검색 URL은 절대 저장하지 않고,
결과 페이지에서 각 공고의 `jobs/view/{id}` 영구 링크를 추출해 저장합니다.

또한 링크드인이 자동 선택한 `currentJobId`를 **믿지 않습니다.** 그 ID는 내 검색어
매칭이 아니라 *나에 대한 추천*입니다. 결과 목록 전체를 파싱해 회사+직무명으로
매칭합니다.

(사이트별 추출 레시피는 의도적으로 이 레포 범위 밖입니다 — 수집은 개념
수준까지만 서술합니다.)

### 정규화: 나중이 아니라 캡처 즉시

"3 hours ago", "Yesterday" 같은 상대 표현은 **캡처하는 순간** 절대 날짜
`YYYY-MM-DD`로 변환합니다 — "Last 24h"는 다음 날 48시간 전이 되어 조용히
틀려지니, 저장 데이터에 상대값은 없습니다. 모든 Posted Date에 라벨이 붙습니다:
상대 표현에서 계산했거나 기본값을 넣은 날짜는 "추정(estimated)", 날짜 자체가 없는
careers 페이지는 "모름(unknown)" — 그리고 정말 날짜가 없던 것만 이메일에
"freshness unconfirmed"로 표시됩니다. careers URL은 한 형식으로 통일합니다 (다음
단계의 전제).

### 하드 게이트: 확정 탈락은 기록 없이 드롭

카드 텍스트만으로 판정하는 게이트 5종: 직무명 허용/제외, 시간 창, 산업 제외,
내가 못 쓰는 언어의 명시적 필수 요건, 카드가 이미 배제하는 위치. 탈락한 공고는 시트에
남기지 않고 버립니다 — 떨어진 것까지 전부 기록하면 정작 봐야 할 것이 묻히기
때문입니다.

실제로 돌리면서 다듬은 규칙 두 가지:

- 공고 국가의 현지어를 필수라고 **추정하지 않습니다** — 그 추정은 비영어권
  공고를 과잉 드롭합니다. 명시적 유창성 요구만 게이트에 걸립니다.
- JD 안에서 뒤늦게 드러난 knockout에도 같은 게이트가 소급 적용됩니다. 예외는
  위치 하나: JD에서만 보인 위치 위반은 드롭 대신 Excluded로 *기록*합니다 (아래
  경계 규칙).

### 중복 제거: 한 공고 = 한 행, 영원히

지문(fingerprint, 공고를 알아보는 고유값) 3개 — 링크드인 jobId, careers 페이지
숫자 ID (URL 통일 후 추출), 회사 + 정규화 직함 — 를 시트 **전체** 행과
대조합니다. 시간 창 없음. 선착순: 최초 발견
행이 살아남고, 절대 재채점하지 않습니다.

v1은 48시간 창 안에서만 중복을 걸렀는데, 실운영에서 같은 공고가 며칠 간격으로
재유입되며 한 직무가 행 3개를 쌓았습니다. 창은 없앴습니다. 시트를 아예 못 읽는
날은 당일 것끼리만 걸러서 계속 진행하고, "과거 대조 못 함 — 중복 가능"을
이메일에 공개합니다 (새 공고를 놓치는 비용이 중복 한 줄보다 큽니다).

## [Decision] — 각 후보가 얼마나 좋은가

### 채점: 인상이 아니라 증거

5단계는 **JD 전문을 여는 유일한 단계**입니다. 살아남은 공고마다 JD의 실제
요구사항을 뽑아 `cv.md`에 적힌 증거와 하나씩 대조해 0-100점을 만듭니다 — credit
등급(Direct / Adjacent / Weak / Missing), 차별 요건에 실리는 가중치, must-have가
비었거나 약할 때 총점을 덮어쓰는 cap. 모든 점수에는 한 줄 **Fit Reason**이
따라붙어 "이게 왜 84점이지?"를 시트 행 하나로 답할 수 있습니다. JD 본문은
신뢰하지 않는 데이터로 취급합니다 — 본문에 섞인 지시문은 무시합니다.

기준표 전체 — 과거 실제 지원 백테스트가 무엇을 바꿨는지 포함 — 는
[`fit-scoring-rubric.md`](fit-scoring-rubric.md)에 있습니다.

### 후처리, 그리고 드롭 vs 기록의 경계

이메일 컷(기본 70, `preferences.md`에서 자유 조정)과 위치 규칙은 마킹만 하고
아무것도 지우지 않습니다. 무엇은 버리고 무엇은 기록하는지, 그 경계는 이렇습니다:

- **카드만으로 확정** → 하드 게이트에서 기록 없이 드롭.
- **JD를 열어야 보임** (카드는 "Remote", 본문은 "US time zones required") →
  채점까지 마친 뒤 `Status: Excluded`로 기록하고 이메일에서 제외 — "왜 이메일에
  없지?"의 답이 항상 시트에 남습니다.

위치 조건은 여러 단계에 등장합니다 (검색 쿼리, Aiming 확장, 하드 게이트, 후처리,
기준표의 Location 배점) — 중복이 아니라 각자 다른 일(검색 비용 절약, Aiming 범위
확대, 탈락, 기록, 순위)을 하는 계층 방어입니다. 합치면 다섯 중 하나가 깨집니다.

### 열림 확인: 죽은 공고 거르기

발송 전에 80점 초과 공고 — 이번 실행 신규 *그리고* 이미 시트에 있던 것 — 를
링크로 확인합니다. 닫힌 건 "Closed (날짜)"로 마킹하고 이메일에서 뺍니다. 닫힘
판정은 양성 증거(명시적 마감 문구, 404)로만 하고, 차단·모호 응답은 unverified로
표시합니다. 요청은 예의 있게 간격을 둡니다 — 크롤이 아니라 확인입니다. 사람이 쓴
Status는 절대 덮어쓰지 않습니다.

## [Reporting] — 유일한 쓰기 구간

### 시트 기록

모든 쓰기는 8단계에서 일어납니다: 중복 제거를 통과한 전부(Excluded 포함)를
추가하고, 위층의 마킹(중복 사본 비우기, Closed 기입)을 실행합니다. 실제로 돌리면서 배운
쓰기 안전 수칙이 있습니다: 마지막 행은 빈칸 없는 열을 기준으로 매번 새로 찾고,
쓰기 직전에 대상 행의 회사·직무명을 다시 확인하고, 이전에 읽어 둔 상태를 그대로
믿지 않습니다 — 시트 주인이 실행 사이에 시트를 손으로 고치기 때문입니다.

- **`Jobs` 탭 10열:** Date Added | Company | Job Title | Location | Source |
  Posted Date | Link | Status | Fit Score | Fit Reason.
- **`Companies` 탭 9열:** Index | Company | Aiming | Match Level | URL |
  Source site | HQ | Topics | Memo.

시트에 쓰는 모든 내용은 **한 언어**로 고정합니다 (`preferences.md`에서 지정) —
셀에 언어가 섞이는 건 대화 언어가 데이터로 새는 증상입니다.

Status 열은 그대로 **outcome loop**가 됩니다: 지원이 진행될 때마다 `Applied` /
`Passed - CV` / `Rejected - CV` / `Lost`를 직접 적고 (리퍼럴 지원은 `(referral)`
접미), 딥스캔 날 리포트가 funnel을 요약합니다 — 점수 밴드별 서류 통과율, cold와
referral 분모 분리. 상세는 기준표의 "피드백 루프 두 개".

정렬은 Date Added 내림차순 — 시트를 열 때 첫 질문이 "오늘 뭐가 있었지?"이고,
날짜는 명확한 반면 점수는 나중에 조정될 수 있어서입니다 (직접 큐레이션하면 생략
가능). 한 공고가 여러 소스에 나오면 Source를 이어 붙입니다 — 소스가 많을수록
진짜이고 최신일 확신이 커집니다.

### 회사 자동 추가

화이트리스트에 없는 회사의 공고는 품질 게이트(그 시장 기준의 성숙도 —
기준값은 `companies.md`) **그리고** Strong/Soft 산업 매칭을 통과할 때만 목록에
추가됩니다. 자동 추가에는 Aiming 플래그가 절대 붙지 않고, 전부 이메일에 표시되어
거부할 수 있습니다. 수동 검토 대기열 없이 잡음을 막습니다.

### 이메일 + 실행 리포트

실행당 이메일 1통: Top(85 이상)·Strong(70-84) 그룹, 헤드헌터 공고 태그, 신선도
미확인 공고 플래그, auto-add 목록과 거부 안내, 시트 링크. **매칭 0건이어도
보냅니다** — "No new matches. System alive." 침묵은 언제나 고장을 의미하게.
메일 도구가 초안만 지원하면 초안 먼저, 그 다음 발송 — 발송이 실패해도 초안이
안전망으로 남습니다.

실행 끝에는 체크리스트 리포트가 붙습니다 — 발견 / 발송 / auto-add / 드롭 사유 /
열림 확인 결과 / 커버리지 공백 / 중복 정리 건수. 사람 없이 도는 시스템은, 뭔가 이상해
보이는 아침에 어젯밤 무엇을 했는지 따져 볼 수 있어야 하기 때문입니다.

## 비용

평상시 한 번 실행에 ~25-30K 토큰 (상한 50K, 딥스캔 날은 상향). 하루 2회는 기각:
24시간 창이 크게 겹쳐 두 번째 실행은 비용만 거의 두 배이고 새 신호는 적습니다.
아침 1회 실행이면 업무 시작 시각에 이메일이 도착합니다.
