# Target companies

> ⚠️ EXAMPLE. The companies below are well-known European consumer brands used
> only to show the format — they are NOT endorsements, and NOT anyone's real
> target list. Copy this file to `companies.md` and put your own list in.
> 예시입니다. 아래 회사는 형식을 보여주려고 넣은 유럽 소비자 브랜드일 뿐,
> 추천(endorsement)도, 누군가의 실제 타겟 목록도 아닙니다. `companies.md`로 복사한 뒤 본인 목록으로
> 바꾸세요.

## Whitelist
One company per line.
`Name | careers URL (optional) | Match level (Strong/Soft) | Aiming | note (optional)`

- GetYourGuide | | Strong | Aiming | activities marketplace — closest to my travel core
- Booking.com | https://careers.booking.com/ | Strong | |
- FlixBus | | Strong | | intercity travel, operations-heavy
- Bolt | https://bolt.eu/en/careers/ | Strong | | ride-hailing + rentals — my APM year domain
- Doctolib | | Soft | | health tech (Soft industry)
- Personio | | Soft | | B2B SaaS — internal tooling adjacency

## Aiming (manual flag)
`Aiming` marks the one to three companies you are actively gunning for right
now. Set it BY HAND only — the system never sets it (auto-added rows always
leave it blank). An Aiming company gets a wider net every run: a 7-day window
instead of 24h, the extended locations from `preferences.md`, and a direct
careers-page check.

## Auto-add rule
When a qualifying posting comes from a company NOT on this list, add it if:
- Quality gate: late-stage for its market — e.g. Series C+, listed, or
  clearly established. (This is the maturity bar the pipeline reads; tune the
  wording to your market.) AND
- Industry matches a Strong or Soft topic in `preferences.md`.
Every auto-add is flagged in the email so you can veto it.
