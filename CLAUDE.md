# Industria Internal Deadline Planner

## Context
Tool voor studentenvereniging **Industria** (KU Leuven Campus Groep T, ~16 praesidiumfuncties, 100+ activiteiten/jaar). Probleem: interne deadlines (aanvragen X weken vóór een event) worden slecht opgevolgd. Gebruiker: Marti Geudens (Schachtenmeester).

## Wat het doet
Statische single-page app voor **GitHub Pages** (geen backend). Flow:
1. Selecteer functie → 2. Voeg activiteiten toe (naam, hoofdverantwoordelijke, event-type, datum) → 3. Deadlines worden automatisch berekend (exact X weken/dagen vóór event, bewust géén weekend-correctie of -waarschuwing; wel waarschuwing bij datum in verleden) → 4. Export.

## Bestanden
- `index.html` — volledige app (UI + logica + embedded fallback-config). ExcelJS via cdnjs.
- `config.json` — deadline-types (met defaultWeeks/defaultDays) + event-types + postTypes + rollenlijst. Wordt via fetch geladen; embedded DEFAULT_CONFIG in index.html is de fallback voor file:// gebruik. **Bij wijziging: beide synchroon houden.**
- **Roles-structuur (sinds juli 2026)**: array van objecten `{ name, headResponsible }` — headResponsible = persoon die de functie momenteel leidt (mag leeg zijn; per functie, niet per activiteit). Beheer incl. headResponsible in de Roles-kaart van de admin-tab. `migrateConfig()` zet oude configs met roles-als-strings automatisch om naar `{ name, headResponsible:'' }`.
- `.github/workflows/deploy.yml` — Pages-deploy; injecteert sha256-hash van secret `ADMIN_PASSWORD` op de plek van placeholder `__ADMIN_HASH__` in index.html (sed, placeholder mag maar 1× letterlijk in het bestand staan!).

## Admin-beveiliging & GitHub-publicatie
- Admin-tab zit achter wachtwoord: client-side sha256-check tegen geïnjecteerde hash. Placeholder niet vervangen (lokaal/secret ontbreekt) ⇒ admin open. **Dit is een deterrent, geen echte security** — echte schrijfbeveiliging = GitHub-permissies.
- Setup: repo → Settings → Secrets and variables → Actions → secret `ADMIN_PASSWORD`; Pages-source op "GitHub Actions" zetten.
- "Publish directly to GitHub" in admin: commit config.json via GitHub Contents API (PUT, met sha van bestaande file). Vereist fine-grained PAT (Contents: read & write, alleen deze repo). Token wordt nooit opgeslagen; owner/repo/branch wel in localStorage (`ghRepoInfo`).
- Let op: commit updatet alleen config.json; embedded DEFAULT_CONFIG in index.html blijft achter (alleen relevant voor file://-gebruik).

## Kernbeslissingen (bevestigd door Marti)
- Deadline-regels zijn **gedeeld** over de vereniging; event-types bepalen welke gelden.
- **Admin-modus in de site**: regels aanpassen in de UI → download nieuwe `config.json` → committen naar repo voor permanentie.
- Geen opslag van jaarplanning nodig; export is genoeg.
- Interface in het **Engels**.
- Exports: **Excel** én **.ics** agenda.

## Excel-formaat (vast, door Marti aangeleverd screenshot)
**Vast celcontract bovenaan (sinds juli 2026):** rij 1 = `A1` label "Hoofdverantwoordelijke functie", **`B1` = headResponsible van de geselecteerde rol**; rij 2 leeg; gegroepeerde content start op rij 3. **B1 niet verplaatsen**: het aparte deadlineNotifications-project (Teams-bot, andere repo) leest die cel uit via de Microsoft Graph API om die persoon te @mentionen.
Daaronder gegroepeerd per activiteit. Headerrij per activiteit (bold, groene bovenrand): `Activiteitnaam | Hoofdverantwoordelijke | Datum activiteit | "Status"`. Daaronder per deadline: `Deadlinenaam | Verantwoordelijke | Datum deadline | Status`. Status heeft dropdown (To Do/Busy/Done) + kleuren: Done=groen (C6EFCE), Busy=geel (FFEB9C), To Do=rood (FFC7CE), plus conditional formatting zodat kleuren mee veranderen in Excel zelf.

## Communication planner (aparte tab)
Voor team Communicatie: gedeelde **postTypes** in config (zoals deadlineTypes: id, naam, channel, `defaultDaysBefore` — negatief = ná het event — en `defaultTime`); per event-type een **commPlan** met referenties `{type, daysBefore, time}` (override per event). Admin: postTypes-kaart + checklist per event-type (zelfde patroon als deadlines). `resolveCommPlan(et)` zet referenties om naar concrete posts; `migrateConfig()` converteert oude inline commPlans automatisch bij load/upload. Zelfde flow als deadline planner: activiteiten toevoegen → postplan gegenereerd → zelfde Excel/ICS-export (functies zijn geparametriseerd: `exportExcel(list, fname)`, `exportICS(list, fname, timed)`). "Other" = checklist van postTypes (aanvinken + offset/tijd overriden, zoals de deadline-checklist in de dl-planner) + optioneel custom posts. Channels als datalist uit `config.commChannels`. Beheer van commPlans (incl. tijden) in de admin-tab per event-type. **Default commPlans zijn gokken van Claude, valideren met team Communicatie.**
- **Tijdbotsing**: posts van *verschillende* activiteiten op zelfde dag+uur schuiven automatisch +1u op (`assignCommTimes`); posts van dezelfde activiteit mogen samenvallen. ⏲-markering in UI. Uur per post editeerbaar, met live her-check bij wijziging (`setCommTime`).
- **ICS voor comm is timed**: 30-minutenblok (18:00–18:30) i.p.v. all-day, zodat zichtbaar in Google Calendar. Deadline planner blijft all-day.
- **Week-exports (vervangen per-activiteit Excel voor comm)**: `exportCommAgenda()` = alle activiteiten per semesterweek met "Comm request"-status per activiteit (`a.commReqStatus`, ook in UI als "req"-dropdown); `exportCommSchedule()` = alle posts chronologisch per week, met finale botsings-resolutie bij export. Weekberekening: `weekLabel()`/`mondayOf()`, semesterstart in `config.academicYear` (S1 2026-09-21, S2 2027-02-08; instelbaar in admin). Weken tellen door per semester (S1 W20 vlak vóór S2 W1); vóór S1 = "pre-S1". **Elke week = een apart werkblad** (sheetnaam bv. "S1 W1 21-09" via `weekSheetName`; titelrij + kolomkoppen via `weekSheet`). Excel-helpers: `excelStatusCell/excelStatusCF/groupByWeek` (excelWeekHeader is legacy, ongebruikt).

## Link page / "linktree" (toegevoegd juli 2026)
- Map `linktree/` = **aparte toekomstige GitHub Pages-repo** (staat in `.gitignore` van deze repo). Bevat: `links.html` (publieke bio-linkpagina), `events.json` (jaarplanning + ticketlinks), `links.json` (standaardlinks + socials — alleen via GitHub-editor te wijzigen, bewuste keuze), `README.md` (setup + comm-workflow). Geen admin, geen workflow: Pages "deploy from branch".
- `links.html` fetcht beide JSON's apart (embedded fallbacks voor file:// — synchroon houden bij wijziging). Toont eerstvolgende `showUpcoming` (default 3) events; voorbije verdwijnen automatisch. `url` ingevuld ⇒ rode ticketknop; leeg ⇒ "Join us at {naam} · {datum}" (bewust géén "tickets soon" — elk event wordt gepromoot).
- **Generator zit in de admin-tab van de planner** (kaart "Link page — generate events.json"): importeert dezelfde jaarplanning-Excel (hergebruikt `HEAD_MAP`/`parseDateCell`/`splitCsvLine`; enkel activityName+activityDate, event-types/rollen genegeerd), preview-overlay `#linkOverlay` met checkbox per rij + optioneel timeNote-veld, download `events.json` met lege urls. Comm uploadt handmatig naar de linktree-repo en vult ticketlinks later in via de GitHub-editor. Bewust geen publish-API voor deze repo.

## Bulk import & persistentie (toegevoegd juli 2026)
- **Import Excel/CSV** in beide tabs (`startImport(ev, mode)` met mode 'dl'|'comm'). Headers via `HEAD_MAP` (EN+NL aliassen): activityName, eventType, activityDate, responsible, role (alleen dl). Role-kolom filtert op geselecteerde functie ('otherrole' = skipped). Datums: Excel-date-cellen, serials, dd/mm/yyyy, yyyy-mm-dd. CSV met , of ;.
- **Preview-modal** (#importOverlay): fixes inline (dropdown voor onbekend type, date-input voor foute datum) vóór confirm. Template-download per tab (`downloadTemplate`).
- **Persistentie**: autosave naar localStorage (`industriaPlannerState`) bij elke render/mutatie. Dates worden gerevived met `reviveList`. "Clear all" per tab. Functie-keuze apart in localStorage (`industriaRole`). Geen savePlan/loadPlan naar JSON-bestand — bewust niet (Marti, juli 2026): exports volstaan om te delen.
- **Pending activities** (`pendingActs.dl/.comm`): importrijen met onbekend type (bv. "Other") worden niet weggegooid maar bewaard in een "Imported without a plan"-kaart per tab. Daar: event-type toekennen (Apply), "Set up manually" (vult add-form met type Other) of verwijderen. Zit mee in autosave/savePlan.

## Bekende omgevingsquirk (Cowork-sessies)
De sandbox-mount van deze projectmap synct traag/onvolledig; node-checks op `/sessions/.../mnt/internalDeadlines/` kunnen stale content zien. Verificatie: functies kopiëren naar /tmp en daar testen, of host-side Grep/Read gebruiken.

## Deadline-types (defaults zijn voorlopige schattingen!)
Communication request (4w), Material request (4w), Material order/new purchase (6w), Drink request (2w), Food request (2w), Industria van request (3w), Province request (8w), Public domain occupation (8w), Room request (6w).
**TODO: echte weken-waarden en per-event-type mapping laten valideren door Marti/praesidium.**

## Open punten
- Event-type ↔ deadline-mapping voor BSA/Cantus/Fakfeest is een gok, moet gevalideerd worden.
- Meer event-types toevoegen (TD, quiz, sport...) via admin-modus.
- Deploy: repo naar GitHub pushen, Settings → Pages → deploy from branch.

## Testen
Excel-exportlogica is geverifieerd in node met exceljs (gegroepeerde layout + kleuren + dropdown OK). App zelf testen door `index.html` in browser te openen.
