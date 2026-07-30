# `Companies` tab — input reference

> ⚠️ REFERENCE, not an input file. This file shows how to fill in the
> `Companies` tab of your Google Sheet — the single source of truth for your
> company list. Do NOT copy it to a local `companies.md`: the pipeline never
> reads a local companies file. It is also not a CSV, and you don't need to
> create one — you type rows straight into the Sheet.
> 참고용이지 입력 파일이 아닙니다. 이 파일은 회사 목록의 유일한 원본인 구글
> 시트 `Companies` 탭을 채우는 방법을 보여줍니다. 로컬 `companies.md`로
> 복사하지 마세요 — 파이프라인은 로컬 회사 파일을 읽지 않습니다. CSV도
> 아니고 별도의 CSV를 만들 필요도 없습니다 — 행은 시트에 직접 입력합니다.

## The 9-column contract

The tab's header row, in this exact order:

| Index | Company | Aiming | Match Level | URL | Source site | HQ | Topics | Memo |
|---|---|---|---|---|---|---|---|---|
| 1 | `<Company name>` | `<Aiming or blank>` | `<Strong or Soft>` | `<careers URL or blank>` | `<source or blank>` | `<HQ or blank>` | `<topics or blank>` | `<note or blank>` |

One row per company. A row with an empty `Company` cell is not treated as a
company. For the optional columns, blank beats a guess — fill them only
where you actually know the value.

## What the columns mean

- **`Aiming`** — a hand-set flag for the 1-3 companies you're actively
  gunning for right now. An Aiming company gets a wider net every run: a
  7-day window instead of 24h, the extended locations from
  `preferences.md`, and a direct careers-page check. The system never sets
  this flag — auto-added rows always leave it blank.
- **`Match Level`** — `Strong` or `Soft`, mirroring the industry classes in
  `preferences.md`. It sets the sweep order on the weekly deep-scan day.
- **`URL`** — the careers page, if you know it. The pipeline uses it for
  direct checks and falls back to job boards when it is blank or broken.
- **`Index` / `Source site` / `HQ` / `Topics` / `Memo`** — bookkeeping: a
  running number, where the company was found, headquarters, industry
  keywords, and free-form notes.

## Auto-add

The pipeline can register new companies it discovers by appending rows to
this tab (Aiming left blank, Memo noting "Auto-added" and the date); those
rows join the company searches from the next run. Whether auto-add is on,
and the maturity bar a new company must clear, are settings in `config.md`
— see [`config.example.md`](config.example.md). Existing rows are never
modified: what you typed stays yours.
