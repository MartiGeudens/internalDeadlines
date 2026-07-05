# Industria Deadline Planner

A static single-page web app for [Industria](https://industria.be) (student association of the Faculty of Engineering Technology, KU Leuven Campus Groep T) that helps praesidium members track **internal deadlines**: requests that must be submitted X weeks before an activity (communication, materials, drinks, room bookings, ...).

**Live site:** enable GitHub Pages (see [Deployment](#deployment)) and the app runs at `https://<owner>.github.io/<repo>/`.

## How it works

1. **Select your function** (Ontspanning, Sport, Cultuur, ...).
2. **Add your activities** for the year: name, main responsible, event type (BSA, Cantus, Fakfeest, or *Other* with custom deadline selection) and date.
3. All internal deadlines are **calculated automatically** (exactly X weeks before the event, with a warning when a deadline falls in a weekend or in the past).
4. **Export everything at once**:
   - **Excel (.xlsx)** — grouped per activity, with a Status column (To Do / Busy / Done) that has a dropdown and colour coding that keeps working inside Excel.
   - **Calendar (.ics)** — import all deadlines into Google/Outlook Calendar.

Nothing is stored on a server; you build your year plan in the browser and export it.

## Admin mode

The **Admin** tab lets you manage the shared deadline rules:

- edit deadline types and their default number of weeks,
- edit event types and which deadlines apply to them,
- add new deadline types and event types.

Changes apply immediately in your session. To make them permanent for everyone, either:

- **Publish directly to GitHub** (recommended): fill in owner/repo/branch plus a fine-grained personal access token (*Contents: read & write*, this repo only) and click *Commit config.json*. The site redeploys automatically. The token is never stored.
- **Download config.json** and commit it manually.

On the deployed site the Admin tab is protected with a password (see below). When the app is opened locally (`file://`), the Admin tab is open — handy for testing.

## Deployment

1. Push this repository to GitHub (branch `main`).
2. **Settings → Secrets and variables → Actions**: add a secret named `ADMIN_PASSWORD` with the admin password of your choice.
3. **Settings → Pages**: set *Source* to **GitHub Actions**.
4. Push (or run the workflow manually). The workflow in `.github/workflows/deploy.yml` injects a SHA-256 hash of the password into the page and deploys it.

> **Security note:** the password check is client-side and is a deterrent, not hard security. Real write protection comes from GitHub permissions: only someone with a valid token can change `config.json`.

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire app (UI + logic + embedded fallback config) |
| `config.json` | Shared rules: deadline types, event types, roles |
| `.github/workflows/deploy.yml` | GitHub Pages deploy + password hash injection |

## Configuration format

```json
{
  "roles": ["Ontspanning", "Sport", "..."],
  "deadlineTypes": [
    { "id": "communication", "name": "Communication request", "defaultWeeks": 4 }
  ],
  "eventTypes": [
    { "id": "bsa", "name": "BSA", "deadlines": [ { "type": "communication", "weeks": 4 } ] }
  ]
}
```

`config.json` is fetched at load time; if that fails (e.g. opening `index.html` directly from disk) the embedded default config in `index.html` is used. When editing by hand, keep both in sync.

## Development

No build step. Open `index.html` in a browser. Excel export uses [ExcelJS](https://github.com/exceljs/exceljs) from cdnjs.
