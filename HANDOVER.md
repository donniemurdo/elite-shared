# Elite Apps and Systems: handover note

Last updated: 23 October 2026.

Read this before starting any piece of work. At the end of each piece of work, update the Done, To do, Infrastructure and Open sections, add a line to the change log, and change the existing line rather than adding a competing one. Where this note and a repo disagree, the repo wins; fix the note.

## 1. Ownership

This project owns the code, the Cloudflare setup, the repos, tests and this note. For anything outside the code (Cloudflare settings, R2, Access, Teams, GitHub): do what the tools allow, otherwise give Greg exact steps or commands and track it in section 9 until Greg confirms it is done.

At the end of every piece of work Greg gets: the changes on GitHub (a PR per repo, no zip needed, Greg 2026-10-23), and this note brought up to date.

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
* Release: `./build.sh` (joins `app/` parts into `public/index.html`), bump the cache name in `public/sw.js` and `APP_VERSION` in `app/_p6.js` to the same number (both v13 as of 2026-10-23), run `npm test`, then `npx wrangler deploy`. Phones only update after the cache name changes. There is an in app update banner, and a sign out and update screen that requires a saved draft file first.
* Deploys can run from the project cloud environment (scoped Cloudflare token). Any deploy needs Greg's go ahead, and anything that might break the live app on site is flagged before shipping.

**Apps**
* Offline first: service worker, localStorage, photos in IndexedDB. The service worker skips `/api/`, `/beds/`, `/curtains` and `/admin`.
* Export: Excel only (ExcelJS), no CSV. The admin Excel download needs `exceljs.min.js` in `public/admin/`.

**Storage (R2 bucket `elite-surveys` in the EU jurisdiction, bound as `SURVEYS` with `"jurisdiction": "eu"`)**
* Any wrangler command on the bucket needs `--jurisdiction eu` (for example `npx wrangler r2 object get elite-surveys/measures/all.json --jurisdiction eu --remote --pipe`). Without it wrangler looks for a bucket that no longer exists.
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
* Survey to FileMaker data flow design agreed in principle: [docs/survey-to-filemaker.md](docs/survey-to-filemaker.md). Key: every measure comes from a FileMaker quote and is keyed by its Q- number (S- once ordered, same serial); FileMaker pushes quote and client data to the Worker so a Q- number fills in the client; FileMaker pulls finished surveys every 15 minutes via a server schedule under Greg's Claris ID (re-save yearly). No extra Claris licence needed. Jobs are created explicitly by the office (FileMaker checkbox or admin page, Measure or Fit only, one job per Q- number). Nothing built.
* Survey review and quoting recommendation: [docs/survey-review-and-quoting.md](docs/survey-review-and-quoting.md). Office reviews each item and clears warnings on a new admin review screen; only Checked surveys go to FileMaker; supplier, product and fabric are picked and priced in FileMaker on a new allocation card off the quote (not on the web page, because FileMaker owns the catalogue, codes and costing). Greg's answers recorded 2026-10-23 (Curtainwise costs fabric metres and making; that send lives in FileMaker). Nothing built; remaining decisions in section 11.
* This shared repo and note created.
* Tests fixed: `npm install`, then `./build.sh && npm test` runs all 7 files from the repo (Playwright for app and admin, Node for the Worker).
* Release v13 (soft furnishings PR #1, live 2026-10-23, version 547e2eaa): office notes (access and parking, notes for the measurer or fitter) shown in full in a Kiwi "Read before you start" box at the top of the job screen, plus a "Notes, read first" chip on job list cards; phone and pin icons on Call and Directions; admin New measure shows only measure fields and New fit only fit fields (Edit and Duplicate show all).

## 7. To do next (in this order unless Greg says otherwise)

1. Check release v13 on a phone (live 2026-10-23).
2. People list in admin: measurers, fitters and sales reps (reps sometimes measure), each with name, email and role. Measurer and fitter fields become dropdowns with a way to enter someone else. Greg alone edits the list and other settings as administrator; the office team adds and edits jobs (Greg, 2026-10-23).
3. External fitter: one outside fitter sees only jobs assigned to him; no full job list, no admin page, no other people's backups. Needs the Worker to allow named external emails and the Access policy changed (Greg makes the Access change; give him the steps). Greg's answers (2026-10-23): measurers see only their own jobs. Measurers are Scot Rigby (external) and David MacFarlane (Elite). Scot is also the one outside fitter. He has no Elite M365 account and signs in with the Cloudflare Access emailed PIN; only Scot may use the PIN. Adding him is deferred until near his first measure or fit.
4. Blind child safety:
   * Hanging height per blind (floor to top of blind).
   * Chain length to order = hanging height minus 1500 mm, rounded down to 10 mm. Chains are made to measure, so any length is allowed.
   * Warn if the chain would be under about 500 mm, or if hanging height is missing (Check screen and when marking measured).
   * Required safety device per chain operated blind: tensioner fixed to wall, breakaway, or chain break.
   * All of it in the Excel export.
5. Survey review: phone app critical choices start blank (agreed 2026-10-23; blind fit, drop reference, heading, lining are pre-filled today, so the office cannot tell chosen from untouched), shared warning rules file, survey locks after Measured; then the admin review screen with edits, warning clearance and Mark as checked. Steps A and B in [docs/survey-review-and-quoting.md](docs/survey-review-and-quoting.md) section 7. No FileMaker change.
6. Later: FileMaker format Excel export (Greg supplies the format), cloud backup for the bed survey, Curtainwise column template, brand fonts and logo, possibly an import of jobs from FileMaker (check the DDR first). Survey to FileMaker stage 0 (UUID survey ids, revision counter) fits here, see section 10.

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
| 1 | Move R2 bucket `elite-surveys` out of North America. The `--location weur` hint is ignored when a bucket name is reused, so the bucket was recreated in the EU jurisdiction (Greg's choice), same name. Cloudflare places it in EEUR; data is guaranteed to stay in the EU. 11 test objects deleted, old bucket deleted, Worker redeployed with the EU binding, write and read checked. | Claude | **Done 2026-10-23.** Greg to confirm one real cloud backup save from the app shows as saved. Greg to merge elite-softfurnishing-survey PR #2 (adds the EU binding). Live is v13 with the EU binding (version 547e2eaa) |
| 2 | Cloudflare Access on the measure app: Microsoft for staff, emailed PIN for Scot only. Until Scot is added: untick One time PIN so only Microsoft shows, and Instant authentication can be on. When Scot is added: re-tick One time PIN, add a policy that allows only Scot's email with login method One time PIN, require the Microsoft login method on the staff policy, and turn Instant authentication off (it only works with one login method). | Greg (steps from Claude) | Open |
| 3 | Teams: `TEAMS_WEBHOOK_URL` is already set as a Worker secret (seen by the 2026-10-23 deploy check). Still to do: live test with a real booking; a card should appear in Measure & Fit with the measurer mentioned. | Greg tests | Open, live test unconfirmed |
| 4 | Measurers move to the new address (save a draft file first, then re add the Home Screen icon). Bed app is at `/beds/`. Retire the old morning-queen-dc99 workers.dev address. | Greg and measurers | Open |
| 5 | Push both repos to GitHub (private). | Claude | **Done 2026-10-23** |
| 6 | Optional: Google Places key for address lookup (`npx wrangler secret put GOOGLE_MAPS_KEY`). | Greg | Open, optional |
| 7 | Measurers see only their own jobs (Greg, 2026-10-23; to build with to do item 3). Still unanswered: import jobs from FileMaker? | Greg | Open |

## 10. Known issues and gotchas

* Never deploy from a branch whose `wrangler.jsonc` lacks `"jurisdiction": "eu"` on SURVEYS: wrangler silently creates a new North America bucket instead (happened once on 2026-10-23).
* Tests: run `./build.sh && npm test` after every change. On a new machine run `npx playwright install chromium` once.
* `wrangler deploy` silently creates a missing R2 bucket in North America. If the `jurisdiction: eu` line is ever lost from `wrangler.jsonc`, the Worker binds to a new empty bucket and the job list looks empty.
* R2 location hints only work the first time a bucket name is ever created. To move a bucket again, use a new name or a jurisdiction.
* `fitOnly` is handled by its own line in `cleanMeasure` and is not in the `MFIELDS` list.
* Fit completion photos are job level, not per item.
* `APP_VERSION` and the service worker cache name are separate; keep them on the same number (v13 and `curtain-survey-v13` from release v13).
* Surveys started without a measure get a 4 character random id: not safe as a key in FileMaker. Needs a full UUID (integration stage 0). No revision counter yet.
* Photos upload inline as base64 in one request (up to 40 MB), which fails on poor signal. Split into one upload per photo.
* Bed survey has no cloud upload at all.
* Soft furnishings app pre-fills critical choices (blind fit "Inside the recess", drop "Track to floor", pencil pleat, lined). Defaults look like measured answers. The app also computes sizes only, no fabric metres.
* Brand mismatches in the soft furnishings app: 11 listed in [design/elite-design.md](design/elite-design.md) section 6. To fix against the design file.

## 11. Open decisions

**Survey to FileMaker** (details in [docs/survey-to-filemaker.md](docs/survey-to-filemaker.md) section 9): all decisions agreed 2026-10-23. Decision 1 changed the same day: only surveys checked and sent by the admin team go to FileMaker.

**Survey review and quoting** (details in [docs/survey-review-and-quoting.md](docs/survey-review-and-quoting.md) section 8). Agreed 2026-10-23: warnings reviewed and cleared on the web admin page only, Missing and Check kinds, each cleared on its own; admin team sends checked surveys to FileMaker with a button; grouping of quote lines done in FileMaker after fabric allocation; critical choices start blank on the phone; Curtainwise returns fabric metres and making cost, and the send to Curtainwise lives in FileMaker (fabric per curtain with roll width and pattern repeat); roller blinds made by Excel (excelgb.com, Blindata portal) and priced there by the admin team, costs typed into FileMaker; roman blinds made by Curtainwise; no bulk clear of warnings; office can edit measurements, every change tracked. Still open:
1. Ask Curtainwise for a CSV or Excel export of their quotation and fabric summary (today two PDFs, Cust Ref = Q- number, sizes in cm, per line net price and fabric metres, per fabric roll width and pattern repeat). Otherwise typed entry first.
2. What input format Curtainwise wants from us, and cm or mm.
3. Printable roller blind list from FileMaker for keying into the Excel portal? Recommended: yes.

**Design** (details in [design/elite-design.md](design/elite-design.md) section 8)
1. Buttons: darker olive #686D1E (passes contrast) or brand olive with larger bold labels?
2. Bed survey brand: Beds (retail) or CF (hotels)?
3. SVG logo files from the designer.

**EliteFM:** see [Docs/STATUS.md](https://github.com/donniemurdo/EliteFM/blob/main/Docs/STATUS.md) "Next session". As of 2026-10-09: finishing the `UI_Quote_Print` sliding test, then the quote send script. Survey import is not on the EliteFM roadmap; no FileMaker survey tables before Greg's go ahead and the Phase 1 gate.

## 12. Change log

| Date | What changed |
|---|---|
| 2026-10-23 | Survey review: Curtainwise example PDFs read (two per job, matched by Q- number and Location); no bulk warning clear; rollers by Excel (blind maker), romans by Curtainwise. |
| 2026-10-23 | Survey review: admin team sends checked surveys to FileMaker (survey design decision 1 changed), grouping in FileMaker, blinds priced via Excel by the admin team, office edits allowed and fully tracked. |
| 2026-10-23 | Survey review and quoting: Greg's answers recorded (web review only, each warning cleared on its own, grouped quote lines, blank defaults, Curtainwise returns metres and making cost, send to Curtainwise from FileMaker). |
| 2026-10-23 | Survey review and quoting recommendation written (docs/survey-review-and-quoting.md): review and warnings on the web admin page, supplier, fabric and pricing in FileMaker. Open decisions added to section 11. Nothing built. |
| 2026-10-23 | R2 bucket `elite-surveys` moved to the EU jurisdiction (location hint ignored for a reused name). Test data deleted. Worker redeployed with `jurisdiction: eu` binding. |
| 2026-10-23 | Soft furnishings: tests runnable from the repo; release v13 (notes stand out, call and directions icons, admin New measure / New fit fix) merged and live. A deploy during the bucket move briefly created a stray North America bucket; removed and redeployed with the EU binding.  |
| 2026-10-23 | Brand settled: the brand PDFs and the shared design file win over the old app colours and fonts. |
| 2026-10-23 | Survey apps pushed to GitHub. Cloud deploy access checked. Design file and survey to FileMaker design drafted. This note created, then merged with Greg's handover from the previous chat. |
