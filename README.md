# 📊 LeetCode Google Sheets Updater

Automatically update the number of LeetCode questions solved by students — directly inside your Google Sheet. No coding experience needed!

## 📌 What Does This Do?

You have a Google Sheet with student names, their LeetCode profile links, and a column for "questions solved". Instead of checking each profile one by one, this script does it **all for you in one click**.

## 🛠️ How to Set It Up

1. Open your Google Sheet.
2. Click **Extensions** > **Apps Script** (in the top menu bar).
3. You'll see some default code like `function myFunction() {}` — **delete all of it**.
4. Copy the code below and **paste it** into the editor:

```javascript
// This function adds a custom button to your Google Sheet menu
function onOpen() {
  SpreadsheetApp.getUi()
    .createMenu('LeetCode Tool')
    .addItem('Update Stats Now', 'updateLeetCodeStats')
    .addToUi();
}

function updateLeetCodeStats() {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
  var data = sheet.getDataRange().getValues();
  var updated = 0;
  var failed = [];

  for (var i = 0; i < data.length; i++) {
    var url = String(data[i][3] || '');  // Column D

    if (url.indexOf('leetcode.com') !== -1) {
      var username = extractUsername(url);

      if (username) {
        var count = getLeetCodeSolved(username);

        if (count !== null) {
          sheet.getRange(i + 1, 2).setValue(count);  // Write to Column B
          updated++;
        } else {
          failed.push('Row ' + (i + 1) + ': ' + data[i][0]);
        }

        Utilities.sleep(1000);
      }
    }
  }

  var msg = 'Updated ' + updated + ' students!';
  if (failed.length > 0) {
    msg += '\n\nFailed ' + failed.length + ':\n' + failed.join('\n');
  }
  SpreadsheetApp.getUi().alert(msg);
}

function extractUsername(url) {
  var cleaned = url.trim();
  if (cleaned.charAt(cleaned.length - 1) === '/') cleaned = cleaned.slice(0, -1);
  var parts = cleaned.split('/');

  var uIndex = parts.indexOf('u');
  if (uIndex !== -1 && uIndex + 1 < parts.length) {
    return parts[uIndex + 1];
  }

  var last = parts[parts.length - 1];
  if (last && last !== 'leetcode.com') return last;
  return null;
}

function getLeetCodeSolved(username) {
  var url = 'https://leetcode.com/graphql';

  var payload = {
    query: 'query getUserProfile($username: String!) { matchedUser(username: $username) { submitStats { acSubmissionNum { difficulty count } } } }',
    variables: { username: username }
  };

  var options = {
    method: 'post',
    contentType: 'application/json',
    payload: JSON.stringify(payload),
    muteHttpExceptions: true
  };

  try {
    var response = UrlFetchApp.fetch(url, options);
    var json = JSON.parse(response.getContentText());
    var stats = json && json.data && json.data.matchedUser && json.data.matchedUser.submitStats && json.data.matchedUser.submitStats.acSubmissionNum;

    if (stats) {
      for (var i = 0; i < stats.length; i++) {
        if (stats[i].difficulty === 'All') return stats[i].count;
      }
    }
  } catch (e) {}
  return null;
}
```
5. Click the **Save** button 💾.
6. Go back to your Google Sheet and **refresh the page**.
7. After a few seconds, you'll see a new menu called **"Update profile"** at the top. That's it — you're done!

## ▶️ How to Use It

1. Click **Update profile** > **Update Stats Now**.
2. Wait for it to finish (it takes about 1 second per student).
3. A popup will say **"Finished updating all LeetCode stats!"** when it's done.
4. Column B will now have the updated solved counts! ✅

## ⚠️ First Time Only — Google Will Ask for Permission

The first time you run it, Google will show a permission popup. This is normal — just follow these steps:

1. Click **Continue**.
2. Pick your Google account.
3. Click **Advanced** (small text at the bottom).
4. Click **Go to Untitled project (unsafe)**.
5. Click **Allow**.

You only have to do this once. After that, it just works every time!

## 📋 Sheet Format

Your Google Sheet should have:

| Column A | Column B | Column C | Column D | Column E |
|----------|----------|----------|----------|----------|
| Student Name | Solved Count (auto-filled) | ... | ... | LeetCode Profile Link |

> **Note:** If your LeetCode links or solved count are in different columns, you can change that in the code. Just open Apps Script and look for the comments that say which column to change.
