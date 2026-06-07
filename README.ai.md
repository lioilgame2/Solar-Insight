# README.ai.md - Deye Solar Insight Technical Reference

This file is an English-only technical handoff for AI assistants and developers working on `app.html`.

It should stay consistent with:

- `README.md`: public user guide and project overview
- `app.html`: the actual single-file application and in-app guide

## 1. Project Snapshot

| Item | Current State |
|---|---|
| Project | Deye Solar Insight |
| Main file | `app.html` |
| App type | Offline-first browser dashboard |
| Stack | HTML / CSS / Vanilla JavaScript / Chart.js / SheetJS / IndexedDB |
| UI language | Thai |
| Documentation language | `README.md` is Thai first with English below; `README.ai.md` is English only |
| Primary input | XLSX exports from Deye hybrid inverter reports |
| Compatibility scope | Built and tested mainly for Deye XLSX exports; other inverter brands are not guaranteed |
| Storage | Browser IndexedDB database `SolarInsight`, with localStorage fallback for settings |
| Best device | Desktop/laptop; not mobile-first |
| PV string comparison | Up to 2 strings |
| Meter mode | Best for TOU meters; pre-TOU periods use flat-rate billing |

Do not hardcode current file size, line count, or gzip size in documentation. `app.html` changes often.

## 2. Architecture

The project intentionally stays as a single static HTML app.

```text
app.html
├── head
│   ├── CSS variables and layout styles
│   ├── SheetJS XLSX CDN
│   └── Chart.js CDN
├── body
│   ├── main navigation tabs
│   ├── daily / weekly / monthly / yearly / all-time panes
│   ├── analysis pane
│   ├── in-app guide pane
│   ├── system settings modal
│   ├── bill history modal
│   ├── file manager modal
│   └── first-run welcome modal
└── script
    ├── configuration constants
    ├── IndexedDB and settings helpers
    ├── XLSX parsing
    ├── daily summary calculation
    ├── real bill calibration and K-Factor projection
    ├── render functions
    ├── Chart.js helpers/plugins
    ├── calendar heatmap renderers
    ├── ROI and simulation logic
    └── backup import/export helpers
```

Avoid splitting into multiple modules unless the user explicitly asks for that direction.

## 3. User-Facing Documentation Contract

The three user-facing documentation surfaces must agree:

| Surface | Role |
|---|---|
| `README.md` | Public user guide, Thai first and English below |
| in-app guide tab | Short onboarding guide inside the app |
| first-run welcome modal | Very short start guidance for GitHub Pages visitors |

Required shared claims:

- The app is built and tested mainly with Deye XLSX exports.
- Other inverter brands may work only if columns and units are similar.
- The app is best for TOU users, but pre-TOU periods are supported with flat-rate billing.
- Data is stored in the current browser; it is not sent to a project server.
- Clearing browser storage can remove saved data.
- Using another browser or another machine will not automatically carry data over.
- The current UI is best on desktop/laptop, not mobile-first.
- PV string comparison currently supports up to 2 strings.
- Simulation results are directional estimates, not guarantees.
- Battery empty %, recharge %, and design cycles are configurable in system settings.
- Manual system settings and bill history can be backed up via Export backup / Import backup.
- Backup files do not include imported inverter rows; users should keep original XLSX files.

## 4. Data Flow

```text
User imports XLSX
  ↓
parseXLSX(buffer)
  ↓
detectC(firstRow)
  ↓
group rows by YYYY-MM-DD
  ↓
calcSum(rows, date)
  ↓
dbPut({ date, rows, summary, cols })
  ↓
refresh()
  ↓
dbAll()
  ↓
computeCycleCalibrations(validBills, getP())
  ↓
applyCalibration(summary, date)
  ↓
renderDay / renderWeek / renderMonth / renderYear / renderAllTime / renderAnalysis
```

`dbAll()` recomputes calibration on read. IndexedDB stores raw imported daily summaries and rows; calibrated currency values are applied in memory.

## 5. Core Configuration

### `CFG`

Representative defaults:

```javascript
const CFG = {
  sys: {
    panels: 10,
    wp: 645,
    peakSunH: 4.5,
    battKwh: 10.24,
    battAmp: 48,
    battModel: 'PYLON LiFePO4',
    invKw: 5,
    battEmptyPct: 21,
    battRechargePct: 25,
    battCycles: 6000,
    pvModel: 'LONGi LR7-72HYD-645M',
    pvEff: 23.9,
  },
  startDate: '2026-04-10',
  touStart: '2026-05-05',
  billDay: 15,
  systemCost: 152350,
  costNote: { solar: 209000, tou: 3350, taxDeduction: 60000 },
  pvSpec: { pmax: 645, voc: 54.12, isc: 15.06, vmp: 44.77, imp: 14.41 },
  strings: { aLabel: 'String A', aSpec: '5 West', bLabel: 'String B', bSpec: '3 South + 2 West' },
  partialDays: ['2026-04-10', '2026-05-16', '2026-05-17'],
};
```

System settings metadata also includes:

```javascript
{
  appName: 'Deye Solar Insight',
  inflationRate: 2.89,
  showSimulation: false,
  stringA: '5 West',
  stringB: '3 South + 2 West',
}
```

`showSimulation` defaults to `false` for new or cleared system profiles because simulation assumptions are system-specific.

### Electricity Rates

`getP()` reads current UI values and falls back to defaults:

```javascript
{
  on:   5.7982,
  off:  2.6369,
  flat: 4.4217,
  ft:   0.0972,
  srv:  24.62,
  sell: 2.20,
}
```

### Thresholds

```javascript
getSocEmptyThreshold()    // default 21
getSocRechargeThreshold() // default 25
MONTHLY_BILL_TARGET = 600
DAILY_BILL_TARGET = 600 / 30.4
```

## 6. Storage

### IndexedDB

| Store | Key | Value |
|---|---|---|
| `days` | `date` | `{ date, rows, summary, cols }` |
| `settings` | `key` | `{ key, value }` |

### Settings Keys

| Key | Shape |
|---|---|
| `systemInfo` | System settings saved from the system modal |
| `histBills` | Pre-solar baseline bills: `[{ m, kwh, cost }]` |
| `actualBills` | Real bills: `[{ y, m, cost, normalKwh, onPeakKwh, offPeakKwh, ft }]` |
| `ui:welcomeDismissed` | Whether the welcome modal was dismissed |
| `ui:welcomeHide` | Whether the welcome modal should stay hidden |

Older docs may mention `actualBills.kwh`. The current save path deletes legacy `kwh` and `serviceFee`, then stores separated kWh fields.

### Backup

`exportSettingsBackup()` exports a JSON file with:

- `systemInfo`
- `histBills`
- `actualBills`

It does not export:

- imported inverter rows
- daily raw IndexedDB data
- original XLSX files

`importSettingsBackup(files)` imports the JSON backup and overwrites the current browser's manual system settings and bill history after confirmation.

## 7. Daily Summary: `calcSum(rows, date)`

Main outputs:

- `solar`, `load`, `imp`, `exp`
- `onPeakImp`, `offPeakImp`
- `costOn`, `costOff`, `costTotal`
- `sellKwh`, `sellCost`
- `socMin`, `socMinT`, `socEmptyT`, `socPreDayEmptyT`
- `chg`, `dis`
- `pv1`, `pv2`
- temperature stats and bands: `bmsT`, `batT`, `dcT`, `acT`, `tempT`
- voltage/current stats
- override flags: `isBilledOverride`, `isProjectedOverride`

Important logic:

- TOU is active when `date >= CFG.touStart`.
- Before `CFG.touStart`, billing uses whole-day flat-rate calculation.
- Weekends and configured Thai holidays are off-peak.
- Service fee is prorated as `P.srv / 30.4`.
- VAT is `1.07`.
- kWh values shown to users remain raw inverter values.
- Currency values can be adjusted by real bill override or K-Factor projection and are marked with `*`.

## 8. Battery Empty Time

Battery empty detection intentionally avoids treating an early-morning low SOC as the current day's empty event.

Current behavior:

- The app waits for meaningful daytime solar/charge recovery before recording same-day empty time.
- Default recharge threshold is 25%, configurable by the user.
- Empty threshold defaults to 21%, configurable by the user.
- Early-morning empty before daytime recharge is stored as `socPreDayEmptyT`.
- `carryNextDayBatteryEmpty(days)` can carry next day's pre-day empty time back to the previous day when appropriate.

Example:

```text
2026-06-06 03:22 is the overnight empty event for 2026-06-05,
not the empty event for 2026-06-06, if the battery had not yet recharged on 2026-06-06.
```

Heatmap interpretation:

- Empty before 22:00 is a warning during TOU because it is still on-peak.
- Empty at 22:00 or later is OK from a bill-cost perspective because off-peak has started.
- Before TOU starts, battery empty status is less meaningful for TOU cost.

## 9. Bill Calibration

Current calibration functions:

- `computeCycleCalibrations(validBills, P)`
- `applyCalibration(summary, dateStr)`
- global `CYCLE_CALIBRATION`

### Cycle Key

`getCycleKeyForDate(date, billDay)` maps a day into a bill cycle ending month.

Example with bill day 15:

```text
2026-05-14 -> 2026-05 cycle
2026-05-15 -> 2026-06 cycle
```

### Real Bill Override

For a bill cycle with a real cost:

```text
EstimatedBill = sum RawCost(day, billFt)
BillRatio = ActualBill / EstimatedBill
CalibratedCost(day) = RawCost(day, billFt) * BillRatio
```

Daily summaries get:

- `isBilledOverride = true`
- `isProjectedOverride = false`
- adjusted `costTotal`, `costOn`, and `costOff`

### K-Factor Projection

For cycles where real cost is not available yet:

```text
K_normal = BillNormal_kWh / InverterNormal_kWh
K_on     = BillOnPeak_kWh / InverterOnPeak_kWh
K_off    = BillOffPeak_kWh / InverterOffPeak_kWh
```

`activeK` uses the latest stable K-Factor set, or the average of the latest 3 when values do not swing too much.

Projected daily summaries get:

- `isBilledOverride = false`
- `isProjectedOverride = true`

Important: kWh display remains raw inverter kWh. Only currency values are adjusted.

## 10. ROI And Simulation

### ROI

ROI uses actual load to estimate a no-solar baseline.

```text
NoSolarCostPerDay = (Load_kWh * (P.flat + P.ft) + P.srv / 30.4) * 1.07
ActualSolarCostPerDay = average(summary.costTotal)
SavingPerDay = max(0, NoSolarCostPerDay - ActualSolarCostPerDay)
MonthlySaving = SavingPerDay * 30.4
YearlySaving = SavingPerDay * 365
Investment = SolarCost + TOUCost - TaxDeduction
PaybackYears = Investment / YearlySaving
NominalPaybackMonths = Investment / MonthlySaving
DiscountedPaybackMonths = first month where
  sum(MonthlySaving / (1 + monthlyInflation)^month) >= Investment
```

The ROI card displays the Thai inflation rate used for discounting.

### Sell-To-MEA Simulation

The sell simulation shows a new payback period, not a monthly saving number.

```text
P90Clipping = renderHourlyAnalysis(rowDays, null, null, allRowsForP90).totalClipped
SellValuePerYear = average((Export_kWh + P90Clipping_kWh) * SellRate) * 365
PaybackWithSell = Investment / (YearlySaving + SellValuePerYear)
```

If row data is not available, the app falls back to `summary.sellCost`.

### Optimizer Simulation

```text
RecoverableWh = max(0, min(max(PV1, PV2) * 2, inverterW) - SolarW) * deltaTime
OptimizerWh = RecoverableWh * 50%
Saving = min(OptimizerWh, GridImportWh) * rate
```

The 50% factor is intentional and matches the UI tooltip.

### Battery 10 -> 16 kWh Simulation

This is a directional estimate, not a full hourly SOC simulation.

```text
ExtraUsableBatt = 6 * (1 - 0.21)
Storeable = min(export + 2 kWh rough clipping allowance, ExtraUsableBatt)
BatterySaving = min(Storeable, nightImport) * offPeakRateWithVAT
```

## 11. Battery Cycle

Function: `renderBattCycles(days)`

Current method is Equivalent Full Cycle (EFC):

```text
usableKwh = CFG.sys.battKwh * 0.9
EFC/day = (Charge_kWh + Discharge_kWh) / (2 * usableKwh)
```

Partial days are excluded.

This replaced the older charge-only method because charge-only overstates cycle count when charge and discharge are imbalanced.

## 12. Temperature Charts

Monthly, yearly, and all-time temperature stack charts use AC temperature bands:

```text
45-50 C
50-55 C
55-60 C
>60 C
```

Weekly temperature chart can still compare multiple temperature sources. That is why weekly temperature naming can differ from monthly/year/all-time AC-specific views.

## 13. Heatmaps

Current placement:

- Weekly tab: no heatmap.
- Monthly tab:
  - self-sufficiency calendar heatmap
  - battery empty calendar heatmap
  - problem day calendar heatmap
  - daily cost calendar heatmap
- Analysis month/year panes use calendar heatmaps where applicable.

Color logic:

- `selfColor()` maps self-sufficiency from red to yellow to green.
- `costColor()` maps daily bill against `DAILY_BILL_TARGET`.
- Current default monthly target is 600 baht/month, so daily target is about 19.74 baht/day.
- `socTimeColor()` treats battery empty before 22:00 as warning because it is still TOU on-peak.
- Battery empty at 22:00 or later is OK from a bill-cost perspective because off-peak has started.

## 14. Render Flow

`refresh()` calls render functions and catches render errors per section.

Key functions:

- `renderDay(date)`
- `renderWeek(startDate)`
- `renderMonth()`
- `renderYear()`
- `renderAllTime()`
- `renderAnalysis()`

`switchTab(t)` calls `renderVisibleTab(t)` to avoid blank Chart.js rendering after hidden canvas tabs become visible.

## 15. Chart Helpers

### `mkChart(id, type, labels, datasets, yOverride, plugins)`

Important behavior:

- Destroys an existing chart before creating a new one.
- Adds bar value labels automatically for non-timeY bar charts.
- Supports `stacked`, `timeY`, `tooltipExtra`, and `legend: false`.

### `barValueLabelsPlugin()`

- Non-stacked bars: label is outside the bar.
- Stacked bars: label is inside the segment when there is enough height.
- Adds top padding/grace to reduce label overlap.

## 16. Known Risks And Technical Debt

| Risk | Notes |
|---|---|
| Single large HTML file | Easy to run, harder to maintain |
| Multiple render functions call `dbAll()` | Works, but can be optimized later |
| Holiday list is hardcoded | Must be updated for future years |
| Simulation constants are rough | Especially bigger-battery and optimizer assumptions |
| Browser storage is local | Clearing browser data deletes saved dashboard data |
| Backup excludes raw inverter rows | Users need original XLSX files for full restore |
| CDN dependency | Chart.js and SheetJS require network access unless cached |

## 17. Safe Editing Rules

1. Do not change IndexedDB schema unless migration is handled.
2. Do not alter `calcSum()` without tracing downstream cards/charts.
3. Do not change calibration without checking both real bill override and K-Factor projection.
4. Do not change kWh display semantics: raw inverter kWh must remain raw.
5. Keep adjusted currency values marked with `*`.
6. Keep `app.html` single-file unless the user explicitly asks for a multi-file app.
7. Keep README, README.ai, the in-app guide, and the welcome modal aligned on tested scope, storage, backup, TOU, and limitations.
8. Before claiming a fix is complete, run the inline script syntax check.

Recommended syntax check:

```powershell
$src=Get-Content -Path app.html -Raw -Encoding UTF8
$scripts=[regex]::Matches($src,'<script[^>]*>([\s\S]*?)</script>')
$i=0
foreach($m in $scripts){
  $i++
  $p=Join-Path $env:TEMP ("solar_app_script_$i.js")
  Set-Content -Path $p -Value $m.Groups[1].Value -Encoding UTF8
  node --check $p
}
"SCRIPT_OK scripts=$i"
```

## 18. Diagnostic Checklist

When a user reports wrong numbers:

```text
[ ] Is CFG.startDate correct?
[ ] Is CFG.touStart correct?
[ ] Is CFG.billDay correct?
[ ] Is the day marked partial?
[ ] Does actualBills have real cost for the bill cycle?
[ ] Does actualBills include normal/on/off kWh if K-Factor is expected?
[ ] Is the value raw kWh or adjusted currency?
[ ] Is the date in a weekend/holiday off-peak period?
[ ] Is the browser opened from the same file path/origin as before?
[ ] Was an overnight battery empty event carried to the previous day correctly?
```

When a chart looks blank or incomplete:

```text
[ ] Was the chart rendered while hidden?
[ ] Did switchTab call renderVisibleTab?
[ ] Is the canvas id unique?
[ ] Did mkChart destroy an existing chart with the same id?
[ ] Are labels and dataset lengths equal?
```

When documentation changes:

```text
[ ] README.md public scope still matches app.html.
[ ] README.ai.md technical scope still matches app.html.
[ ] In-app guide still matches README.md.
[ ] No personal paths, mojibake, internal phase names, or outdated repo URLs leaked.
```
