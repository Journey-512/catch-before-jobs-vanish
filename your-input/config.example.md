# Config

> ⚠️ COPYABLE TEMPLATE. Copy this file to `config.md`, then:
> 1. **Replace before running:** Google Sheet ID, email recipient, alert
>    schedule, and alert timezone.
> 2. **Review and adjust:** email subject, deep-scan weekday, per-tier company
>    cap, and all Company discovery settings.
> 3. **Keep exactly:** `Sheet tabs: Jobs | Companies`.
> Replace every value in angle brackets. Values left below are deliberate
> defaults and can stay unless you want different behavior.
>
> 복사해서 쓰는 템플릿입니다. `config.md`로 복사한 뒤:
> 1. **실행 전 반드시 교체:** 구글 시트 ID, 이메일 수신자, 알림 일정, 알림
>    시간대.
> 2. **내 상황에 맞는지 확인:** 이메일 제목, 딥스캔 요일, tier당 회사 상한,
>    Company discovery 설정 전체.
> 3. **그대로 유지:** `Sheet tabs: Jobs | Companies`.
> 꺾쇠괄호 안의 값은 모두 교체하세요. 나머지 값은 의도적으로 넣어 둔
> 기본값이므로, 다른 동작을 원하지 않는다면 그대로 써도 됩니다.

- Google Sheet ID: <YOUR_GOOGLE_SHEET_ID>
- Sheet tabs: Jobs | Companies
- Email recipient: <YOUR_EMAIL_ADDRESS>
- Email subject: Catch the fresh fish before jobs vanish
- Deep-scan weekday: Tuesday — one day a week the run sweeps your Strong/Soft
  companies (token budget raised that day); "which day is today" is judged in
  the Alert timezone below.
  매주 하루, Strong/Soft 회사 전체를 훑습니다. 그날은 토큰 예산이
  늘어나며, 오늘의 요일은 아래 알림 시간대를 기준으로 판단합니다.
- Per-tier company cap: 8 — maximum companies checked per tier on the
  deep-scan day (5-10 is a sensible range).
  딥스캔 날 tier별로 확인할 최대 회사 수입니다. 5-10개가 적당합니다.

## Alert delivery · 알림 전송

- Alert schedule: <YOUR_ALERT_SCHEDULE> — e.g. Daily at 09:00, in plain
  language, not cron. Use the same wording when you register the scheduled
  task in your agent. The pipeline never reads this line to create or change
  a schedule, and editing it does not update an already-registered task.
  예: `Daily at 09:00`. cron이 아닌 자연어로 적고, 에이전트에 예약
  작업을 등록할 때도 같은 일정을 사용하세요. 이 줄 자체는 예약을 만들거나
  바꾸지 않으며, 수정해도 이미 등록된 예약은 변경되지 않습니다.
- Alert timezone: <YOUR_ALERT_TIMEZONE> — Region/City form, e.g.
  Europe/Amsterdam (keeps your local clock right across daylight saving).
  Governs the run's "today", the deep-scan weekday check, Date Added
  timestamps, and email/report times.
  `Europe/Amsterdam` 같은 지역/도시 형식으로 적습니다. 서머타임이
  바뀌어도 현지 시각을 유지하며, 실행의 오늘 날짜·딥스캔 요일·`Date Added`·
  이메일과 리포트 시각의 기준이 됩니다.

## Company discovery · 새 회사 자동 등록

- Auto-add newly discovered companies: yes — "no" still evaluates and records
  postings from unknown companies in the Jobs tab; it only stops new rows
  being added to the Companies tab.
  `no`로 설정해도 처음 보는 회사의 공고를 평가하고 Jobs 탭에
  기록합니다. Companies 탭에 새 행을 자동으로 추가하는 것만 중단합니다.
- Minimum company maturity: Series C+, listed, or clearly established
  — the quality gate a new company must clear before auto-add; tune the
  wording to your market.
  새 회사를 자동 등록하기 전에 통과해야 하는 최소 기준입니다.
  본인이 찾는 시장에 맞게 문구를 조정하세요.
- Accepted industry match: Strong or Soft — which industry classes from
  preferences.md qualify a new company. These are classes, not company names;
  the companies themselves live in the Sheet's Companies tab.
  `preferences.md`의 산업 분류 중 새 회사 등록을 허용할 범위입니다.
  회사명이 아니라 Strong/Soft 같은 분류를 적고, 실제 회사 목록은 시트의
  Companies 탭에서 관리합니다.
