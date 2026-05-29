# Architecture

**Language:** English (below) · [한국어](#아키텍처-한국어)

How the daily run fits together, and the reasoning behind each design choice.

## The pipeline

```
load config  ->  search (tiered)  ->  filter  ->  score  ->  auto-add  ->  write Sheet  ->  email
```

Each stage is described below.

## Single source of truth: `your-input/` + the Sheet PRD tab

The scheduler prompt is deliberately thin. It carries no personal data and no rules
of its own — at runtime it reads `your-input/` (cv, preferences, companies, config)
and the Sheet's `PRD` tab. On any conflict, the **Sheet PRD wins**, because it's the
surface you can edit fastest (one cell, no file edit, no prompt regen).

Why this matters:

- **Inspectable** — every rule is a plain file or a spreadsheet cell, not buried in
  prompt text.
- **Updatable** — change a target title without regenerating the agent.
- **Shareable** — the prompt has zero personal data, so it lives in a public repo
  unchanged.

## Tiered search

Sources are organized by cost and reliability, not treated equally.

| Tier | Source | When it runs | Cost (rough) |
|---|---|---|---|
| 1 | Indeed | every run | ~1-2K tokens/query |
| 1 | LinkedIn (24h filter) | every run | ~3-5K tokens |
| 2 | Company career pages | only when a Strong-match company didn't surface in Tier 1 | ~5-15K each |

**Tier 1 catches ~90% of postings** because most companies cross-post to LinkedIn
and Indeed. Company-site checks are a *failsafe*, not the primary mechanism — career
pages are a fragile web surface (ATS migrations, JS-only rendering, search-param
churn), so the system is designed to tolerate their failure rather than depend on
them. Running all whitelisted company sites every day would cost 300-700K tokens —
unviable, and unnecessary.

### Capturing durable links

LinkedIn's 24h-filtered search URL (`f_TPR=r86400`) is a **rolling window**: the
result set behind an identical URL changes hour by hour. So the agent never saves a
search URL. Instead it extracts per-posting `jobs/view/{id}` permalinks from the
results page and saves those — they don't rotate.

It also does **not** trust LinkedIn's auto-selected `currentJobId`. That ID is
LinkedIn's recommendation for *you*, not a match for your query, and the same ID can
appear across unrelated searches. The agent parses the whole result list and matches
by company + title.

## Filtering

Three filters, applied in order:

1. **Title allowlist + exclude list.** An exact allowlist (e.g. Product Manager,
   Product Owner, Senior/Principal variants) plus an explicit exclude list (Junior,
   Associate, Director, VP, Head, Chief, Staff, Lead, etc.). The exclude list matters
   as much as the allowlist — without it, near-miss titles dilute the signal.
2. **Strict rolling 24h recency.** Relative labels like "X hours ago" are parsed to
   an absolute `YYYY-MM-DD` **at capture time**. Stored data is never relative — a
   "Last 24h" string is 48h old the next day and silently wrong.
3. **Hard location filter.** Categorical constraints (e.g. non-EU-timezone remote)
   are excluded outright, regardless of score. Excluded postings still get written to
   the Sheet (`Status: Excluded`) for the record, but never reach the email. See the
   rubric doc for why this is a filter, not a score penalty.

## Scoring

A 100-point rubric across five dimensions (Domain 30 / Role 25 / Seniority 20 /
Company 15 / Location 10), detailed in
[`fit-scoring-rubric.md`](fit-scoring-rubric.md). The weights are derived from your
own CV-tailoring methodology, so scores are auditable against criteria you defined.

Scoring is **post-hoc**: pick the email cutoff after seeing a real distribution of
scores, not before. The default cutoff is 60, with sub-tiers (Top ≥85, Strong 70-84,
Maybe 60-69).

## Auto-adding companies

A posting from a company not on your whitelist is added to the watchlist only if it
clears a gate: a stage/quality bar (e.g. Series C+, listed, or late-stage) **and** an
industry match against your Strong/Soft topics. Every auto-add is flagged in the
email for your veto. This avoids a manual review queue while keeping junk out.

## Writing to the Sheet

All postings — including excluded ones — are appended to the `Jobs` tab, then the
tab is sorted by `Date Added` descending. Date-sort beats score-sort because the
first question on opening the sheet is "what happened today?", and the date is
unambiguous while scores can be retuned. When a posting shows up on multiple sources,
the `Source` field is concatenated ("LinkedIn, Indeed, Company site") — more sources
means higher confidence the posting is real and current.

## Email

One HTML email per run, only postings at or above the threshold, grouped into Top /
Strong / Maybe / New Companies, plus a link back to the Sheet. The subject is fixed
("Catch the fresh fish before jobs vanish") — recurring automated mail should read
well on the 50th occurrence, so metadata the mail client already shows (date, time)
is stripped.

If the email tool only supports drafts, the agent creates a draft and then sends it
via browser automation. Draft-first is a deliberate fallback: if the send fails, the
email still exists in Drafts for a manual send.

## Cost

A full run is ~25-30K tokens. Twice-daily runs were rejected: 24h windows overlap
heavily, so a second run roughly doubles cost for little new signal. A single morning
run lands the email in the inbox at the start of the workday.

---

<a name="아키텍처-한국어"></a>

# 아키텍처 (한국어)

**언어:** [English](#architecture) · 한국어

매일 실행이 어떻게 맞물리는지, 그리고 각 설계 결정의 근거.

## 파이프라인

```
설정 로드  ->  검색(단계별)  ->  필터  ->  점수  ->  자동 추가  ->  시트 기록  ->  이메일
```

## 유일한 기준점: `your-input/` + 시트 PRD 탭

스케줄러 프롬프트는 일부러 얇습니다. 자체 규칙이나 개인 데이터를 들고 있지 않고, 실행할 때
`your-input/`(cv·preferences·companies·config)과 시트 `PRD` 탭을 읽습니다. 충돌이 나면
**시트 PRD가 우선** — 가장 빨리 고칠 수 있는 곳이기 때문입니다(셀 하나, 파일 수정 없음,
프롬프트 재생성 없음).

- **들여다보기 쉬움** — 모든 규칙이 평범한 파일이나 시트 셀에 있음.
- **고치기 쉬움** — 직무 하나 바꾸려고 에이전트를 다시 만들 필요 없음.
- **공유하기 쉬움** — 프롬프트에 개인정보가 없어 공개 레포에 그대로 둘 수 있음.

## 단계별(tiered) 검색

소스를 비용과 신뢰도에 따라 차등 취급합니다.

| 단계 | 소스 | 실행 시점 | 비용(대략) |
|---|---|---|---|
| 1 | 인디드 | 매 실행 | 쿼리당 ~1-2K 토큰 |
| 1 | 링크드인 (24h 필터) | 매 실행 | ~3-5K 토큰 |
| 2 | 회사 채용 페이지 | 1차에서 강매칭 회사가 안 잡혔을 때만 | 각 ~5-15K |

대부분 회사가 링크드인·인디드에 교차 게시하기 때문에 **1차가 공고의 약 90%를 잡습니다.**
회사 페이지 확인은 주력이 아니라 *안전망*입니다 — 채용 페이지는 취약한 표면이라(ATS 이전,
JS 전용 렌더링, 검색 파라미터 변경) 의존하기보다 실패를 견디도록 설계했습니다. 화이트리스트
회사 페이지를 매일 다 도는 건 30만~70만 토큰이라 불가능하고 불필요합니다.

### 영구 링크 확보

링크드인의 24시간 필터 검색 URL(`f_TPR=r86400`)은 **움직이는 창**입니다 — 같은 URL인데
뒤의 결과 집합이 시간마다 바뀝니다. 그래서 검색 URL은 절대 저장하지 않고, 결과 페이지에서
각 공고의 `jobs/view/{id}` 영구 링크를 추출해 저장합니다.

또한 링크드인이 자동 선택한 `currentJobId`를 **믿지 않습니다.** 그 ID는 내 검색어 매칭이
아니라 *나에 대한 추천*이고, 같은 ID가 무관한 검색들에 걸쳐 나타날 수 있습니다. 결과 목록
전체를 파싱해 회사+직무명으로 매칭합니다.

## 필터링

순서대로 3개 필터:

1. **직무명 허용 + 제외 목록.** 정확한 허용 목록(예: Product Manager, Product Owner,
   Senior/Principal 변형)과 명시적 제외 목록(Junior, Associate, Director, VP, Head,
   Chief, Staff, Lead 등). 제외 목록도 허용 목록만큼 중요 — 없으면 비슷한 직무명이 신호를
   흐립니다.
2. **엄격한 24시간 최신성.** "X hours ago" 같은 상대 표현은 **캡처 시점에** 절대 날짜
   `YYYY-MM-DD`로 변환합니다. 저장 데이터는 절대 상대값을 쓰지 않음 — "Last 24h"는 다음 날
   48시간 전이 되어 조용히 틀려집니다.
3. **하드 위치 필터.** 범주형 조건(예: EU 시간대 밖 원격)은 점수와 무관하게 즉시 제외.
   제외된 공고도 기록을 위해 시트엔 `Status: Excluded`로 쓰되 이메일엔 안 들어갑니다. 왜
   점수 감점이 아니라 필터인지는 기준표 문서를 보세요.

## 점수 매기기

5개 항목 100점 기준표(도메인 30 / 직무 25 / 시니어리티 20 / 회사 15 / 위치 10),
[`fit-scoring-rubric.md`](fit-scoring-rubric.md)에 상세. 가중치는 내 이력서 맞춤화 방법론에서
가져와, 점수가 내가 정한 기준으로 감사 가능합니다.

점수는 **사후(post-hoc)** — 이메일 컷오프는 미리가 아니라 실제 점수 분포를 보고 정합니다.
기본 컷오프 60점, 하위 등급(Top ≥85, Strong 70-84, Maybe 60-69).

## 회사 자동 추가

화이트리스트에 없는 회사 공고는 게이트를 통과할 때만 목록에 추가됩니다: 단계/품질 기준(예:
Series C+, 상장, 후기 단계) **그리고** 강/약 산업 매칭. 모든 자동 추가는 거부할 수 있도록
이메일에 표시됩니다. 수동 검토 대기열을 없애면서도 잡음을 막습니다.

## 시트 기록

제외된 것 포함 모든 공고를 `Jobs` 탭에 추가한 뒤 `Date Added` 내림차순(최신순)으로 정렬.
시트를 열 때 첫 질문이 "오늘 뭐가 있었지?"이고 날짜는 명확한 반면 점수는 나중에 조정될 수
있어, 날짜 정렬이 점수 정렬을 이깁니다. 한 공고가 여러 소스에 나오면 `Source`를 이어 붙임
("LinkedIn, Indeed, Company site") — 소스가 많을수록 진짜이고 최신일 확신이 높아집니다.

## 이메일

실행당 HTML 이메일 1통, 기준선 이상 공고만 Top / Strong / Maybe / New Companies로 묶고
시트 링크 첨부. 제목은 고정("Catch the fresh fish before jobs vanish") — 반복 자동
메일은 50번째에도 잘 읽혀야 하므로 메일 클라이언트가 이미 보여주는 메타데이터(날짜·시각)는
뺍니다.

이메일 도구가 초안만 지원하면 초안을 만든 뒤 브라우저 자동화로 발송합니다. 초안 먼저는 의도된
안전망 — 발송이 실패해도 초안이 남아 수동 발송이 가능합니다.

## 비용

한 번 실행에 ~25-30K 토큰. 하루 2회는 기각: 24시간 창이 크게 겹쳐 두 번째 실행은 비용만
거의 두 배이고 새 신호는 적습니다. 아침 1회 실행이면 출근 시각에 이메일이 도착합니다.
