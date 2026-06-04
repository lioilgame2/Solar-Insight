## คู่มือใช้งาน SolarInsight

เอกสารนี้เปิดด้วยภาษาไทย ส่วนภาษาอังกฤษจะอยู่ด้านล่าง

SolarInsight เป็น dashboard แบบเปิดจากไฟล์ `app.html` ได้เลย ไม่ต้องติดตั้ง server เพิ่ม

สิ่งที่ควรรู้ก่อนเริ่ม:

- ต้อง `import` ไฟล์จาก inverter เข้ามาในระบบก่อนถึงจะเห็นกราฟและการวิเคราะห์
- โปรเจกต์นี้ทดสอบกับไฟล์ XLSX จาก `Deye` เป็นหลัก
- ยี่ห้ออื่นอาจใช้ได้ถ้าโครงสร้างคอลัมน์ใกล้เคียง แต่ยังไม่การันตีว่าจะไม่มีปัญหา
- ปุ่ม `ระบบ` ใช้ตั้งค่าชื่อระบบ, ค่าไฟ, เงินเฟ้อ, แบตเตอรี่, และตัวเลือก Simulation
- จุดสีแดง = ใช้คำนวณ
- จุดสีฟ้า = ใช้แสดงผล

### เริ่มใช้งานแบบสั้นที่สุด

1. เปิด `app.html` ด้วย Chrome หรือ Edge
2. กดปุ่ม `ระบบ`
3. กรอกข้อมูลหลักของระบบ เช่น วันเริ่มใช้ Solar, วันเริ่ม TOU, ขนาดระบบ, แบตเตอรี่, ค่าไฟ, และต้นทุน
4. กดปุ่ม `inverter data` แล้วเลือกไฟล์ XLSX ที่ export มาจาก inverter
5. ถ้ามีบิลจริง ให้ไปที่ `ประวัติบิล` แล้วกรอกบิลก่อนติดตั้งและหลังติดตั้ง
6. ดูผลที่ tab `รายวัน`, `รายเดือน`, และ `วิเคราะห์`

### ต้องเตรียมอะไรบ้าง

- ไฟล์ XLSX จาก inverter
- วันที่เริ่มใช้งาน Solar
- วันที่เริ่มใช้มิเตอร์ TOU
- ค่าไฟ On-peak / Off-peak / Flat / Ft / ค่าบริการ
- ข้อมูลแบตเตอรี่ที่ใช้งานจริง
- ถ้ามี: บิลค่าไฟจริงจาก MEA/PEA

### เรื่องไฟล์จาก inverter

- เวอร์ชันปัจจุบันพัฒนาจากไฟล์ export ของ `Deye`
- ถ้าใช้ `Deye` โอกาสใช้งานได้ตรงที่สุด
- ถ้าเป็น inverter ยี่ห้ออื่น ระบบอาจยัง import ได้ แต่มีความเสี่ยงเรื่อง:
  - ชื่อคอลัมน์ไม่ตรง
  - หน่วยข้อมูลไม่เหมือนกัน
  - ไม่มีบาง field ที่ระบบใช้คำนวณ เช่น PV string, SOC, temperature, grid purchase/export
- ถ้า import แล้วกราฟหรือค่าคำนวณดูผิด ควรเริ่มจากเช็คว่าโครงสร้างไฟล์ตรงกับ Deye หรือไม่

### ปุ่ม `ระบบ` ใช้ตั้งค่าอะไร

- จุดสีแดง: ค่าเหล่านี้ถูกใช้คำนวณจริง เช่น ค่าไฟ, battery kWh, batt charge, inflation
- จุดสีฟ้า: ค่าเหล่านี้ใช้แสดงผลหรือช่วยอธิบายระบบ เช่น ชื่อระบบ, string spec, battery model
- `เงินเฟ้อไทย (%/ปี)` ใช้กับ ROI แบบหักเงินเฟ้อ
- `Simulation` เป็นตัวเลือกเปิด/ปิดกล่องจำลองใน tab `วิเคราะห์`
- ค่า `Simulation` default เป็น `ปิด` เพื่อไม่ให้แสดงค่าจำลองกับระบบที่ยังไม่ได้ตั้งใจใช้สมมติฐานชุดนี้

### คำแนะนำการใช้งาน

- ถ้าพึ่งเริ่ม ให้ดู `รายวัน` ก่อน เพื่อเช็คว่า import เข้ามาครบและเวลาถูกต้อง
- ถ้าต้องการดูตามรอบบิล ให้ดู `รายเดือน`
- ถ้าต้องการดู ROI, battery cycle, simulation, และ before/after bill ให้ดู `วิเคราะห์`
- ถ้าค่าไฟในระบบยังไม่ตรงบิลจริง อย่าดู ROI ทันที ควรกรอก `ประวัติบิล` ก่อน

### ข้อจำกัดที่ควรรู้

- ข้อมูลทั้งหมดเก็บใน browser ของเครื่องที่เปิดใช้งาน
- ถ้าล้าง browser storage ข้อมูลที่ import อาจหายได้
- Simulation บางส่วนยังเป็นการประมาณเชิงทิศทาง ไม่ใช่ผลการันตีจริง
- ยี่ห้อ inverter อื่นนอกจาก Deye อาจต้องปรับ parsing เพิ่ม

---

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

## System Settings

Click **ระบบ** to adjust the system profile used by the dashboard.

The form uses two colored dots:

| Marker | Meaning |
|---|---|
| Red dot | Used in calculations |
| Blue dot | Used for display/context |

Important settings:

| Setting | Used For |
|---|---|
| Peak sun h | Theoretical daily production and solar efficiency benchmark |
| Battery kWh | Battery cycle estimation |
| Batt Charge (A) | Battery charge-limit and clipping checks |
| Electricity rates | TOU, flat-rate, Ft, service fee, and sell-back calculations |
| Solar cost / TOU cost / Tax deduction | ROI investment basis |
| เงินเฟ้อไทย (%/ปี) | ROI countdown with inflation discounting |
| แสดง Simulation | Shows or hides the Analysis simulation card |

Simulation is optional because it uses directional assumptions such as optimizer recovery, sell-back value, and bigger-battery behavior. For a generic or shared system profile, keeping it off by default is safer. Turn it on only when those assumptions are useful for the system being reviewed.

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
DiscountedPayback = cumulative monthly saving discounted by Thai inflation
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
.
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
