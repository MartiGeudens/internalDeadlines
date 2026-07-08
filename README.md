# Industria Deadline Planner

A static single-page web app for [Industria](https://industria.be) (student association of the Faculty of Engineering Technology, KU Leuven Campus Groep T) that helps praesidium members track **internal deadlines**: requests that must be submitted X weeks before an activity (communication, materials, drinks, room bookings, ...).

**Live site:** enable GitHub Pages (see [Deployment](#deployment)) and the app runs at `https://<owner>.github.io/<repo>/`.

## How it works

### Deadline planner

1. **Select your function** (Ontspanning, Sport, Cultuur, ...).
2. **Add your activities** for the year: name, main responsible, event type (BSA, Fakparty, Cantus, or *Other* with custom deadline selection) and date.
3. All internal deadlines are **calculated automatically** (exactly X weeks or days before the event, with a warning when a deadline falls in the past).
4. **Export everything at once**:
   - **Excel (.xlsx)** — grouped per activity, with a Status column (To Do / Busy / Done) that has a dropdown and colour coding that keeps working inside Excel.
   - **Calendar (.ics)** — import all deadlines into Google/Outlook Calendar.

#### Step-by-step guide

1. **Select your function** in step 1 (Ontspanning, Sport, Cultuur, ...).
2. Skip step 2 for the moment, this is for manual import.
   1. For manual import, skip to step 7.
3. Upload the Excel made after the agenda meeting via the **Import Excel/CSV** button.
4. Check if all the info is correct:
   1. Are all your activities imported?
   2. Are all the dates of your activities correct?
   3. Is the event type correct for predefined event types (like BSA, Cantus, IFB, ...)?
5. Add the (head)responsibles for each deadline of the activity (also possible to do later in Excel).
6. Click on **Confirm import** — all deadlines should pop up on the website chronologically.
7. If not all your activities were in the Excel, you can add them manually:
   1. Add the name of the activity.
   2. Add the main responsible of the event (also possible to do later in Excel).
   3. Select the event type:
      1. For predefined events in the config file the deadlines will be selected automatically.
      2. For *Other* activities, you need to select all deadlines that need to be followed yourself.
   4. Select the date of the event.
   5. Click on **Add activity**.
   6. Repeat for all events not in the activity Excel.
8. Done! Click on **Export Excel** and enjoy a deadline planner for within your function.

### Communication planner

Separate tab for the communication team. Every event type has a standard **post plan** (announcement, reminder, day-of story, aftermovie, ...) with an offset in days and a **default posting time** (e.g. 18:00). There is also a **weekly recurring posts** section for posts that go out every week at the same moment (e.g. the weekly overview): build a list of weekly posts — multiple channels at once, each with its own weekday and time — pick the period (quick presets: Semester 1 / Semester 2 / Whole year), and one post per week is generated for each — with the same collision handling, statuses and exports as regular posts. Everything added together counts as one activity, so those posts may share the same moment. In the "Agenda per week" export a recurring post appears in **every** week of its period, so it is never forgotten. Manually editing a post time immediately re-checks collisions with other activities. Add activities (or import them, see below) and the full posting schedule is generated. If posts of two *different* activities land on the same day at the same time, the later one is automatically shifted one hour and flagged, so posts of different events never overlap — posts belonging to the same activity (e.g. an Instagram and Facebook post together) may share the same moment. Times remain editable per post. The .ics export creates **30-minute calendar blocks** (e.g. 18:00–18:30) so posts are clearly visible in Google Calendar.

The communication Excel exports are **organised per semester week** (semester start dates are configurable in `config.json` / the Admin tab):

1. **Agenda per week** — every activity of the year grouped per week, with a colour-coded "Comm request" status column to tick off that the organising function submitted its communication request. The same status can be tracked in the overview table ("req" dropdown per activity).
2. **Post schedule per week** — every post chronologically per week (date, time, post, activity, responsible, status). Times are guaranteed collision-free: any remaining same-moment conflicts are resolved automatically at export.

### Bulk import (Excel/CSV)

Both planners have an **Import Excel/CSV** button so you don't have to type the whole year plan by hand:

- Columns: `activityName`, `eventType`, `activityDate` (+ optional `responsible`; the deadline planner also accepts `role`).
- With a `role` column, only rows matching your selected function are imported — so the praesidium can maintain one central year-plan file.
- An **import preview** shows what was recognised; unknown event types or invalid dates can be fixed inline before confirming.
- Use the **Import template** button to get a ready-made file with the right headers.
- Dates work as real Excel dates, `dd/mm/yyyy` or `yyyy-mm-dd`; CSV with `,` or `;` separators.

### Saving your work

The plan is **saved automatically in your browser** (localStorage) and survives a refresh. Note: it is tied to this browser on this computer — nothing is stored on a server, so use the Excel/calendar exports to share your planning.

## Admin mode

The **Admin** tab lets you manage the shared deadline rules:

- edit deadline types and their default offset (weeks or days),
- edit event types, which deadlines apply to them, and their communication plans,
- edit the list of roles (praesidium functions) and communication channels,
- set the semester start dates (academic year),
- add new deadline types and event types.

Changes apply immediately in your session; a banner warns while there are unsaved configuration changes (also on closing the tab). To make them permanent for everyone, either:

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
  "postTypes": [
    { "id": "announcement", "name": "Announcement post", "channel": "Instagram post", "defaultDaysBefore": 21, "defaultTime": "18:00" }
  ],
  "eventTypes": [
    { "id": "bsa", "name": "BSA",
      "deadlines": [ { "type": "communication", "weeks": 4 } ],
      "commPlan": [ { "type": "announcement", "daysBefore": 21, "time": "18:00" } ] }
  ]
}
```

Offsets can be given in weeks or days: use `defaultWeeks` **or** `defaultDays` on a deadline type, and `weeks` **or** `days` on an event-type deadline (days take precedence if both are present). The unit is also switchable per deadline in the Admin tab and in the *Other* activity form.

Communication posts work like deadline types: `postTypes` defines the shared list (name, channel, default offset in days, default time), and each event type's `commPlan` references them by id with an optional per-event `daysBefore`/`time` override. Configs in the old format (inline `name`/`channel` per commPlan entry) are migrated automatically at load time.

`config.json` is fetched at load time; if that fails (e.g. opening `index.html` directly from disk) the embedded default config in `index.html` is used. When editing by hand, keep both in sync.

## Development

No build step. Open `index.html` in a browser. Excel export uses [ExcelJS](https://github.com/exceljs/exceljs) from cdnjs.
