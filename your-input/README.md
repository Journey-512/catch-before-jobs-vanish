# `your-input/` — your personal data lives here

**Language:** English (below) · [한국어](#your-input-한국어)

This folder is the **single source of truth** for the whole system. The scheduler
prompt is intentionally thin: at runtime it reads the four files below and does its
work based on what it finds. You configure the system by editing these files — you
never edit the scheduler prompt.

> **Privacy:** everything in this folder is gitignored except this README and
> `.gitkeep`. Your CV, preferences, company list, and config never get committed or
> pushed. See the repo root `.gitignore` for the exact rule.

## Fastest start: copy the examples

You don't have to write these from scratch. This folder ships four **filled-in
example files** for a made-up person. Copy each one and drop the `.example` from the
name, then edit it with your own details:

```bash
cp cv.example.md          cv.md
cp preferences.example.md preferences.md
cp companies.example.md   companies.md
cp config.example.md      config.md
```

(On Windows PowerShell use `Copy-Item cv.example.md cv.md`, etc.)

Now open each new file and replace the example content with yours. The skeletons and
small examples below explain what goes in each one.

## The four files

Create these four files in this folder (or copy the `.example.md` versions above).

| File | What it holds | Used for |
|---|---|---|
| `cv.md` | A career summary — roles, domains, seniority, key strengths | Fit scoring (matching postings to you) |
| `preferences.md` | Target titles, locations, industries, hard filters, score threshold | Search + filter + email cutoff |
| `companies.md` | Your target-company whitelist + auto-add rules | Tier-2 company-site checks, watchlist |
| `config.md` | Sheet ID, email recipient, schedule time, repo path | Wiring the run to your accounts |

---

### `cv.md` — skeleton

```markdown
# CV summary (for fit scoring)

## Positioning
One or two sentences: who you are professionally and the role you're targeting.

## Experience
- Role, domain, seniority, years. Repeat per role.
- Focus on domains and scope, not employer names if you'd rather keep those private.

## Strengths
- 3-6 bullet points the scorer should weight heavily.

## Domains I know well
fintech, marketplace, retail media, ... (the scorer boosts matches here)
```

> ✏️ **Filled example (excerpt):** *"A B2C product manager with 3 years on consumer
> mobile apps… targeting mid-level PM / Product Owner roles."* Full version:
> [`cv.example.md`](cv.example.md).

### `preferences.md` — skeleton

```markdown
# Preferences

## Target titles (allowlist — exact)
- Product Manager
- Product Owner
- Senior Product Manager
- ...

## Excluded titles (never match)
- Junior, Associate, Intern, Director, VP, Head, Chief, Staff, ...

## Locations
- Acceptable: <e.g., Netherlands, EU-remote, hybrid Amsterdam>
- Hard exclude: <e.g., US / Canada / Asia / Australia remote — timezone>

## Industries
- Strong: <list>
- Soft: <list>
- Avoid: <list>

## Recency
Rolling 24-hour window, strict.

## Email threshold
Include postings scoring >= 60. Sub-tiers: Top >=85, Strong 70-84, Maybe 60-69.
```

> ✏️ **Filled example (excerpt):** acceptable locations *"Netherlands, Germany,
> EU-remote"*; hard-exclude *"Asia / Australia remote (timezone)"*. Full version:
> [`preferences.example.md`](preferences.example.md).

### `companies.md` — skeleton

```markdown
# Target companies

## Whitelist
One company per line. Name | careers URL (optional) | match level (Strong/Soft)

## Auto-add rule
When a qualifying posting comes from a company NOT on this list, add it if:
- Quality gate: Series C+ (or listed / late-stage), AND
- Industry matches a Strong or Soft topic above.
Flag every auto-add in the email so you can veto it.
```

> ✏️ **Filled example (excerpt):** *"Spotify | https://www.lifeatspotify.com/jobs |
> Strong"*. Full version: [`companies.example.md`](companies.example.md).

### `config.md` — skeleton

```markdown
# Config

- Google Sheet ID: <YOUR_SHEET_ID>
- Sheet tabs: Jobs | Companies | PRD
- Email recipient: your-email@example.com
- Email subject: Catch the fresh fish before jobs vanish
- Schedule: 0 9 * * *   # 9 AM your timezone, daily
- Career-memory folder (optional): ~/path/to/career-memory/
```

> ✏️ **Filled example (excerpt):** *"Email recipient: you@example.com"*, *"Schedule:
> 0 9 * * *"*. Full version: [`config.example.md`](config.example.md).

---

Once these four files exist, follow [`../setup.md`](../setup.md) to register the
scheduled task.

---

<a name="your-input-한국어"></a>

# `your-input/` — 내 개인 데이터가 들어가는 곳 (한국어)

**언어:** [English](#your-input--your-personal-data-lives-here) · 한국어

이 폴더가 시스템 전체의 **유일한 기준점**입니다. 스케줄러 프롬프트는 일부러 얇게 만들어,
실행할 때 아래 4개 파일을 읽고 그 내용대로 동작합니다. 설정은 이 파일들을 고쳐서 하고,
스케줄러 프롬프트는 절대 손대지 않습니다.

> **개인정보:** 이 폴더 안의 모든 것은 이 README와 `.gitkeep`만 빼고 gitignore 처리됩니다.
> 이력서·선호·회사 목록·설정은 커밋되거나 푸시되지 않습니다. 정확한 규칙은 레포 루트의
> `.gitignore`를 보세요.

## 가장 빠른 시작: 예시 복사하기

처음부터 직접 쓸 필요 없습니다. 이 폴더에는 가공 인물(B2C 3년차 PM)로 **이미 채워진 예시
파일** 4개가 들어 있습니다. 각 파일을 복사해서 이름에서 `.example`만 빼고, 본인 내용으로
고치면 됩니다:

```bash
cp cv.example.md          cv.md
cp preferences.example.md preferences.md
cp companies.example.md   companies.md
cp config.example.md      config.md
```

(윈도우 PowerShell에서는 `Copy-Item cv.example.md cv.md` 처럼 쓰세요.)

복사한 파일을 열어 예시 내용을 본인 것으로 바꾸면 끝입니다. 각 파일에 뭘 넣는지는 아래
설명과 영문 섹션의 골격(skeleton)을 참고하세요.

## 4개 파일

이 폴더에 다음 4개 파일을 만듭니다 (또는 위 `.example.md` 파일을 복사).

| 파일 | 담는 내용 | 용도 |
|---|---|---|
| `cv.md` | 경력 요약 — 직무, 도메인, 연차, 핵심 강점 | 적합도 점수 (공고와 나를 매칭) |
| `preferences.md` | 목표 직무명, 지역, 산업, 하드 필터, 점수 기준선 | 검색 + 필터 + 이메일 컷오프 |
| `companies.md` | 목표 회사 화이트리스트 + 자동 추가 규칙 | 2차 회사 페이지 확인, 관심 목록 |
| `config.md` | 시트 ID, 이메일 수신자, 실행 시각, 레포 경로 | 실행을 내 계정과 연결 |

각 파일의 뼈대(skeleton)는 위 영문 섹션의 코드 블록을 그대로 쓰면 됩니다. 내용을 한국어로
적어도 동작에는 문제없지만, 직무명(예: Product Manager)처럼 실제 채용 사이트에서 검색에
쓰이는 키워드는 영어 표기를 그대로 두는 걸 권장합니다.

- `cv.md` — 점수에 쓸 내 경력 요약 (어떤 일, 도메인, 강점). 회사명을 밝히기 싫으면 도메인과
  스코프 위주로. 채워진 예시: [`cv.example.md`](cv.example.md).
- `preferences.md` — 허용 직무명(정확히), 제외 직무명, 허용/제외 지역, 산업(강/약/회피),
  최신성(엄격한 24시간), 이메일 기준선(기본 60점 이상). 예시: [`preferences.example.md`](preferences.example.md).
- `companies.md` — 회사 목록(한 줄에 하나)과 자동 추가 게이트(후기 단계 + 산업 매칭).
  예시: [`companies.example.md`](companies.example.md).
- `config.md` — 시트 ID, 이메일 수신자, 제목, cron 일정, (선택) 경력 메모 폴더 경로.
  예시: [`config.example.md`](config.example.md).

4개 파일이 준비되면 [`../setup.md`](../setup.md)를 따라 예약 작업을 등록하세요.
