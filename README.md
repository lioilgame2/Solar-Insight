# ⚡ SolarInsight Dashboard

> **แดชบอร์ดวิเคราะห์ระบบโซลาร์เซลล์แบบ Offline-First**  
> ไม่ต้องติดตั้ง · ไม่ต้องมี Server · เปิดไฟล์เดียวใช้ได้เลย

[![HTML](https://img.shields.io/badge/HTML-Single_File-orange?style=flat-square&logo=html5)](app.html)
[![Chart.js](https://img.shields.io/badge/Chart.js-4.4-pink?style=flat-square)](https://www.chartjs.org/)
[![IndexedDB](https://img.shields.io/badge/Storage-IndexedDB-blue?style=flat-square)](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## 🌟 ทำไมต้องใช้ SolarInsight?

ระบบ Solar ส่วนใหญ่มี App ของตัวเองจากผู้ผลิต แต่มักมีข้อจำกัด:  
❌ ดูได้แค่ real-time ไม่มี deep analysis  
❌ ไม่สามารถเปรียบเทียบค่าไฟก่อน-หลังติดได้  
❌ ต้องอาศัย Cloud ถ้า Server ล่มก็ดูไม่ได้  
❌ ไม่รองรับการนำเข้าบิลจริงจาก MEA/PEA  

**SolarInsight แก้ปัญหาทั้งหมดนี้ด้วยไฟล์ HTML ไฟล์เดียว**

---

## ✨ ฟีเจอร์หลัก

### 📊 วิเคราะห์หลายมิติ
| มุมมอง | สิ่งที่เห็นได้ |
|--------|--------------|
| **รายวัน** | กราฟพลังงาน 24 ชั่วโมง · อุณหภูมิ · แรงดัน · กระแส |
| **รายสัปดาห์** | เปรียบเทียบ WoW · String A vs B · Self-sufficiency |
| **รายเดือน (รอบบิล)** | วิเคราะห์ตามรอบบิลจริง · Calendar Heatmap |
| **รายปี** | YoY · Production trend · Battery cycle count |
| **ทั้งหมด** | ROI คืนทุน · Simulation · System health |
| **วิเคราะห์** | เปรียบเทียบค่าไฟก่อน/หลังติด Solar · Clipping detection |

### 🔑 จุดเด่น
- **📋 บิลจริง MEA/PEA** — กรอกบิลจริงเพื่อเปรียบเทียบกับ baseline ก่อนติด Solar
- **🎯 Calibration Factor** — ปรับค่า kWh ให้ตรงกับ meter จริงโดยอัตโนมัติ
- **🔋 Battery Health** — ติดตาม cycle count, SOC minimum, อุณหภูมิ
- **⚡ Clipping Detection** — ตรวจจับพลังงานที่สูญเสียเมื่อแบตเต็ม
- **🌡️ Smart Alerts** — แจ้งเตือน String Imbalance, อุณหภูมิสูง, Voltage ผิดปกติ
- **💰 ROI Calculator** — คำนวณคืนทุนจากข้อมูลจริง พร้อม Simulation
- **🗓️ TOU Rate Support** — รองรับอัตรา TOU (On-peak/Off-peak) ของ MEA
- **📱 Offline-First** — ข้อมูลเก็บใน Browser ไม่ส่งไปที่ไหน

---

## 🚀 เริ่มใช้งานใน 3 ขั้นตอน

### ขั้นตอนที่ 1 — เปิดไฟล์
```
เปิด app.html ด้วย Chrome หรือ Edge (แนะนำ Chrome ล่าสุด)
```
> ⚠️ ต้องเปิดผ่าน Browser โดยตรง ไม่รองรับการเปิดใน iframe หรือ WebView

### ขั้นตอนที่ 2 — ตั้งค่าระบบ
กดปุ่ม **⚙️ ระบบ** แล้วกรอก:
- ข้อมูล Inverter / Panel / Battery
- **วันที่เริ่มใช้ Solar** (สำคัญมาก!)
- **วันที่เปลี่ยนมิเตอร์ TOU**
- ต้นทุนการติดตั้ง (สำหรับคำนวณ ROI)

### ขั้นตอนที่ 3 — นำเข้าข้อมูล Inverter
1. ดาวน์โหลดไฟล์ XLSX จาก **Deye Cloud** (หรือ Inverter App ที่รองรับ)
2. กดปุ่ม **📁 Inverter Data** แล้วเลือกไฟล์
3. รอสักครู่ → ข้อมูลจะแสดงผลทันที

---

## 📱 Inverter ที่รองรับ

| ผู้ผลิต | รุ่น | รูปแบบข้อมูล |
|---------|------|-------------|
| **DEYE** | SUN-5K-SG04LP1 และ Series ใกล้เคียง | XLSX จาก Deye Cloud |
| อื่นๆ | Hybrid Inverter ที่ Export XLSX | ต้องมีคอลัมน์เวลา + kWh |

> Inverter ที่ Export ข้อมูล XLSX พร้อมคอลัมน์พลังงาน Solar / Grid / Load สามารถ Map ได้อัตโนมัติ

---

## 💡 ฟีเจอร์พิเศษ — เปรียบเทียบค่าไฟก่อน/หลัง Solar

กดปุ่ม **💰 ประวัติบิล** แล้ว:
1. กรอก **บิลก่อนติด Solar** (12 เดือนที่แล้วสำหรับ Baseline)
2. กรอก **บิลจริงจาก MEA/PEA หลังติด** ทุกเดือน
3. ระบบจะแสดงกราฟเปรียบเทียบ พร้อมยอดประหยัดสะสม

แดชบอร์ดจะแสดง badge **"หลังติด ⚡"** หรือ **"ก่อนติด 📋"** บนบิลแต่ละรายการให้ชัดเจน

---

## 🛠️ Tech Stack

```
Frontend Only — ไม่มี Backend ไม่มี Server
├── HTML5 / CSS3 / Vanilla JavaScript
├── Chart.js 4.4        — กราฟทุกประเภท
├── SheetJS (XLSX)       — อ่านไฟล์ Excel จาก Inverter
├── IndexedDB            — เก็บข้อมูลใน Browser (persistent)
└── Google Fonts         — Outfit + JetBrains Mono
```

**ขนาดไฟล์**: ~225 KB (HTML เดียว รวมทุกอย่าง)  
**Dependencies**: โหลดจาก CDN (ต้องมี internet ครั้งแรก)

---

## 📁 โครงสร้างไฟล์

```
Test/
├── app.html          ← ไฟล์หลัก (ทุกอย่างอยู่ที่นี่)
├── README.md         ← คุณกำลังอ่านอยู่
└── README.ai.md      ← เอกสารสำหรับ AI/Developer
```

---

## 🔒 Privacy & Security

- **ข้อมูลทั้งหมดอยู่ใน Browser ของคุณ** — ไม่มีการส่งข้อมูลออกไปที่ Server ใดๆ
- เก็บใน `IndexedDB` ของ Browser เฉพาะเครื่องและ Origin นั้น
- ต้องการ Internet เฉพาะตอนโหลด CDN ครั้งแรก (Chart.js, SheetJS)

---

## ⚠️ ข้อจำกัดที่ต้องรู้

- **Single Browser Origin** — ถ้าเปิดจากหลาย path/URL จะเห็น IndexedDB คนละฐาน
- **ข้อมูลอยู่ที่ Browser นั้น** — ถ้า Clear browser data จะหายทั้งหมด
- **วันหยุดราชการ** — ครอบคลุมถึงปี 2027 (ต้องอัปเดต `THAI_HOLIDAYS` ในโค้ดเมื่อมีปีใหม่)
- **ทดสอบกับ DEYE** — Inverter ยี่ห้ออื่นอาจต้องปรับ Column Mapping

---

## 🗺️ Roadmap

- [ ] Export ข้อมูลเป็น PDF/Excel
- [ ] Multi-system support (หลายบ้าน)
- [ ] Notification Alert (Low Battery, High Temp)
- [ ] วันหยุดราชการ configurable ใน UI
- [ ] รองรับ Inverter อื่นเพิ่มเติม (Growatt, Solis, etc.)
- [ ] Progressive Web App (PWA) — ใช้แบบ Offline บน Mobile

---

## 🤝 Contributing

Pull Requests ยินดีต้อนรับ! กรุณาอ่าน `README.ai.md` ก่อนเพื่อเข้าใจ Architecture  
แล้ว open Issue เพื่อพูดคุยก่อน implement

---

## 📄 License

MIT License — ใช้ได้อิสระ ทั้งส่วนตัวและเชิงพาณิชย์

---

<p align="center">
  Made with ☀️ for solar system owners who want real insights, not just pretty graphs.
</p>
