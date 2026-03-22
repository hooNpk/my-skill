# Apps Script Web App — Complete Code Reference

## doPost Handler

The web app receives POST requests from the website and writes directly to Google Sheets.

```javascript
function doPost(e) {
  try {
    var p = e.parameter;
    var type = p.type || 'reservation';
    var now = new Date();
    var timestamp = Utilities.formatDate(now, Session.getScriptTimeZone(), 'yyyy-MM-dd HH:mm:ss');

    var ss = SpreadsheetApp.openById(SHEET_ID);
    var sheet = ss.getSheetByName('시트1') || ss.getSheets()[0];

    // Ensure headers exist
    if (sheet.getLastRow() === 0) {
      sheet.appendRow([
        '타임스탬프','이름','연락처','희망 날짜','시간대','룸선택',
        '인원','촬영목적','입금자명','방문차량','장비렌탈',
        '특이사항','비밀번호','공개여부','상태'
      ]);
    }

    // Insert at row 2 (below header) — newest on top
    sheet.insertRowAfter(1);
    sheet.getRange(2, 1, 1, 15).setValues([[
      timestamp,
      p.name      || '',
      p.phone     || '',
      p.date      || '',
      p.time      || '',
      p.room      || '',
      p.people    || '',
      p.purpose   || '',
      p.depositor || '',
      p.vehicle   || '',
      p.equipment || '',
      p.note      || '',
      p.password  || '',
      p.secret    || '',
      '확인중',       // Default status
    ]]);

    // Add dropdown validation for status column
    var statusRule = SpreadsheetApp.newDataValidation()
      .requireValueInList(['확인중', '예약완료'], true)
      .setAllowInvalid(false)
      .build();
    var statusCell = sheet.getRange(2, 15);
    statusCell.setDataValidation(statusRule);
    statusCell.setFontColor('#cc0000'); // Red for '확인중'

    return ContentService
      .createTextOutput(JSON.stringify({ success: true }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ success: false, error: err.message }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  return ContentService
    .createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

## Deployment Steps

1. Open any Google Spreadsheet → Extensions → Apps Script
2. Paste the code above (replace any existing code)
3. Update `SHEET_ID` with your spreadsheet ID
4. Save (Ctrl+S)
5. Deploy → New deployment
   - Type: **Web app**
   - Execute as: **Me**
   - Access: **Anyone** (including anonymous)
6. Click Deploy → Copy the web app URL
7. Authorize when prompted (first time only)

## Updating After Code Changes

After modifying the Apps Script code:
1. Deploy → Manage deployments
2. Click the pencil/edit icon on the active deployment
3. Version: **New version**
4. Click Deploy

The URL stays the same — no need to update config.js.

## Why Not Google Forms?

Google Forms' `/formResponse` endpoint has become unreliable:
- Returns 400 errors unpredictably
- Required fields with dropdown options cause strict validation failures
- `fbzx` session tokens may be required but change per page load
- `no-cors` mode prevents reading the actual error response

The Apps Script web app approach avoids all of these issues by writing directly to the spreadsheet via the SpreadsheetApp API.

## Status Column Color Coding

To automatically color the status cell when an admin changes it from '확인중' to '예약완료', add an `onEdit` trigger:

```javascript
function onEdit(e) {
  var range = e.range;
  var sheet = range.getSheet();
  if (range.getColumn() === 15) { // Status column (O)
    var value = range.getValue();
    if (value === '예약완료') {
      range.setFontColor('#008000'); // Green
    } else if (value === '확인중') {
      range.setFontColor('#cc0000'); // Red
    }
  }
}
```
