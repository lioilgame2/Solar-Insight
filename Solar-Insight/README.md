# SolarInsight Dashboard

> Offline-first solar analytics for home hybrid inverter owners.
> One HTML file. No server. No login. Your inverter data stays in your browser.

[![HTML](https://img.shields.io/badge/App-Single%20HTML-orange?style=flat-square&logo=html5)](app.html)
[![Chart.js](https://img.shields.io/badge/Charts-Chart.js%204.4-pink?style=flat-square)](https://www.chartjs.org/)
[![Storage](https://img.shields.io/badge/Storage-IndexedDB-blue?style=flat-square)](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)

## What It Is

SolarInsight is a browser-based dashboard for people who want to understand their solar system beyond the default inverter app.

It turns exported inverter XLSX files and real electricity bills into practical answers:

- Is my solar system actually reducing the bill?
- When does the battery run out?
- Which PV string is underperforming?
- Is the inverter getting hot?
- How much should my monthly bill be after calibration with real MEA/PEA bills?
- What is the payback period if my current usage pattern continues?

The app is built for a real home solar setup, but the structure is generic enough for other hybrid inverter exports with similar columns.

## Why This Exists

Most inverter apps are good at showing live data, but weak at long-term ownership questions.

| Problem | SolarInsight Approach |
|---|---|
| Cloud dashboards focus on real-time views | Local daily, weekly, monthly, yearly, and all-time analysis |
| Electricity bills rarely match inverter estimates | Real bill calibration and K-Factor adjustment |
| ROI is usually guessed | ROI uses actual imported load and cost data |
| Battery behavior is hard to read | Battery empty time, SOC trend, and cycle estimation |
| PV string issues are hidden | String A/B comparison and imbalance detection |
| Data privacy matters | Data is stored locally in browser IndexedDB |

## Key Features

### Energy Views

| View | Purpose |
|---|---|
| Daily | 24-hour solar, load, grid, battery, SOC, temperature, voltage, and current |
| Weekly | Week comparison, string production, self-sufficiency, and cost trend |
| Monthly | Billing-cycle analysis, daily cost heatmap, self-sufficiency heatmap, and grouped charts |
| Yearly | Month-by-month yearly trend and year-level summaries |
| All Time | Long-range totals and health patterns |
| Analysis | ROI, simulation, battery cycle, trend snapshot, and before/after bill comparison |

### Bill And ROI Intelligence

- Real MEA/PEA bill input after solar installation
- 12-month pre-solar baseline support
- Before-vs-after monthly bill comparison
- K-Factor calibration from real bill kWh vs inverter kWh
- Marked calibrated values using `*`
- ROI payback using actual imported data
- Monthly bill target heatmap, currently oriented around a ฿600/month goal

### Technical Diagnostics

- PV String A vs B comparison
- Battery empty-time detection after meaningful recharge
- Battery Equivalent Full Cycle (EFC) estimation
- AC temperature band chart
- Clipping and potential recovery estimate
- TOU on-peak/off-peak cost separation
- Weekend and holiday off-peak handling

## Quick Start

1. Download or clone this repository.
2. Open `app.html` in Chrome or Edge.
3. Click **ระบบ** and set your system information:
   - Solar start date
   - TOU meter start date
   - Inverter size
   - Battery size
   - Electricity rates
   - Investment cost for ROI
4. Export an XLSX report from your inverter platform.
5. Click **inverter data** and import the file.
6. Open **ประวัติบิล** and add real electricity bills when available.

No build step is required.

## Supported Data

The app was developed around a DEYE hybrid inverter export from Deye Cloud.

| Source | Status |
|---|---|
| DEYE XLSX day report | Supported |
| MEA/PEA actual bills | Supported by manual input |
| Other hybrid inverter XLSX exports | Possible if the columns are similar |

If your inverter export has time, solar, grid purchase, grid export, load, battery, PV string, SOC, and temperature columns, it is likely adaptable.

## How The Main Calculations Work

### Electricity Cost

TOU cost is calculated from imported grid energy:

```text
OnCost  = OnPeak_kWh  × (OnRate  + Ft) × 1.07
OffCost = (OffPeak_kWh × (OffRate + Ft) + ServiceFee / 30.4) × 1.07
```

If a real bill exists, the app calibrates the displayed cost:

```text
BillRatio = ActualBill / EstimatedBill
CalibratedDailyCost = RawDailyCost × BillRatio
```

If no bill exists yet, the app can apply K-Factor:

```text
K_on  = BillOnPeak_kWh  / InverterOnPeak_kWh
K_off = BillOffPeak_kWh / InverterOffPeak_kWh
```

The kWh values remain raw inverter data. The `*` marker means the currency value was adjusted by a real bill override or K-Factor projection.

### ROI

ROI uses a no-solar baseline based on the same actual load:

```text
NoSolarCostPerDay = (Load_kWh × (FlatRate + Ft) + ServiceFee / 30.4) × 1.07
SavingPerDay = max(0, NoSolarCostPerDay - ActualSolarCostPerDay)
PaybackYears = Investment / (SavingPerDay × 365)
```

The Analysis tab also shows the day-level formula directly, for example:

```text
฿142/day before solar - ฿55/day after solar = ฿87/day saved
```

### Before vs After Bill Comparison

The before/after chart pairs the same billing month:

```text
Before: May 2025 bill or May baseline
After : May 2026 real post-solar bill
```

As more real post-solar bills are added, the chart grows month by month until it can show a full 12-month comparison.

### Battery Cycle

Battery usage is estimated with Equivalent Full Cycle (EFC):

```text
EFC = (Charge_kWh + Discharge_kWh) / (2 × Battery_kWh × 0.9)
```

This is better than using charge-only energy because it reflects total battery throughput.

### Simulation

The simulation cards are directional estimates, not promises.

| Simulation | Main Idea |
|---|---|
| Optimizer | Estimate recoverable imbalance clipping, then assume 50% is practically recovered |
| Sell to MEA | Add possible sell value from export and estimated clipping |
| Bigger battery | Estimate how much export/clipping can be shifted into night usage |

## Privacy

SolarInsight is offline-first:

- No backend
- No account
- No database server
- No telemetry
- Data is stored in your browser using IndexedDB

Important: browser data is local to the browser and file origin. If you clear browser storage, the saved dashboard data can be removed.

## Project Structure

```text
Solar-Insight/
├── app.html      # Main application
├── README.md     # Public project overview
└── README.ai.md  # Developer and AI handoff notes
```

## Best Fit

SolarInsight is useful for:

- Home solar owners with battery storage
- Hybrid inverter users who can export XLSX data
- People on TOU electricity plans
- Owners who want to verify the real financial impact of solar
- Solar installers who want a lightweight local analysis tool for customers

It is not meant to replace official billing records or certified metering.

## Roadmap

- Export summary reports to PDF or Excel
- Configurable public holidays
- Better inverter column mapping UI
- More inverter presets
- Optional backup/restore for IndexedDB data
- Progressive Web App packaging

## Contributing

Issues and pull requests are welcome.

For implementation context, see `README.ai.md`.

## License

MIT-style usage is intended, but a formal `LICENSE` file should be added before treating the repository as fully licensed for public reuse.
