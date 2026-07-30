# `your-input/` — your personal data lives here

**Language:** English (below) · [한국어](#your-input-한국어)

This folder is the **single home for everything personal** in the system. The
whole pipeline has exactly two rule sources: the scheduler prompt (the logic —
never edited) and these four files (the data — yours). At runtime the prompt
reads this folder and does its work based on what it finds. You configure the
system by editing these files; the Google Sheet holds records, never rules.

> **Privacy:** everything in this folder is gitignored except this README,
> `.gitkeep`, and the `*.example.md` files. Your real CV, preferences, company
> list, and config never get committed or pushed. See the repo root
> `.gitignore` for the exact rule.

## Fastest start: copy the examples

You don't have to write these from scratch. This folder ships four **filled-in
example files** for a made-up person — a 5-year mobility & travel PM based in
Berlin, invented end to end for the demo. Copy each one, drop the `.example`
from the name, then replace the contents with your own details:

```bash
cp cv.example.md          cv.md
cp preferences.example.md preferences.md
cp companies.example.md   companies.md
cp config.example.md      config.md
```

(On Windows PowerShell use `Copy-Item cv.example.md cv.md`, etc.)

## The four files

| File | What it holds | Used for |
|---|---|---|
| `cv.md` | Career evidence (roles, domains, numbers) + a **Skill calibration** section | Evidence scoring — every requirement in a JD is graded against what's written here |
| `preferences.md` | Target titles, locations, industries, recency, email cutoff, home timezone, output language + an **Eligibility** section (knockouts) | Search + hard gates + email cutoff |
| `companies.md` | Target-company whitelist with Match Level and a manual **Aiming** flag + the auto-add rule | Company-tier checks, the deep-scan sweep, watchlist growth |
| `config.md` | Sheet ID, email recipient, subject, schedule, deep-scan weekday, per-tier cap | Wiring the run to your accounts |

---

### `cv.md` — skeleton

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
[`../skills/job-alert/fit-scoring-rubric.md`](../skills/job-alert/fit-scoring-rubric.md). An agent may
draft it from your CV — you confirm it (that sign-off is the point). It also
works empty: scoring then runs on the "as written, never above" meta-rule
alone. Full example: [`cv.example.md`](cv.example.md).

### `preferences.md` — skeleton

```markdown
# Preferences

## Target titles (allowlist — exact)
- Product Manager
- Senior Product Manager
- ...

## Excluded titles (never match)
- Junior, Associate, Intern, Lead, Director, VP, Head, Chief, Group, ...

## Locations
- Acceptable: <e.g., Netherlands, Germany, EU-remote, hybrid Berlin>
- Extended locations (Aiming companies only): <e.g., London>
- Hard exclude: <e.g., Asia / Australia remote — timezone>
- Remote anchor rule: exclude remote roles anchored more than N hours from
  your home timezone (default 4).
- Home timezone: <e.g., Europe/Berlin>

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

### `companies.md` — skeleton

```markdown
# Target companies

## Whitelist
Name | careers URL (optional) | Match level (Strong/Soft) | Aiming | note

## Aiming (manual flag)
Mark 1-3 companies you're actively gunning for. Hand-set only; Aiming
companies get a 7-day window, extended locations, and a daily careers check.

## Auto-add rule
Add unknown companies only past a quality gate (late-stage for its market)
AND an industry match. Every auto-add is flagged in the email for veto.
```

Full example: [`companies.example.md`](companies.example.md).

### `config.md` — skeleton

```markdown
# Config

- Google Sheet ID: <YOUR_SHEET_ID>
- Sheet tabs: Jobs | Companies
- Email recipient: your-email@example.com
- Email subject: Catch the fresh fish before jobs vanish
- Schedule: 0 9 * * *        # used when you register the task
- Deep-scan weekday: Tuesday # weekly Strong/Soft company sweep
- Per-tier company cap: 8    # deep-scan day, per tier
```

Full example: [`config.example.md`](config.example.md).

---

Once these four files exist, follow [`../setup.md`](../setup.md) to create the
Sheet and register the scheduled task.

---

<a name="your-input-한국어"></a>

# `your-input/` — 내 개인 데이터가 들어가는 곳 (한국어)

**언어:** [English](#your-input--your-personal-data-lives-here) · 한국어

이 폴더에는 **시스템이 쓰는 개인 데이터 전부**가 들어갑니다. 이 시스템에서
규칙이 들어 있는 곳은 딱 두 군데입니다. 하나는 스케줄러 프롬프트(로직 담당 —
손대지 않습니다), 다른 하나는 이 폴더의 4개 파일(내 데이터)입니다. 실행할 때
프롬프트가 이 폴더를 읽어 그 내용대로 동작하니, 설정을 바꾸려면 이 파일들만
고치면 됩니다. 구글 시트에는 기록만 쌓이고 규칙은 두지 않습니다.

> **개인정보:** 이 폴더 안의 파일은 이 README와 `.gitkeep`, `*.example.md`를
> 빼면 전부 gitignore 처리됩니다. 실제 이력서·선호·회사 목록·설정은 커밋되거나
> 푸시되지 않습니다. 정확한 규칙은 repo 루트의 `.gitignore`에서 확인하세요.

## 가장 빠른 시작: 예시 복사하기

처음부터 직접 쓸 필요 없습니다. 이 폴더에는 **이미 채워진 예시 파일** 4개가
들어 있습니다. 내용을 채운 인물은 베를린에 사는 모빌리티·여행 5년차 PM인데,
데모용으로 처음부터 끝까지 창작한 가공 인물입니다. 각 파일을 복사해서 이름에서
`.example`만 빼고 본인 내용으로 고치면 됩니다:

```bash
cp cv.example.md          cv.md
cp preferences.example.md preferences.md
cp companies.example.md   companies.md
cp config.example.md      config.md
```

(윈도우 PowerShell에서는 `Copy-Item cv.example.md cv.md`처럼 쓰세요.)

## 4개 파일

| 파일 | 담는 내용 | 용도 |
|---|---|---|
| `cv.md` | 경력 증거 (직무·도메인·수치) + **Skill calibration** 섹션 | Evidence 채점 — JD의 요구사항 하나하나를 여기 적힌 증거와 대조 |
| `preferences.md` | 목표 직무명, 지역, 산업, 최신성, 이메일 컷, 홈 타임존, 출력 언어 + **Eligibility** 섹션 (지원이 무의미해지는 조건) | 검색 + 필터 + 이메일 컷 |
| `companies.md` | 관심 회사 목록 (Match Level + 직접 적는 **Aiming** 표시) + 자동 추가 규칙 | 회사 단위 확인, 딥스캔, 관심 목록 자동 확장 |
| `config.md` | 시트 ID, 이메일 수신자, 제목, 실행 일정, 딥스캔 요일, tier당 회사 상한 | 실행을 내 계정과 연결 |

각 파일의 뼈대(skeleton)는 위 영문 섹션의 코드 블록을 그대로 가져다 쓰면
됩니다. 내용을 한국어로 적어도 동작에는 문제없지만 직무명(예: Product
Manager)처럼 실제 채용 사이트 검색에 쓰이는 키워드는 영어 표기를 그대로 두는
편이 좋습니다.

- `cv.md` — 채점에 쓸 경력 정리. 회사명을 밝히기 싫으면 "[단계 | 도메인 | 규모]"
  한 줄 소개로 대체하면 됩니다. **Skill calibration**은 스킬마다 "실제로 어디까지
  해봤는지"를 한 줄씩 적는 목록입니다 — AI 채점기가 단어만 보고 점수를 후하게
  주지 못하게 막는 장치라서 초안은 에이전트에게 시키더라도 확정은 본인이 합니다.
  비워 둬도 동작합니다. 예시: [`cv.example.md`](cv.example.md).
- `preferences.md` — 허용/제외 직무명, 지역(+ Aiming 회사에만 적용되는 확장
  지역), 산업(강/약/회피), 24시간 기준, 이메일 컷(기본 70), 홈 타임존, 출력
  언어, **Eligibility**. Eligibility에는 못 갖추면 지원 자체가 무의미해지는
  조건(필수 언어, work authorization)을 적습니다 — 여기 걸리는 공고는 점수를
  매기기 전에 걸러집니다. 예시:
  [`preferences.example.md`](preferences.example.md).
- `companies.md` — 관심 회사 목록(한 줄에 하나) + 직접 적는 **Aiming** 표시
  (표시한 회사는 7일치 검색, 확장 지역, 채용 페이지 매일 확인) + 자동 추가 규칙
  (충분히 성숙한 회사 + 산업 매칭일 때만). 예시:
  [`companies.example.md`](companies.example.md).
- `config.md` — 시트 ID, 이메일 수신자, 제목, 실행 일정, 딥스캔 요일, tier당
  회사 수 제한. 예시: [`config.example.md`](config.example.md).

4개 파일이 준비되면 [`../setup.md`](../setup.md)를 따라 시트를 만들고 예약
작업을 등록하세요.
