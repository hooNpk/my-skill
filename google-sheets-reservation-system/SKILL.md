---
name: google-sheets-reservation-system
description: |
  Build a complete reservation system backed by Google Sheets — reservation form, reservation board, and schedule calendar. Use this skill when the user wants to create a booking/reservation system, appointment scheduler, or any form-to-spreadsheet workflow with a public-facing board and calendar view. Also use when the user mentions Google Sheets integration for form submissions, Apps Script web app proxy, or wants to display spreadsheet data on a website. Covers the full stack: HTML form → Apps Script web app → Google Sheets storage → public read via Sheets API.
---

# Google Sheets Reservation System

A complete reservation system with three components:
1. **Reservation Form** — submit reservations via Apps Script web app proxy
2. **Reservation Board** — public listing with pagination, masking, detail modal
3. **Schedule Calendar** — shows confirmed reservations on a monthly calendar

## Architecture Overview

```
[Website Form] --POST--> [Apps Script Web App] --insertRow--> [Google Sheets]
                                                                    |
[Website Board] <--GET (Sheets API)-- [Google Sheets] <--read-------+
[Website Calendar] <--GET (Sheets API)-- [Google Sheets] (filter: 예약완료)
```

Google Forms HTTP POST submission is unreliable (returns 400 frequently). Instead, use an **Apps Script web app as a proxy** that receives form data and writes directly to the spreadsheet.

## 1. Reservation Form

### HTML Structure

The form needs these field groups:

```html
<form id="reservationForm">
  <!-- Basic info: name, phone -->
  <!-- Date/Time: date picker + start/end time dropdowns (30-min intervals, 09:00~20:00) -->
  <!-- Selection fields: room, people count, purpose -->
  <!-- Additional: depositor name, vehicle count, equipment, notes -->
  <!-- Privacy: secret post toggle + password -->
  <!-- Required: privacy consent checkbox + CAPTCHA -->
  <button type="submit">예약 신청</button>
</form>
```

### Key Form Features

**Time Selection** — Two separate dropdowns for start and end time:
```html
<select name="time_start">
  <option>09:00</option><option>09:30</option>...<option>20:00</option>
</select>
<select name="time_end"><!-- same options --></select>
```
Validate: `if (time_start >= time_end) { alert('종료 시간은 시작 시간보다 늦어야 합니다.'); return; }`

**CAPTCHA** — Simple math-based auto-registration prevention:
```html
<div class="captcha-box">
  <span id="captchaQuestion">3 + 7 = ?</span>
  <input name="captcha" type="number" required>
</div>
```
Generate random numbers on page load; validate answer before submission.

**Privacy Consent** — Checkbox that must be checked before submission:
```html
<label><input type="checkbox" name="privacy_agree" required> 개인정보 수집 및 이용에 동의합니다</label>
```

**Secret Post** — Toggle for private posting with password:
```html
<label><input type="checkbox" id="secretToggle"> 비밀글</label>
<input type="password" name="password" placeholder="비밀번호 (4자리)">
```

### Fire-and-Forget Async Submission

Submit without waiting for the Apps Script response. The web app redirects (302), which is slow — don't wait for it:

```javascript
// Fire-and-forget: don't await the response
const params = new URLSearchParams({ type: 'reservation', name, phone, date, time, ... });
fetch(CONFIG.APPS_SCRIPT_URL, {
  method: 'POST',
  mode: 'no-cors',
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
  body: params.toString(),
});
// Immediately show success and update UI
alert('예약 신청이 완료되었습니다.');
allData.unshift({ /* new item */ });
renderList();
```

This makes submission feel instant (~0ms) instead of waiting 3-5 seconds for the Apps Script round-trip.

## 2. Apps Script Web App (Server Side)

See `references/apps-script.md` for the complete Apps Script code.

### Key Design Decisions

**Why not Google Forms?** Google Forms HTTP POST submission (`/formResponse`) has become unreliable — it frequently returns 400 errors. The Apps Script web app approach is more reliable because it uses the SpreadsheetApp API directly.

**insertRow vs appendRow:**
- `insertRowAfter(1)` + `setValues()` — new data at row 2 (below header). O(n) but keeps newest on top in the spreadsheet.
- `appendRow()` — new data at bottom. O(1) but newest is at bottom.

Choose `insertRowAfter(1)` when the admin frequently views the spreadsheet directly (newest on top is convenient). The website always controls its own sort order regardless.

### Web App Deployment

1. Open Google Apps Script editor (Extensions → Apps Script)
2. Paste the code from `references/apps-script.md`
3. Deploy → New deployment → Type: Web app
4. Execute as: Me / Access: Anyone (including anonymous)
5. Copy the web app URL to `config.js`

### Spreadsheet Headers

**Reservation sheet** (15 columns):
```
타임스탬프 | 이름 | 연락처 | 희망 날짜 | 시간대 | 룸선택 | 인원 | 촬영목적 | 입금자명 | 방문차량 | 장비렌탈 | 특이사항 | 비밀번호 | 공개여부 | 상태
```

The `상태` column gets a dropdown validation: `확인중` (red) / `예약완료` (green).

## 3. Reservation Board

### Reading Data from Sheets

Use Google Sheets API v4 with an API key (spreadsheet must be shared as "Anyone with the link"):

```javascript
const url = `https://sheets.googleapis.com/v4/spreadsheets/${sheetId}/values/${tab}!A:O?key=${apiKey}`;
const response = await fetch(url);
const data = await response.json();
```

Try multiple tab names since Google creates tabs with different names depending on locale:
```javascript
const tabCandidates = ['설문지 응답 시트1', '설문지 응답 시트2', '폼 응답 1', 'Form Responses 1', 'Sheet1'];
```

### Name Masking

Protect privacy by masking the middle characters of names:
```javascript
function maskName(name) {
  if (name.length <= 2) return name[0] + '*';
  return name[0] + '*'.repeat(name.length - 2) + name[name.length - 1];
}
// '박정훈' → '박*훈', '메가코드' → '메**드'
```

### Pagination

```javascript
const PAGE_SIZE = 20;
let currentPage = 1;

function renderPage(page) {
  const start = (page - 1) * PAGE_SIZE;
  const pageData = allData.slice(start, start + PAGE_SIZE);
  // render rows...
  // render pagination controls (prev / page numbers / next)
}
```

### Detail Modal

Click a row to open a modal showing full reservation details:
```javascript
function viewPost(index) {
  const post = allData[index];
  // If secret post, show password prompt first
  if (post['공개여부'] === '비밀글') {
    showPasswordModal(index);
    return;
  }
  showDetailModal(post);
}
```

## 4. Schedule Calendar

The calendar reads from the **same reservation spreadsheet** but filters only confirmed bookings.

### Data Loading

```javascript
const all = SheetsAPI.toObjects(rows);
scheduleData = all
  .filter(r => r['상태'] === '예약완료')  // Only confirmed
  .map(r => ({
    '날짜': r['희망 날짜'],
    '시간상세': r['시간대'],
    '룸': parseRoom(r['룸선택']),
    '예약자': maskName(r['이름']),
  }));
```

### Desktop View — Monthly Grid

Standard calendar grid with events shown in each cell:
```html
<table class="calendar">
  <thead><tr><th>일</th><th>월</th>...<th>토</th></tr></thead>
  <tbody id="calendarBody"></tbody>
</table>
```

Each day cell shows color-coded room badges:
- Room A: pink, Room B: blue, Room C: green
- A+B: purple, A+C: orange, Full: gold

### Mobile View — List

Shows every day of the month in a vertical list (including days with no reservations):
```html
<div class="calendar__mobile-item">
  <div class="calendar__mobile-date">22(토)</div>
  <div><!-- event badges or empty --></div>
</div>
```

## Config Structure

```javascript
const CONFIG = {
  GOOGLE_SHEETS_API_KEY: "AIza...",
  RESERVATION_SHEET_ID: "spreadsheet-id",
  APPS_SCRIPT_URL: "https://script.google.com/macros/s/.../exec",
  RESERVATION_SHEET_NAME: "시트1",
};
```
