# Survey to FileMaker data flow: design

Status: design agreed in principle, 2026-10-23 (decisions in section 9). Nothing built. Soft furnishings survey first; bed survey on the backburner and follows the same pattern later.

## 1. Summary

- Phones keep saving to the Worker as now. The Worker becomes a holding area and exposes a small, locked down feed.
- Every job starts as a FileMaker quote. Its number `Q-000123` is the key everywhere; when ordered it becomes `S-000123` (same serial). FileMaker pushes quote and client details to the Worker, so the office (or app) enters a Q- number and the client data fills in.
- FileMaker Cloud pulls finished surveys from the feed on a server schedule using Insert from URL, attaches them to the quote by its number, then confirms receipt.
- No extra Claris licence is needed: the schedule runs as Greg's existing Claris ID.
- Xero only ever receives data from FileMaker (Roadmap Phase 3 and 4, unchanged).
- Both apps are unfinished, so the recommendation is **agree the shape now, build later**, and make a few cheap choices in the soft furnishings app now so it is ready (section 8, stage 0).

```
Phone (offline first) ──PUT──▶ Worker + R2 (holding area, feed)
                                   ▲            │
               Insert from URL     │            │ GET feed / POST ack
               (server schedule)   └────────────┘
                                   FileMaker Cloud ──▶ Xero (Phase 3/4)
```

## 2. Why pull, not push (Claris docs checked 2026-10-23)

| Point | Status | Source |
|---|---|---|
| Data API, OData and Admin API on FileMaker Cloud must log in with a Claris ID, "not an external IdP account" | Confirmed | [Using Claris ID for external authentication](https://help.claris.com/en/customer-console-help/content/create-fmid-token.html) |
| Each user needs a unique, individually licensed Claris ID; accounts cannot be shared | Confirmed | [FileMaker Cloud FAQ](https://support.claris.com/s/article/Claris-FileMaker-Cloud-FAQ?language=en_US) |
| Custom OAuth / external IdP (Entra, Okta etc.) only changes how a licensed person signs in. It does not give a seat free account | Confirmed by the two points above | [External IdP](https://help.claris.com/en/customer-console-help/content/authenticate-with-external.html) |
| Data API tokens last one hour; login is a Cognito flow, with MFA if enabled. Awkward for a Worker | Confirmed | [Data API login](https://help.claris.com/en/data-api-guide/content/log-in-database-session-claris-id.html) |
| Insert from URL runs in server side scripts on FileMaker Cloud | Confirmed | [Insert from URL](https://help.claris.com/en/pro-help/content/insert-from-url.html) |
| Script schedules run as the schedule creator's Claris ID, which must be in the file's security settings | Confirmed | [Script schedules](https://help.claris.com/en/cloud-help/content/create-script-schedules.html) |
| Schedules expire after one year and must be re-saved; schedules saved by an inactive Claris ID stop | Confirmed | same page |
| Schedules can repeat every N minutes within a time range; no minimum interval stated | Assumed 15 min works, test it | same page |
| A schedule does not use an extra seat | Assumed (runs as Greg's existing licensed ID; docs do not say either way) | |

Answer to the custom OAuth question: no, it would not give the Worker access without a licence.

## 3. What the soft furnishings app has now (from the code)

- Survey structure: job details (customer, contact, site, ref, measurer, job type, supplier, notes), a fabrics list (name, supplier, code, colour, roll width, repeat, FR), rooms, windows with measurements, and per window treatments: curtain, voile, blind types, perfect fit, pelmet, hardware. Plus a fit section (notes, photos, signature).
- Saved locally on the phone, then uploaded whole to `PUT /api/backup/curtains/<id>` when there is signal. Photos are inside the same upload as base64, up to 40 MB.
- Surveys opened from a measure use the measure id (stable, unique). Surveys started without a measure get a **4 character random id**: not safe as a key in FileMaker.
- Measures (jobs) are one JSON file in R2, edited in /admin/. Customer and site are free text; nothing links to a FileMaker company.
- "Finished" signals exist: measure status `measured`, fit status `fitted`.
- Every Worker request needs a Cloudflare Access login with a company email. A machine caller has no email, so FileMaker needs its own path.

## 4. Auth between Worker and FileMaker

- Cloudflare Access **service token** for FileMaker (Client ID and Secret), Service Auth policy only on `/api/fm/*`.
- FileMaker sends it as headers in the Insert from URL cURL options: `-H "CF-Access-Client-Id: …" -H "CF-Access-Client-Secret: …"`.
- Worker: on `/api/fm/*` accept only the service token (checked by its Client ID); everywhere else keep requiring a staff email. Each side cannot reach the other's endpoints.
- Secret held in one field on `AppSettings`, Full Access only. One year expiry, Teams reminder to rotate.

## 5. Endpoints (new, Worker)

| Endpoint | Purpose |
|---|---|
| `GET /api/fm/surveys?after=<cursor>&limit=25` | Finished surveys since the cursor. Flattened JSON: header, fabrics, and one row per treatment with its room and window. No photo data, only photo keys. |
| `POST /api/fm/ack` `{ "items":[{"surveyId","revision","fmId"}] }` | FileMaker confirms what it committed. Worker records it and the last good poll time. |
| `GET /api/fm/photo/<key>` | One photo for a web viewer link. Photos stay in R2, not FileMaker containers (Cloud storage is limited per user). |
| `PUT /api/fm/quotes` | FileMaker pushes soft furnishing quotes: `QuoteSerialNumber`, Q- number, S- number once ordered, `CompanyID`, client name, contact, phone, email, site / delivery address and postcode, customer reference. The Worker keeps a copy. |
| `GET /api/quote/Q-000123` (staff) | Admin page and app look up a Q- number and fill in the client data from that copy. |

Feed: when a survey is marked measured (or fitted), the Worker writes a small index object `feed/<ISO time>-<surveyId>-<revision>.json` in the existing R2 bucket. Cursor is the last key read (R2 `list` with `startAfter`). No new storage service.

The Worker does the flattening, so the FileMaker script is a plain loop of `JSONGetElement` and Set Field, and changes to the phone app's internal layout don't break FileMaker as long as the feed format stays the same.

### Q- number lookup: how it really works

The Worker cannot ask FileMaker live (that needs the Data API and a licence, section 2). So FileMaker sends quote details to the Worker ahead of time, and the lookup reads that copy:

- When a soft furnishing quote is saved in FileMaker, a script sends its details to the Worker with Insert from URL (from the user's own FileMaker session, so no extra licence).
- The 15 minute schedule also sends any quotes changed since the last run, as a backstop if a send failed.
- Typing `Q-000123` on the admin page (or in the app) fills in client, contact and site from the copy. A measure is created against that number; its id becomes the Q- number, so the survey id is the Q- number too.
- When the quote is ordered, FileMaker sends the S- number and the measure shows as a fit job under the same serial.
- The phone keeps the jobs list for offline use, as it does now.

Customer details are then edited in FileMaker only; the admin page shows them read only.

## 6. FileMaker side (when it is built)

Server schedule (every 15 to 30 minutes in working hours, avoiding the maintenance window), script `SYNC_Surveys_Pull`:

1. **Open.** Insert from URL into a variable, check `Get ( LastError )` and the HTTP status.
2. **Open.** For each survey: find by `ExternalSurveyID`. New → create; newer revision → update and set `NeedsReview`; same revision → skip. Commit.
3. **Open.** POST the ack for committed ones only. One `SurveySyncLog` row per run.

Proposed records (names fitted to Standards §25 when scoped; none exist yet):

- **Surveys**: `SurveyID` (UUID key), user facing number, `ExternalSurveyID`, `Revision`, `Kind` (Soft furnishing now, Bed later), `Status`, `CompanyID` (empty until matched), `QuoteID`, customer and site text as entered, survey date, measurer, `ImportedAt`, `NeedsReview`, `SourceJSON`.
- **SurveyItems**: one row per treatment: room, window, type, dimensions, quantity, fabric, lining, heading, fitting, notes, photo keys.
- **SurveyFabrics**: fabric name, supplier, code, colour, roll width, repeat, FR. Matched to `FabricDesignRanges` by the office, not automatically.
- **SurveySyncLog**: run time, counts, errors.

Matching: by `QuoteSerialNumber` (from the Q- number), so the survey links to its quote and, through it, its company. **Never by name**, same rule as Xero. Fabrics are matched to `FabricDesignRanges` by the office, not automatically.

Flow: quote (Q-) → measure survey against that quote → survey sizes checked against the quote lines → order (S-) → fit → POs and invoice to Xero. Updating quote lines from survey sizes automatically needs a mapping from treatment type to Products: later, and never silently (differences flagged for review).

## 7. Poor signal and errors

- Phone: every change saved locally first (already the case). Uploads retry on reconnect and app open (already the case for curtains). The phone shows "Sent to office" only after the Worker confirms.
- Split photos from the survey: each photo as its own small upload. One 40 MB request on poor signal keeps failing.
- Uploads are safe to repeat: same id and revision overwrites; previous copy kept (already the case).
- FileMaker down or schedule stopped: nothing lost, items wait in the feed until acked. Pull is idempotent.
- Worker daily check: Teams message if FileMaker has not polled successfully for 24 hours, or a survey has waited unacked for over a day.
- Survey changed after import: new revision with `NeedsReview` set. The office's quote is never changed automatically.

## 8. Fit with the roadmap and staging

EliteFM is in Phase 1 (quote PDF and email); there are no Survey, Sale or Order tables yet. Survey import is not on the roadmap. Proposed as a "Parallel" roadmap item, staged:

| Stage | Work | When |
|---|---|---|
| 0 | Soft furnishings app: every survey starts from a measure that carries the Q- number; add a `revision` counter; keep fabrics and treatments as structured fields, not free text | Now, while the app is being built. Cheap. |
| 1 | Worker: quotes copy and Q- lookup, feed, ack, photo endpoint, service token, photo split upload, daily Teams check | When the survey app is stable. No FileMaker changes. |
| 2 | FileMaker: send quote on save, Surveys, SurveyItems, SurveyFabrics, SurveySyncLog, pull script, schedule, review list | Greg's call; not before the Phase 1 gate |
| 3 | Survey sizes compared with quote lines, differences flagged (treatment to product mapping) | After stage 2 |
| 4 | S- number and fit jobs driven from FileMaker orders | With Phase 2 (orders) |
| Later | Bed survey: add cloud upload (it has none today) and reuse stages 1 to 4 with `Kind = Bed` | Off the backburner |

## 9. Decisions (Greg, 2026-10-23)

1. **Agreed.** Only finished (measured) surveys go to FileMaker.
2. **Agreed.** Every measure comes from a FileMaker quote and is keyed by its Q- number (`"Q-" & Right ( "000000" & QuoteSerialNumber ; 6 )`). Ordered jobs become S- with the same serial and turn into a fit.
3. **Agreed.** FileMaker owns client data. Entering a Q- number pulls it through.
4. **Agreed.** Photos stay in Cloudflare R2, viewed by link.
5. **Agreed.** The schedule runs under Greg's Claris ID; Greg re-saves it in Admin Console once a year.
6. **Agreed.** Pull every 15 minutes in working hours. Cost: Insert from URL is not the Data API, so the Data API allowance does not apply. About 40 calls a day on the Cloudflare side, far inside the Workers and R2 free allowances.
7. **Open.** Which quotes count as soft furnishing (for example a quote type or product category), so only those are sent to the Worker.

## 10. Not checked

- No handover note found in the project files to check against.
- Claris Connect (now included with FileMaker licences per Claris) could be an alternative link; not assessed.
- Cloudflare Access service token JWT claim name (`common_name`) to confirm when building.
