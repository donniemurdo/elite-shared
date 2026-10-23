# Elite Apps and Systems: handover note

Last updated: 23 October 2026.

Read this before starting any piece of work. At the end of each piece of work, update the Done, To do, Infrastructure and Open sections, add a line to the change log, and change the existing line rather than adding a competing one. Where this note and a repo disagree, the repo wins; fix the note.

## How we work

This is the working method unless a better way comes up (Greg, 2026-10-23).

* **GitHub is the link** between this project (claude.ai) and Claude Code in VS Code on Greg's Mac. There is no live sync between them.
* **Pull before, push after.** Every session pulls the repos before starting and pushes when done, so both sides see the same code.
* **This note is the shared memory.** Claude Code on the Mac does not see the project chat or its memory. Each repo's `CLAUDE.md` points here and to `design/`.
* **Where work happens:** FileMaker work in VS Code with agentic-fm on Greg's Mac. App and Cloudflare work in this project.
* **Remote Control:** a project thread can run Claude Code on Greg's Mac by Remote Control, with Greg's approval, when a task needs the Mac.
* **EliteFM:** its `CLAUDE.md` is left as it is because agentic-fm relies on it, so it does not point here. When FileMaker work affects the survey apps or this note, update this note as well.

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
* Release: `./build.sh` (joins `app/` parts into `public/index.html`), bump the cache name in `public/sw.js` and `APP_VERSION` in `app/_p6.js` to the same number (both v17 as of 2026-10-23), run `npm test`, then `npx wrangler deploy`. Phones only update after the cache name changes. There is an in app update banner, and a sign out and update screen that requires a saved draft file first.
* Deploys can run from the project cloud environment (scoped Cloudflare token). Any deploy needs Greg's go ahead, and anything that might break the live app on site is flagged before shipping.

**Apps**
* Offline first: service worker, localStorage, photos in IndexedDB. The service worker skips `/api/`, `/beds/`, `/curtains` and `/admin`.
* Export: Excel only (ExcelJS), no CSV. The admin Excel download needs `exceljs.min.js` in `public/admin/`.

**Storage (R2 bucket `elite-surveys` in the EU jurisdiction, bound as `SURVEYS` with `"jurisdiction": "eu"`)**
* Any wrangler command on the bucket needs `--jurisdiction eu` (for example `npx wrangler r2 object get elite-surveys/measures/all.json --jurisdiction eu --remote --pipe`). Without it wrangler looks for a bucket that no longer exists.
* Surveys: `curtains/<email slug>/<id>.json` plus `.prev.json` copies.
* `bymeasure/<id>.json` pointers so a fitter loads the latest survey for a job whoever saved it.
* Jobs: one object, `measures/all.json`, written with an etag check and retry; previous copy in `all.prev.json`. Fine for hundreds of jobs; move to D1 if it grows.
* People list (from PR #3): `people/all.json`, previous copy in `people/all.prev.json`.

**Login**
* Cloudflare Access with Microsoft Entra (M365). The Worker also verifies the Access token itself and fails closed.
* `wrangler.jsonc`: `TEAM_DOMAIN`, `POLICY_AUD` (one or more Access app audience tags, comma separated), `ALLOWED_DOMAIN` (elitecontractfurniture.co.uk), `ADMINS` (greg), `OFFICE` (amen, betsy, greg).
* Staff already signed in to M365 get in without a prompt (expected).
* Live since v14 (PR #3 carried in PR #4): `ADMINS` and `OFFICE` see every job and use `/admin/`. Everyone else sees only jobs where they are measurer or fitter (no change history), can only change status on those, can only open the survey for those, and is sent away from `/admin/`. Only `ADMINS` edits the people list at `/admin/people.html`; office reads it for the job form dropdowns. Someone outside the company domain gets in only while Active in the people list (and only if Access lets them in); they cannot open `/beds/` or bed backups.

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
* Soft furnishings app and admin pages use the brand since v14 (Poppins and Lato bundled, navy bar, olive #686D1E main buttons, logos). New components there: top bar with menu sheet (sign out in its own last section), job card with step list, segmented tap buttons for lists of 4 or fewer, "choose one" flag on blank critical fields. The bed survey still uses the old styling.

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
* People list, own jobs only and external user support (soft furnishings [PR #3](https://github.com/donniemurdo/elite-softfurnishing-survey/pull/3), live in v14). Measurer and fitter fields are dropdowns from the list, sales reps appear as measurers, "Someone else..." for a one off.
* Release v13 (soft furnishings PR #1, live 2026-10-23, version 547e2eaa): office notes (access and parking, notes for the measurer or fitter) shown in full in a Kiwi "Read before you start" box at the top of the job screen, plus a "Notes, read first" chip on job list cards; phone and pin icons on Call and Directions; admin New measure shows only measure fields and New fit only fit fields (Edit and Duplicate show all).
* Release v14 (soft furnishings [PR #4](https://github.com/donniemurdo/elite-softfurnishing-survey/pull/4), live 2026-10-23, version c169667a, includes PR #3): app style redesign per the UI plan (top bar, menu top right with sign out in its own section that saves first, new job home with job card and steps, tap buttons for short choices, brand fonts, colours and logos, admin side navigation); heading, lining, drop reference and blind fit start blank and are flagged; fabric allocation removed from the app (FileMaker does it), a "fabric chosen by the customer" note kept; all sizes in cm with one decimal, mm surveys converted once on load (phone, cloud, draft file), Excel headings in cm.
* Release v15 ([PR #5](https://github.com/donniemurdo/elite-softfurnishing-survey/pull/5), live 2026-10-23, version ff4b203e): fixed users being stuck after the 24h Access session expired (service worker served the cached page so the login never appeared; pages now network first with offline fallback, `/signin` route and Sign in button). Office only Fabrics and Allocate fabrics screens restored as an interim until the new FileMaker allocates fabric (confirmed by Greg 2026-10-23). The app needs no FileMaker link: Q- ref optional, jobs entered by hand.
* Release v16 ([PR #6](https://github.com/donniemurdo/elite-softfurnishing-survey/pull/6), live 2026-10-23, version c6ebff7d): Supplier data Excel and CSV leave out client details and all notes, name the file by job ref, and add a Location column (room and window) per item; it is the file the office sends Curtainwise until the new FileMaker produces it. New test `tests/tsup.mjs`.
* Release v17 ([PR #7](https://github.com/donniemurdo/elite-softfurnishing-survey/pull/7), live 2026-10-23, version dcdd5935): People page email box no longer loses focus after the first character; the External label shows only for a complete outside email.

## 7. To do next (in this order unless Greg says otherwise)

1. Greg reviews v14 on a phone (live 2026-10-23); testing starts Monday 2026-10-26. Backup recheck after v14 handed to the R2 thread.
2. People list: live in v14. Greg opens `/admin/people.html` and adds David MacFarlane (measurer) and any fitters and reps. Until someone is in the list, the office uses "Someone else..." on the job form. Measurers now see only their own jobs.
3. Scot Rigby (external measurer and fitter, emailed PIN, deferred until near his first job): Greg adds him on the People page, then sets up Access as in section 9 item 2. The Worker side is already in PR #3.
4. Blind child safety:
   * Hanging height per blind (floor to top of blind).
   * Chain length to order = hanging height minus 150 cm, rounded down to 1 cm. Chains are made to measure, so any length is allowed.
   * Warn if the chain would be under about 50 cm, or if hanging height is missing (Check screen and when marking measured).
   * Required safety device per chain operated blind: tensioner fixed to wall, breakaway, or chain break.
   * All of it in the Excel export.
   * Excel (the blind maker) already takes Install Height and calculates Control Length: hanging height feeds Install Height, and our chain length becomes a check against theirs.
5. Survey review: critical choices start blank (done in v14); next a shared warning rules file, survey locks after Measured; then the admin review screen with edits, warning clearance and Mark as checked. Steps A and B in [docs/survey-review-and-quoting.md](docs/survey-review-and-quoting.md) section 7. No FileMaker change.
6. Later: FileMaker format Excel export (Greg supplies the format), cloud backup for the bed survey, Curtainwise column template, possibly an import of jobs from FileMaker (check the DDR first). Survey to FileMaker stage 0 (UUID survey ids, revision counter) fits here, see section 10.

## 8. Rules

* Never ask Greg to paste secrets, passwords, keys or tokens into chat. Secrets are set with `npx wrangler secret put`. Never print secret values in notes, logs or commits.
* The Entra client secret must never be repeated anywhere; rotate it if it ever appears.
* Keep all repos private.
* Cloudflare Access: the cloud token has Access Apps and Policies Edit (Greg, 2026-10-23). Claude changes Access only when Greg asks, and reports exactly what changed. The measure app is a zone level Access app (`/zones/<zone>/access/apps`; the account level list shows nothing). Other security settings stay Greg's; give him steps.
* Never delete data, buckets or files without asking first and saying exactly what will be lost.
* Say before shipping if something might break the live app measurers use on site.
* If a request could mean two things, ask one short question.
* Check the repos and this note before building. Reuse the existing login, storage, admin and Teams setup, and keep every app consistent with the shared design file.
* Style: short and structured, no flattery, say plainly when something is wrong. Emails and documents for Greg: avoid hyphens where possible, no sign offs.

## 9. Infrastructure actions (tracked until Greg confirms)

| # | Action | Who | Status |
|---|---|---|---|
| 1 | Move R2 bucket `elite-surveys` out of North America. The `--location weur` hint is ignored when a bucket name is reused, so the bucket was recreated in the EU jurisdiction (Greg's choice), same name. Cloudflare places it in EEUR; data is guaranteed to stay in the EU. 11 test objects deleted, old bucket deleted, Worker redeployed with the EU binding, write and read checked. | Claude | **Done and confirmed 2026-10-23.** Real cloud backups from Greg's phone landed in the EU bucket on v13 (10:46 UK) and on v15 (11:09 UK, cm format). |
| 2 | Cloudflare Access: Microsoft for staff, emailed PIN for Scot only. **Staff part done 2026-10-23:** the measure app (zone level Access app) allows one login method (Microsoft) with Instant Auth on; Greg confirmed a private window goes straight to Microsoft. **When Scot is added, two options (Greg to choose; recommended A):** A) separate address `fit.elitecontractfurniture.co.uk` on the same Worker with its own Access app, One time PIN only, policy allows only Scot's email; Claude can create it (token has Access edit) and adds the route and its audience tag to `POLICY_AUD`. Staff app untouched. B) same address: re-tick One time PIN, add a policy allowing only Scot's email with login method One time PIN, require Microsoft on the staff policy, turn Instant Auth off; staff then see a two button login screen. | Claude when Greg asks | Staff part **Done 2026-10-23**. Scot part open: A or B not chosen, deferred until near his first job |
| 3 | Teams: `TEAMS_WEBHOOK_URL` is already set as a Worker secret (seen by the 2026-10-23 deploy check). Still to do: live test with a real booking; a card should appear in Measure & Fit with the measurer mentioned. | Greg tests | Open, live test unconfirmed |
| 4 | Measurers move to the new address (save a draft file first, then re add the Home Screen icon). Bed app is at `/beds/`. Retire the old morning-queen-dc99 workers.dev address. | Greg and measurers | **Done 2026-10-23.** David uses the new address; Greg deleted the morning-queen-dc99 Worker (checked: only elite-surveys remains, its workers.dev and preview URLs are off). Leftover: an unused Access app "morning-queen-dc99 - Cloudflare Workers", safe to delete |
| 5 | Push both repos to GitHub (private). | Claude | **Done 2026-10-23** |
| 6 | Optional: Google Places key for address lookup (`npx wrangler secret put GOOGLE_MAPS_KEY`). | Greg | Open, optional |
| 7 | Measurers see only their own jobs (Greg, 2026-10-23): built in PR #3, live with the combined UI release. Still unanswered: import jobs from FileMaker? | Greg | Open |

## 10. Known issues and gotchas

* Never deploy from a branch whose `wrangler.jsonc` lacks `"jurisdiction": "eu"` on SURVEYS: wrangler silently creates a new North America bucket instead (happened once on 2026-10-23).
* Tests: run `./build.sh && npm test` after every change. On a new machine run `npx playwright install chromium` once.
* `wrangler deploy` silently creates a missing R2 bucket in North America. If the `jurisdiction: eu` line is ever lost from `wrangler.jsonc`, the Worker binds to a new empty bucket and the job list looks empty.
* Expired Access session (24h) leaves users stuck: `sw.js` serves navigations cache-first, so the Access login never shows and the app says "Sign in again with signal". Workaround: open `/admin/` (the service worker skips it, so Access prompts). Fix handed to the release thread (2026-10-23): a `/signin` route and button, or network-first navigations.
* R2 location hints only work the first time a bucket name is ever created. To move a bucket again, use a new name or a jurisdiction.
* `fitOnly` is handled by its own line in `cleanMeasure` and is not in the `MFIELDS` list.
* Fit completion photos are job level, not per item.
* `APP_VERSION` and the service worker cache name are separate; keep them on the same number (v17 and `curtain-survey-v17` from release v17).
* Surveys started without a measure get a 4 character random id: not safe as a key in FileMaker. Needs a full UUID (integration stage 0). No revision counter yet.
* Photos upload inline as base64 in one request (up to 40 MB), which fails on poor signal. Split into one upload per photo.
* Bed survey has no cloud upload at all.
* Soft furnishings app works in mm; Greg wants cm everywhere (one decimal), to change with the review work.
* Soft furnishings app pre-fills critical choices (blind fit "Inside the recess", drop "Track to floor", pencil pleat, lined). Defaults look like measured answers. The app also computes sizes only, no fabric metres.
* Brand mismatches in the soft furnishings app: 11 listed in [design/elite-design.md](design/elite-design.md) section 6. To fix against the design file.

## 11. Open decisions

**Survey to FileMaker** (details in [docs/survey-to-filemaker.md](docs/survey-to-filemaker.md) section 9): all decisions agreed 2026-10-23. Decision 1 changed the same day: only surveys checked and sent by the admin team go to FileMaker.

**Survey review and quoting** (details in [docs/survey-review-and-quoting.md](docs/survey-review-and-quoting.md) section 8). Agreed 2026-10-23: warnings reviewed and cleared on the web admin page only, Missing and Check kinds, each cleared on its own; admin team sends checked surveys to FileMaker with a button; grouping of quote lines done in FileMaker after fabric allocation; critical choices start blank on the phone; Curtainwise returns fabric metres and making cost, and the send to Curtainwise lives in FileMaker (fabric per curtain with roll width and pattern repeat); roller blinds made by Excel (excelgb.com, Blindata portal) and priced there by the admin team, costs typed into FileMaker; roman blinds made by Curtainwise; no bulk clear of warnings; office can edit measurements, every change tracked. Still open:
Also agreed: everything in cm (one decimal allowed), and Curtainwise CSV or Excel exports are imported into FileMaker, matched by Q- number and Location, confirmed by the office.
1. What input format Curtainwise wants from us; a sample of their CSV or Excel export for the import mapping.
2. Excel (rollers): keyed by hand, sizes in mm, Q- number in Customer Reference. Measure To: Recess Size for inside the recess (Excel deducts, send recess as measured, app roller deduction stays 0), Blind Size for outside (send our order size); Cloth Size unused. Excel calculates control length from install height. Recommended: printable roller list from FileMaker.
3. `FabricDesignRanges` has no roll width or pattern repeat (Greg, 2026-10-23). To add in EliteFM: `RollWidth` and `PatternRepeat`, Number, cm, on `UI_Fabric_Edit_Card`. On the EliteFM roadmap, not gated.

**Design:** nothing open. Bed survey uses Elite CF; no SVG logos exist, the PNGs in `design/logos/` are the masters ([design/elite-design.md](design/elite-design.md) section 8).

**EliteFM:** see [Docs/STATUS.md](https://github.com/donniemurdo/EliteFM/blob/main/Docs/STATUS.md) "Next session". As of 2026-10-09: finishing the `UI_Quote_Print` sliding test, then the quote send script. Survey import is not on the EliteFM roadmap; no FileMaker survey tables before Greg's go ahead and the Phase 1 gate.

## 12. Change log

| Date | What changed |
|---|---|
| 2026-10-23 | "How we work" section added (GitHub is the link, pull before and push after, FileMaker in VS Code, apps and Cloudflare in the project). `CLAUDE.md` pointers added to elite-shared, elite-softfurnishing-survey and elite-bed-survey. EliteFM `CLAUDE.md` unchanged. |
| 2026-10-23 | Old morning-queen-dc99 Worker deleted by Greg and checked (infra item 4 done). |
| 2026-10-23 | Soft furnishings v17 (PR #7) live, version dcdd5935: People page email box fix. |
| 2026-10-23 | Design decisions closed: bed survey brand is Elite CF, no SVG logos (PNGs are masters), olive #686D1E confirmed. |
| 2026-10-23 | `FabricDesignRanges` confirmed to have no roll width or pattern repeat; two fields specified and added to the EliteFM roadmap. |
| 2026-10-23 | Soft furnishings v16 (PR #6) live, version c6ebff7d: supplier Excel is the Curtainwise file for now, with no client details or notes and a Location per item. Excel Measure To options recorded. |
| 2026-10-23 | Soft furnishings v15 live (PR #5): expired sign in fix, office fabric screens back as an interim. |
| 2026-10-23 | Excel roller form read: mm, own recess deductions, install height and control length, options priced per blind. |
| 2026-10-23 | Survey designs: FileMaker link is an add-on behind one Worker setting; until the new EliteFM is built the app and admin work as now with hand typed jobs and client details (Greg). |
| 2026-10-23 | Excel blind portal read: hand keyed, no upload; fields recorded in docs/survey-review-and-quoting.md section 8b. |
| 2026-10-23 | Soft furnishings v14 live (PR #4, includes PR #3): UI redesign with menu and separate sign out, brand applied, blank critical choices, fabric allocation left to FileMaker, all sizes in cm (mm surveys converted on load). Olive #686D1E buttons approved by Greg. |
| 2026-10-23 | Access: staff login confirmed Microsoft only with Instant Auth (infra item 2 staff part done). Token given Access Apps and Policies Edit; Claude changes Access only when Greg asks. |
| 2026-10-23 | Soft furnishings PR #3: people list (Greg edits), measurers and fitters see only their own jobs, external email PIN users allowed only while in the list, admin dropdowns. Not deployed. Access item 2 split into "Microsoft only now" and the Scot options. |
| 2026-10-23 | Survey review: everything in cm; Curtainwise CSV or Excel export imported into FileMaker. |
| 2026-10-23 | Survey review: Curtainwise example PDFs read (two per job, matched by Q- number and Location); no bulk warning clear; rollers by Excel (blind maker), romans by Curtainwise. |
| 2026-10-23 | Survey review: admin team sends checked surveys to FileMaker (survey design decision 1 changed), grouping in FileMaker, blinds priced via Excel by the admin team, office edits allowed and fully tracked. |
| 2026-10-23 | Survey review and quoting: Greg's answers recorded (web review only, each warning cleared on its own, grouped quote lines, blank defaults, Curtainwise returns metres and making cost, send to Curtainwise from FileMaker). |
| 2026-10-23 | Survey review and quoting recommendation written (docs/survey-review-and-quoting.md): review and warnings on the web admin page, supplier, fabric and pricing in FileMaker. Open decisions added to section 11. Nothing built. |
| 2026-10-23 | R2 bucket `elite-surveys` moved to the EU jurisdiction (location hint ignored for a reused name). Test data deleted. Worker redeployed with `jurisdiction: eu` binding. |
| 2026-10-23 | Soft furnishings: tests runnable from the repo; release v13 (notes stand out, call and directions icons, admin New measure / New fit fix) merged and live. A deploy during the bucket move briefly created a stray North America bucket; removed and redeployed with the EU binding.  |
| 2026-10-23 | Brand settled: the brand PDFs and the shared design file win over the old app colours and fonts. |
| 2026-10-23 | Survey apps pushed to GitHub. Cloud deploy access checked. Design file and survey to FileMaker design drafted. This note created, then merged with Greg's handover from the previous chat. |
