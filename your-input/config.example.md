# Config

> ⚠️ EXAMPLE values. Copy this file to `config.md` and replace with your own
> Sheet ID, email, and schedule. The Sheet ID below is a fake placeholder.
> 예시 값입니다. `config.md`로 복사한 뒤 본인 시트 ID·이메일·일정으로 바꾸세요.
> 아래 시트 ID는 가짜 자리표시자입니다.

- Google Sheet ID: abcde-REPLACE-WITH-YOUR-OWN-SHEET-ID-12345
- Sheet tabs: Jobs | Companies
- Email recipient: you@example.com
- Email subject: Catch the fresh fish before jobs vanish
- Schedule: 0 9 * * *   # daily at 9 AM your timezone — used when you REGISTER
                        # the scheduled task; the prompt itself never reads it
- Deep-scan weekday: Tuesday   # one day a week the run sweeps your Strong/Soft
                               # companies (token budget raised that day)
- Per-tier company cap: 8      # max companies checked per tier on the
                               # deep-scan day (5-10 is a sensible range)
