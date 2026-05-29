# Fit-scoring rubric

**Language:** English (below) · [한국어](#적합도-점수-기준표-한국어)

> **In one line:** this is how each job posting is scored out of 100 for how well it
> fits *you* — so the email can sort the best matches to the top and skip the rest.

Each surviving posting is scored out of 100 across five dimensions. The score is used
to sort and to apply the email cutoff. The weights below are a starting point —
adapt them in your own `your-input/cv.md` and `preferences.md` to reflect what you
actually optimize for.

The principle: **a score should trace back to criteria you wrote.** These five
dimensions correspond to a "check before applying" self-assessment, so when you ask
"why did this get 85?" the answer points at your own rules, not arbitrary defaults.

## The five dimensions

| Dimension | Weight | What it measures |
|---|---|---|
| Domain Fit | 30 | How well the company's domain matches industries you know |
| Role Fit | 25 | How closely the responsibilities match your actual experience |
| Seniority Scope | 20 | Whether the level/scope fits where you are in your career |
| Company Quality | 15 | Stage, reputation, trajectory of the company |
| Location/Logistics | 10 | Commute/timezone/remote compatibility |
| **Total** | **100** | |

### Domain Fit — 30 pts

The single heaviest dimension. Score high when the company's domain is one you've
worked in or know deeply (the Strong topics in your preferences); lower for adjacent
domains; near-zero for domains you'd be starting cold in. Domain is weighted highest
because it's the strongest predictor of both interview success and on-the-job ramp.

### Role Fit — 25 pts

How well the posting's actual responsibilities map to what you've done. A "Product
Manager" title with a job description full of work you've shipped scores high; the
same title describing work outside your experience scores lower. Read the
description, not just the title.

### Seniority Scope — 20 pts

Is the level right? Both directions cost points: a role too junior wastes your
experience, a role too senior is a stretch that won't shortlist you. Score highest
where the scope matches your current trajectory.

### Company Quality — 15 pts

Stage and standing — funded and growing, listed, or a well-regarded scale-up scores
higher than an unknown early-stage shop. Intentionally only 15 points: a great role
at a less famous company should still be able to outscore a mediocre role at a famous
one.

### Location/Logistics — 10 pts

The lightest dimension, because the *binary* location question is handled by a hard
filter, not here (see below). These 10 points capture softer logistics among
*acceptable* locations — fully on-site vs. hybrid vs. local-remote.

## Hard filters override scores

Some constraints are categorical, not continuous. The clearest example: a remote role
in a timezone too far from yours makes daily synchronous collaboration impractical —
no matter how strong the role is on every other dimension.

A pure rubric mishandles this. Give a US-remote role 2/10 on Location but 85+ across
the other four dimensions and it still surfaces as a "Top Match" — exactly the false
positive you don't want. So location-incompatibility is enforced as a **hard filter
applied after scoring**: such postings are excluded from the email regardless of
score (they're still recorded in the Sheet with `Status: Excluded`).

**Rule of thumb:** if your preference is "I won't do X under any circumstances,"
that's a hard filter, not a low score. Filters and rubrics are complementary — most
real systems need both.

## Choosing the cutoff

Score everything first, *then* pick the cutoff from the real distribution. The
default:

- **Top Match** — ≥ 85
- **Strong Match** — 70–84
- **Maybe Fit** — 60–69
- Below 60 — recorded in the Sheet, not emailed

Erring toward a lower cutoff (60) is a reasonable default early on: it's easier to
ignore a borderline entry in an email than to never have seen it. Raise the cutoff
once you trust the rubric.

---

<a name="적합도-점수-기준표-한국어"></a>

# 적합도 점수 기준표 (한국어)

**언어:** [English](#fit-scoring-rubric) · 한국어

> **한 줄 요약:** 공고 하나하나가 *나*랑 얼마나 맞는지 100점으로 매기는 방식입니다 — 그래야
> 이메일이 잘 맞는 공고를 위로 정렬하고 나머지는 건너뛸 수 있어요.

살아남은 공고마다 5개 항목 100점 만점으로 점수를 매깁니다. 점수는 정렬과 이메일 컷오프에
쓰입니다. 아래 가중치는 출발점일 뿐 — 본인의 `your-input/cv.md`와 `preferences.md`에서
실제로 중요하게 보는 것에 맞게 바꾸세요.

원칙: **점수는 내가 쓴 기준으로 거슬러 올라가야 한다.** 이 5개 항목은 "지원 전 점검"
자가진단에 대응하므로, "이게 왜 85점이지?"가 임의 기본값이 아니라 내 규칙을 가리킵니다.

## 5개 항목

| 항목 | 가중치 | 측정하는 것 |
|---|---|---|
| 도메인 적합 | 30 | 회사 도메인이 내가 아는 산업과 얼마나 맞나 |
| 직무 적합 | 25 | 직무 책임이 내 실제 경험과 얼마나 맞나 |
| 시니어리티 스코프 | 20 | 레벨/스코프가 내 커리어 단계에 맞나 |
| 회사 품질 | 15 | 회사의 단계·평판·성장 궤도 |
| 위치/물류 | 10 | 통근/시간대/원격 적합성 |
| **합계** | **100** | |

### 도메인 적합 — 30점

가장 무거운 단일 항목. 회사 도메인이 내가 일했거나 잘 아는 분야(선호의 Strong 주제)면 높게,
인접 도메인이면 낮게, 완전히 새로 시작할 도메인이면 거의 0점. 도메인은 면접 통과와 입사 후
적응 둘 다의 가장 강한 예측 변수라 최고 가중치입니다.

### 직무 적합 — 25점

공고의 실제 책임이 내가 해온 일과 얼마나 맞나. "Product Manager"라도 내가 해본 일로 가득한
JD면 높게, 같은 직함이라도 경험 밖 일을 적었으면 낮게. 직함만 보지 말고 설명을 읽으세요.

### 시니어리티 스코프 — 20점

레벨이 맞나? 양방향 모두 감점: 너무 주니어면 경험 낭비, 너무 시니어면 서류 통과가 안 되는
무리수. 현재 궤도에 맞을 때 최고점.

### 회사 품질 — 15점

단계와 위상 — 투자받고 성장 중이거나 상장사거나 평판 좋은 스케일업이, 무명 초기 회사보다
높음. 일부러 15점만: 덜 유명한 회사의 좋은 자리가 유명 회사의 평범한 자리를 이길 수 있어야
하니까요.

### 위치/물류 — 10점

가장 가벼운 항목 — *이분법적* 위치 질문은 여기가 아니라 하드 필터가 처리하기 때문(아래
참고). 이 10점은 *허용된* 위치들 사이의 부드러운 물류 차이(완전 출근 vs. 하이브리드 vs.
현지 원격)를 잡습니다.

## 하드 필터가 점수를 덮어쓴다

어떤 조건은 연속적이 아니라 범주형입니다. 가장 명확한 예: 시간대가 너무 먼 원격 직무는 다른
모든 항목이 아무리 강해도 매일 동기 협업이 비현실적입니다.

순수 기준표는 이걸 잘못 처리합니다. 미국 원격에 위치 2/10을 줘도 나머지 4개 항목이 85+면
여전히 "Top Match"로 떠오릅니다 — 바로 원치 않는 거짓 양성이죠. 그래서 위치 비호환은 **점수
이후 적용하는 하드 필터**로 강제합니다: 그런 공고는 점수와 무관하게 이메일에서 제외(시트엔
`Status: Excluded`로 기록).

**경험칙:** "어떤 경우에도 X는 안 한다"는 선호라면 그건 낮은 점수가 아니라 하드 필터입니다.
필터와 기준표는 상보적 — 실제 시스템은 대개 둘 다 필요합니다.

## 컷오프 정하기

먼저 전부 점수 매기고, *그 다음* 실제 분포에서 컷오프를 고릅니다. 기본값:

- **Top Match** — ≥ 85
- **Strong Match** — 70–84
- **Maybe Fit** — 60–69
- 60 미만 — 시트엔 기록, 이메일엔 안 보냄

초반엔 낮은 컷오프(60)가 합리적 기본값: 이메일의 애매한 항목은 무시하기 쉬워도, 아예 못 본
건 되돌릴 수 없으니까요. 기준표를 신뢰하게 되면 컷오프를 올리세요.
