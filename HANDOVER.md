# Elite Apps and Systems: handover note

Last updated: 23 October 2026.

Read this before starting any piece of work. At the end of each piece of work, update the sections below with what changed and what is still open. Change the existing line rather than adding a competing one.

## 1. Purpose

Build and maintain the internal apps Elite Contract Furniture uses to run the business. Apps are linked so information is entered once and shared.

## 2. Target architecture

FileMaker is the central system and holds all information.

```
Survey apps (phones, offline first)  ->  FileMaker (sale and order)  ->  Xero (invoicing)
```

* Xero only ever receives data from FileMaker (EliteFM Roadmap Phases 3 and 4).
* Match companies by ID, never by name (CompanyID in FileMaker, XeroContactID for Xero).

## 3. Apps and repos

| App | Repo | Status |
|---|---|---|
| EliteFM (FileMaker) | [donniemurdo/EliteFM](https://github.com/donniemurdo/EliteFM) | In build. Phase 1 (quote PDF and email) |
| Soft furnishings survey + office admin page + Worker | [donniemurdo/elite-softfurnishing-survey](https://github.com/donniemurdo/elite-softfurnishing-survey) | In build |
| Bed survey | [donniemurdo/elite-bed-survey](https://github.com/donniemurdo/elite-bed-survey) | On the backburner |
| Shared files (this repo) | [donniemurdo/elite-shared](https://github.com/donniemurdo/elite-shared) | Handover note, design, docs |

**EliteFM:** do not copy its rules here. Read its own files first:
[CLAUDE.md](https://github.com/donniemurdo/EliteFM/blob/main/CLAUDE.md),
[Docs/STATUS.md](https://github.com/donniemurdo/EliteFM/blob/main/Docs/STATUS.md),
[Docs/ROADMAP.md](https://github.com/donniemurdo/EliteFM/blob/main/Docs/ROADMAP.md),
[Standards/FileMakerStandards.md](https://github.com/donniemurdo/EliteFM/blob/main/Standards/FileMakerStandards.md).
Built by Greg with Claude Code and agentic-fm on his Mac (`/Users/gwins/Development/FileMaker/EliteFM`). The repo holds docs only; the .fmp12 files are not in git. Claude never edits the .fmp12 directly; Greg pastes and confirms. Hosted on FileMaker Cloud. Don't build ahead of the roadmap phase gates without Greg's OK.

**Survey apps:**
* One Cloudflare Worker (`elite-surveys`) serves everything at measure.elitecontractfurniture.co.uk, behind Cloudflare Access with Microsoft 365 login. Storage is the R2 bucket `elite-surveys`.
* The bed survey is not deployed from its own repo. A built copy sits in `elite-softfurnishing-survey/public/beds/` and is served at `/beds/`. To release a bed change, copy it into `public/beds/` and deploy from the soft furnishings repo.
* Teams notifications use the `TEAMS_WEBHOOK_URL` Wrangler secret. `GOOGLE_MAPS_KEY` is also a Wrangler secret. No secrets are in any repo; keep the repos private.

## 4. Priorities

1. EliteFM and the soft furnishings app: both in build, both the focus.
2. Bed survey: on the backburner.
3. Survey to FileMaker link: agree the design now, build later (soft furnishings first, beds reuse the pattern).

## 5. Decisions made

| Date | Decision |
|---|---|
| 2026-10-23 | Cloud environment "Elite cloud" (project default): scoped Cloudflare API token (expires 2027-10-23), account ID, wrangler from the setup script, full network. Read access verified; write access not proven until the first real deploy. Any deploy needs Greg's go ahead. |
| 2026-10-23 | No spare Claris licence, and custom OAuth or an external IdP does not avoid one (FileMaker Cloud Data API needs a licensed Claris ID). So FileMaker **pulls** from a secured Worker feed on a server schedule using Insert from URL, running as Greg's Claris ID. Design: [docs/survey-to-filemaker.md](docs/survey-to-filemaker.md). Status: draft, not approved. |
| 2026-10-23 | Brand: Elite CF proposed for all internal apps (soft furnishings survey, admin page, FileMaker). Beds and FF&E styling only on what those customers see. Design file: [design/elite-design.md](design/elite-design.md), tokens in `design/elite-brand.css`. |
| 2026-10-23 | Brand PDFs win over the older app colours and fonts. [design/elite-design.md](design/elite-design.md) is the source of truth for styling. |
| 2026-10-23 | Shared files (this note, design, docs) live in this repo. Each app keeps its own copy of the brand CSS and fonts so it still works offline. |

## 6. Open items

**Survey to FileMaker, decisions for Greg** (details in [docs/survey-to-filemaker.md](docs/survey-to-filemaker.md) section 9)
1. Import only finished (measured) surveys, or every draft save? Recommended: finished only.
2. Must every survey start from a measure? Recommended: yes.
3. Who owns jobs long term: FileMaker or the admin page? Recommended: FileMaker.
4. Photos: keep in R2 and view by link, or copy into FileMaker containers? Recommended: R2.
5. Is a schedule tied to Greg's Claris ID, re-saved yearly, acceptable for now?
6. Pull frequency: every 15 minutes in working hours, or less?

**Design, questions for Greg** (details in [design/elite-design.md](design/elite-design.md) section 8)
1. Buttons: darker olive #686D1E (passes contrast) or brand olive with larger bold labels?
2. Bed survey brand: Beds (retail) or CF (hotels)?
3. SVG logo files from the designer.

**Soft furnishings app issues**
* Test scripts in `tests/` point at file paths from an old scratch environment and won't run until fixed.
* Surveys started without a measure get a 4 character random id. Not safe as a key in FileMaker; needs a full UUID (stage 0 of the integration design).
* No revision counter on surveys yet (stage 0).
* Photos upload inline as base64 in one request (up to 40 MB). Fails on poor signal; split into one upload per photo.
* Design mismatches against the brand (wrong navy, cream backgrounds, fonts never loaded, navy buttons, typed wordmark): 11 listed in the design file section 6. No app code changed yet.

**Bed survey**
* No cloud upload at all. Needed before it can feed FileMaker.

**EliteFM**
* See [Docs/STATUS.md](https://github.com/donniemurdo/EliteFM/blob/main/Docs/STATUS.md) "Next session". As of 2026-10-09: finishing the `UI_Quote_Print` sliding test, then the quote send script.
* Survey import is not on the EliteFM roadmap yet. Proposed as a Parallel item; no FileMaker survey tables before Greg's go ahead and the Phase 1 gate.

## 7. Change log

| Date | What changed |
|---|---|
| 2026-10-23 | Survey apps pushed to GitHub. Cloud deploy access checked. Design file drafted. Survey to FileMaker design drafted. This note created. |
