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
  const ui = SpreadsheetApp.getUi();
  ui.createMenu('Update prfile')
      .addItem('Update Stats Now', 'updateLeetCodeStats')
      .addToUi();
}

function updateLeetCodeStats() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  
  // Get all data from the sheet
  const dataRange = sheet.getDataRange();
  const data = dataRange.getValues();
  
  // Loop through every row
  for (let i = 0; i < data.length; i++) {
    const row = data[i];
    
    // Column E is index 4   update accordingly    (the LeetCode URL)
    const url = row[4]; 
    
    if (url && typeof url === 'string' && url.includes('leetcode.com')) {
      const username = extractUsername(url);
      
      if (username) {
        const count = getLeetCodeSolved(username);
        
        if (count !== null) {
          // Update Column B (which is column number 2 in Sheets)
          sheet.getRange(i + 1, 2).setValue(count);
          
          // Wait so LeetCode doesn't block the script
          Utilities.sleep(500); 
        }
      }
    }
  }
  
  SpreadsheetApp.getUi().alert("Finished updating all LeetCode stats!");
}

function extractUsername(url) {
  let cleaned = url.trim();
  if (cleaned.endsWith('/')) cleaned = cleaned.slice(0, -1);
  const parts = cleaned.split('/');
  
  const uIndex = parts.indexOf('u');
  if (uIndex !== -1 && uIndex + 1 < parts.length) {
    return parts[uIndex + 1];
  }
  
  const last = parts[parts.length - 1];
  if (last && last !== 'leetcode.com') {
    return last;
  }
  return null;
}

function getLeetCodeSolved(username) {
  const url = "https://leetcode.com/graphql";
  
  const payload = {
    "query": `
      query getUserProfile($username: String!) {
        matchedUser(username: $username) {
          submitStats {
            acSubmissionNum {
              difficulty
              count
            }
          }
        }
      }
    `,
    "variables": { "username": username }
  };
  
  const options = {
    "method": "post",
    "contentType": "application/json",
    "payload": JSON.stringify(payload),
    "muteHttpExceptions": true
  };
  
  try {
    const response = UrlFetchApp.fetch(url, options);
    const json = JSON.parse(response.getContentText());
    
    const stats = json?.data?.matchedUser?.submitStats?.acSubmissionNum;
    if (stats) {
      for (let i = 0; i < stats.length; i++) {
        if (stats[i].difficulty === 'All') {
          return stats[i].count;
        }
      }
    }
  } catch (e) {
    // Skip if there's an error
  }
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
