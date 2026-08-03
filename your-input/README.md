<a name="your-input-english"></a>

# `your-input/` — your personal data lives here

**Language:** English · [한국어](#your-input-한국어)

Use this guide after completing Step 3 in [`setup.md`](../setup.md). Setup
explains how to create and copy the files; this page explains **what to put in
them**.

The daily pipeline reads three local files — `cv.md`, `preferences.md`, and
`config.md` — plus the `Companies` tab in your Google Sheet. The scheduled
prompt contains the fixed logic; do not put personal settings in it. The
company list lives only in the Sheet, not in a local file.

> **Privacy:** everything in this folder is ignored by Git except this README,
> `.gitkeep`, the two templates, and the `cv` / `Companies` reference examples.
> Your real CV and settings are not committed or pushed. The company list stays
> in your Google Sheet. See [`.gitignore`](../.gitignore) for the exact rules.

## Inputs at a glance

| Input | What you own | What the pipeline uses it for |
|---|---|---|
| `cv.md` | Career evidence and your reviewed `Skill calibration` | Evidence-based fit scoring |
| `preferences.md` | Titles, locations, eligibility, industries, recency, cutoff, and output language | Search, hard filters, location checks, and email inclusion |
| `config.md` | Sheet and email connection, alert delivery, deep-scan, and company discovery settings | Running and reporting the alert |
| Sheet: `Companies` | Company names, `Aiming`, match level, careers URLs, and notes | Company-specific and deep-scan searches |

`cv-original.md` is a private, one-time source file used to create or update
`cv.md`. The daily pipeline never reads it.

## `cv.md` — create it from your real CV

1. Create `your-input/cv-original.md` and paste the complete text of your
   existing CV under this heading. Plain text is fine; no Markdown cleanup is
   required.

   ```markdown
   # Original CV

   <Paste the complete text of your existing CV here>
   ```

   If you only have a PDF or DOCX, ask your agent to extract all of its text
   into `cv-original.md` first.
2. Give your agent the conversion request below.
3. Review the result against your original CV using the checklist below.
4. Register the scheduled task only after `cv.md` is approved.

The request is safe for both first-time setup and later updates. On the first
run it creates `cv.md`. If a reviewed `cv.md` already exists, it creates
`cv.generated.md` instead and leaves the live scoring file untouched.

**Conversion request (copy and paste):**

```text
Read `your-input/cv-original.md`.

If `your-input/cv.md` does not exist, create it.
If `your-input/cv.md` already exists, do not overwrite it. Create
`your-input/cv.generated.md`, compare it with the existing `cv.md`, and list
the proposed changes for my review.

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

If an existing `cv.md` contains owner-reviewed Skill calibration, preserve it
unless the updated CV provides explicit evidence for a change. Put any proposed
calibration change in the review summary; do not silently replace it.

If evidence is missing or ambiguous, omit the claim instead of guessing.
Do not anonymize company names unless I explicitly ask.
After creating the file, show me what I need to review before it is used.
```

**Review before using the file:**

1. Company names, titles, dates, and numbers match the original.
2. No experience, achievement, or skill was invented or inflated.
3. `Strengths` and `Domains I know well` stay within the written evidence.
4. The `Skill calibration` Core / Transferable / Gap split matches your actual
   level of experience.

For a later update, compare `cv.generated.md` with the existing `cv.md`. Merge
only the changes you approve, then keep the reviewed result at
`your-input/cv.md`; the pipeline ignores `cv.generated.md`.

`Skill calibration` prevents the scorer from crediting experience above the
level you can support. The agent may draft it, but you approve it and continue
to adjust it from real scoring results. See the
[fit-scoring rubric](../skills/job-alert/fit-scoring-rubric.md) and the
[fictional output example](cv.example.md).

## `preferences.md` — field guide

[`preferences.template.md`](preferences.template.md) is the runtime skeleton.
Keep its headings and field labels, replace every `<PLACEHOLDER>`, and add more
list rows when a section needs several values. Use `NONE` only for optional
fields with no value; do not leave placeholders in the finished file.

| Field | How to write it | Example or rule |
|---|---|---|
| Target titles | One exact job title per list row | `- Product Manager` and `- Senior Product Manager` |
| Excluded titles | One word or title fragment per list row; use `NONE` if there are none | `- Intern`, `- Director`, `- Product Owner` |
| Acceptable locations | A semicolon-separated list, ordered from most preferred to broadest acceptable | `Amsterdam; Netherlands; European Union; Remote within Europe` |
| Extended locations | Extra locations allowed only for `Aiming` companies; otherwise `NONE` | `London; Berlin` |
| Hard exclude | Places you will not accept under any circumstances; otherwise `NONE` | `United States; Canada` |
| Work-hours timezone | The timezone whose working hours you intend to keep, in Region/City form | `Europe/Amsterdam` |
| Maximum timezone difference | The largest acceptable gap from your work-hours timezone | Default: `4 hours` |
| Languages I work in | Every language you can actually use at work, including level when useful | `Korean (native); English (professional)` |
| Work authorization | State authorization and sponsorship needs by region | `EU: authorized, no sponsorship; UK: sponsorship required` |
| Strong industries | Industries where you have strong evidence or clear preference | `Travel; Mobility; Consumer marketplaces` |
| Soft industries | Adjacent industries you would consider; otherwise `NONE` | `Fintech; Digital health` |
| Avoid industries | Industries to exclude; otherwise `NONE` | `Gambling; Tobacco` |
| Rolling window | Keep the freshness window at `24 hours` | Aiming companies use the wider window defined by the pipeline |
| Cutoff | Whole-number minimum score for the email | Default: `70`; lower means more email results |
| Sheet and email language | The single language to use for all generated Sheet and email text | `English` or `Korean` |

The **Eligibility** fields are knockout conditions, not scoring preferences.
An explicitly required language you do not work in or missing work
authorization can remove a posting before scoring, so describe these fields
factually rather than aspirationally.

The two timezone settings have different jobs. `Work-hours timezone` here is
used only to judge remote-role compatibility. `Alert timezone` in `config.md`
controls dates, weekdays, and report times. Both must use Region/City form, such
as `Europe/Amsterdam`, `Asia/Seoul`, or `America/New_York`.

## The `Companies` tab — company input guide

Enter and maintain your company list directly in the Google Sheet. The
pipeline reads the `Companies` tab at the start of every run and never reads a
local `companies.md` file.

You can add companies manually, and the pipeline can append qualifying new
companies when auto-add is enabled in `config.md`. Auto-added rows always leave
`Aiming` blank; existing rows and values you entered are never overwritten.
See [`companies.example.md`](companies.example.md) for the exact nine columns,
three fictional row examples, and each field's behavior.

<a name="config-field-guide"></a>

## `config.md` — field guide

[`config.template.md`](config.template.md) contains four placeholders. Replace
all four and keep `Sheet tabs: Jobs | Companies` exactly as written.

| Field | How to write it | Example or rule |
|---|---|---|
| Google Sheet ID | Copy only the string between `/d/` and `/edit` in the Sheet URL | Do not paste the full URL |
| Sheet tabs | Keep the exact tab names | `Jobs` and `Companies` |
| Email recipient | The address that receives the digest | `name@example.com` |
| Email subject | Any subject you will recognize | The template value is ready to use |
| Deep-scan weekday | A weekday name in English | Default: `Tuesday` |
| Per-tier company cap | A positive whole number | Default: `8` |
| Alert schedule | Plain language matching the task registered in your agent | `Daily at 09:00` |
| Alert timezone | Region/City timezone used for run dates and report times | `Asia/Seoul` |
| Auto-add newly discovered companies | `yes` or `no` | Default: `yes` |
| Minimum company maturity | A plain-language minimum for auto-add | Default: `Series C+, listed, or clearly established` |
| Accepted industry match | Which preference classes may be auto-added | `Strong or Soft`; use `Strong` for a narrower list |

`Alert schedule` is a record for you and your scheduler; the pipeline does not
create or change a scheduled task from this line. If you edit the schedule or
alert timezone later, update the task registered in your agent too and verify
its next-run time.

## Ready to continue

Before returning to Step 4 in [`setup.md`](../setup.md), confirm:

1. `cv.md`, `preferences.md`, and `config.md` exist.
2. No `<PLACEHOLDER>` remains in `preferences.md` or `config.md`.
3. `cv.md` has been reviewed against the original CV.
4. The Sheet has exact `Jobs` and `Companies` tab names, and the `Companies`
   header follows [`companies.example.md`](companies.example.md).

---

<a name="your-input-한국어"></a>

# `your-input/` — 내 개인 데이터를 입력하는 곳 (한국어)

**언어:** [English](#your-input-english) · 한국어

[`setup.md`](../setup.md)의 3단계를 완료한 뒤 이 안내를 사용하세요. 파일을
만들고 복사하는 방법은 Setup에서 설명하고, 이 문서는 각 파일에 **무엇을 어떤
형식으로 입력할지** 설명합니다.

매일 실행되는 파이프라인은 로컬의 `cv.md`, `preferences.md`, `config.md`와
구글 시트의 `Companies` 탭을 읽습니다. 예약 작업의 프롬프트에는 고정 로직만
두고 개인 설정을 넣지 마세요. 회사 목록은 로컬 파일이 아니라 시트에서만
관리합니다.

> **개인정보:** 이 README, `.gitkeep`, 템플릿 2개와 `cv` / `Companies` 참고
> 예시를 제외한 이 폴더의 모든 파일은 Git에서 제외됩니다. 실제 CV와 설정은
> 커밋되거나 푸시되지 않고, 회사 목록은 구글 시트에 남습니다. 정확한 규칙은
> [`.gitignore`](../.gitignore)에서 확인할 수 있습니다.

## 입력 항목 한눈에 보기

| 입력 | 사용자가 관리하는 내용 | 파이프라인의 사용 목적 |
|---|---|---|
| `cv.md` | 경력 증거와 사용자가 검토한 `Skill calibration` | 증거 기반 적합도 채점 |
| `preferences.md` | 직무, 지역, 지원 자격, 산업, 최신성, 기준 점수, 출력 언어 | 검색, 필터, 지역 판정, 이메일 포함 여부 |
| `config.md` | 시트·이메일 연결, 알림, 딥스캔, 회사 자동 발견 설정 | 알림 실행과 결과 보고 |
| 시트의 `Companies` 탭 | 회사명, `Aiming`, 적합 산업, 채용 페이지, 메모 | 회사별 검색과 딥스캔 |

`cv-original.md`는 `cv.md`를 처음 만들거나 갱신할 때만 사용하는 비공개 원본
파일입니다. 매일 실행되는 파이프라인은 이 파일을 읽지 않습니다.

## `cv.md` 만들기 (한국어 안내)

1. `your-input/cv-original.md`를 만들고 아래 제목 밑에 기존 이력서의 전체
   텍스트를 붙여넣으세요. 일반 텍스트면 충분하며 Markdown으로 다시 정리할
   필요는 없습니다.

   ```markdown
   # Original CV

   <여기에 기존 이력서의 전체 텍스트를 붙여넣으세요>
   ```

   PDF나 DOCX만 있다면 먼저 에이전트에게 전체 내용을 빠짐없이 추출해
   `cv-original.md`에 저장해 달라고 요청하세요.
2. 아래 변환 요청문을 에이전트에게 전달하세요.
3. 아래 체크리스트를 사용해 결과를 원본 이력서와 대조하세요.
4. `cv.md` 검토가 끝난 뒤에만 예약 작업을 등록하세요.

이 요청문은 최초 설정과 이후 갱신을 모두 안전하게 처리합니다. 처음에는
`cv.md`를 만들고, 이미 검토된 `cv.md`가 있다면 그 파일을 덮어쓰지 않고
`cv.generated.md`를 만듭니다.

**변환 요청문 (복사해서 사용):**

```text
`your-input/cv-original.md`를 읽어줘.

`your-input/cv.md`가 없다면 새로 만들어줘.
기존 `your-input/cv.md`가 있다면 덮어쓰지 말고
`your-input/cv.generated.md`를 만들어줘. 기존 `cv.md`와 비교해 변경 제안을
내가 검토할 수 있게 정리해줘.

`your-input/cv.example.md`는 출력 구조 참고용으로만 사용하고, 가공 인물의
사실은 절대 가져오지 마.

결과 구조:

# CV summary (for fit scoring)
## Positioning
## Experience
## Strengths
## Domains I know well
## Skill calibration

회사·직책·날짜·수치·성과·skill은 `cv-original.md`가 뒷받침하는 그대로
보존해. 원문에 없는 경험을 지어내거나 부풀리거나 추론하지 마.

타깃 직무와 산업을 이해하기 위해 `preferences.md`를 읽어도 되지만, 선호는
맥락일 뿐 경력 증거가 아니야.

Skill calibration은 CV의 명시적 증거를 바탕으로 보수적으로 작성해줘:
- Core: 직접적이고 반복된 경험
- Transferable: 부분적이거나 인접한 경험과 정직한 상한
- Gap: CV에 적혀 있지 않다는 이유만으로 Gap이라고 추정하지 말 것

기존 `cv.md`에 사용자가 검토한 Skill calibration이 있다면, 갱신된 CV에
변경을 뒷받침하는 명시적 증거가 없는 한 그대로 보존해. 바꿀 필요가 있는
항목은 검토 요약에서 별도로 제안하고 조용히 교체하지 마.

증거가 없거나 모호한 주장은 추측하지 말고 빼줘.
내가 명시적으로 요청하지 않는 한 회사명을 익명화하지 마.
파일을 만든 뒤, 사용하기 전에 내가 검토해야 할 부분을 보여줘.
```

**사용 전 검토 체크리스트:**

1. 회사명, 직책, 날짜, 수치가 원본과 일치하는가?
2. 새로운 경험·성과·skill이 만들어지거나 부풀려지지 않았는가?
3. `Strengths`와 `Domains I know well`이 실제 증거 안에 머무르는가?
4. `Skill calibration`의 Core / Transferable / Gap 분류가 실제 경험 수준과
   맞는가?

이후 CV를 갱신할 때는 `cv.generated.md`와 기존 `cv.md`를 비교하세요. 승인한
변경만 병합한 뒤 검토된 최종 파일을 `your-input/cv.md`로 유지합니다.
파이프라인은 `cv.generated.md`를 읽지 않습니다.

`Skill calibration`은 채점기가 사용자의 증거보다 높은 수준의 경험을
인정하지 못하게 하는 장치입니다. 에이전트가 초안을 만들 수 있지만 사용자가
확정하고, 실제 채점 결과를 보면서 계속 조정합니다. 자세한 원리는
[한국어 채점 기준표](../docs/fit-scoring-rubric.ko-KR.md), 완성 형태는
[가공 CV 예시](cv.example.md)를 참고하세요.

## `preferences.md` 필드 안내

[`preferences.template.md`](preferences.template.md)가 실행에 사용되는
뼈대입니다. 제목과 필드 이름은 유지하고, 모든 `<자리표시자>`를 실제 값으로
바꾸세요. 값이 여러 개면 목록 행을 추가합니다. 선택 항목에 값이 없을 때만
`NONE`을 사용하고, 완성된 파일에는 자리표시자를 남기지 마세요.

| 필드                          | 입력 방법                                   | 예시 또는 규칙                                                       |
| --------------------------- | --------------------------------------- | -------------------------------------------------------------- |
| Target titles               | 목록 행 하나에 정확한 직무명 하나                     | `- Product Manager`, `- Senior Product Manager`                |
| Excluded titles             | 목록 행 하나에 제외할 단어나 직무명 조각 하나. 없으면 `NONE`  | `- Intern`, `- Director`, `- Product Owner`                    |
| Acceptable locations        | 가장 선호하는 곳부터 넓은 허용 범위 순서로 세미콜론(`;`)으로 구분 | `Amsterdam; Netherlands; European Union; Remote within Europe` |
| Extended locations          | `Aiming` 회사에만 추가로 허용할 지역. 없으면 `NONE`    | `London; Berlin`                                               |
| Hard exclude                | 어떤 경우에도 지원하지 않을 지역. 없으면 `NONE`          | `United States; Canada`                                        |
| Work-hours timezone         | 실제로 유지하려는 근무 시간대를 지역/도시 형식으로 입력         | `Europe/Amsterdam`                                             |
| Maximum timezone difference | 근무 시간대와 허용할 최대 시차                       | 기본값 `4 hours`                                                  |
| Languages I work in         | 실제 업무가 가능한 언어를 모두 적고 필요하면 수준도 표시        | `Korean (native); English (professional)`                      |
| Work authorization          | 지역별 취업 허가와 비자 후원 필요 여부를 구체적으로 작성        | `EU: authorized, no sponsorship; UK: sponsorship required`     |
| Strong industries           | 경력 증거가 강하거나 우선순위가 높은 산업                 | `Travel; Mobility; Consumer marketplaces`                      |
| Soft industries             | 인접해 있어 검토할 산업. 없으면 `NONE`               | `Fintech; Digital health`                                      |
| Avoid industries            | 제외할 산업. 없으면 `NONE`                      | `Gambling; Tobacco`                                            |
| Rolling window              | 최신성 기준은 `24 hours`로 유지                  | `Aiming` 회사에는 파이프라인의 확장 기간이 적용됨                                |
| Cutoff                      | 이메일에 포함할 최소 점수를 정수로 입력                  | 기본값 `70`; 낮추면 이메일 결과가 늘어남                                      |
| Sheet and email language    | 시트와 이메일의 생성 문구에 사용할 언어 하나               | `English` 또는 `Korean`                                          |

**Eligibility** 필드는 점수 선호가 아니라 탈락 조건입니다. 업무가 불가능한
언어가 필수이거나 취업 허가가 없으면 채점 전에 공고가 제외될 수 있으므로,
희망이 아니라 현재 사실을 적으세요.

시간대 필드 두 개는 역할이 다릅니다. 여기의 `Work-hours timezone`은 remote
공고의 근무 가능 여부만 판단합니다. `config.md`의 `Alert timezone`은 실행
날짜·요일과 보고 시각을 정합니다. 둘 다 `Europe/Amsterdam`, `Asia/Seoul`,
`America/New_York` 같은 지역/도시 형식을 사용하세요.

## `Companies` 탭 입력 안내

회사 목록은 구글 시트에서 직접 입력하고 관리합니다. 파이프라인은 매 실행
시작 시 `Companies` 탭을 읽으며 로컬 `companies.md` 파일은 읽지 않습니다.

회사는 사용자가 직접 추가할 수 있고, `config.md`에서 자동 추가를 켜면 조건을
통과한 새 회사가 파이프라인을 통해 추가될 수도 있습니다. 자동 추가된 행의
`Aiming`은 항상 빈칸이며, 기존 행과 사용자가 입력한 값은 덮어쓰지 않습니다.
정확한 9개 열, 가공된 행 예시 3개와 각 필드의 동작은
[`companies.example.md`](companies.example.md)를 참고하세요.

## `config.md` 필드 안내

[`config.template.md`](config.template.md)에는 자리표시자 4개가 있습니다. 모두
실제 값으로 바꾸고 `Sheet tabs: Jobs | Companies`는 그대로 유지하세요.

| 필드 | 입력 방법 | 예시 또는 규칙 |
|---|---|---|
| Google Sheet ID | 시트 URL의 `/d/`와 `/edit` 사이 문자열만 복사 | 전체 URL을 붙여넣지 않음 |
| Sheet tabs | 정확한 탭 이름 유지 | `Jobs`와 `Companies` |
| Email recipient | 결과 이메일을 받을 주소 | `name@example.com` |
| Email subject | 알아보기 쉬운 이메일 제목 | 템플릿 기본값을 그대로 사용해도 됨 |
| Deep-scan weekday | 영어 요일 이름 | 기본값 `Tuesday` |
| Per-tier company cap | 1 이상의 정수 | 기본값 `8` |
| Alert schedule | 에이전트에 등록한 예약 작업과 같은 내용을 자연어로 입력 | `Daily at 09:00` |
| Alert timezone | 실행 날짜와 보고 시각의 기준이 되는 지역/도시 시간대 | `Asia/Seoul` |
| Auto-add newly discovered companies | `yes` 또는 `no` | 기본값 `yes` |
| Minimum company maturity | 자동 추가할 회사의 최소 성숙도를 자연어로 입력 | 기본값 `Series C+, listed, or clearly established` |
| Accepted industry match | 자동 추가를 허용할 선호 산업 등급 | `Strong or Soft`; 더 좁히려면 `Strong` |

`Alert schedule`은 사용자와 예약 실행 도구를 위한 기록이며, 이 줄만으로 예약
작업이 만들어지거나 바뀌지는 않습니다. 나중에 일정이나 `Alert timezone`을
수정하면 에이전트에 등록한 예약 작업도 함께 수정하고 다음 실행 시각을
확인하세요.

## 다음 단계로 넘어가기 전

[`setup.md`](../setup.md)의 4단계로 돌아가기 전에 다음을 확인하세요.

1. `cv.md`, `preferences.md`, `config.md`가 모두 존재한다.
2. `preferences.md`와 `config.md`에 `<자리표시자>`가 남아 있지 않다.
3. `cv.md`를 원본 이력서와 대조해 검토했다.
4. 시트 탭 이름이 정확히 `Jobs`, `Companies`이고, `Companies` 헤더가
   [`companies.example.md`](companies.example.md)의 구조와 일치한다.
