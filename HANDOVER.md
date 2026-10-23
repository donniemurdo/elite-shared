# Elite Apps and Systems: handover note

Last updated: 23 October 2026.

Read this before starting any piece of work. At the end of each piece of work, update the Done, To do, Infrastructure and Open sections, add a line to the change log, and change the existing line rather than adding a competing one. Where this note and a repo disagree, the repo wins; fix the note.

## 1. Ownership

This project owns the code, the Cloudflare setup, the repos, tests and this note. For anything outside the code (Cloudflare settings, R2, Access, Teams, GitHub): do what the tools allow, otherwise give Greg exact steps or commands and track it in section 9 until Greg confirms it is done.

At the end of every piece of work Greg gets: an updated zip of the changed app, and this note brought up to date.

## 2. About Greg and the goal

Greg is a director at Elite Contract Furniture, Glasgow (B2B contract furniture, family business since 1954). Goal: AI built apps for the business, made as and when needed, linked so information is entered once, with an excellent and consistent UI.

Target architecture: FileMaker is the central system and holds all information.

```
Survey apps (phones, offline first)  ->  FileMaker (sale and order)  ->  Xero (invoicing)
```

* Xero only ever receives data from FileMaker (EliteFM Roadmap Phases 3 and 4).
* Match companies by ID, never by name (CompanyID in FileMaker, XeroContactID for Xero).
* Every supplier and product has its own FileMaker code, so importing jobs from FileMaker is not simple. Check the DDR before proposing links.

## 3. Apps and repos

| App | Repo | Status |
|---|---|---|
| Soft furnishings survey (curtains, blinds, pelmets, voiles, hardware) at the root of measure.elitecontractfurniture.co.uk, office admin page at `/admin/`, the Worker | [donniemurdo/elite-softfurnishing-survey](https://github.com/donniemurdo/elite-softfurnishing-survey) (private; the zip folder is named elite-curtain-survey) | In build, focus |
| Bed survey at `/beds/` | [donniemurdo/elite-bed-survey](https://github.com/donniemurdo/elite-bed-survey) (private) | Backburner |
| EliteFM (FileMaker) | [donniemurdo/EliteFM](https://github.com/donniemurdo/EliteFM) (docs only, .fmp12 not in git) | In build, focus. Phase 1 (quote PDF and email) |
| Shared files (this repo) | [donniemurdo/elite-shared](https://github.com/donniemurdo/elite-shared) | Handover note, `design/`, `docs/` |

**EliteFM:** do not copy its rules here. Read its own files first:
[CLAUDE.md](https://github.com/donniemurdo/EliteFM/blob/main/CLAUDE.md),
[Docs/STATUS.md](https://github.com/donniemurdo/EliteFM/blob/main/Docs/STATUS.md),
[Docs/ROADMAP.md](https://github.com/donniemurdo/EliteFM/blob/main/Docs/ROADMAP.md),
[Standards/FileMakerStandards.md](https://github.com/donniemurdo/EliteFM/blob/main/Standards/FileMakerStandards.md).
Built by Greg with Claude Code and agentic-fm on his Mac (`/Users/gwins/Development/FileMaker/EliteFM`). Claude never edits the .fmp12 directly; Greg pastes and confirms. Hosted on FileMaker Cloud. Don't build ahead of the roadmap phase gates without Greg's OK.

**Priorities:** EliteFM and soft furnishings first. Bed survey on the backburner. Survey to FileMaker link: agree the design now, build later.

## 4. How the survey apps work

**Hosting and deploy**
* One Cloudflare Worker, `elite-surveys`, serves both apps. `run_worker_first` is on, so the Worker handles every request. workers.dev and preview URLs are off on purpose.
* The bed app is not deployed from its own repo: copy its files into `public/beds/` of the soft furnishings repo and deploy from there.
* Release: `./build.sh` (joins `app/` parts into `public/index.html`), bump the cache name in `public/sw.js` and `APP_VERSION` in `app/_p6.js` (they are separate), then `npx wrangler deploy`. Phones only update after the cache name changes. There is an in app update banner, and a sign out and update screen that requires a saved draft file first.
* Deploys can run from the project cloud environment (scoped Cloudflare token). Any deploy needs Greg's go ahead, and anything that might break the live app on site is flagged before shipping.

**Apps**
* Offline first: service worker, localStorage, photos in IndexedDB. The service worker skips `/api/`, `/beds/`, `/curtains` and `/admin`.
* Export: Excel only (ExcelJS), no CSV. The admin Excel download needs `exceljs.min.js` in `public/admin/`.

**Storage (R2 bucket `elite-surveys`, bound as `SURVEYS`)**
* Surveys: `curtains/<email slug>/<id>.json` plus `.prev.json` copies.
* `bymeasure/<id>.json` pointers so a fitter loads the latest survey for a job whoever saved it.
* Jobs: one object, `measures/all.json`, written with an etag check and retry; previous copy in `all.prev.json`. Fine for hundreds of jobs; move to D1 if it grows.

**Login**
* Cloudflare Access with Microsoft Entra (M365). The Worker also verifies the Access token itself and fails closed.
* `wrangler.jsonc`: `TEAM_DOMAIN`, `POLICY_AUD`, `ALLOWED_DOMAIN` (elitecontractfurniture.co.uk), `ADMINS` (greg), `OFFICE` (amen, betsy, greg).
* Office users add and edit jobs. Everyone signed in sees the job list and updates their own status. Staff already signed in to M365 get in without a prompt (expected).

**Jobs**
* One job can have a measure visit and a fit visit. Fit only jobs (`fitOnly = '1'`) have no measure visit; the client supplied sizes.
* Each job keeps a change history (who changed what).
* Fields: customer, site, postcode, ref, contact, phone, email, measure date and time, measurer, fit date and time, fitter, supplier, access and parking notes, notes for the measurer, notes for the fitter, status, fit status.

**Teams**
* A Teams Workflow ("Send webhook alerts to a chat") posts adaptive cards with an @mention of the measurer or fitter into the group chat Measure & Fit (4 people) when a measure or fit is booked or changed, or a job is cancelled.
* Worker secret `TEAMS_WEBHOOK_URL`. The link is the secret: never print it. The admin form has a tick box to skip the message per save.

**Address lookup (admin)**
* Free OpenStreetMap by default: postcode gives streets, the user types the house number. Cannot list house numbers or find many businesses.
* Optional `GOOGLE_MAPS_KEY` secret switches to Google Places (better results, small cost).

## 5. Design and brand

* The brand PDFs are the source (Greg, 2026-10-23). All colours, fonts, buttons, forms and logos come from the shared design file: [design/elite-design.md](design/elite-design.md), tokens in `design/elite-brand.css`, logos in `design/logos/`. Do not invent styles; add anything missing to the design file first.
* Elite CF for all internal apps; Beds and FF&E styling only on what those customers see. Each app keeps its own copy of the CSS and fonts so it works offline.
* Light theme only.
* The apps still use older colours and fonts; bringing them in line is section 10.

## 6. Done

**Soft furnishings survey and admin (previous chat)**
* Full UI redesign with hub tiles and accordions.
* Jobs start screen with search and filters (today panel, mine, type, date).
* Office admin page: job form, duplicate, history, Excel download, New measure and New fit buttons, filters, address lookup.
* Fit visits show the survey the measurer entered. Fitter notes at job level only.
* Track or pole width as the default measurement with optional window width; rail suggestion inside the Window and wall section.
* Window type help with a casement default.
* Fit completion screen (tick each item, snag notes, job photos, fit notes, customer signature) feeding the Excel job sheet.
* Checks before Mark as measured and Mark as fitted. Size sanity warnings.
* Visible backup status on the job screen. Call and Directions buttons. Update banner. Access and parking notes. Change history. Teams notifications. Security assessment.

**This project (2026-10-23)**
* Both survey repos pushed to GitHub, private (infra item 5).
* Cloud environment set up with a scoped Cloudflare token (expires 2027-10-23); read access to the Worker, secrets, R2 and the zone verified. Write access not proven until the first real deploy.
* Shared design file drafted from the brand PDFs.
* Survey to FileMaker data flow design drafted: [docs/survey-to-filemaker.md](docs/survey-to-filemaker.md).
* This shared repo and note created.

## 7. To do next (in this order unless Greg says otherwise)

1. Make the admin notes (access and parking, notes for the fitter or measurer) stand out much more on the job screen.
2. Add icons to the Call and Directions buttons.
3. Admin form bug: New measure also shows the fitting fields. New measure should show only measure fields, New fit only fit fields.
4. People list in admin: measurers, fitters and sales reps (reps sometimes measure), each with name, email and role. Measurer and fitter fields become dropdowns with a way to enter someone else. **Ask Greg:** who can edit the list (just Greg, or the office team too)?
5. External fitter: one outside fitter sees only jobs assigned to him; no full job list, no admin page, no other people's backups. Needs the Worker to allow named external emails and the Access policy changed (Greg makes the Access change; give him the steps). **Ask Greg:** does he have a Microsoft account, or should he use a one time PIN?
6. Blind child safety:
   * Hanging height per blind (floor to top of blind).
   * Chain length to order = hanging height minus 1500 mm, rounded down to 10 mm. Chains are made to measure, so any length is allowed.
   * Warn if the chain would be under about 500 mm, or if hanging height is missing (Check screen and when marking measured).
   * Required safety device per chain operated blind: tensioner fixed to wall, breakaway, or chain break.
   * All of it in the Excel export.
7. Later: FileMaker format Excel export (Greg supplies the format), cloud backup for the bed survey, Curtainwise column template, brand fonts and logo, possibly an import of jobs from FileMaker (check the DDR first). Survey to FileMaker stage 0 (UUID survey ids, revision counter) fits here, see section 10.

## 8. Rules

* Never ask Greg to paste secrets, passwords, keys or tokens into chat. Secrets are set with `npx wrangler secret put`. Never print secret values in notes, logs or commits.
* The Entra client secret must never be repeated anywhere; rotate it if it ever appears.
* Keep all repos private.
* Cloudflare Access and security settings are Greg's to change: give him the steps.
* Never delete data, buckets or files without asking first and saying exactly what will be lost.
* Say before shipping if something might break the live app measurers use on site.
* If a request could mean two things, ask one short question.
* Check the repos and this note before building. Reuse the existing login, storage, admin and Teams setup, and keep every app consistent with the shared design file.
* Style: short and structured, no flattery, say plainly when something is wrong. Emails and documents for Greg: avoid hyphens where possible, no sign offs.

## 9. Infrastructure actions (tracked until Greg confirms)

| # | Action | Who | Status |
|---|---|---|---|
| 1 | Move R2 bucket `elite-surveys` from North America to Western Europe. Location cannot be changed: empty, delete, then `npx wrangler r2 bucket create elite-surveys --location weur`. Worker binds by name, so only a redeploy is needed. Contents are test data, but list exactly what will be lost and get Greg's OK before deleting. | Claude, after Greg's OK | Open |
| 2 | Cloudflare Access on the measure app: untick One time PIN / Cloudflare login so only Microsoft shows, and turn on Instant authentication. | Greg (steps from Claude) | Open |
| 3 | Teams: `TEAMS_WEBHOOK_URL` is already set as a Worker secret (seen by the 2026-10-23 deploy check). Still to do: live test with a real booking; a card should appear in Measure & Fit with the measurer mentioned. | Greg tests | Open, live test unconfirmed |
| 4 | Measurers move to the new address (save a draft file first, then re add the Home Screen icon). Bed app is at `/beds/`. Retire the old morning-queen-dc99 workers.dev address. | Greg and measurers | Open |
| 5 | Push both repos to GitHub (private). | Claude | **Done 2026-10-23** |
| 6 | Optional: Google Places key for address lookup (`npx wrangler secret put GOOGLE_MAPS_KEY`). | Greg | Open, optional |
| 7 | Still unanswered: should measurers see all jobs (now everyone does, with a Mine filter)? Import jobs from FileMaker? | Greg | Open |

## 10. Known issues and gotchas

* Tests in `tests/` (Playwright for the app and admin, plain Node for the Worker with mocked R2 and Access tokens) have hardcoded paths from the old scratch environment (for example `/home/claude/elite-surveys/src/worker.js`). Fix the paths, then run all of them after every change.
* `fitOnly` is handled by its own line in `cleanMeasure` and is not in the `MFIELDS` list.
* Fit completion photos are job level, not per item.
* `APP_VERSION` and the service worker cache name are separate; update both on release. At push time they were out of step (`v11` and `curtain-survey-v12`).
* Surveys started without a measure get a 4 character random id: not safe as a key in FileMaker. Needs a full UUID (integration stage 0). No revision counter yet.
* Photos upload inline as base64 in one request (up to 40 MB), which fails on poor signal. Split into one upload per photo.
* Bed survey has no cloud upload at all.
* Brand mismatches in the soft furnishings app: 11 listed in [design/elite-design.md](design/elite-design.md) section 6. To fix against the design file.

## 11. Open decisions

**Survey to FileMaker** (details in [docs/survey-to-filemaker.md](docs/survey-to-filemaker.md) section 9). No spare Claris licence, and custom OAuth does not avoid one, so the draft design has FileMaker pull from a secured Worker feed on a server schedule. Not approved.
1. Import only finished (measured) surveys, or every draft save? Recommended: finished only.
2. Must every survey start from a measure? Recommended: yes.
3. Who owns jobs long term: FileMaker or the admin page? Recommended: FileMaker.
4. Photos: keep in R2 and view by link, or copy into FileMaker containers? Recommended: R2.
5. Is a schedule tied to Greg's Claris ID, re-saved yearly, acceptable for now?
6. Pull frequency: every 15 minutes in working hours, or less?

**Design** (details in [design/elite-design.md](design/elite-design.md) section 8)
1. Buttons: darker olive #686D1E (passes contrast) or brand olive with larger bold labels?
2. Bed survey brand: Beds (retail) or CF (hotels)?
3. SVG logo files from the designer.

**EliteFM:** see [Docs/STATUS.md](https://github.com/donniemurdo/EliteFM/blob/main/Docs/STATUS.md) "Next session". As of 2026-10-09: finishing the `UI_Quote_Print` sliding test, then the quote send script. Survey import is not on the EliteFM roadmap; no FileMaker survey tables before Greg's go ahead and the Phase 1 gate.

## 12. Change log

| Date | What changed |
|---|---|
| 2026-10-23 | Brand settled: the brand PDFs and the shared design file win over the old app colours and fonts. |
| 2026-10-23 | Survey apps pushed to GitHub. Cloud deploy access checked. Design file and survey to FileMaker design drafted. This note created, then merged with Greg's handover from the previous chat. |
