# Personal Shopper Network — Landing Page Setup

This guide walks you through wiring the signup form to a Google Sheet so every email from `index.html` lands in a spreadsheet you control. Total time: about 5 minutes.

## Step 1 — Create the Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new blank spreadsheet.
2. Rename it something like **PSN Beta Signups**.
3. In **row 1**, add these column headers exactly:

   | A           | B     | C            | D         |
   |-------------|-------|--------------|-----------|
   | Timestamp   | Email | Source       | UserAgent |

## Step 2 — Add the Apps Script

1. In your sheet, click **Extensions → Apps Script**. A new tab opens.
2. Delete the placeholder code in `Code.gs` and paste in the script below:

```javascript
function doPost(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    const data = JSON.parse(e.postData.contents);

    // Append a new row
    sheet.appendRow([
      new Date(),
      data.email || '',
      data.source || 'unknown',
      (e.parameter && e.parameter.ua) || ''
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ success: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ success: false, error: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet() {
  return ContentService
    .createTextOutput(JSON.stringify({ status: 'PSN signup endpoint live' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Click the **Save** icon (or Ctrl/Cmd + S). Give the project a name like **PSN Signup Endpoint**.

## Step 3 — Deploy as a Web App

1. Click **Deploy → New deployment** (top right).
2. Click the gear icon next to "Select type" and choose **Web app**.
3. Fill in:
   - **Description**: `PSN beta signup`
   - **Execute as**: **Me** (your Google account)
   - **Who has access**: **Anyone** (this is required so the public form can post to it — only your script can write to your sheet)
4. Click **Deploy**.
5. Google will ask you to **Authorize access** the first time — accept the permissions for your own account. You may see a "Google hasn't verified this app" warning; click **Advanced → Go to (project name)** to proceed (this is normal because it's your private script).
6. Copy the **Web app URL** that ends in `/exec`. It will look like:

   ```
   https://script.google.com/macros/s/AKfycby...long-string.../exec
   ```

## Step 4 — Paste the URL into the landing page

1. Open `index.html` in any text editor.
2. Find both occurrences of `REPLACE_WITH_YOUR_GOOGLE_APPS_SCRIPT_URL` (there are two — one in the hero form, one in the bottom CTA form).
3. Replace each with the URL you just copied. Keep the quotes around it.

Example:

```html
data-endpoint="https://script.google.com/macros/s/AKfycby.../exec"
```

4. Save the file.

## Step 5 — Test it

1. Open `index.html` in a browser (double-click it, or drag it into a tab).
2. Enter a test email and click **Request invite**.
3. You should see "You're on the list" appear under the form.
4. Switch to your Google Sheet — a new row should appear within a second or two.

If you don't see the row, check the browser console (F12) for errors.

## Step 6 — Publish the site

The page is a single static HTML file with no build step, so you can host it anywhere:

- **Netlify Drop** (easiest — [app.netlify.com/drop](https://app.netlify.com/drop)): drag `index.html` onto the page, get a free URL in seconds.
- **Vercel**: `vercel deploy` from the folder.
- **GitHub Pages**: commit `index.html` to a repo, enable Pages.
- **Cloudflare Pages**: connect a repo or upload directly.

You can attach a custom domain (e.g. `personalshoppernetwork.com`) on any of these.

## Updating the deployment later

If you change the Apps Script code, you must redeploy:

1. **Deploy → Manage deployments**
2. Click the pencil icon on your existing deployment
3. Change **Version** to **New version**
4. Click **Deploy**

The URL stays the same.

## Troubleshooting

**"Backend not yet configured"** appears when submitting → You haven't replaced `REPLACE_WITH_YOUR_GOOGLE_APPS_SCRIPT_URL` in `index.html`.

**Form says success but no row appears in the sheet** → Make sure you deployed with **Who has access: Anyone**. Re-check by going to Deploy → Manage deployments.

**"Something went wrong"** → Open the browser console (F12 → Console tab) and submit again. The error there will tell you whether it's a network issue or a script issue.

**Spam protection** — if you start getting bot signups, you can add a simple honeypot field to the form, or front the endpoint with Cloudflare Turnstile. Easy to add later; let me know when it's needed.
