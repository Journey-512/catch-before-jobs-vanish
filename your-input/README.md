# `your-input/` — your personal data lives here

**Language:** English (below) · [한국어](#your-input-한국어)

This folder is the home for your **local personal inputs**. The whole
pipeline has exactly three sources: the scheduler prompt (the logic — never
edited), the three files in this folder (your local settings), and the
`Companies` tab of your Google Sheet (your company registry). At runtime the
prompt reads this folder and that tab and does its work based on what it
finds. You configure the system by editing these files and the tab — your
company list itself lives in the Sheet, never in a local file.

> **Privacy:** everything in this folder is gitignored except this README,
> `.gitkeep`, and the `*.example.md` files. Your real CV — both the pasted
> original (`cv-original.md`) and the scoring file (`cv.md`) — plus your
> preferences and config never get committed or pushed. Your company list
> never touches this repo at all, because it lives in your Google Sheet.
> See the repo root `.gitignore` for the exact rule.

## Fastest start: copy the examples

Two of this folder's example files are meant to be copied — drop the
`.example` from the name, then replace the contents with your own details:

```bash
cp preferences.example.md preferences.md
cp config.example.md      config.md
```

(On Windows PowerShell use `Copy-Item preferences.example.md preferences.md`,
etc.)

The other two examples are references, never copied:
[`cv.example.md`](cv.example.md) shows what a converted `cv.md` looks like,
and [`companies.example.md`](companies.example.md) shows how to fill the
`Companies` tab of your Google Sheet. The persona in the examples — a
3-year travel & mobility PM applying from Seoul with EU work authorization
— is invented end to end for the demo.

`cv.md` is neither copied nor written from scratch: you convert it once
from your real CV — see the `cv.md` section below.

## The three files (plus one Sheet tab)

| File | What it holds | Used for |
|---|---|---|
| `cv.md` | Career evidence (roles, domains, numbers) + a **Skill calibration** section — converted once from your `cv-original.md`, then reviewed by you | Evidence scoring — every requirement in a JD is graded against what's written here |
| `preferences.md` | Target titles, locations, industries, recency, email cutoff, **Remote-work compatibility** (work-hours timezone + maximum timezone difference), output language + an **Eligibility** section (knockouts) | Search + hard gates + email cutoff |
| `config.md` | Sheet ID, email recipient, subject, **Alert delivery** (alert schedule + alert timezone), deep-scan weekday, per-tier cap + the **Company discovery** settings (auto-add on/off, maturity bar, accepted industry match) | Wiring the run to your accounts + the auto-add gate |

Company names, `Aiming` flags, `Match Level`, careers `URL`s, and
per-company memos are all managed in the Sheet's **`Companies` tab** — see
[the tab section below](#the-companies-tab--your-company-registry).

One more file lives here but is NOT a runtime input: `cv-original.md`, the
pasted text of your real CV. The one-time conversion below reads it; the
daily run never does.

---

### `cv.md` — converted from your real CV, then reviewed by you

`cv.md` is created once, in four steps:

1. Create `your-input/cv-original.md` and paste the complete text of your
   existing CV under this heading — plain text copied straight out of your
   CV is fine, no Markdown cleanup needed:

   ```markdown
   # Original CV

   <Paste the complete text of your existing CV here>
   ```

   Only have a PDF or DOCX? Ask your agent to extract the full text into
   `cv-original.md` first — the daily pipeline never reads PDF or DOCX
   files directly.
2. Give your agent the conversion request below.
3. Review the generated `cv.md` line by line (checklist below).
4. Only then register the scheduled task.

`cv-original.md` stays yours and private (gitignored like every
non-example file here), and the daily job-alert run never reads it — only
`cv.md` is used for scoring. Nothing regenerates automatically: if your CV
changes later, rerun the conversion request and review the new `cv.md`.
Already have a good, reviewed `cv.md` from an earlier version? It stays
valid — no `cv-original.md` needed.

**Conversion request (copy-paste):**

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

**Review before the first run:**

1. Company names, titles, dates, and numbers match the original.
2. No new experience or achievements were invented.
3. `Strengths` and `Domains I know well` stay within the actual evidence.
4. The `Skill calibration` Core / Transferable / Gap split matches your
   real level of experience.

The output structure the conversion produces:

```markdown
# CV summary (for fit scoring)

## Positioning
One or two sentences: who you are professionally and the role you're targeting.

## Experience
- Role, domain, seniority, years — one block per role. A one-line company
  profile ([stage | domain | scale]) works better than a brand name if you'd
  rather keep employers out of the file.
- Bullets: verb + outcome number + how / with whom.

## Strengths
- 3-6 bullet points the scorer should weight heavily.

## Domains I know well
on-demand mobility, travel booking, ... (the scorer boosts matches here)

## Skill calibration
One line per skill, syntax: `skill: one-line scope. [req type] = [max credit]`
- Core: <experience that always matches>
- Transferable: <partial experience> — with explicit caps
- Gap: <absent but you apply anyway>
```

The **Skill calibration** section is the anti-inflation control described in
[`../skills/job-alert/fit-scoring-rubric.md`](../skills/job-alert/fit-scoring-rubric.md).
The conversion request drafts it from your CV — you confirm it (that
sign-off is the point). It also works empty: scoring then runs on the "as
written, never above" meta-rule alone. Format reference:
[`cv.example.md`](cv.example.md) — never copy its facts.

### `preferences.md` — skeleton

```markdown
# Preferences

## Target titles (allowlist — exact)
- Product Manager
- Senior Product Manager
- ...

## Excluded titles (never match)
- Intern, Lead, Director, VP, Head, Chief, Group, ... (whatever levels you
  don't want)

## Locations
- Acceptable: <e.g., Netherlands, Germany, EU-remote, hybrid Berlin>
- Extended locations (Aiming companies only): <e.g., London>
- Hard exclude: <e.g., Asia / Australia remote — timezone>

## Remote-work compatibility
- Work-hours timezone: <e.g., Europe/Berlin — where you'll actually work>
- Maximum timezone difference: <hours; default 4 — remote roles anchored
  further away are excluded>

## Eligibility (knockout conditions)
- Languages I work in: <e.g., English, Korean>
- Work authorization: <regions where you hold it / regions needing sponsorship>

## Industries
- Strong: <list>   - Soft: <list>   - Avoid: <list>

## Recency
Rolling 24-hour window, strict.

## Email threshold
Cutoff 70 (default — change freely). Top >= 85, Strong 70-84; below the
cutoff is recorded in the Sheet, never emailed.

## Output language
Sheet and email language: <e.g., English>
```

The **Eligibility** section is where knockouts live — conditions that make an
application pointless (a language you don't work in, missing work
authorization). The hard gates enforce them by dropping postings outright; the
scorer never sees them. Full example:
[`preferences.example.md`](preferences.example.md).

### The `Companies` tab — your company registry

Your company list lives in the Google Sheet, not in this folder. The
`Companies` tab is the single source of truth for the list and every
per-company attribute, and the pipeline reads it at the start of every run.
Its 9 columns, in this exact order:

| Index | Company | Aiming | Match Level | URL | Source site | HQ | Topics | Memo |
|---|---|---|---|---|---|---|---|---|
| 1 | `<Company name>` | `<Aiming or blank>` | `<Strong or Soft>` | `<careers URL or blank>` | `<source or blank>` | `<HQ or blank>` | `<topics or blank>` | `<note or blank>` |

- **`Aiming`** — hand-set only, for the 1-3 companies you're actively
  gunning for (7-day window, extended locations, daily careers check).
  Auto-added rows always leave it blank.
- **`Match Level`** — `Strong` or `Soft`, mirroring your industry classes in
  `preferences.md`; it orders the weekly deep-scan sweep.
- The auto-add policy (whether new companies get appended, and the maturity
  bar) is configured in `config.md` — see the skeleton below.

Format reference for the tab: [`companies.example.md`](companies.example.md)
— a Markdown file to look at while filling in the Sheet, never copied to a
local file.

### `config.md` — skeleton

```markdown
# Config

- Google Sheet ID: <YOUR_SHEET_ID>
- Sheet tabs: Jobs | Companies
- Email recipient: your-email@example.com
- Email subject: Catch the fresh fish before jobs vanish
- Deep-scan weekday: Tuesday # weekly Strong/Soft company sweep — "which
                             # day is today" is judged in the Alert timezone
- Per-tier company cap: 8    # deep-scan day, per tier

## Alert delivery

- Alert schedule: Daily at 09:00
- Alert timezone: Asia/Seoul

## Company discovery

- Auto-add newly discovered companies: yes
- Minimum company maturity: Series C+, listed, or clearly established
- Accepted industry match: Strong or Soft
```

Two notes on the time fields:

- **Timezones use Region/City form** — `Europe/Amsterdam`, `Europe/Berlin`,
  `Asia/Seoul`, `America/New_York` — not a city name alone, a GMT offset, or an
  abbreviation like CET or KST. Region/City keeps your local clock time
  correct when daylight saving shifts the offset. The Alert timezone here
  drives the run's dates, the deep-scan weekday, `Date Added`, and
  email/report times; the Work-hours timezone in `preferences.md` is a
  different setting for a different job (judging remote roles) — the two
  can hold the same value but are never merged.
- **The `Alert schedule` line runs nothing by itself.** The actual trigger
  is the scheduled task registered in your agent, and the pipeline never
  reads this line to create or change a schedule. Editing `config.md` does
  not update an already-registered task — change the scheduler's
  registration too, and check that its next-run time matches the local
  time you intended.

Full example: [`config.example.md`](config.example.md).

---

Once `cv.md` is converted and reviewed and the three files exist, follow
[`../setup.md`](../setup.md) to create the Sheet, fill the `Companies` tab,
and register the scheduled task.

---

<a name="your-input-한국어"></a>

# `your-input/` — 내 개인 데이터가 들어가는 곳 (한국어)

**언어:** [English](#your-input--your-personal-data-lives-here) · 한국어

이 폴더에는 **로컬에 두는 개인 입력**이 들어갑니다. 이 시스템의 소스는 세
곳입니다. 하나는 스케줄러 프롬프트(로직 담당 — 손대지 않습니다), 다른 하나는
이 폴더의 3개 파일(내 로컬 설정), 마지막은 구글 시트의 `Companies` 탭(회사
registry)입니다. 실행할 때 프롬프트가 이 폴더와 그 탭을 읽어 그 내용대로
동작합니다. 회사 목록은 로컬 파일이 아니라 시트에서 직접 관리합니다.

> **개인정보:** 이 폴더 안의 파일은 이 README와 `.gitkeep`, `*.example.md`를
> 빼면 전부 gitignore 처리됩니다. 실제 이력서 — 붙여넣은 원문
> `cv-original.md`와 채점용 `cv.md` 둘 다 — 와 선호·설정은 커밋되거나
> 푸시되지 않습니다. 회사 목록은 구글 시트에 있어서 애초에 이 repo에
> 들어오지 않습니다. 정확한 규칙은 repo 루트의 `.gitignore`에서 확인하세요.

## 가장 빠른 시작: 예시 복사하기

이 폴더의 예시 파일 중 복사해서 쓰는 것은 2개입니다 — 이름에서 `.example`만
빼고 본인 내용으로 고치면 됩니다:

```bash
cp preferences.example.md preferences.md
cp config.example.md      config.md
```

(윈도우 PowerShell에서는 `Copy-Item preferences.example.md preferences.md`처럼
쓰세요.)

나머지 예시 2개는 복사하지 않는 참고 자료입니다.
[`cv.example.md`](cv.example.md)는 변환된 `cv.md`가 어떤 모습인지,
[`companies.example.md`](companies.example.md)는 구글 시트 `Companies` 탭을
어떻게 채우는지 보여줍니다. 예시 속 인물(서울에서 EU 이직을 준비하는
여행·모빌리티 3년차 PM, EU work authorization 보유)은 데모용으로 처음부터
끝까지 창작한 가공 인물입니다.

`cv.md`는 복사하는 파일도, 처음부터 새로 쓰는 파일도 아닙니다. 본인의 기존
이력서에서 한 번 변환해 만듭니다 — 아래 `cv.md` 부분을 보세요.

## 3개 파일 (+ 시트 탭 1개)

| 파일 | 담는 내용 | 용도 |
|---|---|---|
| `cv.md` | 경력 증거 (직무·도메인·수치) + **Skill calibration** 섹션 — `cv-original.md`에서 한 번 변환한 뒤 본인이 검토 | Evidence 채점 — JD의 요구사항 하나하나를 여기 적힌 증거와 대조 |
| `preferences.md` | 목표 직무명, 지역, 산업, 최신성, 이메일 컷, **Remote-work compatibility** (근무 시간대 + 최대 시간대 차이), 출력 언어 + **Eligibility** 섹션 (지원이 무의미해지는 조건) | 검색 + 필터 + 이메일 컷 |
| `config.md` | 시트 ID, 이메일 수신자, 제목, **Alert delivery** (알림 일정 + 알림 시간대), 딥스캔 요일, tier당 회사 상한 + **Company discovery** 설정 (자동 추가 여부·회사 성숙도 기준·허용 산업 분류) | 실행을 내 계정과 연결 + 자동 추가 게이트 |

회사명, `Aiming` 표시, `Match Level`, careers `URL`, 회사별 메모는 전부 구글
시트의 **`Companies` 탭**에서 관리합니다. 탭의 9열 계약과 입력 형식은
[`companies.example.md`](companies.example.md)에 있습니다 (복사 대상 아님).

이 폴더에 하나 더 두지만 런타임 입력은 아닌 파일이 `cv-original.md`입니다.
본인 이력서 원문을 붙여넣는 파일로, 아래의 1회성 변환만 읽고 매일 실행되는
파이프라인은 절대 읽지 않습니다.

## `cv.md` 만들기 (한국어 안내)

`cv.md`는 다음 네 단계로 한 번만 만듭니다:

1. `your-input/cv-original.md`를 만들고 아래 제목 밑에 기존 이력서의 전체
   텍스트를 그대로 붙여넣습니다 — 이력서에서 복사한 일반 텍스트면 충분하고,
   Markdown으로 다시 정리할 필요 없습니다:

   ```markdown
   # Original CV

   <여기에 기존 이력서의 전체 텍스트를 붙여넣으세요>
   ```

   PDF나 DOCX만 있다면, 에이전트에게 내용을 빠짐없이 추출해
   `cv-original.md`로 저장해 달라고 먼저 요청하세요. 매일 실행되는
   파이프라인이 PDF·DOCX를 직접 읽는 일은 없습니다.
2. 아래 변환 요청문을 에이전트에게 줍니다.
3. 만들어진 `cv.md`를 한 줄씩 검토합니다 (아래 체크리스트).
4. 검토가 끝난 다음에만 예약 작업을 등록합니다.

`cv-original.md`는 본인이 보관하는 비공개 파일이고(예시가 아닌 파일은 전부
gitignore), 매일 실행되는 job-alert 파이프라인은 읽지 않습니다 — 채점에는
`cv.md`만 쓰입니다. 자동으로 다시 만들어지는 일도 없습니다: 이력서가 바뀌면
변환 요청을 다시 실행하고 새 `cv.md`를 검토하세요. 이전 버전에서 이미 잘
검토된 `cv.md`를 쓰고 있다면 그대로 유효합니다 — `cv-original.md`를 새로
만들 필요 없습니다.

**변환 요청문 (복사해서 사용):**

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

**첫 실행 전 검토 체크리스트:**

1. 회사명·직책·날짜·수치가 원본과 일치하는가
2. 새로운 경력이나 성과가 만들어지지 않았는가
3. `Strengths`와 `Domains I know well`이 실제 증거를 벗어나지 않는가
4. `Skill calibration`의 Core·Transferable·Gap 분류가 실제 경험 수준과
   맞는가

## 나머지 파일 안내

각 파일의 뼈대(skeleton)는 위 영문 섹션의 코드 블록을 그대로 가져다 쓰면
됩니다. 내용을 한국어로 적어도 동작에는 문제없지만 직무명(예: Product
Manager)처럼 실제 채용 사이트 검색에 쓰이는 키워드는 영어 표기를 그대로 두는
편이 좋습니다.

- `preferences.md` — 허용/제외 직무명, 지역(+ Aiming 회사에만 적용되는 확장
  지역), 산업(강/약/회피), 24시간 기준, 이메일 컷(기본 70), 출력 언어,
  **Eligibility**, 그리고 **Remote-work compatibility**: `Work-hours
  timezone`(실제로 근무하려는 시간대)과 `Maximum timezone difference`(기본
  4시간)를 적으면, 그보다 먼 시간대에 묶인 remote 공고는 제외됩니다.
  Eligibility에는 못 갖추면 지원 자체가 무의미해지는 조건(필수 언어, work
  authorization)을 적습니다 — 여기 걸리는 공고는 점수를 매기기 전에
  걸러집니다. 예시: [`preferences.example.md`](preferences.example.md).
- `Companies` 탭 — 관심 회사 목록은 이 폴더가 아니라 구글 시트에서 직접
  관리합니다. **Aiming** 표시(표시한 회사는 7일치 검색, 확장 지역, 채용
  페이지 매일 확인)도 이 탭에 적고, 시스템이 자동 추가한 행에는 절대 붙지
  않습니다. **Match Level**(`Strong`/`Soft`)은 `preferences.md`의 산업
  분류를 따라 적으며 주간 딥스캔의 순서를 정합니다. 자동 추가 기준은
  `config.md`의 Company discovery에서 관리합니다. 9열 계약과 각 열의 의미,
  입력 형식: [`companies.example.md`](companies.example.md) (복사하지 않는
  참고 자료).
- `config.md` — 시트 ID, 이메일 수신자, 제목, 딥스캔 요일, tier당 회사 수
  제한 + **Alert delivery** + **Company discovery**. Alert delivery에는
  알림 일정(`Alert schedule: Daily at 09:00`처럼 cron이 아닌 자연어)과 알림
  시간대(`Alert timezone`)를 적습니다. Alert timezone은 실행의 "오늘" 날짜,
  딥스캔 요일 판정, `Date Added`, 이메일·리포트 시각의 기준입니다. 예시:
  [`config.example.md`](config.example.md).

시간대 값 2가지 유의점:

- **시간대는 지역/도시 형식으로 적습니다** — `Europe/Amsterdam`,
  `Europe/Berlin`, `Asia/Seoul`, `America/New_York`. 도시 이름만, GMT
  오프셋(GMT+9), CET·KST 같은 약어는 쓰지 않습니다. 지역/도시 형식이어야 서머타임으로 오프셋이 바뀌어도 현지
  시각이 유지됩니다. `Alert timezone`(알림·실행 기준)과 `Work-hours
  timezone`(remote 판정 기준)은 값이 같을 수는 있어도 역할이 달라 합치지
  않습니다.
- **`Alert schedule` 줄 자체는 아무것도 실행하지 않습니다.** 실제 실행
  트리거는 에이전트에 등록한 예약 작업이고, 파이프라인이 이 줄을 읽어 예약을
  만들거나 바꾸는 일은 없습니다. `config.md`를 고쳐도 이미 등록된 예약은
  바뀌지 않으니 스케줄러의 예약도 같이 수정하고, 예약 후 표시되는 다음 실행
  시각이 원하는 현지 시각과 맞는지 확인하세요.

`cv.md` 변환·검토가 끝나고 3개 파일이 준비되면
[`../setup.md`](../setup.md)를 따라 시트를 만들고 `Companies` 탭을 채운 뒤
예약 작업을 등록하세요.
