# Config

> ⚠️ EXAMPLE values. Copy this file to `config.md` and replace with your own
> Sheet ID, email, and schedule. The Sheet ID below is a fake placeholder.
> 예시 값입니다. `config.md`로 복사한 뒤 본인 시트 ID·이메일·일정으로 바꾸세요.
> 아래 시트 ID는 가짜 자리표시자입니다.

- Google Sheet ID: abcde-REPLACE-WITH-YOUR-OWN-SHEET-ID-12345
- Sheet tabs: Jobs | Companies
- Email recipient: you@example.com
- Email subject: Catch the fresh fish before jobs vanish
- Deep-scan weekday: Tuesday   # one day a week the run sweeps your Strong/Soft
                               # companies (token budget raised that day);
                               # "which day is today" is judged in the Alert
                               # timezone below
- Per-tier company cap: 8      # max companies checked per tier on the
                               # deep-scan day (5-10 is a sensible range)

## Alert delivery

- Alert schedule: Daily at 09:00
  # plain language, not cron — you repeat this when you REGISTER the
  # scheduled task in your agent; the pipeline never reads it to create or
  # change a schedule, and editing this line does not update an
  # already-registered task
- Alert timezone: Asia/Seoul
  # Region/City form (keeps your local clock right across daylight saving).
  # Governs the run's "today", the deep-scan weekday check, Date Added
  # timestamps, and email/report times

## Company discovery

- Auto-add newly discovered companies: yes
  # "no" still evaluates and records postings from unknown companies in the
  # Jobs tab — it only stops new rows being added to the Companies tab
- Minimum company maturity: Series C+, listed, or clearly established
  # the quality gate a new company must clear before auto-add; tune the
  # wording to your market
- Accepted industry match: Strong or Soft
  # which industry classes from preferences.md qualify a new company —
  # classes, not company names; the companies themselves live in the
  # Sheet's Companies tab
