# Changelog

**Language:** English (below) · [한국어](#changelog-한국어)

All notable changes to this project, with the *reason* next to each change —
the version history is part of the product here. Format follows
[Keep a Changelog](https://keepachangelog.com/); versions track generations of
the pipeline logic.

## [3.0.0] — 2026-07-28

The pipeline ran daily in production for a month — evolving into the private
2.x generation below — and its scoring was then backtested against past real
application outcomes. This release rewrites the template and docs around what
that month proved and disproved.

### Changed

- **Scoring: impression rubric → evidence rubric.** v1 scored five
  impression-level dimensions; in practice most PM postings share the same
  generic asks, so scores saturated in one band. v3 extracts each JD's actual
  requirements and grades them against written evidence in `cv.md` (credit
  grades Direct/Adjacent/Weak/Missing, weighted toward differentiating
  requirements, with caps that override the total). See
  [`docs/fit-scoring-rubric.md`](docs/fit-scoring-rubric.md).
- **Three rules tightened by the backtest** (past real applications, outcomes
  known): generic PM competencies may never be promoted to must-have (they
  saturated the evidence score and discriminated nothing); below-level
  postings score 8/20 with a "downlevel warning" (cold applications to
  below-level roles are screened out more reliably — v1 pointed the other way);
  referral applications carry a `(referral)` status suffix so pass-rate
  denominators stay clean.
- **Email cutoff 60 → 70, "Maybe" tier retired.** With calibrated scoring,
  60-69 was inbox noise. Those rows are still recorded in the Sheet; the
  cutoff remains a personal setting you can change in `preferences.md`.
- **Dedup: 48-hour window → three fingerprints against all rows.** The window
  let the same posting re-enter after multi-day gaps (one role reached three
  rows in production). Now: LinkedIn jobId, careers-site numeric ID, and
  company + normalized title, matched with no time window; the earliest row
  survives and is never re-scored.
- **Pipeline reorganized into 10 steps in 4 layers.** Steps 0-7 write
  nothing; all writes happen in steps 8-9, transcribing what earlier steps
  marked. Each rule now lives in exactly one step — the dedup bug survived v1
  precisely because the rule lived in two places and only one got fixed.

### Added

- **Normalize step** — relative dates become absolute at capture time, careers
  URLs are canonicalized, and every Posted Date carries an
  estimated/unknown label (feeding a "freshness unconfirmed" flag in the
  email).
- **Liveness check step** — postings scoring above 80 are verified still open
  before the email goes out; closures need positive evidence.
- **Skill calibration** (in `cv.md`) and **Eligibility knockouts** (in
  `preferences.md`) — user-confirmed controls that cap the scorer's optimism
  and drop pointless applications at the gates.
- **Aiming flag + weekly deep scan** — 1-3 hand-picked companies get a 7-day
  window, extended locations, and a daily careers check; the full Strong/Soft
  company sweep runs once a week under a per-tier cap.
- **Outcome loop** — a Status vocabulary (`Applied` / `Passed - CV` /
  `Rejected - CV` / `Lost`, plus `(referral)`) turns the Sheet into labeled
  outcome data, reported as a funnel on the deep-scan day.
- **Heartbeat + run report** — zero matches still sends "System alive", and
  every run ends with an auditable checklist (found / emailed / dropped /
  gaps / duplicates cleaned).
- **Injection guard** — JD text is treated as untrusted data; instructions
  embedded in postings are ignored.
- **Headhunter rule** — intermediary postings are labeled, never
  score-discounted: information scarcity is not evidence of mismatch, and the
  outcome loop judges empirically whether they convert.
- **Fictional demo persona** — the `your-input/` examples now model a 5-year
  mobility & travel PM (invented end to end) whose files exercise the new
  calibration, eligibility, and Aiming features.
- **Korean README split out** — `README.ko-KR.md` now carries the full Korean
  page; the other docs stay bilingual within one file.
- This changelog.

### Removed

- **The Sheet PRD tab.** v1 kept a second, "wins on conflict" rule surface in
  the Sheet; two rule homes produced half-updated rules. The Sheet now holds
  records only — logic lives in the template, data in `your-input/`.
- **The 48-hour dedup window** (see Changed).
- The unused career-memory folder line in `config.md` — a v1 leftover the
  runtime never read.

## [2.x] — 2026-06 ~ 2026-07-28 (private, never published)

The generation that only ever ran in production. Over a month of daily runs it
grew, one fix at a time, the rules that 3.0.0 documents — the 70 cutoff, the
three-fingerprint dedup, the liveness check. Nothing from this line was
released; 3.0.0 is its public form.

## [1.0.0] — 2026-05-29

Initial public release: thin scheduler prompt with zero personal data +
gitignored `your-input/` as the personal-data home; tiered 24h search
(LinkedIn + Indeed) with per-posting permalink capture; five-dimension
100-point rubric with email cutoff 60; hard location filters; auto-add
company watchlist with a quality gate; draft-then-send email.

---

<a name="changelog-한국어"></a>

# Changelog (한국어)

**언어:** [English](#changelog) · 한국어

이 프로젝트의 주요 변경 기록입니다. 변경마다 *이유*를 함께 적습니다 — 여기서는
버전 이력 자체가 제품의 일부입니다. 형식은
[Keep a Changelog](https://keepachangelog.com/)를 따르고, 버전 번호는
파이프라인 로직의 세대를 나타냅니다.

## [3.0.0] — 2026-07-28

파이프라인을 한 달간 매일 실운영했고(그 사이 아래의 비공개 2.x 세대로
진화했습니다), 채점을 과거 실제 지원의 결과로 백테스트했습니다. 이 릴리스는
그 한 달이 증명한 것과 반증한 것을 중심으로 템플릿과 문서를 다시 쓴 것입니다.

### 바뀐 것

- **채점: 인상 기준표 → 증거 기준표.** v1은 인상 수준 5개 항목을 매겼는데, PM
  공고 대부분이 비슷한 일반 요건을 공유해 점수가 한 구간에 몰렸습니다. v3는
  JD의 실제 요구사항을 뽑아 `cv.md`의 증거와 대조합니다 (credit 4등급, 차별
  요건 가중, 총점을 덮어쓰는 cap).
- **백테스트가 조인 규칙 3개** (과거 실제 지원): 일반 PM 역량의 must-have 승격
  금지 (Evidence를 포화시켜 변별 0) · 다운레벨 공고 8/20 + 경고 (cold 다운레벨
  지원 서류에서 걸러지기 쉬움 — v1 배점은 방향이 반대였음) · 리퍼럴 지원의 `(referral)`
  접미 (통과율 분모 오염 방지).
- **이메일 컷 60 → 70, "Maybe" 등급 폐지.** 보정된 채점에서 60-69는
  받은편지함의 소음이었습니다. 시트엔 계속 기록되고, 컷은 `preferences.md`에서
  자유롭게 바꾸는 개인 설정입니다.
- **중복 제거: 48시간 창 → 지문 3개 x 전체 행.** 창 방식은 같은 공고를 며칠
  간격으로 재유입시켰습니다 (실운영에서 한 직무가 행 3개). 이제 링크드인
  jobId · careers 숫자 ID · 회사+정규화 직함을 시간 창 없이 대조하고, 최초 행이
  살아남으며 재채점하지 않습니다.
- **파이프라인을 4층 10단계로 재구성.** 0-7단계는 아무것도 안 쓰고, 모든 쓰기는
  8-9단계에서 상류의 마킹을 받아 적습니다. 규칙은 딱 한 단계에만 적습니다 — v1의
  중복 버그가 살아남은 이유가 같은 규칙이 두 곳에 적혀 있어서였습니다.

### 새로 생긴 것

- **정규화 단계** — 상대 날짜의 캡처 즉시 절대화, careers URL 통일,
  추정/모름 라벨 (+ 이메일의 "freshness unconfirmed" 플래그).
- **열림 확인 단계** — 80점 초과 공고는 발송 전 생존 확인. 닫힘 판정은 양성
  증거로만.
- **Skill calibration**(`cv.md`)과 **Eligibility knockout**(`preferences.md`)
  — 채점기가 과하게 후한 점수를 주지 못하게 막고, 조건이 안 되는 공고는 점수를
  매기기 전에 걸러 내는, 본인이 확정하는 통제 장치.
- **Aiming 플래그 + 주 1회 딥스캔** — 직접 고른 1-3개 회사는 7일 창·확장 지역·
  careers 매일 확인, Strong/Soft 전체 스윕은 tier당 상한을 걸고 주 1회.
- **Outcome loop** — Status 어휘(`Applied` / `Passed - CV` / `Rejected - CV` /
  `Lost` + `(referral)`)로 시트가 라벨 데이터가 되고, 딥스캔 날 funnel로
  요약됩니다.
- **하트비트 + 실행 리포트** — 0건이어도 "System alive" 발송, 매 실행 끝에
  무엇을 했는지 확인할 수 있는 체크리스트.
- **Injection guard** — 공고 본문은 신뢰하지 않는 데이터. 본문 속 지시문은
  무시.
- **헤드헌터 규칙** — 중개 공고는 점수 할인 없이 라벨만: 정보 부족은 부적합의
  증거가 아니고, 전환 여부는 outcome loop가 실측으로 판정.
- **가공 인물 데모** — `your-input/` 예시가 완전 창작된 모빌리티·여행 5년차
  PM으로 바뀌어, 새 calibration·eligibility·Aiming 기능을 그대로 시연합니다.
- **한국어 README 분리** — `README.ko-KR.md`가 한국어 페이지 전체를 담습니다.
  나머지 문서는 기존대로 한 파일에 영어·한국어를 같이 둡니다.
- 이 changelog.

### 없어진 것

- **시트 PRD 탭.** v1은 시트 안에 "충돌 시 우선"인 두 번째 규칙 표면을 뒀고, 그
  결과가 반쪽만 고쳐지는 규칙이었습니다. 이제 시트에는 기록만 — 로직은 템플릿에,
  데이터는 `your-input/`에.
- **48시간 중복 창** (위 참조).
- `config.md`의 안 쓰이던 career-memory 폴더 줄 — 런타임이 읽지 않던 v1 잔재.

## [2.x] — 2026-06 ~ 2026-07-28 (비공개 운영 전용)

실운영에서만 살았던 세대입니다. 한 달간 매일 돌면서 3.0.0에 문서화된 규칙들 —
컷 70, 지문 3개 중복 제거, 열림 확인 — 을 하나씩 만들어 냈습니다. 이 계보에서
공개된 것은 없고, 3.0.0이 그 공개판입니다.

## [1.0.0] — 2026-05-29

최초 공개: 개인 데이터 0의 얇은 스케줄러 프롬프트 + 개인 데이터를 담는
gitignored `your-input/`, 공고별 영구 링크를 저장하는 24시간 단계별 검색
(링크드인 + 인디드), 5개 항목 100점 기준표와 이메일 컷 60, 하드 위치 필터, 품질
게이트가 있는 회사 자동 추가, draft-then-send 이메일.
