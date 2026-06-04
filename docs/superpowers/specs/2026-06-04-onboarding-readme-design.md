# Onboarding And README In-App Design

## Goal

Improve first-time use of GitHub Pages version of Deye Solar Insight so a new user understands:

- this tool is mainly for Deye XLSX exports
- desktop/laptop is recommended
- data stays in the current browser
- import is required before charts become useful

Also provide a persistent in-app help surface after the first visit.

## Scope

Add two UI elements:

1. A first-run welcome modal
2. A new top-level `คู่มือ` tab beside the existing main tabs

No changes to calculations, IndexedDB schema for day data, or analysis logic.

## User Experience

### First-run welcome modal

Show on first visit to the Pages app, and also when no imported data exists unless the user has chosen to hide it.

Content should be short and task-focused:

- `Deye Solar Insight (Public Beta)`
- tested mainly with Deye exports
- desktop/laptop recommended
- data is stored in this browser only
- clearing browser storage may remove saved data
- quick start:
  1. open `ระบบ`
  2. import `inverter data`
  3. add `ประวัติบิล` when real bills are available

Actions:

- `เริ่มใช้งาน`
- `อ่านคู่มือ`
- checkbox or toggle: `ไม่ต้องแสดงอีกบนเครื่องนี้`

Behavior:

- `เริ่มใช้งาน` closes the modal
- `อ่านคู่มือ` closes the modal and switches to the new `คู่มือ` tab
- hide preference is stored in browser settings/local storage

### Main `คู่มือ` tab

Add a new main tab at the same level as `รายวัน` and `วิเคราะห์`.

Purpose:

- persistent in-app guide for new users
- quick place to re-read setup notes without opening GitHub README

Sections:

1. เริ่มใช้งาน
2. รองรับไฟล์แบบไหน
3. ข้อมูลเก็บที่ไหน
4. ข้อจำกัดสำคัญ
5. คำศัพท์สั้น ๆ เช่น TOU, K-Factor, Simulation

This is not a full dump of `README.md`; it should be a concise in-app guide optimized for scanning.

Include clear CTA buttons near the top:

- `เปิดระบบ`
- `เลือกไฟล์ inverter data`
- `เปิดประวัติบิล`

## Data And State

Add lightweight browser-only preference keys for onboarding UI, for example:

- `welcomeDismissed`
- `welcomeHide`

These should be separate from imported day data and must not require schema migration.

## Rendering And Navigation

- Extend main tab switching to support the new `คู่มือ` tab
- Add a content pane for the guide
- When the guide tab is active, show no date navigation
- `อ่านคู่มือ` from the modal should call the same tab switch path as a normal tab click

## Visual Design

- Match current dark theme and card language
- Keep the modal compact, readable, and non-technical
- Make the guide tab look like a product help page inside the app, not a raw markdown dump
- Avoid large walls of text; prefer grouped cards, short bullets, and action buttons

## Error Handling

- If settings persistence fails, the modal may reappear on next visit; this is acceptable
- If imported data already exists, the guide still remains accessible from the `คู่มือ` tab

## Testing

Manual checks:

1. New browser profile: modal appears on first load
2. Click `เริ่มใช้งาน`: modal closes
3. Click `อ่านคู่มือ`: switches to `คู่มือ`
4. Enable `ไม่ต้องแสดงอีกบนเครื่องนี้`: modal does not reappear after refresh
5. `คู่มือ` tab renders with no overlap and no date-nav artifacts
6. CTA buttons open the expected modal/file picker
7. Existing tabs still render normally

Technical checks:

- inline script syntax check for `app.html`
- `git diff --check`

## Out Of Scope

- full markdown renderer for README
- mobile redesign
- any calculation or chart formula changes
- server/backend changes
