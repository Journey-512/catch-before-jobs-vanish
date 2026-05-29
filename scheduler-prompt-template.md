# Scheduler prompt template

**Language:** English (below) · [한국어](#스케줄러-프롬프트-템플릿-한국어)

This is the prompt the scheduled task runs every morning. **Do not fill in any
placeholders here** — this prompt deliberately contains no personal data. It reads
everything it needs from `your-input/` at runtime.

Copy the block below into your scheduled task (see [`setup.md`](setup.md)). The only
edit you make is the repo path on the first line, so it knows where to find
`your-input/`.

---

```text
You are running the daily job-alert routine for the "catch-before-jobs-vanish"
system. Today is the current date. Work autonomously end to end and produce one
email.

## Step 0 — Load configuration (single source of truth)
Read these four files from the repo's your-input/ folder
(repo path: <REPLACE WITH YOUR LOCAL REPO PATH>/your-input/):
  - cv.md          -> the operator's profile, for fit scoring
  - preferences.md -> target titles, excluded titles, locations, industries,
                      recency window, email threshold
  - companies.md   -> target-company whitelist + auto-add rule
  - config.md      -> Sheet ID, email recipient, subject, schedule
If any file is missing, stop and report which one — do not guess.
Also read the PRD tab of the Google Sheet (ID from config.md) for any rule that
has been updated since these files were written. The Sheet PRD wins on conflict.

## Step 1 — Search (tiered, cost-aware)
Tier 1 (primary, do every run):
  - Indeed: query each target title from preferences.md, filtered to the last 24h.
  - LinkedIn: same titles, location filter from preferences.md, f_TPR=r86400.
    On the results page, extract per-posting jobIds and build permalink URLs
    (jobs/view/{id}). NEVER save a search URL — it is a moving 24h window.
    Do NOT trust LinkedIn's auto-selected currentJobId; parse the full result list
    and match by company + title.
Tier 2 (fallback, only when a Strong-match company from companies.md didn't surface
    in Tier 1): check that company's career page. Treat company pages as best-effort
    — tolerate timeouts/404s, don't burn budget verifying fragile URLs.
Respect the token guidance: target ~25-30K tokens, hard cap 50K. Skip low-value
Tier 2 work if you're near the cap.

## Step 2 — Filter
  - Title must be on the allowlist AND not on the excluded list (preferences.md).
  - Posting must be within the strict rolling 24h window. Parse "X hours ago" /
    "Yesterday" / "Today" into an absolute YYYY-MM-DD at capture time. NEVER store
    "Last 24h".
  - Apply hard location filter from preferences.md. Non-acceptable locations
    (e.g. non-EU remote) are EXCLUDED from the email regardless of score — but still
    written to the Jobs sheet with Status "Excluded (location)" for the record.
  - Deduplicate by company + similar title within 48h. If a posting appears on
    multiple sources, concatenate the Source field ("LinkedIn, Indeed, Company site").

## Step 3 — Score (100-point rubric)
Score every surviving posting against cv.md using the rubric in
docs/fit-scoring-rubric.md:
  Domain Fit 30 | Role Fit 25 | Seniority Scope 20 | Company Quality 15 |
  Location/Logistics 10.
Record the score and a one-line reason per posting.

## Step 4 — Auto-add new companies
For any qualifying posting from a company NOT on the whitelist, apply the auto-add
gate from companies.md (quality gate AND industry match). If it passes, append the
company to the Companies tab and flag it in the email ("New company added: X —
OK?"). If it fails the gate, leave it out of the watchlist but it may still appear
in the email if it scored above threshold.

## Step 5 — Write to the Sheet
Append all postings (including excluded ones) to the Jobs tab with columns:
Date Added | Company | Job Title | Location | Source | Posted Date (YYYY-MM-DD) |
Link | Status | Fit Score | Fit Reason.
Then sort the Jobs tab by Date Added descending (newest first).

## Step 6 — Email
Build an HTML email to the recipient in config.md with the subject from config.md.
Include only postings scoring >= the threshold (default 60), grouped:
  - Top Match (>=85)
  - Strong Match (70-84)
  - Maybe Fit (60-69)
  - New Companies Added (with veto prompt)
  - Link to the Sheet
If your email tool only supports drafts, create the draft, then send it via browser
automation (navigate to Drafts, click Send). Draft-first gives a fallback if send
fails.

## Step 7 — Report
End with a one-line summary: N found, M emailed, K auto-added companies.
```

---

## Why this prompt is thin

All personal data lives in `your-input/` and the Sheet PRD tab. The benefits:

- **Inspectable** — rules live in plain files and a spreadsheet you already use,
  not buried in a prompt.
- **Updatable** — change a target title by editing `preferences.md` or a Sheet cell;
  no need to regenerate the scheduler.
- **Shareable** — this prompt contains zero personal data, so it can live in a
  public repo unchanged.

---

<a name="스케줄러-프롬프트-템플릿-한국어"></a>

# 스케줄러 프롬프트 템플릿 (한국어)

**언어:** [English](#scheduler-prompt-template) · 한국어

이건 예약 작업이 매일 아침 실행하는 프롬프트입니다. **여기에 개인정보를 채우지 마세요** —
이 프롬프트엔 일부러 개인 데이터가 하나도 없습니다. 필요한 건 전부 실행 시점에
`your-input/`에서 읽어옵니다.

위(영문 섹션)의 코드 블록을 그대로 복사해 예약 작업에 넣으세요 ([`setup.md`](setup.md) 참고).
**직접 고치는 곳은 딱 한 군데** — 첫 줄(Step 0)의 레포 경로뿐입니다. 거기에 `your-input/`이
있는 실제 위치를 적어주면 됩니다. 예:

```
repo path: C:\Users\<본인>\...\catch-before-jobs-vanish\public-repo/your-input/
```

이 한 줄 외에 본문은 전혀 손대지 않습니다. 직무·지역·점수 기준·이메일 주소 같은 건 모두
실행할 때 `your-input/`의 4개 파일에서 자동으로 읽어옵니다.

프롬프트 블록 자체를 영어로 둔 이유: 에이전트 실행용 지시문이라 영어로 두는 게 안정적입니다.
내용(취향)은 한국어로 적어도 되는 `your-input/` 쪽에서 정하면 됩니다.

## 이 프롬프트가 얇은 이유

개인 데이터는 전부 `your-input/`과 시트 PRD 탭에 둡니다. 장점:

- **들여다보기 쉬움** — 규칙이 프롬프트에 묻혀 있지 않고, 평범한 파일과 늘 쓰던 시트에 있음.
- **고치기 쉬움** — 직무 하나 추가하려면 `preferences.md`나 시트 셀만 고치면 됨. 스케줄러를
  다시 만들 필요 없음.
- **공유하기 쉬움** — 이 프롬프트엔 개인정보가 없어서 공개 레포에 그대로 둘 수 있음.
