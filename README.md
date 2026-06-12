> This README starts with a Thai user guide. The English section is below.
> เอกสารนี้เปิดด้วยภาษาไทย ส่วนภาษาอังกฤษอยู่ด้านล่าง

# คู่มือใช้งาน Deye Solar Insight (Public Beta)


[เปิดใช้งานผ่าน GitHub Pages](https://lioilgame2.github.io/Solar-Insight/) · [อ่านรายละเอียดเทคนิคสำหรับ AI/Developer](README.ai.md)

Deye Solar Insight เป็น dashboard สำหรับอ่านไฟล์ export จาก inverter แล้วช่วยดูพลังงาน Solar, Battery, Grid, ค่าไฟ, รอบบิล, และ ROI จาก browser โดยไม่ต้องมี server ของตัวเอง

## สิ่งที่ควรรู้ก่อนเริ่ม

- ต้อง import ไฟล์ XLSX จาก inverter ก่อน จึงจะเห็นกราฟและการวิเคราะห์
- โปรเจกต์นี้ทดสอบกับไฟล์ export จาก `Deye` เป็นหลัก
- ยี่ห้ออื่นอาจใช้ได้ถ้าโครงสร้างคอลัมน์ใกล้เคียง แต่ยังไม่รับประกัน
- เหมาะกับมิเตอร์แบบ `TOU` มากที่สุด แต่ช่วงก่อนเริ่ม TOU ใช้งานได้ โดยระบบจะคิดค่าไฟแบบ `Flat rate` ทั้งวันจนกว่าจะถึง `TOU start date`
- วัน Off-peak ทั้งวัน = เสาร์–อาทิตย์ + วันหยุดราชการตามประกาศ กกพ./MEA โดย **วันหยุดชดเชย วันพืชมงคล และวันหยุดพิเศษของ ครม. ไม่นับเป็น off-peak**
- รายการวันหยุด off-peak ต้องยืนยันใหม่ทุกปีตามประกาศ — ปีที่แอปยืนยันแล้วคือ `2569` ถ้า import ข้อมูลปีที่ยังไม่ยืนยัน แอปจะขึ้นแถบเตือน และกรอกวันหยุดเพิ่มเองได้ในปุ่ม `ระบบ`
- กราฟเปรียบเทียบ PV string เวอร์ชันนี้รองรับสูงสุด `2 strings`
- ข้อมูลทั้งหมดเก็บใน browser ของเครื่องที่ใช้งาน ไม่ได้ส่งไปเก็บบน server ของโปรเจกต์
- เวอร์ชันนี้เหมาะกับ desktop/laptop มากกว่ามือถือ

## เริ่มใช้งานแบบสั้นที่สุด

1. เปิด [GitHub Pages](https://lioilgame2.github.io/Solar-Insight/) หรือเปิด `app.html` ด้วย Chrome/Edge
2. กด `ระบบ`
3. กรอกข้อมูลหลัก เช่น วันเริ่มใช้ Solar, วันเริ่ม TOU, ขนาดระบบ, ค่าไฟ, เงินเฟ้อ, แบตเตอรี่, และต้นทุน
4. กด `inverter data` แล้วเลือกไฟล์ XLSX ที่ export จาก Deye
5. ถ้ามีบิลจริง ให้ไปที่ `ประวัติบิล` แล้วกรอกบิลก่อนติดตั้งและหลังติดตั้ง
6. ดูผลที่ tab `รายวัน`, `รายเดือน`, และ `วิเคราะห์`

## ข้อมูลที่ควรเตรียม

| ข้อมูล | ใช้ทำอะไร |
|---|---|
| ไฟล์ XLSX จาก inverter | แหล่งข้อมูลหลักสำหรับกราฟและสรุปรายวัน |
| วันเริ่มใช้ Solar | แยกช่วงก่อน/หลังติดตั้ง |
| วันเริ่ม TOU | เปลี่ยนสูตรค่าไฟจาก Flat เป็น TOU |
| ค่าไฟ On-peak / Off-peak / Flat / Ft / ค่าบริการ | คำนวณค่าไฟรายวันและรอบบิล |
| Battery empty %, recharge %, design cycles | วิเคราะห์แบตหมดและประเมิน battery cycle |
| เงินเฟ้อไทย (%/ปี) | ใช้กับ ROI countdown แบบคิดมูลค่าเงินในอนาคต |
| บิลจริงจาก MEA/PEA | ใช้ calibrate ค่าไฟให้ใกล้บิลจริง |

## เรื่องไฟล์จาก inverter

เวอร์ชันปัจจุบันพัฒนาจากไฟล์ export ของ `Deye` เป็นหลัก

ถ้าใช้ inverter ยี่ห้ออื่น ระบบอาจ import ได้ แต่มีความเสี่ยงเรื่อง:

- ชื่อคอลัมน์ไม่ตรง
- หน่วยข้อมูลไม่เหมือนกัน
- ไม่มี field ที่ระบบใช้คำนวณ เช่น PV string, SOC, temperature, grid purchase/export
- เวลาในไฟล์ไม่ตรงกับ timezone หรือรูปแบบที่ระบบคาดไว้

ถ้า import แล้วกราฟหรือค่าคำนวณดูผิด ให้เริ่มจากเช็กว่าไฟล์มีข้อมูล solar, load, grid, battery, SOC, temperature และ PV string ครบหรือไม่

## ปุ่ม `ระบบ` ใช้ตั้งค่าอะไร

| หมวด | ตัวอย่าง | ใช้คำนวณหรือแสดงผล |
|---|---|---|
| ชื่อระบบ / inverter / PV model | Deye Solar Insight, DEYE model, LONGi model | แสดงผล |
| ขนาดระบบ / panels / W per panel / peak sun | 5 kW, 10 panels, 645 W, 4.5 h | คำนวณ |
| Battery kWh / charge A / empty % / recharge % / design cycles | 10.24 kWh, 48 A, 21%, 25%, 6000 cycles | คำนวณ |
| ค่าไฟ / Ft / ค่าบริการ / ขายคืน | On-peak, Off-peak, Flat, Ft | คำนวณ |
| เงินเฟ้อไทย | 2.89%/ปี | คำนวณ ROI |
| เป้าค่าไฟ (฿/เดือน) | 600 | สี heatmap ค่าไฟรายวัน |
| วันหยุด Off-Peak เพิ่มเติม | 2027-01-01,... | คำนวณ (สำหรับปีที่ยังไม่มีรายการในแอป) |
| Simulation | เปิด/ปิดกล่องจำลองใน tab วิเคราะห์ | แสดงผล |

ในหน้า `ระบบ` มีสีแยกไว้:

- จุดสีแดง = ใช้คำนวณ
- จุดสีฟ้า = ใช้แสดงผล

## ประวัติบิลและ K-Factor

`K-Factor` คืออัตราปรับค่าเงินจากข้อมูล inverter ให้ใกล้บิลจริงมากขึ้น โดยไม่เปลี่ยนค่า kWh ดิบ

หลักการ:

```text
K-Factor = ค่าไฟจริงจากบิล / ค่าไฟที่ระบบคำนวณจาก inverter
ค่าไฟที่ปรับแล้ว = ค่าไฟจาก inverter × K-Factor
```

ตัวอย่าง:

```text
ค่าไฟจาก inverter = 520 บาท
ค่าไฟจริงจากบิล = 560 บาท
K-Factor = 560 / 520 = 1.077
ค่าไฟรายวันที่อยู่ในรอบบิลนั้นจะถูกคูณด้วย 1.077
```

หมายเหตุ:

- ค่าไฟที่มี `*` คือค่าที่ผ่านการปรับจากบิลจริงหรือ K-Factor
- ค่า kWh ดิบไม่ถูกแก้
- ถ้ายังไม่มีบิลจริง ระบบจะใช้ค่าประมาณจากสูตรค่าไฟและข้อมูล inverter

## Backup / ย้ายเครื่อง

ข้อมูลใน browser ไม่ตามไปเองเมื่อเปลี่ยนเครื่องหรือเปลี่ยน browser

ในปุ่ม `ระบบ` มี:

- `Export backup` สำหรับ export ไฟล์ JSON
- `Import backup` สำหรับนำไฟล์ JSON กลับมาใช้

Backup นี้รวม:

- ข้อมูลระบบที่กรอกเอง
- บิลก่อนติดตั้ง
- บิลจริงหลังติดตั้ง

Backup นี้ไม่รวม:

- ไฟล์ inverter XLSX ที่ import แล้ว
- raw data รายวันใน IndexedDB

ถ้าจะย้ายเครื่อง แนะนำให้ export backup และเก็บไฟล์ XLSX ต้นฉบับจาก inverter ไว้ด้วย

## Tab หลักในแอป

| Tab | ใช้ดูอะไร |
|---|---|
| รายวัน | พลังงานรายชั่วโมง, SOC, อุณหภูมิ, ค่าไฟวันล่าสุด |
| รายสัปดาห์ | pattern ระยะสั้นและการเปรียบเทียบรายสัปดาห์ |
| รายเดือน | รอบบิล, heatmap รายวัน, ค่าไฟ, self-sufficiency |
| รายปี | trend ระยะยาวและภาพรวมรายเดือน |
| ทั้งหมด | สรุปภาพรวมทุกข้อมูลที่ import |
| วิเคราะห์ | ROI, battery cycle, simulation, before/after bill |
| คู่มือ | คู่มือสั้นในตัวแอป |

ถ้าต้องการรายละเอียดมากกว่าหน้า `คู่มือ` ในแอป ให้อ่านไฟล์นี้ (`README.md`) บน GitHub และถ้าต้องการรายละเอียดเชิงเทคนิคหรือให้ AI ช่วยต่อยอด ให้อ่าน `README.ai.md`

## ข้อจำกัดสำคัญ

- Public Beta ยังอาจมี edge case จากไฟล์ inverter บางรูปแบบ
- ข้อมูลทั้งหมดอยู่ใน browser ของเครื่องที่ใช้งาน ถ้าล้าง browser storage ข้อมูลที่ import ไว้อาจหาย
- ไม่ควรใช้บนเครื่องคนอื่นหรือเครื่องสาธารณะ ถ้าจำเป็นควรล้างข้อมูลหลังใช้งาน
- Simulation เป็นการประมาณเชิงทิศทาง ไม่ใช่ผลการันตี
- การคำนวณค่าไฟควรถูกเทียบกับบิลจริงเสมอเมื่อมีข้อมูล
- เวอร์ชันนี้ไม่ใช่ mobile-first

## โครงสร้างไฟล์

```text
Solar-Insight/
├── app.html      # แอปหลักแบบ static HTML
├── index.html    # redirect/entry สำหรับ GitHub Pages
├── README.md     # คู่มือผู้ใช้และภาพรวม public
└── README.ai.md  # technical reference สำหรับ developer/AI
```

## สำหรับคนที่อยากช่วยทดสอบ

ลองใช้งานกับไฟล์ Deye XLSX ของตัวเอง แล้วดูว่า:

- import ได้ครบหรือไม่
- ค่าไฟใกล้บิลจริงหรือไม่
- เวลาแบตหมดสมเหตุสมผลหรือไม่
- heatmap และกราฟอ่านง่ายหรือไม่
- มี field ของ inverter รุ่นอื่นที่ควรรองรับเพิ่มหรือไม่

---

# Deye Solar Insight (Public Beta)

> Offline-first solar analytics for Deye hybrid inverter owners.
> One HTML file. No server. No login. Your inverter data stays in your browser.

[Open the live app](https://lioilgame2.github.io/Solar-Insight/) · [Technical reference for AI/developers](README.ai.md)

## What It Is

Deye Solar Insight is a browser-based dashboard for understanding home solar behavior beyond the default inverter app.

It turns exported inverter XLSX files and real electricity bills into practical answers:

- Is the solar system reducing the bill?
- When does the battery run out?
- Which PV string is underperforming?
- Is the inverter getting hot?
- How much should the monthly bill be after real-bill calibration?
- What is the payback period if the current usage pattern continues?

The app is built and tested primarily around Deye hybrid inverter XLSX exports. Other inverter brands may work if their exported columns are similar, but compatibility is not guaranteed.

## Key Scope

| Area | Current support |
|---|---|
| App type | Static browser app |
| Main file | `app.html` |
| Hosting | GitHub Pages or local file |
| Storage | Browser IndexedDB/localStorage fallback |
| Tested inverter export | Deye XLSX |
| Meter mode | Best for TOU; pre-TOU flat-rate period is supported |
| Off-peak holidays | Per ERC/MEA annual announcement; substitution holidays, Royal Ploughing Day, and special cabinet holidays are NOT off-peak; verified year list ships in-app (2026) with a warning + custom-date field for newer years |
| PV string comparison | Up to 2 strings |
| Best device | Desktop/laptop |

## Quick Start

1. Open the [live app](https://lioilgame2.github.io/Solar-Insight/) or open `app.html` in Chrome/Edge.
2. Click `ระบบ` and configure system information.
3. Click `inverter data` and import Deye XLSX day-report files.
4. Add real electricity bills in `ประวัติบิล` when available.
5. Review daily, monthly, and analysis tabs.

## Data And Privacy

Deye Solar Insight is offline-first:

- Imported inverter data is stored in the current browser.
- The project does not receive or store your inverter data.
- Data does not automatically move to another browser or machine.
- Clearing browser storage can remove saved data.
- Use `Export backup` in `ระบบ` to save manually entered system settings and bill history.

The backup file does not include imported inverter rows. Keep the original XLSX files if you want a full restore on another machine.

## Calibration

Real bills can be used to calibrate calculated electricity cost:

```text
K-Factor = real bill cost / calculated inverter-based cost
adjusted cost = inverter-based cost × K-Factor
```

Adjusted cost values are marked with `*`. Raw kWh values are not changed.

## Documentation

- `README.md`: public user guide and project overview
- `README.ai.md`: technical reference, formulas, implementation notes, and AI handoff context

## Status

Public Beta. Useful for Deye users today, but still expected to need improvements for other inverter brands and edge cases.
