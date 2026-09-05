# Legends Organisation — Mock Donation Intake Portal

A static test fixture for **WF0 `Portal_DownloadDeliveryNotes`** (UiPath Studio Web).
No backend. Hosts on GitHub Pages as-is.

The portal is operated by the charity, not by a donor. Donors publish their
delivery notes to it; the **coordinator** signs in and retrieves them. WF0
automates the coordinator's side.

All donor organisations, contacts and delivery notes are fictional and were
created by the project team. No real donor, charity or personal data is used.

---

## Files

```
index.html          Login page
dashboard.html      Delivery-note listing, gated on a session token
assets/portal.css   Styles. No external fonts or CDNs
README.md           This file
```

The delivery-note PDFs are **not in this repo**. They are hosted on Google Drive
and linked from `dashboard.html`. See "PDF hosting" below — this has consequences
for WF0 that you must resolve before building.

## Publish on GitHub Pages

1. Push these files to the repo root on the `main` branch.
2. Settings → Pages → Source: **Deploy from a branch** → `main` / `/ (root)`.
3. Wait for the build, then open `https://<user>.github.io/<repo>/`.

## Test credentials

| Field | Value |
|---|---|
| Username | `coordinator` |
| Password | `Rescue2026` |

Do **not** type these as literals into the `Type Into` activity. Store them as an
Orchestrator credential asset and read them with `Get Credential`. The `.uis`
export is submitted and shared with the team; plaintext credentials in it read as
a build-practice failure.

---

## PDF hosting — read this before building WF0

The three delivery notes are Google Drive files. Two things follow, and both
change how WF0 has to be written.

**1. The HTML `download` attribute does not work across origins.** Browsers
ignore `download` when the `href` points at a different domain. Drive is a
different domain. Clicking a link therefore *navigates* the tab to Drive rather
than saving a file to disk. The attribute is still present in the markup, but it
has no effect.

**2. A `/view` link returns a web page, not a file.** Use the direct form:

```
https://drive.usercontent.google.com/download?id=<FILE_ID>&export=download
```

### Before anyone writes a workflow

- [ ] Open each of the three links in a **private browser window**. If any of
      them prompts for a Google sign-in, that file is not shared correctly. Set
      it to **Anyone with the link → Viewer**.
- [ ] Click one link on the machine the robot will run on. Confirm whether a
      file lands on disk, or whether the browser navigates to a Drive page.
- [ ] Record the answer here before building. WF0's design depends on it.

### Two viable WF0 designs, depending on that answer

**A — A file lands on disk.** Keep the shape in "Suggested WF0 shape" below.
`Path Exists` is a valid check. Note that Drive supplies the saved filename, not
the `download` attribute, so verify what it actually writes — a re-download can
produce `... (1).pdf`, which will break WF1's `SourcePDFName` uniqueness guard.

**B — The browser navigates to Drive.** `Path Exists` will fail every run. Either
add browser steps to handle the Drive page, or switch the retrieval step to the
Google Workspace **`Download File`** (Drive) activity and fetch by file ID. Option
B-2 is more reliable, but it makes WF0 data automation rather than UI automation,
which drops your feature count from five to four. That is a project-level
decision, not a developer one — take it to Alex.

**The lowest-risk option remains hosting the PDFs in the repo** under a `notes/`
folder and linking them same-origin (`href="notes/file.pdf"`). `download` then
works, the filename is exactly what you specify, and Google is removed from the
chain entirely. This was the earlier setup and it was replaced deliberately; if
the Drive route costs more than an afternoon, reverting is one commit.

---

## Selector reference for WF0

Every interactive element carries a stable `id`, `name` and `data-testid`. None
of them are generated at runtime, so selectors will not drift between page loads.

### index.html

| Element | id | name | data-testid |
|---|---|---|---|
| Username input | `username` | `username` | `login-username` |
| Password input | `password` | `password` | `login-password` |
| Sign in button | `btnLogin` | `btnLogin` | `login-submit` |
| Error message | `loginError` | `loginError` | `login-error` |

The error element carries `hidden` until a failure occurs. Use its visibility as
a negative check in `Check App State`.

### dashboard.html

| Element | id | data-testid |
|---|---|---|
| Signed-in username | `sessionUser` | `session-user` |
| Page-ready heading | `dashboardReady` | `dashboard-ready` |
| Note count | `noteCount` | `note-count` |
| Notes table | `deliveryNotesTable` | `delivery-notes-table` |
| Table body | `deliveryNotesBody` | `delivery-notes-body` |
| Each row | — | `note-row` (also `data-note-ref`) |
| Filename cell | — | `note-filename` |
| Download link | `downloadAlphaSupermart`, `downloadBravoSupermart`, `downloadCharlieSupermart` | `download-link` (also `data-note-ref`) |
| Sign out | `btnLogout` | `logout-button` |

**Suggested WF0 shape**

1. `Use Application/Browser` → portal URL
2. `Type Into` → `username`, `password` (values from `Get Credential`)
3. `Click` → `btnLogin`
4. `Check App State` → `dashboardReady` exists → proves the login worked
5. `Extract Table Data` → `deliveryNotesTable` → read the filename column
6. `For Each Row` → `Click` the matching `download-link`
7. `Path Exists` on each expected file → `Throw` a `BusinessRuleException` if missing

Step 5 matters. Reading filenames from the table rather than hardcoding them is
what makes the UI automation a real read-and-react step instead of three clicks on
known links.

Step 7 is only valid under design A above.

## The login actually gates the dashboard

`dashboard.html` checks `sessionStorage` before the body paints and redirects to
`index.html?auth=required` if no session token is present. Navigating straight to
the dashboard URL does not work — the robot must complete the login.

This is the answer to "was each feature used properly rather than for show." Say
it on camera: the portal is a simulated intake portal, and the login gates access
rather than decorating it.

## Chrome download behaviour

Chrome may open PDFs in its built-in viewer instead of saving them. Setting
`chrome://settings/content/pdfDocuments` → **Download PDFs** makes it save
instead. This does not fix the cross-origin problem above; it only affects how
the browser treats a PDF it has decided to fetch.

Confirm where files land on the execution target you will use for the demo — a
local robot and a cloud robot resolve the download folder differently.

---

## Delivery notes and the scenarios they carry

| File | Row ref | Scenario it forces |
|---|---|---|
| `Food Donation Form_Alpha Supermart_20260822.pdf` | `DN-AS-20260822-01` | Contains an item dated 22 Aug 2025 and another dated 15 Aug 2026 — both already expired at the 22 Aug 2026 handover. Tests the `DaysToExpiry >= 0` filter. Without it, expired stock scores the highest urgency in the system. Also carries `Halal = No` on one line and `Minimal Leakage` in the condition column |
| `Food Donation Form_Bravo Supermart_20260825.pdf` | `DN-BS-20260825-01` | **Not yet documented.** Open the PDF, record which scenario it covers, and fill in the `note-lines` and `note-storage` cells in `dashboard.html` — both currently read `TBC` |
| `Food Donation Form_Charlie Supermart_20260820.pdf` | `DN-CS-20260820-01` | **Not yet documented.** Same as above |

Scenarios still needed across the full test set, per the build plan:

- **Chilled or frozen stock** — exercises the `HasColdStorage` hard constraint
- **A batch that must be split** — quantity exceeds one recipient's `MaxCapacityKgPerPickup`
- **A contested item** — two eligible recipients, insufficient supply. This is
  what forces the fairness rule to visibly decide something on camera
- **No eligible recipient** — proves the `Unallocated-AtRisk` path
- **A deliberately messy PDF** — skewed scan, inconsistent column widths or a
  handwritten quantity, so the 0.70 confidence gate actually fires in testing

Harry owns this set. Without it the recorded demo contains only straightforward
matches and the agentic feature looks decorative.

## Adding a note

1. Upload the PDF to the team Drive folder.
2. Set sharing to **Anyone with the link → Viewer**. Verify in a private window.
3. Copy the file ID out of the share URL and build the direct link:
   `https://drive.usercontent.google.com/download?id=<FILE_ID>&export=download`
4. Copy a `<tr>` block in `dashboard.html`. Change the seven cell values, the
   `id`, both `data-note-ref` attributes, and both the `href` and `download`
   filenames.
5. Add a comment above the `<a>` naming which PDF the file ID points at. A bare
   ID is not diagnosable without clicking it.
6. The note count updates itself from the row count — nothing else to change.

Filenames contain spaces, so any URL built from them needs `%20` escaping.
`SourcePDFName` is the uniqueness key for WF1's duplicate guard: if Drive ever
serves a file as `... (1).pdf`, that guard stops matching and duplicate inventory
rows return silently.