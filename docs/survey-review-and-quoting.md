# Survey review and quoting: recommendation

Status: recommendation, 2026-10-23. Nothing built. Decisions open in section 8. Builds on [survey-to-filemaker.md](survey-to-filemaker.md).

## 1. Short answer

Split the job in two, by who owns the data:

| Step | Where | Why |
|---|---|---|
| Review every item, see warnings, fix or clear them | **Web admin page** (office browsers) | The survey data, photos, sketches and all the size and warning logic already live in the web app. Rebuilding that in FileMaker means two copies of every rule. |
| Pick supplier, product and fabric per item, price it, make the quote | **FileMaker** | FileMaker owns suppliers, products, fabric ranges, their codes, costs, markup, carriage and VAT. The pickers, "clear stale values" rules and costing button already exist there. |

So: not supplier and fabric on the web page. Picking them on the web means copying the whole catalogue to the Worker, building a second picker with its own stale value rules, and the office still has to open FileMaker to price it. Two screens for one job, and the web copy is always slightly out of date.

What Greg asked for ("next to each window pick a supplier and fabric") is delivered, but inside FileMaker, on one allocation screen that lists every window and item from the survey.

## 2. Recommended flow

```
Measurer (phone)        Office (web admin)                 Office (FileMaker)
measure ──▶ Measured ──▶ Review screen: all data, ──▶ Checked ──▶ pulled in (15 min)
                         warnings fixed or cleared            Allocate: product + fabric per item
                                                              Create quote lines ──▶ cost ──▶ Q- PDF
```

0. **Start (Greg, 2026-10-23):** the office creates a soft furnishing measure or fit, either by the "Soft furnishing job" tick box on the quote in FileMaker (already linked to its Q- number) or on the admin page (where they type the Q- number). The first question is "Measure or Fit only?". One job per Q- number ([survey-to-filemaker.md](survey-to-filemaker.md) section 5). A measure follows steps 1 to 6. A fit only job (client supplied sizes) has no measurer: the office enters the client's sizes on the same review screen (step 2), the same warnings apply, and it carries on from step 3.
1. Measurer marks the survey **Measured**. The phone copy locks for the measurer (office can reopen it).
2. Office opens the survey on the admin page review screen. Every room, window and item with every value entered, photos and sketches, and the warnings.
3. Office fixes values or clears each warning ("checked, correct"). Every edit and clearance is logged with who and when.
4. When nothing is open, office presses **Mark as checked**. Only then does the survey appear in the FileMaker feed.
5. FileMaker pulls it against the quote (by Q- number, as agreed). On the quote, office opens **Allocate from survey**: one row per item, pick product and fabric, bulk apply to similar rows.
6. **Create quote lines**: FileMaker writes headings, quote lines and fabric option rows, then the normal Update Costs / Sales Price workflow prices them. From there it is the normal quote (PDF, send, Accepted, S-).

## 3. Warnings: what to flag and how clearing works

Two kinds:

| Kind | Example | Can it be cleared without changing data? |
|---|---|---|
| **Missing** (critical field blank) | Curtain drop, blind width, inside or outside recess not chosen | No. Enter the value, or set it to TBC with a note (same idea as the TBC colour rule in FileMaker). TBC items go to FileMaker marked TBC. |
| **Check** (value looks wrong) | Drop of 240 (cm not mm), widths differ by 15 mm, rail narrower than window | Yes. "Checked, correct" records who and when. |

Clearing rule: a clearance is stored against the item, the warning code and the values it was raised on. If any of those values change later, the warning comes back. No silent carry over.

**Critical fields per item type** (Missing if blank):

| Type | Critical fields |
|---|---|
| Curtain, voile | Quantity, rail or pole width, drop readings (or exact finished drop), what the drop is measured to, heading, pair or single, lining, fixing type, hardware supply |
| Roller, roman, venetian, vertical, pleated, panel, perfect fit | Quantity, inside or outside recess, width and drop readings (order size computed), control side and type, bracket; for inside recess also recess depth; chain operated also hanging height and safety device (to do item 4) |
| Pelmet | Width, depth, projection |
| Hardware | Fixing type, length, supply type |

**Found in the code, needs fixing for this to work:** the phone app pre-fills several critical choices: blind fit defaults to "Inside the recess", drop defaults to "Track to floor", heading to pencil pleat, lining to lined. A default looks exactly like a measured answer, so the office cannot tell "chosen" from "never touched". Recommend these start blank and count as Missing until chosen. This is the main reason "is it recess or exact?" cannot be checked today.

**Check warnings already in the app** (keep, reuse): unit slips (under 100 mm), readings spread (curtain drops over 20 mm, blind readings over 10 mm), unusual sizes, rail narrower than window or short extension, rail overhang beyond wall space, shallow recess under 60 mm, inward opening window, handle clearance, wide or long blinds, pelmet narrower than rail or too little projection, duplicate room numbers.

**New check worth adding for contract jobs:** outliers across similar items. Hotels repeat the same window dozens of times; flag an item whose size differs by more than a set amount from the other items of the same type and window label on the job (for example room 214 drop 2140 when 30 others are 2410). This catches typos the single item rules cannot.

The warning rules must be one shared file used by the phone app, the admin page and the Worker, so all three always agree.

## 4. What changes in the web app

**Phone app**
* Critical choices start blank (section 3).
* Survey locks for the measurer after Measured; office can reopen.
* Warning rules moved into one shared file; each warning gets a stable code.
* The office status list in the phone app (checked, fabrics allocated, sent to supplier, priced, approved, ordered) retires once FileMaker is live; FileMaker owns everything after Checked.

**Admin page: new survey review screen** (computers, browser)
* Opened from the job row, or by typing the Q- number.
* Left: rooms in number order with a count of open warnings each. Main: each window as a card with its measurements, photos, sketch, then each item with every value entered and the computed order sizes (finished drop, blind order width x drop).
* Warnings shown next to the field they belong to, with **Edit** and **Checked, correct** buttons.
* Filter: only items with open warnings. Header: "12 of 14 warnings cleared".
* Table view: one row per item across the whole job, sortable by type and size. Fastest way to eyeball a 100 room hotel.
* **Mark as checked** enabled only with nothing open. Sets the survey revision that FileMaker will pull.
* Office edits create a new revision, logged as old value, new value, who, when. If the phone later uploads an older revision it is held for review, not allowed to overwrite.

**Worker**
* Store edits, clearances and the Checked state on the survey; the feed (stage 1 of the FileMaker design) lists Checked surveys, not Measured ones.
* Include in the feed: per item computed sizes, raw readings, the measurer's fabric note, the clearance log and photo keys.

## 5. What changes in FileMaker

All of this is survey integration stage 2 and 3, so **after the Phase 1 gate** and only with Greg's go ahead. No changes to existing tables are needed before then.

* **Start tick box** on the quote ("Soft furnishing job", then Measure or Fit only?), as agreed in [survey-to-filemaker.md](survey-to-filemaker.md) section 5. No change here.
* **Survey tables** as already proposed (`Surveys`, `SurveyItems`, `SurveyFabrics`, `SurveySyncLog`). `SurveyItems` gets two new fields: `ProductID` and the chosen fabric (`SupplierCompanyID`, `DesignRangeID`, colour text), plus `QuoteLineID` once a line is made.
* **Quote edit card:** a Survey panel in the reserved right hand panel on `UI_Quote_Edit_Card` (kept free for a quote level feature, ROADMAP Parallel). Shows survey status, revision, open TBCs, a link to the web review screen, and the **Allocate from survey** button.
* **Allocation card** (new): one row per survey item grouped by room and window. Columns: room, window, item, order size, measurer's fabric note, Product, Fabric (supplier, range, colour). Uses the existing Product Picker and fabric Supplier to Design/Range pickers. Tick several rows and **Apply to selected** so 60 identical rooms take one pick, not 60.
* **Create quote lines** script: writes heading rows (`LineType = "Heading"`) and quote lines with `SurveyItemID` kept on the line, plus `Main Fabric` option rows with metres. Then the normal costing workflow. Never changes an existing line silently: a later survey revision flags differences on the line for review (stage 3).
* **Fabric metres:** the survey app deliberately does not compute fabric quantities. Fabric option rows cost by metres x per metre cost, so something has to work out metres (section 8, decision 3).

Why this fits the roadmap: making each item a structured quote line with a product and a fabric option row (supplier, range, metres) is exactly what Phase 2 needs to build the BOM and one PO per supplier (maker, fabric supplier, blind supplier). Free text supplier and fabric picked on the web would have to be matched again later.

## 6. Data that moves each way

| Direction | Data | When |
|---|---|---|
| FileMaker to Worker | Quote and client details by Q- number (already agreed) | On quote save, plus 15 min backstop |
| FileMaker to Worker (optional, later) | Read only list of active fabric ranges: supplier code, range code, name | Nightly. Lets the measurer pick the customer's chosen fabric instead of typing it, so FileMaker can pre-fill the allocation by ID. Not needed for the first version |
| Worker to FileMaker | Checked survey: header, rooms, windows, items (raw readings, computed sizes, fit, options), measurer's fabric notes, TBC list, clearance and edit log, photo keys, revision | 15 min pull |
| FileMaker to Worker (with Phase 2, S- orders) | Per item product and fabric chosen, so the fitter's job screen shows what is going up | When ordered |

Codes: the web side never needs supplier or product codes. They stay in FileMaker. The only identifiers crossing are the Q-/S- number, survey id and revision, item id, and (optionally) fabric range IDs.

## 7. Phasing

| Step | Work | When |
|---|---|---|
| A | Phone: blank critical defaults, shared warning file, lock after Measured | Now. Soft furnishings app is in build |
| B | Admin review screen, edits with log, warning clearance, Mark as checked | Now. No FileMaker dependency, and useful on its own (checked data going to the supplier Excel) |
| C | Worker feed of Checked surveys (survey design stage 1) | When B is stable |
| D | FileMaker survey tables, Survey panel, allocation card, Create quote lines | After the Phase 1 gate, Greg's go ahead |
| E | Revision differences flagged on quote lines | After D |
| F | Product and fabric back to the fit job screen | With Phase 2 |

## 8. Open decisions for Greg

1. **Send to FileMaker on Checked, not Measured.** Changes agreed decision 1 in the FileMaker design. Recommended: yes, FileMaker only receives office checked data.
2. **Quote line shape.** One quote line per surveyed item (100 room hotel = 100+ lines), or one line per identical spec (type, product, fabric, order size) with a quantity and the room numbers listed? Recommended: grouped by identical spec, with headings per floor or room type; the full room by room schedule stays in the survey and job sheet.
3. **Fabric metres.** FileMaker calculation (widths = rail x fullness / roll width rounded up, cut drop = finished drop + allowances rounded to the pattern repeat) shown as an editable suggestion, or the maker's figure typed in? Recommended: FileMaker calculation, editable. Needs Greg's allowances for hems and headings per heading type.
4. **How making and blinds are priced.** The app today has "Sent to supplier for pricing" (for example Curtainwise). If makers and blind suppliers quote each job, FileMaker only needs their cost typed per line. If they have price lists by width and drop, FileMaker needs a price grid per product, which is new scope and probably Phase 2. This answer decides most of the FileMaker build.
5. **Blank defaults on the phone.** Measurers will have to choose fit, drop reference, heading and lining on every item. Recommended: yes for fit and drop reference at least; a "same as last item" button keeps it fast.
6. **Office edits to measurements.** Allowed on the web review screen with a full log? Recommended: yes, measurer copy locked after Measured.

## 9. Not checked

* The current FileMaker product structure for curtain making and blinds (how they are set up as `Products`, unit of measure, whether a curtain is one product with a fabric option). Need a short extract or Greg's description before scoping step D.
* `FabricDesignRanges` field list (roll width, repeat, per metre cost) beyond what Standards mentions; needed for decision 3.
