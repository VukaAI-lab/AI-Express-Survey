# VUKA AI Express Survey

End-of-year AI survey for VUKA, presented as a boarding pass. People answer 10 short questions across 7 "zones", then watch a plane take off from an airport. It is a single static file with no build step.

- **Page:** `index.html` (HTML, CSS and JavaScript in one file, logo embedded)
- **Answers:** sent as one flat JSON row per person to a URL you set (see [Collecting answers](#collecting-answers))
- **Anonymous:** no name, email or login is collected

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The whole survey. This is the only file you need to host. |
| `power-automate-schema.json` | Paste into the Power Automate trigger so each field is available as a column. |
| `README.md` | This guide. |

## Quick start

1. Open `index.html` in a browser to try it. No server needed.
2. Set up a place for answers to land (next section).
3. Host the file (see [Going live](#going-live)).

Until an endpoint is set, the survey works and looks right but **answers go nowhere**. Do not send the link to staff before that is done.

## Collecting answers

A web page cannot write straight into an Excel file, so a small middle step receives the answers. The easiest no-code option is Power Automate.

### Option A: Power Automate to Excel (about 10 minutes)

1. In SharePoint or OneDrive, create an Excel file, for example `VUKA AI Survey 2026.xlsx`.
2. Add a **table** with exactly these column headers:

   `submittedAt`, `persona`, `score`, `q1`, `q2`, `q3`, `q4`, `q5`, `q6`, `q7`, `q8`, `q9`, `q10`

3. In [Power Automate](https://make.powerautomate.com), create an **Instant cloud flow** with the trigger **When an HTTP request is received**.
   - Set **Who can trigger the flow** to **Anyone**.
   - Click **Use sample payload to generate schema** and paste the contents of `power-automate-schema.json`.
4. Add the action **Excel Online (Business), Add a row into a table**. Choose your file, then the table, then map each column to the matching field from the trigger.
5. Save the flow. The trigger now shows an **HTTP POST URL**. Copy it.
6. In `index.html`, find the `CONFIG` block near the top of the script and paste the URL:

   ```js
   const CONFIG = {
     endpoint: 'PASTE-THE-FLOW-URL-HERE',
     surveyName: 'VUKA AI Journey 2026'
   };
   ```

7. Open the page, complete the survey once, and check that a new row appears in the Excel table.

### Option B: SharePoint List

Same flow, but use the action **SharePoint, Create item** and create a List with the same column names (all as single line of text).

### Option C: IT's own system

If Darika or IT already have an endpoint that stores form data (as for other VUKA forms), point `CONFIG.endpoint` at it. The page sends an HTTP `POST` with the body below.

### What the page sends

One JSON object per completed survey:

```json
{
  "submittedAt": "2026-10-22T09:30:00.000Z",
  "persona": "Workflow Wizard",
  "score": 12,
  "q1": "Daily",
  "q2": "Copilot; Claude; ChatGPT",
  "q3": "Used AI to draft, edit or summarise; Shown or taught a colleague",
  "q4": "Look for a way to speed it up with AI",
  "q5": "Skim it and tidy it up",
  "q6": "Fairly confident",
  "q7": "1 to 3 hours",
  "q8": "Yes, once or twice",
  "q9": "Energised",
  "q10": "Hands-on training; Protected time to experiment"
}
```

- Multi-select answers are joined with `; `.
- The request uses `Content-Type: text/plain` and `mode: 'no-cors'`. This lets it work with Power Automate, which does not send CORS headers. The page cannot read the reply, so it never shows a success or error message after the take-off screen.
- If the request cannot be sent at all (for example, no internet), the take-off screen shows **Copy my answers** and **Download** buttons as a fallback.

## Going live

Pick one. All of them just serve `index.html` as a static page.

### Company domain (recommended)

Give `index.html` to Darika to host on the VUKA domain, the same way as the carbon footprint form. The page needs no server code. It loads the Inter font from `fonts.googleapis.com` and `fonts.gstatic.com`; allow these if the site uses a strict content security policy. If blocked, it falls back to Segoe UI.

### GitHub Pages

1. Open the repo **Settings, then Pages**.
2. Set **Source** to **Deploy from a branch**, branch **main**, folder **/ (root)**, then **Save**.
3. After 1 to 2 minutes the site is at `https://vukaai-lab.github.io/AI-Express-Survey/`.

If Pages is greyed out, you need admin rights on the organisation, or the repo needs to be public (depends on the GitHub plan).

### Netlify Drop (fastest)

Put only `index.html` in a folder and drag it onto <https://app.netlify.com/drop>. You get a live link in seconds, and can rename it or attach a custom domain later.

### Not SharePoint

SharePoint pages and document libraries block or download custom HTML like this, so do not host the page there. The Excel file or List can live in SharePoint; the page itself should be hosted elsewhere.

## Before you share the link

- [ ] Endpoint set in `CONFIG` (not empty)
- [ ] One test submission done and the row appears in Excel
- [ ] Checked on a phone (the page is responsive)
- [ ] Link works for people outside your own computer or network
- [ ] Decide who can see the Excel file (answers are anonymous, but still internal data)

## Editing the survey

Everything is in the `<script>` block of `index.html`.

- **Questions and options:** edit the `QUESTIONS` array. Each question has an `id` (`q1` to `q10`), `type` (`single` or `multi`), `style` and `options`. Keep the `id` values stable, because they are the Excel column names. If you add a question, add a matching column in Excel and a field in the flow.
- **Zones (the stamps at the top):** edit the `SCENES` array. Each question points at a zone through its `scene` field.
- **Select limit:** a multi-select question can have `max: 2`, as Q10 does.
- **Exclusive option:** an option with `exclusive: true` (such as "None" in Q2) clears the other choices.
- **Wording on the boarding pass and take-off screen:** search for `renderIntro` and `renderResult`.

### Persona

The survey calculates a persona for each person from their answers and saves it in the `persona` and `score` columns. It is not shown on screen. Scoring lives in the `score()` function and reads Q1, Q3, Q5, Q6 and Q7 by option position, so **if you reorder the options of those questions, update `score()`**. The personas are Packed & Ready, Prompt Explorer, Workflow Wizard, Future Builder, AI Pioneer and Automation Architect.

## Design notes

- Brand colours: navy `#174083`, gold `#E2BE55`, teal `#2A7FB0`, with a white form.
- Font: Inter (Google Fonts), with Segoe UI as a fallback.
- No dark mode; the form is always light.
- Single-choice questions do not move on automatically. People press **Next** when ready.

## Troubleshooting

| Problem | Fix |
| --- | --- |
| Nothing arrives in Excel | Check `CONFIG.endpoint` is set and is the full flow URL. Open the flow's **Run history** to see whether it ran and where it failed. |
| Flow runs but a column is empty | The column header does not match the field name exactly (they are case-sensitive). |
| Rows are added but columns are shifted | Re-map the columns in the **Add a row into a table** action. |
| GitHub link shows 404 | Pages is not switched on yet. See [GitHub Pages](#github-pages). |
| Logo or fonts missing | The logo is embedded in the file. Fonts come from Google Fonts; check the site is not blocking it. |
| Copy and Download buttons appear at the end | The answers could not be sent. Check the endpoint URL and the user's internet connection. |
