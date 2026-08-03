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

Do not type the angle brackets shown in the contract row. The fictional rows
below show actual cell values for three common cases:

| Index | Company | Aiming | Match Level | URL | Source site | HQ | Topics | Memo |
|---|---|---|---|---|---|---|---|---|
| 1 | Northstar Mobility | Aiming | Strong | `https://example.com/careers` | Manual research | Amsterdam | mobility; consumer | Priority target |
| 2 | Cedar Health |  | Soft |  | LinkedIn | Berlin | digital health |  |
| 3 | Orbit Travel |  | Strong |  | LinkedIn |  | travel technology | Auto-added 2026-01-15 |

All names and values above are examples. Row 1 is a manually maintained
priority company, row 2 shows that optional cells may stay blank, and row 3
shows an auto-added row: `Aiming` is blank and `Memo` records the date.

위 회사명과 값은 모두 가공된 예시입니다. 1행은 사용자가 직접 관리하는
`Aiming` 회사, 2행은 선택 항목을 빈칸으로 두는 경우, 3행은 `Aiming`을 비우고
`Memo`에 날짜를 남긴 자동 추가 행을 보여줍니다.

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

### 한국어로 보는 열 의미

- **`Aiming`** — 지금 집중해서 지원할 회사 1~3곳에만 사용자가 직접
  `Aiming`을 입력합니다. 자동 추가된 행은 항상 빈칸입니다.
- **`Match Level`** — `preferences.md`의 산업 분류에 맞춰 `Strong` 또는
  `Soft`를 입력하며, 딥스캔 순서를 정하는 데 사용됩니다.
- **`URL`** — 확인된 회사 채용 페이지 주소입니다. 모르면 추측하지 말고
  빈칸으로 두세요.
- **`Index` / `Source site` / `HQ` / `Topics` / `Memo`** — 순번, 회사를 발견한
  곳, 본사, 산업 키워드, 자유 메모입니다.

## Auto-add

The pipeline can register new companies it discovers by appending rows to
this tab (Aiming left blank, Memo noting "Auto-added" and the date); those
rows join the company searches from the next run. Whether auto-add is on,
and the maturity bar a new company must clear, are settings in `config.md`
— see the [`config.md` field guide](README.md#config-field-guide).
Existing rows are never modified: what you typed stays yours.
