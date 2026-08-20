# Home Vitals Tracker — Setup Guide

A free, private daily BP & SpO₂ logging system for two people (labeled **Father** and **Mother**), syncing straight into your own Google Sheet — no subscriptions, no third-party health app, no paid backend.

---

## What you get

- **`vitals-tracker.html`** — a mobile-friendly web app. Pick Father or Mother, pick Morning/Afternoon/Night, enter 3 BP readings (systolic/diastolic/pulse) and 3 oximeter readings (SpO₂/pulse). It averages them live and saves the entry.
- **A Google Apps Script** — a small free backend that lives inside your own Google Sheet and writes each entry into a **Father** tab or **Mother** tab automatically.
- Entries always save locally on your device first (`localStorage`), so nothing is lost if you're offline — they sync to the Sheet automatically, and you can retry any entry manually from the history list.

---

## 1. Create the Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new blank spreadsheet.
2. Name it something like **"Parents Vitals Tracker"**.
3. Don't create any tabs manually — the script creates **Father** and **Mother** automatically the first time each is used.

---

## 2. Add the sync script

1. With the Sheet open, go to **Extensions → Apps Script**. (Opening it this way is important — it binds the script to *this* spreadsheet specifically.)
2. Delete any starter code and paste in the script below.
3. Save (disk icon or `Ctrl`/`Cmd` + `S`).

```javascript
function doPost(e) {
  var data = JSON.parse(e.postData.contents);
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheetName = data.person === 'father' ? 'Father' : 'Mother';
  var sheet = ss.getSheetByName(sheetName);

  if (!sheet) {
    sheet = ss.insertSheet(sheetName);
    sheet.appendRow([
      'Date', 'Time', 'Person', 'Time of Day',
      'Systolic Avg', 'Diastolic Avg', 'BP Pulse Avg',
      'SpO2 Avg (%)', 'Oximeter Pulse Avg',
      'BP Raw Readings (Sys/Dia/Pulse)', 'SpO2 Raw Readings (SpO2/Pulse)'
    ]);
    sheet.setFrozenRows(1);
  }

  sheet.appendRow([
    data.date, data.time, data.person, data.period,
    data.sysAvg, data.diaAvg, data.bpPulseAvg,
    data.spo2Avg, data.oxPulseAvg,
    data.bpRaw, data.spo2Raw
  ]);

  return ContentService.createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

## 3. Deploy it as a Web App

1. Click **Deploy → New deployment**.
2. Click the gear icon → choose **Web app**.
3. Set **Execute as: Me**, and **Who has access: Anyone**.
4. Click **Deploy**.
5. You'll hit a "Google hasn't verified this app" screen — this is normal for your own private script. Click **Advanced**, then click the project link that appears (e.g. *"Go to Vitals Sync (unsafe)"*), then **Allow**.
6. Copy the URL Google gives you, ending in `/exec`. **Save it somewhere** — you'll paste it into the app next.

> **Verify it worked:** paste that `/exec` URL into a new browser tab. Seeing `Script function not found: doGet` is expected and means the deployment is live. A 404 or access-denied page means something's wrong with the deployment settings — recheck step 3.

---

## 4. Host the app so you can open it on your phone

The simplest free option: **GitHub Pages**.

1. Push `vitals-tracker.html` to any public GitHub repo.
2. In that repo, go to **Settings → Pages**, and enable Pages (source = your main branch).
3. GitHub gives you a URL like:
   `https://<your-username>.github.io/<repo-name>/vitals-tracker.html`
4. Open that URL on your phone's browser, then use the browser menu → **Add to Home Screen** for a full-screen, app-like icon.

*(You can also just open the HTML file locally on desktop/VS Code for testing — everything works the same, it just won't be reachable from your phone unless it's hosted somewhere.)*

---

## 5. Connect the app to your Sheet

1. Open the app (hosted or local).
2. Scroll down and tap **⚙ Sheet sync & export**.
3. Paste your `/exec` URL from step 3 into the field.
4. Tap **Save URL**, then **Test connection**.
5. Check your Google Sheet — a test row should appear in the **Father** or **Mother** tab.

---

## Daily use

1. Open the app.
2. Tap **Father** or **Mother**.
3. Confirm or change the time period (Morning/Afternoon/Night — auto-suggested).
4. Enter all 3 BP readings and all 3 oximeter readings.
5. Tap **Save entry** — it saves locally and syncs to your Sheet immediately.
6. If you're offline, it still saves locally; open **history** later and tap **⟳ Retry sync**, or use **Sync all pending** in settings once you're back online.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `Cannot read properties of undefined (reading 'postData')` | You clicked **Run** on `doPost` inside the Apps Script editor | Ignore it — `doPost` only works via real requests to the `/exec` URL, never via Run. Test through the app instead. |
| App says "Synced" but the Sheet shows nothing | The sync request uses `no-cors` mode, so the app can't actually confirm success | Check **Apps Script → Executions** — that's the real source of truth, not the app's status message. |
| Executions log shows failures from source **"Editor"** | Those are manual Run-button tests, not real traffic | Ignore them; look for entries with source **"Web app"** instead. |
| No new executions appear at all when you tap Save/Test | No live deployment exists yet, or the URL in the app doesn't match your actual deployment | Open your `/exec` URL directly in a browser to confirm it's live, then re-paste the confirmed URL into the app's settings. |
| Data doesn't persist between app sessions | Running via an environment where `localStorage` is blocked (rare, e.g. some in-app browsers) | Open the page in a standard mobile browser (Chrome/Safari) rather than an embedded webview. |

---

## Notes

- This is a personal record-keeping tool, not a certified medical device — it doesn't replace professional medical advice or emergency care.
- All data lives in **your own** Google Sheet and your device's local storage — nothing is sent to any third-party server other than Google's own Apps Script infrastructure that you control.
