# CID Waypoint — Personnel Whereabouts Board
**DepEd · Curriculum Implementation Division**

Weekly tiles, month and year views, printable activity logs with the scanned Travel Authorities, an upcoming-activities calendar, and a passcode-protected admin console.

## The files

| File | What it is | Where it goes |
|---|---|---|
| `index.html` | The whole board in one file (styles, script and demo data inside) | GitHub repository root → GitHub Pages |
| `sw.js`, `manifest.webmanifest`, `icon-*.png` | Lets phones install the board as an app and open the last loaded week offline | Same folder as `index.html` |
| `test.html` | Systems check: tests the backend for reliability, speed and security | Same repository (or open it locally) |
| `Code.gs` | The Apps Script backend | Apps Script editor |
| `appsscript.json` | Apps Script manifest (time zone, permissions, web-app settings) | Apps Script editor |

Already created in your Google Drive:

```
CID Waypoint/                     mother folder
  CID Waypoint — Database         spreadsheet (tabs are built by setup)
  Logos/  Personnel Photos/  Templates/  Exports/
  2026-11 November/               created by setup, then every 1st of the month
    Maria Santos, EPS [P…]/       one folder per personnel — their TAs go here
```
`Code.gs` already holds these IDs (`APP.MOTHER_ID`, `APP.SHEET_ID`).

## 1 · Backend (10 minutes)

1. Open **script.google.com → New project**, name it `CID Waypoint API`.
2. Paste `Code.gs`. In **Project Settings** tick *Show "appsscript.json"*, then paste `appsscript.json`.
3. Choose **`setup`** → **Run** → approve permissions. It builds the tabs `Personnel`, `Activities`, `Events`, `Logos`, `Districts`, `Schools`, `Audit`, `Settings`, the month folders and the monthly trigger.
4. **Deploy → New deployment → Web app** — Execute as **Me**, Who has access **Anyone**. Copy the URL ending in `/exec`.
5. Later code changes: paste the new `Code.gs`, run **`setup`** once (it adds any new tabs and settings rows and keeps your data), then **Deploy → Manage deployments → ✎ → New version** (same URL).

## 2 · Test it

Open `test.html`, paste the `/exec` URL and the admin passcode, press **Run all checks**. It checks connection speed, caching, security refusals (no session, forged token, wrong passcode, disguised files, unknown file ids), a full write round-trip with real uploads to Drive, then deletes everything it created. With the URL empty it tests the built-in demo backend.

## 3 · Frontend on GitHub Pages

1. In `index.html`, find this line near the top and paste your URL:
   ```html
   <meta name="cid-api" content="https://script.google.com/macros/s/…/exec">
   ```
   (Edit only this tag — the page's security policy is locked to the exact script inside, so editing the script itself will stop it running.)
2. Upload `index.html`, `test.html`, `sw.js`, `manifest.webmanifest` and the three `icon-*.png` files to a GitHub repository → **Settings → Pages → Deploy from branch → main / root**.

Empty `cid-api` = demo mode with sample personnel.

## 4 · First sign-in

1. **Admin** → passcode `CID2026` → you are taken to **Settings** to change it.
2. **Branding** → upload the app logo (PNG with transparent background works), office name, the Chief's name and title for “Noted by”, and the **board footer** (up to 6 lines; web and email addresses become links, or switch it off).
3. **Personnel** → position (EPS, PSDS, …), name, employee no., learning area / assignment, **office / section** (tiles are grouped by this), photo.
4. **Coverage areas** → add the division's districts. Type each name; the system finds it on the map.
5. **Log activity** → pick personnel, type, dates, whereabouts, TA / memo no., **Scan with camera** each TA page → **Save**.
6. **Calendar** → click a day, add the activity, tag personnel. It shows on their tiles and the Upcoming rail.

## Passwords in the database

The admin passcode and viewer PIN are stored in the **Settings** tab of the database — as salted, iterated SHA-256 hashes, never as readable text. The tab is protected (warning on edit).

- **Forgot the passcode?** Type a new one (6+ characters) into the value cell of `NEW_ADMIN_PASSCODE`. Within about 2 minutes the system hashes it, clears the cell and signs out every admin.
- **Viewer PIN from the sheet:** type it into `NEW_VIEWER_PIN` (or `OFF` to remove).
- Office names, region, division and Chief's details are also editable there.
- Never type into the `PASS_HASH`, `PASS_SALT` or `*_EPOCH` rows.

## Security — what's in place, and what's on you

In place: server-side checks on every action; random 256-bit sessions that expire (2 h, 30 min idle) and are revoked on passcode change; lockout after 5 wrong tries per device and 20 overall; uploads checked by type and real file signature; Travel Authorities stay private in Drive and are served only for files the database lists; formula-injection and HTML escaping; script-hash Content-Security-Policy; full audit trail.

No website can be “impossible to hack”. The remaining risks:
- The board is public to anyone with its link until you set a **viewer PIN**. Personnel whereabouts are personal data under the Data Privacy Act — set one.
- Anyone who can open the database spreadsheet sees all records. Share it with no one; the board doesn't need it shared.
- Turn on **2-Step Verification** on the Google account that owns the script.
- Personnel photos are “anyone with the link can view” so the board can display them. The app logo and letterhead are served through the API instead.

## Troubleshooting

| Symptom | Fix |
|---|---|
| test.html: “Not JSON” | Deployment access must be **Anyone**; redeploy a new version. |
| “Backend not set up” | Run `setup` once. |
| Page is blank after editing | You edited the script, not the `cid-api` meta tag. Use the original `index.html` and edit only the tag. |
| Photos don't show | Drive file must be link-viewable; files added through the console are set automatically. |

## Rebuilding after source changes

The readable source is in `frontend/` (`template.html`, `styles.css`, `app.js`, `mock.js`, `test-template.html`). Run `python3 frontend/build.py` to produce `dist/index.html` and `dist/test.html`. If `esbuild` is installed (`npm i -g esbuild`), the build is minified (about 41 KB over the network with GitHub's compression).

## Print letterhead

Every printed log, summary and attachment page uses the header and footer from the division's template (`frontend/assets/template-source.pdf`). The header has the DepEd seal, Republic of the Philippines, Department of Education, Caraga Region and Schools Division of Agusan del Norte. The footer has the DepEd, Bagong Pilipinas and SDO Agusan del Norte logos with the address block. They repeat on every page, and no other logos are added. To change them, go to **Admin → Branding → Replace header / footer** (full-width PNG); the replacement is stored in the Drive *Templates* folder.

## Speed notes

- Sign-in: the console opens as soon as the passcode is accepted, and admin details load behind it. The server is woken while you type the passcode.
- Saving: the board refreshes in the background after a save, and audit entries are queued and written every 10 minutes by the `housekeeping` trigger (installed by `setup`).
- Uploads: each scanned page starts uploading as soon as it's added, into a `_Uploading` folder, and is filed into the person's month folder when you save. Files never saved are cleared after 24 hours.

## Alerts, digest, coverage and workload

- **Conflict alerts.** The board shows a red bar when someone tagged in a calendar activity is on leave or logged elsewhere, when leave overlaps travel, or when a large share of one section or learning area is away on the same day. The default share is 50%; change it in **Admin → Settings**. Saving a conflicting log or calendar entry still saves it, then lists the conflicts. Affected tiles get a red **!**.
- **Monday digest.** In **Admin → Settings → Alerts & weekly digest**, add recipients such as the Chief. Every Monday at 7:00 AM they get an email listing who is out that week, the alerts, the coming activities and a button to open the board. **Send test email now** sends one immediately. This needs the email permission, so run `setup` once after updating `Code.gs` and approve it.
- **Coverage areas (admin).** **Admin → Coverage areas** has two lists, **Districts** and **Schools**, each with a filter, **Add**, **Edit**, a red bin button on every row (it works on phones too), tick boxes with **Select all** and **Delete selected**. Deleting a district asks whether to delete its schools as well. Renaming a district moves its schools to the new name.
  - **Districts:** type a name (and the town if the name is not a place) and the system finds it on the map: Google's map search built into Apps Script, then OpenStreetMap. The **search area** (from the division name, e.g. *Agusan del Norte*) keeps “Santiago District” in your province. A red dot means not found, an open red ring means found far from the others: add the town, press **Find again**, or paste coordinates from Google Maps. Paste a whole list to add many; they are located 8 at a time with a progress count. Opening a district lists its schools, where you can add or delete them directly.
  - **Schools (optional):** name, School ID, district, type and an optional location. Without a location the system looks for the school near its district (within 35 km, so a same-named school elsewhere is never used). **Find missing locations** looks up the rest 10 at a time. Paste from a spreadsheet to import many.
- **Logging visits.** For monitoring, technical assistance and LAC, pick the **districts visited**; picking a school adds its district automatically. Older logs are matched by district, town or school names in the whereabouts text.
- **Coverage map (OpenStreetMap).** **Insights → Coverage** and the map in **Admin → Coverage areas** use OpenStreetMap: free, no account or key. Every district is a coloured circle with a name tag (blue = monitoring and TA, light blue = one of them, grey = not yet visited; the tag shows the visit count). **Map / Satellite** switches between OpenStreetMap streets and Esri satellite imagery with place names (the choice is remembered); **Fit all**; full screen; scroll-wheel zoom after clicking the map. Tap a district's name on a card to go to it; **Show schools on map** adds the schools. In the admin map, tapping a district opens it for editing. Name tags that would overlap shrink to dots until you zoom in. Without internet the map says so and the district cards still show everything.
- **Travel days.** **Insights → Travel days** totals days out of office per month, quarter or school year. It groups them by section or district, learning area, or position, and flags anyone at 1.5× or under half of their group's average.
- **Automatic status.** A weekday with nothing logged counts as **In office** (grey). Saturdays and Sundays show as **Weekend · no office** (lavender). A logged TA, leave or activity always takes over, including on weekends.
- **Friday work from home.** **Admin → Settings → Work week** has a switch that turns Fridays with nothing logged into **Work from home**, shown as a hollow blue ring. Logged activities still come first.
- **Tiles by office.** Tiles are grouped under each person's **Office / section**. **Admin → Settings → Board arrangement**: move offices up or down, open an office to order its people (or Sort A–Z), then **Save arrangement**. Switches there turn off grouping or the pinned group. Pinned: everyone out of office today is listed first, and their office header says how many are listed above. The year view uses the same groups.
- **QR codes.** Each printed log has a QR code that opens that person's record for the same period on the board. The summary printout's QR code opens Insights.
- **Phone app.** Open the board in Chrome (Android) and use **Install**, or in Safari (iPhone) use Share → Add to Home Screen. Without signal, the app opens with the last loaded week and says so.
