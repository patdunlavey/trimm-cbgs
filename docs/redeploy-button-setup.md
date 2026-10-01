# Google Sheet "Redeploy Website" Menu

A custom menu appears in the Google Sheet menu bar for authorized users only. Clicking it commits a timestamp to `.deploy-trigger` in the GitHub repo, which triggers GitHub Pages to rebuild the site with the latest Google Sheet data.

---

## Step 1: Add the Script

In the Google Sheet: **Extensions → Apps Script**

Paste the following and save (Ctrl+S):

```javascript
function onOpen() {
  var email = Session.getActiveUser().getEmail();
  var authorized = ['patdunlavey@gmail.com', 'gourdbox21ljt@gmail.com'];
  if (authorized.indexOf(email) !== -1) {
    SpreadsheetApp.getUi()
      .createMenu('Wayne Trimm Archive')
      .addItem('↺ Redeploy Website', 'redeployToGitHub')
      .addToUi();
  }
}

function redeployToGitHub() {
  var props = PropertiesService.getScriptProperties();
  var token = props.getProperty('GITHUB_TOKEN');

  if (!token) {
    SpreadsheetApp.getUi().alert('No GitHub token found. Add GITHUB_TOKEN to Script Properties.');
    return;
  }

  var apiUrl = 'https://api.github.com/repos/patdunlavey/trimm-cbgs/contents/.deploy-trigger';

  // Get current file SHA (required by GitHub API to update a file)
  var get = UrlFetchApp.fetch(apiUrl, {
    headers: { 'Authorization': 'token ' + token, 'Accept': 'application/vnd.github.v3+json' }
  });
  var sha = JSON.parse(get.getContentText()).sha;

  // Write new timestamp
  var timestamp = Utilities.formatDate(new Date(), Session.getScriptTimeZone(), 'yyyy-MM-dd HH:mm:ss');
  var put = UrlFetchApp.fetch(apiUrl, {
    method: 'PUT',
    headers: {
      'Authorization': 'token ' + token,
      'Accept': 'application/vnd.github.v3+json',
      'Content-Type': 'application/json'
    },
    payload: JSON.stringify({
      message: 'Trigger redeploy - ' + timestamp,
      content: Utilities.base64Encode('Redeployed: ' + timestamp + '\n'),
      sha: sha
    })
  });

  var code = put.getResponseCode();
  if (code === 200 || code === 201) {
    SpreadsheetApp.getUi().alert('Redeploy triggered! The site will rebuild in a minute or two.');
  } else {
    SpreadsheetApp.getUi().alert('Something went wrong: ' + put.getContentText());
  }
}
```

Replace the two email addresses in `onOpen()` with the actual authorized emails.

---

## Step 2: Store the GitHub Token

Still in Apps Script: **Project Settings (gear icon) → Script Properties → Add property**

| Property | Value |
|----------|-------|
| `GITHUB_TOKEN` | your GitHub Personal Access Token |

**To create the token:** GitHub → Settings → Developer Settings → Personal access tokens → Fine-grained tokens → New token
- Repository: `patdunlavey/trimm-cbgs`
- Permission: **Contents → Read and Write**

---

## Step 3: Authorize the Script

Close and reopen the Google Sheet. A **"Wayne Trimm Archive"** menu will appear in the menu bar. The first time you click **↺ Redeploy Website**, Google will ask you to authorize the script — click through once and it will never ask again.

Other editors who open the sheet will not see the menu at all.

---

## How It Works

- `onOpen()` runs when the sheet is opened and adds the menu only for authorized email addresses
- Clicking the menu item calls `redeployToGitHub()`, which:
  1. Fetches the current `.deploy-trigger` file from the GitHub API (needed to get its SHA)
  2. Writes a new timestamp to the file and pushes a commit
  3. GitHub Pages detects the new commit and rebuilds the site

The site typically rebuilds within 1–2 minutes of clicking the menu item.