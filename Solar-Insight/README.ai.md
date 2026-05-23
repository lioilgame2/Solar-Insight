# README.ai.md — AI Technical Reference for SolarInsight Dashboard

> **วัตถุประสงค์**: เอกสารนี้สำหรับ AI Assistant หรือ Developer ที่ต้องการเข้าใจโปรเจกต์อย่างลึกซึ้ง  
> เพื่อช่วย debug, เพิ่มฟีเจอร์, หรือ refactor โดยไม่ทำลาย logic เดิม

---

## 1. Project Identity

| ข้อมูล | รายละเอียด |
|-------|-----------|
| **ชื่อโปรเจกต์** | SolarInsight Dashboard |
| **ไฟล์หลัก** | `app.html` (เดียว, ~225KB, ~3,700+ บรรทัด) |
| **ภาษา** | Thai (ภาษาไทย) — UI ทั้งหมดเป็นภาษาไทย |
| **Target Hardware** | DEYE SUN-5K-SG04LP1-EU-SM2 Hybrid Inverter + PYLON LiFePO4 10.24kWh |
| **Target Meter** | MEA TOU (Time-of-Use) meter — On-peak 09:00–22:00 |
| **Owner System** | 10 × LONGi 645W panels (5.25kWp), String A: 5 West, String B: 3S+2W |

---

## 2. Architecture Overview

```
Single HTML File Architecture
──────────────────────────────────────────────
[app.html]
├── <head>
│   ├── CSS (inline ~500 lines) — Dark theme, CSS variables
│   ├── Chart.js 4.4 (CDN)
│   └── SheetJS XLSX (CDN)
│
├── <body> — Tab-based navigation
│   ├── Tab D: Daily
│   ├── Tab W: Weekly
│   ├── Tab M: Monthly (Bill Cycle)
│   ├── Tab Y: Yearly
│   ├── Tab T: All Time
│   └── Tab A: Analysis (with sub-tabs: Overview/Day/Week/Month/Year)
│
└── <script> (inline ~2,700 lines)
    ├── Constants & Config (CFG, CLR, THR, THAI_HOLIDAYS)
    ├── Utility functions (fn, fmtD, tMin, weekStartKey, billPeriodKey)
    ├── Column Detection (detectC)
    ├── IndexedDB Layer (initDB, dbPut, dbGet, dbAll, dbDel, dbClear)
    ├── Settings Layer (settingsGet, settingsPut)
    ├── XLSX Parser (parseXLSX)
    ├── Core Calculator (calcSum)
    ├── App State Management
    ├── System Info Management (CFG modal)
    ├── Bill Management (histBills, actualBills)
    ├── Calibration System (updateCalibrationFactor)
    ├── Render Functions (renderDay, renderWeek, renderMonth, renderYear, renderAllTime, renderAnalysis)
    ├── Chart Builders (buildPowerChart, buildTempChart, mkChart, mkPie)
    ├── Chart Plugins (touPlugin, threshPlugin, syncCrosshairPlugin)
    ├── Heatmap Renderers (renderHeatmap, renderCostHeatmap, renderCalendarHeatmap)
    ├── Analysis Functions (renderHistROI, renderBattHealth, renderBattCycles, renderHourlyAnalysis, renderStringComparison, calcROI)
    └── Boot (bootApp)
```

---

## 3. Data Flow

```
[XLSX File Import]
      │
      ▼
parseXLSX(buf)
  → detectC(rows[0])        ← Auto-detect column names
  → group rows by date
      │
      ▼
calcSum(rows, date)
  → Compute: solar, load, imp, exp, onPeakImp, offPeakImp
  → Compute: costOn, costOff, costTotal (with VAT 7%)
  → Compute: sellKwh (exp + clipping allowance, Capped at 5kWh/day)
  → Compute: socMin, temps, voltages, pv1, pv2
  → Returns: summary object
      │
      ▼
dbPut({date, rows, summary, cols})
  → memoryDays.set(date, data)
  → IndexedDB.put(data)        ← Persists across sessions
      │
      ▼
refresh()
  → dbAll()
      → updateCalibrationFactor(all)    ← Adjusts kwhFactor, costFactor
      → Apply calibration to each summary
  → renderDay() / renderWeek() / renderMonth() / renderYear() / renderAnalysis()
      │
      ▼
[Chart.js Charts + HTML Cards + Heatmaps]
```

---

## 4. Key Constants & Configuration

### `CFG` Object (Global Config)
```javascript
CFG = {
  startDate: '2026-04-10',     // วันที่ติด Solar — CRITICAL สำหรับแยก Pre/Post Solar
  touStart:  '2026-05-05',     // วันที่เปลี่ยนมิเตอร์ TOU
  billDay:   15,                // วันตัดรอบบิล (1-28)
  systemCost: 152350,           // ต้นทุนสุทธิหลังลดหย่อนภาษี (บาท)
  sys: { panels:10, wp:645, peakSunH:4.5, battKwh:10.24, invKw:5 },
  pvSpec: { voc:54.12, vmp:44.77, isc:15.06, imp:14.41 },
  strings: { aSpec:'5 West', bSpec:'3 South + 2 West', aLabel:'String A', bLabel:'String B' },
  costNote: { solar:209000, tou:3350, taxDeduction:60000 },
  partialDays: ['2026-05-16','2026-05-17'],  // วันที่ข้อมูลไม่สมบูรณ์
}
```

### Electricity Rate `P` Object (จาก `getP()`)
```javascript
P = {
  on:   4.1682,   // On-peak rate (บาท/kWh, ก่อน FT+VAT)
  off:  2.6369,   // Off-peak rate
  flat: 3.2484,   // Flat rate (ก่อนมี TOU)
  ft:   0.0093,   // Ft adjustment
  srv:  38.22,    // ค่าบริการรายเดือน (บาท)
  sell: 2.20,     // ราคาขายไฟ Feed-in Tariff (บาท/kWh)
}
```

### Color Constants (`CLR`)
```javascript
CLR = {
  solar:'#4ade80', load:'#fb923c', batt:'#818cf8',
  grid:'#38bdf8',  soc:'#c084fc',
  battT:'#34d399', dc:'#38bdf8', ac:'#f87171',
}
```

---

## 5. Storage Architecture

### IndexedDB: `SolarInsight`
| Store | Key | Value | หมายเหตุ |
|-------|-----|-------|---------|
| `days` | `date` (YYYY-MM-DD) | `{date, rows, summary, cols}` | ข้อมูล XLSX แยกตามวัน |
| `settings` | `key` (string) | `value` (any) | ตั้งค่าทุกอย่าง |

### Settings Keys
| Key | Value Type | Content |
|-----|-----------|---------|
| `systemInfo` | Object | CFG ทั้งหมด (panels, dates, costs, etc.) |
| `histBills` | Array | `[{y,m,kwh,cost}]` — บิลก่อนติด Solar (12 เดือน baseline) |
| `actualBills` | Array | `[{y,m,kwh,cost}]` — บิลจริงจาก MEA ทุกเดือน |

### Memory Cache: `memoryDays` (Map)
- เป็น in-memory cache ของ IndexedDB
- `dbGet()` เช็ค cache ก่อนเสมอ
- `dbAll()` อ่าน IndexedDB ทั้งหมดแล้ว overwrite cache

---

## 6. Core Functions Reference

### `calcSum(rows, date)` → summary object
**หัวใจของระบบ** คำนวณสรุปรายวันจาก raw rows

Critical logic:
- ใช้ค่า meter ณ เวลา 09:00 และ 22:00 เพื่อแยก On/Off-peak
- `touActive(date)` เช็คว่าวันนั้นใช้ TOU แล้วหรือยัง (เปรียบเทียบกับ `CFG.touStart`)
- `THAI_HOLIDAYS.includes(date)` เช็ควันหยุด → ถ้าใช่ ให้ใช้ Off-peak rate ทั้งวัน
- **Clipping Detection:** นับจำนวนช่วง 5 นาทีที่ `SOC >= 95% AND batt > -100W AND solar > 200W` แล้วแปลงเป็น kWh (ให้สูงสุด 1.5kW rate, Capped ที่ 5kWh/day)
- ค่า Clipping นี้จะถูกบวกเข้าไปใน `sellKwh` ทันที เพื่อใช้จำลองการขายไฟ (ถ้าปลดกันย้อน)
- VAT 7% คูณกับทุก cost item

### `updateCalibrationFactor(all)` → sets `kwhFactor`, `costFactor`
- หา post-solar actual bill ล่าสุดจาก `actualBills`
- หา inverter-estimated kWh ในช่วงเดียวกัน
- `kwhFactor = MEA_kwh / inverter_kwh`
- `costFactor = MEA_cost / inverter_cost`
- Factor ถูก Apply ใน `dbAll()` ทุกครั้งที่เรียก

⚠️ **Important**: `_orig` (ค่าดั้งเดิมก่อน calibrate) ถูก save ใน memory เท่านั้น ไม่ได้ save ลง IndexedDB ดังนั้น IndexedDB เก็บค่า RAW เสมอ → Calibration Apply ใหม่ทุกครั้งที่ `dbAll()` ถูกเรียก

### `renderHistROI(days)`
กราฟ **"เปรียบเทียบค่าไฟก่อน/หลังติด Solar"**

Priority logic สำหรับแท่งสีฟ้า (หลังติด):
1. ดู `actualBills` ที่เป็น Post-Solar ก่อน (บิลจริงจาก MEA)
2. ถ้าไม่มี ใช้ inverter estimate จาก `mCostInv`
3. แสดง badge `✅ ใช้บิลจริงจาก MEA/PEA` เมื่อมีข้อมูลจริง

Priority logic สำหรับแท่งสีเทา (ก่อนติด):
1. ดู `actualBills` ที่เป็น Pre-Solar ก่อน
2. ถ้าไม่มี ใช้ `histBills` (12-month baseline)

### `renderHourlyAnalysis(days)`
วิเคราะห์รายชั่วโมงเพื่อหา **Potential P90** (ศักยภาพสูงสุดของระบบ)
- นำข้อมูลทุกวันมากองรวมกันแยกตามชั่วโมง (เช่น 13:00 น. ของทุกวัน)
- หาค่า **Percentile 90 (P90)** ของการผลิต (ตัด 10% ยอดแหลมที่อาจเป็น Noise ทิ้ง)
- ใช้เส้น P90 นี้เป็น Baseline เทียบกับการผลิตจริง (Actual) เพื่อวาดกราฟหาช่วงที่เกิด Clipping

### `billPeriodKey(date)` → `"YYYY-MM"` string
แปลวันที่เป็น billing period key ตาม `CFG.billDay`
- ถ้าวันที่ >= billDay → period = เดือนถัดไป
- ถ้าวันที่ < billDay → period = เดือนปัจจุบัน

### `touActive(date)` → boolean
ตรวจว่าวันนั้นใช้ TOU แล้วหรือยัง  
```javascript
const touActive = (date) => CFG.touStart && date >= CFG.touStart
```

---

## 7. Column Auto-Detection (`detectC`)

ระบบ Auto-detect ชื่อคอลัมน์จาก XLSX:
```javascript
C = {
  time:     // คอลัมน์เวลา
  solar:    // Total DC Input Power (W)
  solarK:   // Daily Production (kWh)
  grid:     // Total Grid Power (W)
  feedIn:   // Daily Grid Feed-in (kWh)
  purchase: // Daily Energy Purchased (kWh) ← KEY for on/off peak calc
  load:     // Total Consumption Power (W)
  loadK:    // Daily Consumption (kWh)
  soc:      // BMS SOC (%)
  batt:     // Battery Power (W)
  pv1:      // DC Power PV1 (W)
  pv2:      // DC Power PV2 (W)
  // + voltage/current columns
}
```

Column matching ใช้ case-insensitive substring search:  
`f('total dc input power')` จะหา key ที่มีทุก word นี้ใน column name

---

## 8. Known Issues & Technical Debt

### แก้ไขแล้ว ✅
| Bug / Feature | การแก้ไข |
|-----|---------|
| `renderHistROI` ใช้ inverter estimate แทน MEA bill จริง | เพิ่ม `postSolarActualCost` lookup จาก `actualBills` |
| `mkPie` ไม่ถูกนิยาม → Tab "ทั้งหมด" crash | เพิ่ม `function mkPie(id,labels,data,colors)` |
| `HOLIDAYS` hardcoded ปี 2026 เท่านั้น | ย้ายเป็น `const THAI_HOLIDAYS` ครอบคลุมถึงปี 2027 |
| May 16-17 patch รัน global (ไม่ระบุปี) | เปลี่ยนเป็น `=== '2026-05-16'` exact match |
| Modal บิลไม่แสดง Pre/Post Solar status | เพิ่ม `getStatusBadge()` + badge UI |
| ชื่อ Heatmap ไม่สื่อความหมาย | สลับ Title เป็น "พึ่งตัวเอง %" และ Subtitle เป็น "Calendar Heatmap" |
| ผู้ใช้ไม่ทราบว่าฟิลด์ไหนสำคัญ | เพิ่ม Asterisk สีแดง (`*`) ที่ฟิลด์ config ที่ระบบนำไปคำนวณจริง |
| Tooltip & Glossary อธิบายขายไฟไม่ชัดเจน | อัปเดตข้อความให้ระบุชัดเจนว่ารวม Clipping (สมมติปลดกันย้อน) แล้ว |

### ยังมีอยู่ ⚠️
| ปัญหา | ตำแหน่ง | Priority |
|------|---------|---------|
| `dbAll()` ถูกเรียก 6 ครั้งต่อ `refresh()` | ทุก render function | Medium |
| Code ซ้ำใน `renderCalendarHeatmap` (first week + remaining weeks) | line 3193+ | Low |
| วันหยุดราชการต้อง update manual ทุกปี | `THAI_HOLIDAYS` | Low |
| `alert()` แทน Toast — block UI | หลายที่ | Low |
| ไม่มี Loading state ระหว่าง import | line ~1548 | Low |
| Magic numbers ในการคำนวณ simulation | line ~3371 | Low |

---

## 9. UI Component Map

### Modals
| ID | เปิดด้วย | เนื้อหา |
|----|---------|--------|
| `sysModal` | `openSystemInfo()` | ตั้งค่า System (inverter, panel, battery, dates, costs) |
| `billModal` | `openBillModal()` | ประวัติบิลค่าไฟ (histBills + actualBills) |
| `fmModal` | `openFileManager()` | จัดการข้อมูล XLSX ที่นำเข้าแล้ว |

### Tab System
```javascript
switchTab('D'|'W'|'M'|'Y'|'T'|'A')   // เปลี่ยน Tab หลัก
switchAnalysis('overview'|'day'|'week'|'month'|'year')  // เปลี่ยน Sub-tab ใน Analysis
```

---

## 10. Future Improvement Suggestions

### Phase 2 — Quick Wins (ไม่กระทบ Architecture)
1. **Toast Notification** แทน `alert()` — สร้าง function `showToast(msg, type)` ใน CSS+JS
2. **Loading Progress Bar** ระหว่าง Import — เพิ่ม progress element และ update ใน file loop
3. **วันหยุดราชการ configurable** — เพิ่ม textarea ใน sysModal สำหรับกรอก HOLIDAYS เพิ่ม
4. **Export to PDF** — ใช้ `window.print()` + print CSS
5. **Export to CSV** — Aggregate `allDaysSummary` แล้ว download

### Phase 3 — Architecture Improvements
6. **Refactor `refresh()`** — Pass `all` data ลงทุก render function แทนให้แต่ละตัว call `dbAll()`
   ```javascript
   // ปัจจุบัน: แต่ละ function เรียก dbAll() เอง (6 ครั้ง/refresh)
   // เป้าหมาย:
   async function refresh() {
     const all = await dbAll()  // เรียกครั้งเดียว
     await renderDay(cur, all)
     await renderWeek(cur, all)
     // ...
   }
   ```
7. **Split into modules** — แยก JS เป็น modules เมื่อไฟล์ใหญ่ขึ้น
8. **Web Worker** สำหรับ `calcSum` เพื่อไม่ block UI ระหว่าง import หลายไฟล์

### Phase 4 — Feature Expansion
9. **Multi-system support** — เพิ่ม system selector, แยก IndexedDB per system
10. **Inverter compatibility** — เพิ่ม column mapping profiles สำหรับ Growatt, Solis, Huawei
11. **PWA** — เพิ่ม Service Worker, manifest.json สำหรับ install บน Mobile
12. **Real-time mode** — WebSocket / API polling สำหรับ Inverter ที่มี local API

---

## 11. Code Conventions & Patterns

### Naming
- `render*()` — functions ที่ update DOM/Charts
- `calc*()` — functions ที่คำนวณและ return ค่า
- `load*()` — functions ที่โหลดจาก storage
- `save*()` — functions ที่ save ลง storage
- `open*()` / `close*()` — functions เปิด/ปิด Modal

### DOM Pattern
```javascript
const set = (id, v) => { const el = document.getElementById(id); if(el) el.innerHTML = v }
// ใช้ if(el) guard เสมอเพื่อป้องกัน null reference
```

### Chart Pattern
```javascript
// ทุก chart ต้อง destroy ก่อน recreate
if(charts[id]) charts[id].destroy()
charts[id] = new Chart(...)

// ใช้ mkChart() สำหรับ bar/line/pie ทั่วไป
mkChart(id, type, labels, datasets, yOverride, plugins)

// ใช้ mkPie() สำหรับ Pie chart สั้นๆ
mkPie(id, labels, dataArray, colorsArray)
```

### Bill Period Key Pattern
```javascript
// billPeriodKey('2026-05-20') with billDay=15 → '2026-06'  (หลัง 15 → รอบถัดไป)
// billPeriodKey('2026-05-10') with billDay=15 → '2026-05'  (ก่อน 15 → รอบนี้)
```

---

## 12. Critical Date/Period Logic

```
CFG.startDate = '2026-04-10'   ← วันที่ติด Solar (CRUCIAL)
CFG.touStart  = '2026-05-05'   ← วันที่เปลี่ยนมิเตอร์เป็น TOU
CFG.billDay   = 15             ← ตัดรอบบิลวันที่ 15

Timeline:
  Before startDate   → Pre-Solar bill (ใช้ flat rate calculation)
  startDate to touStart → Post-Solar, flat rate
  After touStart     → Post-Solar, TOU rate (On/Off peak)

actualBills classification:
  endStr = `${y}-${m}-${billDay-1}`
  isPostSolar = endStr >= CFG.startDate
```

---

## 13. File Modification Safety Guidelines

### 🛑 Code Safety Guidelines (กฎสำคัญสำหรับ AI)

1. **ห้ามเปลี่ยนโครงสร้างข้อมูล IndexedDB:** (ถ้าไม่จำเป็นจริงๆ) เพราะผู้ใช้อาจมีข้อมูลเก่าอยู่แล้ว
2. **ห้ามย้าย logic ออกจากไฟล์ `app.html`:** จุดประสงค์คือ Single-File Dashboard
3. **ฟังก์ชันที่ห้ามแก้ถ้าไม่เข้าใจ 100%:** `calcSum()`, `processFile()`, `renderYear()`
4. **Git Push:** **⚠️ ต้องถามผู้ใช้ (Ask for permission) ก่อนทำการ `git push` ทุกครั้ง ห้าม push อัตโนมัติเด็ดขาด**

เมื่อแก้ไข `app.html` ให้ระวัง:

1. **อย่าแก้ `calcSum()`** โดยไม่เข้าใจ meter logic ก่อน — ผลกระทบต่อตัวเลขทุกตัวในระบบ
2. **อย่าเปลี่ยน IndexedDB schema** โดยไม่ increment version และ handle `onupgradeneeded`
3. **`CFG.startDate` สำคัญที่สุด** — ถ้าผิดจะทำให้การแยก Pre/Post Solar ผิดทั้งหมด
4. **`kwhFactor` และ `costFactor`** ถูก apply ทุกครั้งที่ `dbAll()` ถูกเรียก ไม่ต้อง apply เอง
5. **Chart IDs** ต้อง unique ทั่วทั้งไฟล์ — ถ้าซ้ำจะ destroy chart อื่นโดยไม่ตั้งใจ
6. **`THAI_HOLIDAYS`** ต้อง update ทุกปี (ก่อนสิ้นปี) เพื่อให้ On-peak calculation ถูกต้อง

---

## 14. Quick Diagnostic Checklist

เมื่อ User รายงานปัญหา ถามสิ่งเหล่านี้:

```
[ ] CFG.startDate ตั้งถูกต้องหรือไม่?
[ ] CFG.touStart ตั้งถูกต้องหรือไม่?
[ ] CFG.billDay ตรงกับรอบบิลจริงหรือไม่?
[ ] actualBills มีข้อมูลหรือไม่? (เปิด Modal > ดู "บิลจริง")
[ ] ข้อมูล XLSX มีคอลัมน์ครบหรือไม่? (diagnoseDB() แจ้งจำนวน rows)
[ ] Browser เปิดจาก path/URL เดิมหรือไม่? (IndexedDB แยกตาม Origin)
[ ] partialDays มีวันที่ผิดปกติที่ควรตัดออกหรือไม่?
```

---

*อัปเดตล่าสุด: พฤษภาคม 2569 (2026) — หลังแก้ไข 8 bugs รอบแรก*
