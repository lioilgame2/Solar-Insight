# README.ai.md — SolarInsight Technical Reference

เอกสารนี้มีไว้ให้ AI assistant หรือ developer เข้าใจ `app.html` โดยไม่ต้อง reverse engineer ใหม่ทุกครั้ง

> สถานะเอกสาร: อัปเดตให้ตรงกับ `app.html` หลังงาน bill calibration, ROI, simulation, heatmap, battery empty time และ Glossary ล่าสุด

---

## 1. Project Snapshot

| รายการ | ค่า |
|---|---|
| Project | SolarInsight Dashboard |
| Main file | `app.html` ไฟล์เดียว |
| App type | Offline-first browser dashboard |
| Stack | HTML / CSS / Vanilla JavaScript / Chart.js / SheetJS / IndexedDB |
| UI language | ไทย |
| Primary data | XLSX export จาก hybrid inverter |
| Storage | Browser IndexedDB ชื่อ `SolarInsight` |
| Main hardware profile | DEYE hybrid inverter, 10 × 645 W panels, PYLON LiFePO4 10.24 kWh |

หมายเหตุ: ตัวเลขขนาดไฟล์และจำนวนบรรทัดไม่ควรถูก hardcode ในเอกสาร เพราะ `app.html` เปลี่ยนบ่อย

---

## 2. Current Architecture

```text
app.html
├── <head>
│   ├── CSS variables + layout styles
│   ├── SheetJS XLSX CDN
│   └── Chart.js CDN
├── <body>
│   ├── Daily tab
│   ├── Weekly tab
│   ├── Monthly billing-cycle tab
│   ├── Yearly tab
│   ├── All-time tab
│   ├── Analysis tab
│   ├── System modal
│   ├── File manager modal
│   └── Bill history modal
└── <script>
    ├── Config/constants
    ├── IndexedDB + settings helpers
    ├── XLSX parsing
    ├── Daily summary calculation
    ├── Bill-cycle calibration
    ├── Render functions
    ├── Chart builders/plugins
    ├── Heatmap renderers
    └── Analysis/ROI/simulation functions
```

Important design constraint: the project intentionally stays as a single-file dashboard. Do not split into modules unless the user explicitly asks.

---

## 3. Data Flow

```text
User imports XLSX
  ↓
parseXLSX(buf)
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

`dbAll()` is important because it recomputes calibration every time. IndexedDB should keep raw daily summaries; calibrated values are applied in memory.

---

## 4. Core Config

### `CFG`

```javascript
const CFG = {
  sys: {
    panels: 10,
    wp: 645,
    peakSunH: 4.5,
    battKwh: 10.24,
    battAmp: 48,
    invKw: 5,
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
}
```

### `getP()`

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

ค่าเหล่านี้มาจาก input ใน UI ถ้ามี ถ้าไม่มีจะใช้ default ข้างบน

### Thresholds

```javascript
SOC_EMPTY_THRESHOLD = 21
SOC_RECHARGED_THRESHOLD = 25
MONTHLY_BILL_TARGET = 600
DAILY_BILL_TARGET = 600 / 30.4
```

---

## 5. Storage

### IndexedDB

| Store | Key | Value |
|---|---|---|
| `days` | `date` | `{ date, rows, summary, cols }` |
| `settings` | `key` | `{ key, value }` |

### Settings keys

| Key | Shape |
|---|---|
| `systemInfo` | system config saved from system modal |
| `histBills` | pre-solar baseline: `[{ m, kwh, cost }]` |
| `actualBills` | real bills: `[{ y, m, cost, normalKwh, onPeakKwh, offPeakKwh, ft }]` |

Legacy note: older docs mention `actualBills.kwh`. Current save path deletes legacy `kwh` and `serviceFee`, then stores separated kWh fields.

---

## 6. `calcSum(rows, date)`

Daily summary calculator. This is the highest-impact function in the app.

Main outputs:

- `solar`, `load`, `imp`, `exp`
- `onPeakImp`, `offPeakImp`
- `costOn`, `costOff`, `costTotal`
- `sellKwh`, `sellCost`
- `socMin`, `socMinT`, `socEmptyT`
- `chg`, `dis`
- PV string totals: `pv1`, `pv2`
- temperature stats and bands: `bmsT`, `batT`, `dcT`, `acT`, `tempT`
- voltage/current stats
- override flags: `isBilledOverride`, `isProjectedOverride`

Important logic:

- TOU active when `date >= CFG.touStart`
- Weekends and `THAI_HOLIDAYS` are off-peak
- Service fee is prorated daily as `P.srv / 30.4` in raw daily calculation
- VAT is `1.07`
- Clipping estimate:
  - count samples where solar is active and battery is full/charge-limited
  - convert sample count into kWh using interval minutes
  - cap at `1.5 kW` and `5 kWh/day`
- `sellKwh = exp + clipKwh` is a daily summary fallback. Visible sell-if cards and sell simulation should use dynamic P90 clipping via `estimateSellProjection()` / `renderHourlyAnalysis()` when row data exists.

P90 reference note:

- Full-export trial days are useful P90 reference days because production is less battery-limited
- In this project, Apr 16-19 were used as full-production trial context
- Early in the dataset, a few very clear full-export days can make P90 slightly optimistic
- The app only counts P90 gap as clipping when battery is full or charge-limited, so cloudy days are less likely to be falsely counted

Battery empty time:

- The app does not simply use minimum SOC time
- It waits until battery has meaningfully recharged first (`>= 25%`)
- Then records first SOC `<= 21%`
- This avoids false empty time near midnight or after stale data

---

## 7. Bill Calibration

Current calibration system is **not** `updateCalibrationFactor()`, `kwhFactor`, or `costFactor`.

Current functions:

- `computeCycleCalibrations(validBills, P)`
- `applyCalibration(summary, dateStr)`
- global `CYCLE_CALIBRATION`

### Cycle key

`getCycleKeyForDate(date, billDay)` maps a day into a bill cycle ending month.

Example with bill day 15:

```text
2026-05-14 → 2026-05 cycle
2026-05-15 → 2026-06 cycle
```

### Real bill override

For a bill cycle with a real cost:

```text
EstimatedBill = Σ RawCost(day, billFt)
BillRatio = ActualBill / EstimatedBill
CalibratedCost(day) = RawCost(day, billFt) × BillRatio
```

The daily rows get:

- `isBilledOverride = true`
- `isProjectedOverride = false`
- `costTotal`, `costOn`, `costOff` adjusted

### K-Factor projection

For cycles where the real cost is not available yet:

```text
K_normal = BillNormal_kWh / InverterNormal_kWh
K_on     = BillOnPeak_kWh / InverterOnPeak_kWh
K_off    = BillOffPeak_kWh / InverterOffPeak_kWh
```

`activeK` uses the latest stable K-Factor set, or average of the latest 3 when values do not swing too much.

Projected daily costs get:

- `isBilledOverride = false`
- `isProjectedOverride = true`

Important: kWh display remains raw inverter kWh. Only currency values are adjusted. The UI marks adjusted currency with `*`.

---

## 8. `renderHistROI(days)`

This renders the before-vs-after electricity bill comparison chart in Analysis Overview.

Current behavior:

- The chart only uses post-solar real bills as the main month keys
- For each post-solar bill month:
  - After bar = actual post-solar bill from `actualBills`
  - Before bar = same-month pre-solar actual bill if available
  - Otherwise before bar = `histBills` baseline for that month
- It does not fall back to inverter-estimated monthly cost for the after bar
- Bars are rendered as separated labels:
  - `ก่อน / พ.ค. 68`
  - `หลัง / พ.ค. 69`
  - then the next month pair

This means the chart grows month by month as real post-solar bills are added. With 12 real post-solar bills, it becomes a full 12-month before/after comparison.

---

## 9. ROI And Simulation

### `calcROI()`

ROI uses actual load to create a no-solar baseline.

```text
NoSolarCostPerDay = (Load_kWh × (P.flat + P.ft) + P.srv / 30.4) × 1.07
ActualSolarCostPerDay = average(summary.costTotal)
SavingPerDay = max(0, NoSolarCostPerDay - ActualSolarCostPerDay)
MonthlySaving = SavingPerDay × 30.4
YearlySaving = SavingPerDay × 365
PaybackYears = Investment / YearlySaving
```

The ROI card also displays the daily formula:

```text
Before Solar avg/day - After Solar avg/day = Saving/day
```

### Sell-to-MEA payback

The sell simulation in `simSellRoi` is a new payback period, not a monthly saving number.

```text
P90Clipping = renderHourlyAnalysis(rowDays, null, null, allRowsForP90).totalClipped
SellValuePerYear = avg((Export_kWh + P90Clipping_kWh) × SellRate) × 365
PaybackWithSell = Investment / (YearlySaving + SellValuePerYear)
```

If row data is not available, the app falls back to `summary.sellCost`.

### Optimizer simulation

The optimizer simulation estimates recoverable imbalance energy:

```text
RecoverableWh = max(0, min(max(PV1, PV2) × 2, inverterW) - SolarW) × Δt
OptimizerWh = RecoverableWh × 50%
Saving = min(OptimizerWh, GridImportWh) × rate
```

The `50%` factor is intentional and matches the UI tooltip.

### Battery 10 → 16 kWh simulation

This is a rough directional estimate:

```text
ExtraUsableBatt = 6 × (1 - 0.21)
Storeable = min(export + 2 kWh rough clipping allowance, ExtraUsableBatt)
BatterySaving = min(Storeable, nightImport) × offPeakRateWithVAT
```

It is not a full hourly SOC simulation.

---

## 10. Battery Cycle

Function: `renderBattCycles(days)`

Current method is Equivalent Full Cycle (EFC):

```text
usableKwh = CFG.sys.battKwh × 0.9
EFC/day = (Charge_kWh + Discharge_kWh) / (2 × usableKwh)
```

Partial days are excluded.

This replaced the older charge-only method because charge-only overstates cycle count when charge and discharge are imbalanced.

---

## 11. Temperature Charts

Monthly, yearly, and all-time temperature stack charts use AC temperature bands:

```text
45–50°C
50–55°C
55–60°C
>60°C
```

Weekly temperature chart still compares multiple temperature sources. That is why weekly title is `อุณหภูมิ`, while monthly/year/all-time use `อุณหภูมิ (AC)`.

---

## 12. Heatmaps

Current heatmap placement:

- Weekly tab: no heatmap
- Monthly tab:
  - self-sufficiency heatmap
  - daily cost heatmap
  - problem heatmap
- Analysis/month and year panes use calendar heatmaps where applicable

Color logic:

- `selfColor()` maps self-sufficiency from red → yellow → green
- `costColor()` maps daily bill against `DAILY_BILL_TARGET`
- Current default monthly target is `฿600/month`, so daily target is about `฿19.74/day`
- `socTimeColor()` treats battery empty before 22:00 as warning because it is still TOU on-peak
- Battery empty at 22:00 or later is OK from a bill-cost perspective because TOU off-peak has started

---

## 13. Important Render Flow

`refresh()` calls render functions and catches render errors per section.

Key functions:

- `renderDay(date)`
- `renderWeek(startDate)`
- `renderMonth()`
- `renderYear()`
- `renderAllTime()`
- `renderAnalysis()`

`switchTab(t)` calls `renderVisibleTab(t)` to avoid blank Chart.js rendering after hidden canvas tabs become visible.

---

## 14. Chart Helpers

### `mkChart(id, type, labels, datasets, yOverride, plugins)`

Handles normal bar/line/pie chart creation.

Important behavior:

- Destroys existing chart before creating a new one
- Adds bar value labels automatically for non-timeY bar charts
- Supports:
  - `stacked`
  - `timeY`
  - `tooltipExtra`
  - `legend: false`

### `barValueLabelsPlugin()`

- Non-stacked bars: label is outside the bar
- Stacked bars: label is inside segment when enough height exists
- Adds top padding/grace to avoid overlap

---

## 15. Known Risks / Technical Debt

| Risk | Notes |
|---|---|
| Single large HTML file | Easy to run, harder to maintain |
| Multiple render functions call `dbAll()` | Works, but can be optimized later |
| Holiday list is hardcoded | Must be updated for future years |
| Some simulation constants are rough | Especially `+2 kWh` clipping allowance for bigger battery simulation |
| Browser storage is local | Clearing browser data deletes saved dashboard data |
| CDN dependency | Needs internet the first time Chart.js/SheetJS are loaded |

---

## 16. Safe Editing Rules

1. Do not change IndexedDB schema unless migration is handled.
2. Do not alter `calcSum()` without tracing downstream cards/charts.
3. Do not change calibration without checking both real bill override and K-Factor projection.
4. Do not change kWh display semantics: raw inverter kWh must remain raw.
5. Keep adjusted currency values marked with `*`.
6. Keep `app.html` single-file unless the user explicitly asks for a multi-file app.
7. Before claiming a fix is complete, run inline script syntax check:

```powershell
node -e "const fs=require('fs'),vm=require('vm');const code=fs.readFileSync('app.html','utf8');const scripts=[...code.matchAll(/<script(?!(?:[^>]*src=))[^>]*>([\s\S]*?)<\/script>/gi)].map(m=>m[1]);for(const [i,s] of scripts.entries()){new vm.Script(s,{filename:'inline-script-'+(i+1)+'.js'});}console.log('SCRIPT_OK scripts='+scripts.length+' lines='+code.split(/\r?\n/).length);"
```

---

## 17. Diagnostic Checklist

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
```

When a chart looks blank or incomplete:

```text
[ ] Was the chart rendered while hidden?
[ ] Did switchTab call renderVisibleTab?
[ ] Is the canvas id unique?
[ ] Did mkChart destroy an existing chart with the same id?
[ ] Are labels and dataset lengths equal?
```
