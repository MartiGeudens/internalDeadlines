# Industria Internal Deadline Planner

## Context
Tool voor studentenvereniging **Industria** (KU Leuven Campus Groep T, ~16 praesidiumfuncties, 100+ activiteiten/jaar). Probleem: interne deadlines (aanvragen X weken vóór een event) worden slecht opgevolgd. Gebruiker: Marti Geudens (Schachtenmeester).

## Wat het doet
Statische single-page app voor **GitHub Pages** (geen backend). Flow:
1. Selecteer functie → 2. Voeg activiteiten toe (naam, hoofdverantwoordelijke, event-type, datum) → 3. Deadlines worden automatisch berekend (exact X weken vóór event, bewust géén weekend-correctie — wel visuele waarschuwing) → 4. Export.

## Bestanden
- `index.html` — volledige app (UI + logica + embedded fallback-config). ExcelJS via cdnjs.
- `config.json` — deadline-types (9 stuks, met defaultWeeks) + event-types (BSA, Cantus, Fakfeest) + rollenlijst. Wordt via fetch geladen; embedded DEFAULT_CONFIG in index.html is de fallback voor file:// gebruik. **Bij wijziging: beide synchroon houden.**
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
Gegroepeerd per activiteit. Headerrij per activiteit (bold, groene bovenrand): `Activiteitnaam | Hoofdverantwoordelijke | Datum activiteit | "Status"`. Daaronder per deadline: `Deadlinenaam | Verantwoordelijke | Datum deadline | Status`. Status heeft dropdown (To Do/Busy/Done) + kleuren: Done=groen (C6EFCE), Busy=geel (FFEB9C), To Do=rood (FFC7CE), plus conditional formatting zodat kleuren mee veranderen in Excel zelf.

## Deadline-types (defaults zijn voorlopige schattingen!)
Communication request (4w), Material request (4w), Material order/new purchase (6w), Drink request (2w), Food request (2w), Industria van request (3w), Province request (8w), Public domain occupation (8w), Room request (6w).
**TODO: echte weken-waarden en per-event-type mapping laten valideren door Marti/praesidium.**

## Open punten
- Event-type ↔ deadline-mapping voor BSA/Cantus/Fakfeest is een gok, moet gevalideerd worden.
- Meer event-types toevoegen (TD, quiz, sport...) via admin-modus.
- Deploy: repo naar GitHub pushen, Settings → Pages → deploy from branch.

## Testen
Excel-exportlogica is geverifieerd in node met exceljs (gegroepeerde layout + kleuren + dropdown OK). App zelf testen door `index.html` in browser te openen.
