# Onboarding And README Help Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a first-run welcome modal and a new main `คู่มือ` tab so GitHub Pages users understand how to start, what data is supported, and where their data is stored.

**Architecture:** Keep the single-file `app.html` structure intact. Add one new main tab pane, one lightweight modal overlay, and small browser-local preference helpers so onboarding state can be persisted without changing the day-data schema.

**Tech Stack:** HTML, CSS, vanilla JavaScript, IndexedDB/localStorage fallback, existing modal/tab helpers

---

## File Structure

- Modify: `D:\TMA Drive\OneDrive - Toyota Asia\Temp\AI Project\Google\Test\app.html`
  - Add styles for onboarding modal and guide layout
  - Add `คู่มือ` main tab and content pane
  - Add welcome modal markup
  - Add lightweight onboarding preference helpers
  - Extend tab switching and boot flow
- Modify: `D:\TMA Drive\OneDrive - Toyota Asia\Temp\AI Project\Google\Test\README.md`
  - Add short note that the live app now includes an in-app guide and first-run onboarding

### Task 1: Add onboarding and guide UI structure

**Files:**
- Modify: `D:\TMA Drive\OneDrive - Toyota Asia\Temp\AI Project\Google\Test\app.html`

- [ ] **Step 1: Add the failing UI targets in markup**

Add a new main tab button, a guide content section, and a hidden welcome modal shell near the existing tab/content markup.

```html
<div class="tab" id="tbR" onclick="switchTab('R')">📘 คู่มือ</div>

<section id="tabR" style="display:none">
  <div class="guide-hero">
    <div>
      <div class="guide-kicker">Deye Solar Insight (Public Beta)</div>
      <h1>เริ่มใช้งานให้ถูกตั้งแต่ครั้งแรก</h1>
      <p>ใช้กับไฟล์ export จาก Deye เป็นหลัก ข้อมูลเก็บใน browser เครื่องนี้ และควรใช้งานบน desktop/laptop</p>
    </div>
    <div class="guide-cta-row">
      <button class="btn btn-primary" onclick="openSystemInfo()">ระบบ</button>
      <button class="btn btn-sec" onclick="triggerImportPicker()">inverter data</button>
      <button class="btn btn-sec" onclick="openHistModal()">ประวัติบิล</button>
    </div>
  </div>
</section>

<div id="welcomeOverlay" class="modal-overlay" style="display:none">
  <div class="modal welcome-modal">
    <div class="modal-ttl">👋 ยินดีต้อนรับสู่ Deye Solar Insight</div>
  </div>
</div>
```

- [ ] **Step 2: Add guide and welcome styles**

Extend the existing CSS with focused classes for the guide layout and compact welcome modal.

```css
.guide-page{display:grid;gap:14px}
.guide-hero{background:var(--surface);border:1px solid var(--border);border-radius:var(--r);padding:18px;display:grid;gap:14px}
.guide-kicker{font-size:.74rem;color:var(--solar);text-transform:uppercase;letter-spacing:.08em;font-weight:700}
.guide-cta-row{display:flex;gap:8px;flex-wrap:wrap}
.guide-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(260px,1fr));gap:14px}
.guide-card{background:var(--surface);border:1px solid var(--border);border-radius:var(--r);padding:16px}
.guide-card h3{font-size:1rem;margin-bottom:8px}
.guide-card ul{padding-left:18px;color:var(--muted);display:grid;gap:6px}
.welcome-modal{width:min(720px,92vw)}
.welcome-list{padding-left:18px;display:grid;gap:7px;color:var(--muted)}
.welcome-actions{display:flex;justify-content:space-between;gap:10px;flex-wrap:wrap;align-items:center}
.welcome-note{font-size:.82rem;color:var(--muted);line-height:1.6}
```

- [ ] **Step 3: Verify the file still parses after markup/style insertion**

Run:

```powershell
node -e "const fs=require('fs'),vm=require('vm');const code=fs.readFileSync('app.html','utf8');const scripts=[...code.matchAll(/<script(?!(?:[^>]*src=))[^>]*>([\s\S]*?)<\/script>/gi)].map(m=>m[1]);for(const [i,s] of scripts.entries()){new vm.Script(s,{filename:'inline-script-'+(i+1)+'.js'});}console.log('SCRIPT_OK scripts='+scripts.length+' lines='+code.split(/\r?\n/).length);"
```

Expected: `SCRIPT_OK ...`

- [ ] **Step 4: Commit the UI skeleton**

```bash
git add app.html
git commit -m "Add onboarding modal and guide tab skeleton"
```

### Task 2: Add onboarding state, navigation, and actions

**Files:**
- Modify: `D:\TMA Drive\OneDrive - Toyota Asia\Temp\AI Project\Google\Test\app.html`

- [ ] **Step 1: Add preference helpers and file-trigger helper**

Create lightweight helpers near the existing settings/local storage helpers.

```javascript
async function getUiPref(key, fallback=null){
  const v=await loadSetting('ui:'+key).catch(()=>fallback)
  return v==null?fallback:v
}
async function setUiPref(key, value){
  return saveSetting('ui:'+key, value).catch(()=>{})
}
function triggerImportPicker(){
  const input=document.getElementById('file-input')
  if(input) input.click()
}
```

- [ ] **Step 2: Add welcome modal control functions**

Implement functions to open/close the modal, persist hide preference, and route `อ่านคู่มือ` into the new tab.

```javascript
function openWelcome(){const el=document.getElementById('welcomeOverlay'); if(el) el.style.display='flex'}
function closeWelcome(){const el=document.getElementById('welcomeOverlay'); if(el) el.style.display='none'}
async function dismissWelcome(){
  const hide=document.getElementById('welcomeHideToggle')?.checked
  await setUiPref('welcomeDismissed', true)
  if(hide) await setUiPref('welcomeHide', true)
  closeWelcome()
}
async function openGuideFromWelcome(){
  const hide=document.getElementById('welcomeHideToggle')?.checked
  await setUiPref('welcomeDismissed', true)
  if(hide) await setUiPref('welcomeHide', true)
  closeWelcome()
  switchTab('R')
}
```

- [ ] **Step 3: Extend tab switching for the guide tab**

Update tab activation/deactivation and date-nav visibility so `R` behaves like a main content tab with no date navigation.

```javascript
document.getElementById('tbR').classList.remove('active')
const navR = false
document.getElementById('tabDateNav').style.display=t==='D'?'flex':'none'
if(document.getElementById('tabDateNavA')) document.getElementById('tabDateNavA').style.display=t==='A'?'flex':'none'
['D','W','M','Y','T','A','R'].forEach(x=>{
  const pane=document.getElementById('tab'+x)
  if(pane) pane.style.display=x===t?'':'none'
})
```

- [ ] **Step 4: Show the welcome modal from boot flow**

After `loadSystemInfo()`, `ensureDB()`, and first `refresh()`, decide whether to show onboarding.

```javascript
async function maybeShowWelcome(){
  const hide=await getUiPref('welcomeHide', false)
  const dismissed=await getUiPref('welcomeDismissed', false)
  if(hide) return
  const hasData=Array.isArray(allDaysSummary)&&allDaysSummary.length>0
  if(!dismissed || !hasData) openWelcome()
}
```

Call it from `bootApp()` after refresh settles.

- [ ] **Step 5: Verify script parsing again**

Run:

```powershell
node -e "const fs=require('fs'),vm=require('vm');const code=fs.readFileSync('app.html','utf8');const scripts=[...code.matchAll(/<script(?!(?:[^>]*src=))[^>]*>([\s\S]*?)<\/script>/gi)].map(m=>m[1]);for(const [i,s] of scripts.entries()){new vm.Script(s,{filename:'inline-script-'+(i+1)+'.js'});}console.log('SCRIPT_OK scripts='+scripts.length+' lines='+code.split(/\r?\n/).length);"
```

Expected: `SCRIPT_OK ...`

- [ ] **Step 6: Commit the behavior layer**

```bash
git add app.html
git commit -m "Wire onboarding state and guide navigation"
```

### Task 3: Fill guide content, polish welcome copy, and document it

**Files:**
- Modify: `D:\TMA Drive\OneDrive - Toyota Asia\Temp\AI Project\Google\Test\app.html`
- Modify: `D:\TMA Drive\OneDrive - Toyota Asia\Temp\AI Project\Google\Test\README.md`

- [ ] **Step 1: Fill the in-app guide with concise sections**

Add compact cards and bullets for:

```html
<div class="guide-grid">
  <article class="guide-card">
    <h3>เริ่มใช้งาน</h3>
    <ul>
      <li>กด ระบบ แล้วตรวจวันเริ่ม Solar / TOU / ค่าไฟ</li>
      <li>กด inverter data เพื่อนำเข้าไฟล์ XLSX</li>
      <li>มีบิลจริงเมื่อไร ค่อยกรอกที่ ประวัติบิล</li>
    </ul>
  </article>
</div>
```

Also include cards for supported files, browser-local data, key limitations, and short glossary.

- [ ] **Step 2: Fill the welcome modal body**

Use short Thai copy with one short list and one storage/desktop caution block.

```html
<p class="welcome-note">รองรับไฟล์ export จาก Deye เป็นหลัก และแนะนำให้ใช้บน desktop/laptop</p>
<ul class="welcome-list">
  <li>กด <b>ระบบ</b> เพื่อตั้งค่าวันเริ่มใช้งานและค่าไฟ</li>
  <li>กด <b>inverter data</b> เพื่อ import ไฟล์ XLSX</li>
  <li>ถ้ามีบิลจริง ให้กรอกเพิ่มที่ <b>ประวัติบิล</b></li>
</ul>
```

- [ ] **Step 3: Add a short README note**

Update `README.md` with one short note in the quick-start/help area that the live Pages app includes:

```markdown
- In the live GitHub Pages app, a first-run welcome dialog and in-app `คู่มือ` tab help new users start without opening the repo README first.
```

- [ ] **Step 4: Run final verification commands**

Run:

```powershell
node -e "const fs=require('fs'),vm=require('vm');const code=fs.readFileSync('app.html','utf8');const scripts=[...code.matchAll(/<script(?!(?:[^>]*src=))[^>]*>([\s\S]*?)<\/script>/gi)].map(m=>m[1]);for(const [i,s] of scripts.entries()){new vm.Script(s,{filename:'inline-script-'+(i+1)+'.js'});}console.log('SCRIPT_OK scripts='+scripts.length+' lines='+code.split(/\r?\n/).length);"
git diff --check
```

Expected:
- `SCRIPT_OK ...`
- no whitespace/diff errors; CRLF warnings are acceptable on this Windows checkout

- [ ] **Step 5: Manual browser checks**

Verify in the live app or local file:

```text
1. Welcome modal appears on first load
2. เริ่มใช้งาน closes modal
3. อ่านคู่มือ opens main guide tab
4. ไม่ต้องแสดงอีกบนเครื่องนี้ survives refresh
5. คู่มือ tab hides date navigation
6. CTA buttons open ระบบ / file picker / ประวัติบิล
```

- [ ] **Step 6: Commit the finished feature**

```bash
git add app.html README.md
git commit -m "Add in-app onboarding and guide tab"
```
