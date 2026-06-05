# Project Memo: Deye Solar Insight

> Memo นี้ไว้กันทำงานผิดโฟลเดอร์, ลืม push, หรือเข้าใจผิดว่า GitHub Pages deploy แล้ว ทั้งหมดนี้เป็น note สำหรับ maintainer ก่อนแก้/ปล่อยเวอร์ชันใหม่

## Repo ที่ถูกต้อง

ให้ทำงานจาก repo root ที่มีไฟล์เหล่านี้อยู่ระดับเดียวกัน:

- `.git/`
- `app.html`
- `index.html`
- `README.md`
- `README.ai.md`
- `docs/`

เช็คด้วยคำสั่ง:

```powershell
git rev-parse --show-toplevel
git status -sb
```

ถ้าอยู่ในโฟลเดอร์ซ้อนชื่อ `Solar-Insight/` ให้ระวัง เพราะในเครื่องนี้โฟลเดอร์นั้นเป็นไฟล์/ข้อมูลที่ Git มองเป็น untracked ไม่ใช่ source หลักของแอป

## ก่อนบอกว่า "เรียบร้อยแล้ว"

ต้องเช็คอย่างน้อย:

```powershell
git status -sb
git log --oneline --decorate -5
```

ถ้าแก้ `app.html` ให้เช็ค script inline:

```powershell
@'
const fs=require('fs');
const vm=require('vm');
const html=fs.readFileSync('app.html','utf8');
const scripts=[...html.matchAll(/<script>([\s\S]*?)<\/script>/g)].map(m=>m[1]);
scripts.forEach((src,i)=>new vm.Script(src,{filename:'inline-'+(i+1)+'.js'}));
console.log('SCRIPT_OK scripts='+scripts.length);
'@ | node
```

และเช็ค whitespace:

```powershell
git diff --check
```

หมายเหตุ: บน Windows อาจมี warning เรื่อง `LF will be replaced by CRLF` ได้ ถ้าไม่มี error จาก `git diff --check` ถือว่าไม่ใช่ blocker

## GitHub Pages

หลัง `git push` ให้แยกเช็ค 2 ชั้น:

1. Raw GitHub: ยืนยันว่า commit อยู่บน GitHub แล้ว
2. GitHub Pages: ยืนยันว่าเว็บ live deploy แล้ว

GitHub Pages อาจช้ากว่า raw GitHub ประมาณ 20-60 วินาที ถ้า raw มีของใหม่แต่ Pages ยังไม่มี ให้รอแล้วเช็คซ้ำก่อนสรุป

ถ้าผู้ใช้เปิดเว็บแล้วยังเห็นของเก่า ให้ลอง hard refresh หรือเปิด private window เพราะ browser อาจ cache ไฟล์เก่า

## Untracked files ที่ตั้งใจไม่แตะ

ถ้าเห็นสถานะแบบนี้:

```text
?? Solar-Insight/
?? prompt.xlsx
```

อย่าเพิ่งลบหรือ add อัตโนมัติ

- `Solar-Insight/` อาจมีไฟล์ inverter, note เก่า, `node_modules`, หรือ `V1a`
- `prompt.xlsx` เป็นไฟล์นอก source หลัก

ให้แตะเฉพาะเมื่อผู้ใช้สั่งชัดเจน

## Data และ Privacy

แอปนี้เป็น static web app ไม่มี backend ของเรา

- ไฟล์ inverter ที่ import ไม่ถูกส่งขึ้น GitHub
- ข้อมูลที่ผู้ใช้ import/save อยู่ใน browser storage เช่น IndexedDB/localStorage
- ถ้าใช้เครื่องอื่น browser อื่น หรือ browser ที่ล้าง storage แล้ว ข้อมูลเดิมจะไม่ตามไปด้วย
- ก่อนล้าง browser storage ควร export/backup ข้อมูลก่อน

## Lessons Learned

### 1. Repo root ต้องชัดก่อนทำงาน

ความผิดพลาดที่เจอคือมีโฟลเดอร์ `Solar-Insight/` ซ้อนอยู่ ทำให้เข้าใจผิดได้ง่ายว่าเป็น repo จริง ทั้งที่ source หลักอยู่ที่ repo root

วิธีป้องกัน:

- เริ่มงานด้วย `git rev-parse --show-toplevel`
- ดูว่า `app.html` อยู่ระดับเดียวกับ `.git/`
- อย่าใช้ชื่อโฟลเดอร์เป็นหลัก ให้ใช้ Git root เป็นหลัก

### 2. GitHub push แล้ว ไม่ได้แปลว่า Pages deploy แล้วทันที

Raw GitHub อาจอัปเดตก่อน แต่ Pages ยังเป็น cache เก่า

วิธีป้องกัน:

- เช็ค raw ก่อน
- เช็ค Pages หลังรอสั้น ๆ
- ถ้า Pages ยังเก่า ให้บอกว่า deploy/cache ยังตามไม่ทัน ไม่ใช่สรุปว่า push fail

### 3. Warning ต้องดูพฤติกรรมทั้งช่วง ไม่ใช่จุดแรกกับจุดสุดท้าย

เคส `แบตไม่ชาร์จช่วง Solar` เคยเตือนผิด เพราะดู SOC ตอนต้นช่วง 09:00-15:00 เทียบกับท้ายช่วงอย่างเดียว ทั้งที่ระหว่างวันแบตเคยชาร์จขึ้นไปแล้ว

แนวทางที่ดีกว่า:

- ใช้ `max SOC` ระหว่างช่วงช่วยตัดสิน
- ถ้า SOC เคยเพิ่มชัดเจน หรือแตะระดับสูงแล้ว ไม่ควรเตือนว่าแบตไม่ชาร์จ

### 4. Glossary ต้องเก็บทั้งสูตรและหลักคิด

สูตรอย่างเดียวช่วย verify code ได้ แต่ผู้ใช้ต้องการเข้าใจหลักคิดด้วย เช่น P90, clipping, K-Factor, ROI, pre-TOU flat billing

แนวทางที่ดีกว่า:

- ทุกสูตรสำคัญควรมีหลักคิด
- มีสมการ
- มีตัวอย่างแทนค่า
- ระบุข้อจำกัดของสูตร

### 5. Calibration ต้องแยก kWh กับเงินบาท

หลักที่ตกลงไว้:

- kWh จาก inverter ไม่ควรถูกเปลี่ยน
- ค่าเงินบาทที่มี `*` คือค่าที่ใช้ bill override หรือ K-Factor
- ต้องเขียนให้ชัดใน UI/Glossary เพราะผู้ใช้จะใช้เทียบกับบิลจริง

### 6. Pre-TOU สำคัญสำหรับผู้ใช้ใหม่

หลายบ้านติด solar ก่อนติดมิเตอร์ TOU 1-4 เดือน จึงต้องรองรับช่วงก่อน TOU

หลักปัจจุบัน:

- เก็บข้อมูล import ได้ตามที่มี
- ก่อนวันเริ่ม TOU ให้คิดค่าไฟแบบ flat
- หลังวันเริ่ม TOU จึงแยก on-peak/off-peak

### 7. Public beta ต้องเตือนเรื่องขอบเขตให้ชัด

แอปนี้เหมาะกับ:

- inverter data format แบบ Deye ที่ทดสอบแล้ว
- TOU meter
- PV string ไม่เกิน 2 string
- ใช้งานบน desktop browser เป็นหลัก

สิ่งที่ยังไม่ควร claim:

- รองรับ inverter ทุกยี่ห้อ
- ใช้แทนบิลการไฟฟ้าได้ 100%
- simulation แม่นเท่าการจำลองระบบไฟฟ้าจริงรายชั่วโมงทุกกรณี

## Quick Release Checklist

```text
[ ] อยู่ repo root ถูกต้อง
[ ] git status ไม่มี tracked change ที่หลงเหลือ
[ ] ถ้าแก้ app.html: SCRIPT_OK
[ ] git diff --check ผ่าน
[ ] commit message ชัด
[ ] push แล้ว
[ ] raw GitHub มีของใหม่
[ ] GitHub Pages มีของใหม่
[ ] แจ้ง untracked files ถ้ายังมี แต่ไม่แตะเอง
```
