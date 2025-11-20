# Google Sheets Backend Setup Guide

This guide will help you set up Google Sheets as a database to track student quiz submissions.

## Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Create a new blank spreadsheet
3. Name it "T543 Quiz Submissions"
4. In the first row, add these headers:
   - A1: `Timestamp`
   - B1: `Student Name`
   - C1: `Class`
   - D1: `Score`
   - E1: `Total Questions`
   - F1: `Time Spent (seconds)`
   - G1: `Q1 Correct`
   - H1: `Q2 Correct`
   - I1: `Q3 Correct`
   - J1: `Wrong Questions`

## Step 2: Create Google Apps Script

1. In your Google Sheet, click **Extensions** → **Apps Script**
2. Delete any existing code
3. Copy and paste the following code:

```javascript
function doPost(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    const data = JSON.parse(e.postData.contents);

    // Extract wrong questions
    const wrongQuestions = [];
    for (let q in data.answers) {
      if (!data.answers[q].isCorrect) {
        wrongQuestions.push(q.toUpperCase());
      }
    }

    // Append row to sheet
    sheet.appendRow([
      data.timestamp,
      data.studentName,
      data.studentClass || '',
      data.score,
      data.totalQuestions,
      data.timeSpent,
      data.answers.q1 ? data.answers.q1.isCorrect : false,
      data.answers.q2 ? data.answers.q2.isCorrect : false,
      data.answers.q3 ? data.answers.q3.isCorrect : false,
      wrongQuestions.join(', ')
    ]);

    return ContentService.createTextOutput(JSON.stringify({ success: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ success: false, error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    const data = sheet.getDataRange().getValues();
    const headers = data[0];
    const rows = data.slice(1);

    const submissions = rows.map(row => {
      const answers = {
        q1: { isCorrect: row[6] },
        q2: { isCorrect: row[7] },
        q3: { isCorrect: row[8] }
      };

      return {
        timestamp: row[0],
        studentName: row[1],
        studentClass: row[2],
        score: row[3],
        totalQuestions: row[4],
        timeSpent: row[5],
        answers: answers
      };
    });

    return ContentService.createTextOutput(JSON.stringify(submissions))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

4. Click **Save** (disk icon)
5. Name your project "T543 Quiz Backend"

## Step 3: Deploy the Script

1. Click **Deploy** → **New deployment**
2. Click the gear icon ⚙️ next to "Select type"
3. Choose **Web app**
4. Configure:
   - Description: "T543 Quiz Submission Handler"
   - Execute as: **Me**
   - Who has access: **Anyone** (important!)
5. Click **Deploy**
6. Click **Authorize access**
7. Choose your Google account
8. Click **Advanced** → **Go to T543 Quiz Backend (unsafe)**
9. Click **Allow**
10. **COPY THE WEB APP URL** - you'll need this!

The URL will look like:
```
https://script.google.com/macros/s/AKfycby.../exec
```

## Step 4: Update Your Quiz Files

### Update nucleotide-structure.html

1. Open `quizzes/nucleotide-structure.html`
2. Find this line near the top of the `<script>` section:
   ```javascript
   const SUBMIT_URL = ''; // Leave empty for now, we'll add this later
   ```
3. Replace it with your Web App URL:
   ```javascript
   const SUBMIT_URL = 'https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec';
   ```

### Update teacher-dashboard.html

1. Open `teacher-dashboard.html`
2. Find this line:
   ```javascript
   const SHEET_URL = ''; // We'll add this after setting up Google Sheets
   ```
3. Replace it with the SAME Web App URL:
   ```javascript
   const SHEET_URL = 'https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec';
   ```

## Step 5: Test the Integration

1. Go to your quiz (student role)
2. Fill in your name and class
3. Complete the quiz
4. Check your Google Sheet - a new row should appear!
5. Go to the teacher dashboard (teacher role, code: t543)
6. You should see the submission

## Troubleshooting

### Submissions not appearing?
- Make sure you copied the EXACT Web App URL (including `/exec` at the end)
- Check that the script has "Anyone" access
- Look at the Apps Script execution logs: **Executions** tab in Apps Script editor

### "Authorization required" error?
- Redeploy the script and re-authorize

### Need to make changes?
After editing the Apps Script code:
1. Click **Deploy** → **Manage deployments**
2. Click the pencil icon ✏️ to edit
3. Change version to "New version"
4. Click **Deploy**
5. The URL stays the same - no need to update your HTML files!

## Privacy & Security Notes

- The Web App URL should be kept somewhat private (don't share publicly)
- Consider adding password protection if needed
- All submissions are stored in your Google Sheet
- Only you (the sheet owner) can access the data
- Students cannot see other students' submissions
