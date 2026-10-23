# Survey review and quoting: recommendation

Status: recommendation, 2026-10-23, updated with Greg's answers the same day (section 8). Nothing built. Builds on [survey-to-filemaker.md](survey-to-filemaker.md).

## 1. Short answer

Split the job in two, by who owns the data:

| Step | Where | Why |
|---|---|---|
| Review every item, see warnings, fix or clear them | **Web admin page** (office browsers) | The survey data, photos, sketches and all the size and warning logic already live in the web app. Rebuilding that in FileMaker means two copies of every rule. |
| Pick supplier, product and fabric per item, price it, make the quote | **FileMaker** | FileMaker owns suppliers, products, fabric ranges, their codes, costs, markup, carriage and VAT. The pickers, "clear stale values" rules and costing button already exist there. |

So: not supplier and fabric on the web page. Picking them on the web means copying the whole catalogue to the Worker, building a second picker with its own stale value rules, and the office still has to open FileMaker to price it. Two screens for one job, and the web copy is always slightly out of date.

What Greg asked for ("next to each window pick a supplier and fabric") is delivered, but inside FileMaker, on one allocation screen that lists every window and item from the survey.

**Works without FileMaker (Greg, 2026-10-23):** the review screen, warnings, office edits and tracking do not depend on FileMaker and are built first. Until the new FileMaker link is switched on ([survey-to-filemaker.md](survey-to-filemaker.md), "Works without FileMaker"), the last step is **Mark as checked** and the office uses the existing Excel exports and keys suppliers by hand, as now. When the link is on, the same button becomes **Send to FileMaker**.

## 2. Recommended flow

```
Measurer (phone)        Office (web admin)                 Office (FileMaker)
measure ──▶ Measured ──▶ Review screen: all data, ──▶ Send to  ──▶ pulled in (15 min)
                         warnings fixed or cleared   FileMaker  Allocate: product + fabric per item
                                                              Create quote lines ──▶ cost ──▶ Q- PDF
```

0. **Start (Greg, 2026-10-23):** the office creates a soft furnishing measure or fit, either by the "Soft furnishing job" tick box on the quote in FileMaker (already linked to its Q- number) or on the admin page (where they type the Q- number). The first question is "Measure or Fit only?". One job per Q- number ([survey-to-filemaker.md](survey-to-filemaker.md) section 5). A measure follows steps 1 to 8. A fit only job (client supplied sizes) has no measurer: the office enters the client's sizes on the same review screen (step 2), the same warnings apply, and it carries on from step 3.
1. Measurer marks the survey **Measured**. The phone copy locks for the measurer (office can reopen it).
2. Office opens the survey on the admin page review screen. Every room, window and item with every value entered, photos and sketches, and the warnings.
3. Office fixes values or clears each warning ("checked, correct"). Every edit and clearance is logged with who and when.
4. When nothing is open, the admin team presses **Send to FileMaker** (Greg, 2026-10-23). Only then does the survey appear in the FileMaker feed.
5. FileMaker pulls it against the quote (by Q- number, as agreed). On the quote, office opens **Allocate from survey**: one row per item, pick product and fabric (supplier, range, colour), bulk apply to similar rows.
6. **Send to Curtainwise for costing** (Greg, 2026-10-23: lives in FileMaker). FileMaker produces the file for the maker: every curtain with its room, window, sizes, heading, lining, and the fabric allocated to it with supplier, range, colour, roll width and pattern repeat. FileMaker is the right place because the fabric allocation is made there.
7. Curtainwise works out fabric metres (heading allowance and other workroom allowances) and the making cost, and sends them back. Office enters them in FileMaker: metres on each fabric option row, making cost on the line.
8. **Create quote lines** (or update them with the returned figures): headings, grouped quote lines, fabric option rows, then the normal Update Costs / Sales Price workflow. From there it is the normal quote (PDF, send, Accepted, S-).

## 3. Warnings: what to flag and how clearing works

Two kinds:

| Kind | Example | Can it be cleared without changing data? |
|---|---|---|
| **Missing** (critical field blank) | Curtain drop, blind width, inside or outside recess not chosen | No. Enter the value, or set it to TBC with a note (same idea as the TBC colour rule in FileMaker). TBC items go to FileMaker marked TBC. |
| **Check** (value looks wrong) | Drop of 240 (cm not mm), widths differ by 15 mm, rail narrower than window | Yes. "Checked, correct" records who and when. |

Each warning has its own **Checked, correct** button (Greg, 2026-10-23). Clearing one never clears another. No bulk clear, not even for identical looking warnings on identical items, in case they are not identical (Greg, 2026-10-23).

Clearing rule: a clearance is stored against the item, the warning code and the values it was raised on. If any of those values change later, the warning comes back. No silent carry over.

**Critical fields per item type** (Missing if blank):

| Type | Critical fields |
|---|---|
| Curtain, voile | Quantity, rail or pole width, drop readings (or exact finished drop), what the drop is measured to, heading, pair or single, lining, fixing type, hardware supply |
| Roller, roman, venetian, vertical, pleated, panel, perfect fit | Quantity, inside or outside recess, width and drop readings (order size computed), control side and type, bracket; for inside recess also recess depth; chain operated also hanging height and safety device (to do item 4) |
| Pelmet | Width, depth, projection |
| Hardware | Fixing type, length, supply type |

**Found in the code, needs fixing for this to work:** the phone app pre-fills several critical choices: blind fit defaults to "Inside the recess", drop defaults to "Track to floor", heading to pencil pleat, lining to lined. A default looks exactly like a measured answer, so the office cannot tell "chosen" from "never touched". Agreed (Greg, 2026-10-23): these start blank and count as Missing until chosen. This is the main reason "is it recess or exact?" cannot be checked today.

**Check warnings already in the app** (keep, reuse): unit slips (under 100 mm), readings spread (curtain drops over 20 mm, blind readings over 10 mm), unusual sizes, rail narrower than window or short extension, rail overhang beyond wall space, shallow recess under 60 mm, inward opening window, handle clearance, wide or long blinds, pelmet narrower than rail or too little projection, duplicate room numbers.

**New check worth adding for contract jobs:** outliers across similar items. Hotels repeat the same window dozens of times; flag an item whose size differs by more than a set amount from the other items of the same type and window label on the job (for example room 214 drop 2140 when 30 others are 2410). This catches typos the single item rules cannot.

The warning rules must be one shared file used by the phone app, the admin page and the Worker, so all three always agree.

## 4. What changes in the web app

**Phone app**
* Critical choices start blank (section 3).
* All measurements in cm with one decimal place allowed; warning thresholds and help text change from mm to cm (for example the unit slip warning becomes "under 10 cm").
* Survey locks for the measurer after Measured; office can reopen.
* Warning rules moved into one shared file; each warning gets a stable code.
* The office status list in the phone app (checked, fabrics allocated, sent to supplier, priced, approved, ordered) retires once FileMaker is live; FileMaker owns everything after Checked.

**Admin page: new survey review screen** (computers, browser)
* Opened from the job row, or by typing the Q- number.
* Left: rooms in number order with a count of open warnings each. Main: each window as a card with its measurements, photos, sketch, then each item with every value entered and the computed order sizes (finished drop, blind order width x drop).
* Warnings shown next to the field they belong to, with **Edit** and **Checked, correct** buttons.
* Filter: only items with open warnings. Header: "12 of 14 warnings cleared".
* Table view: one row per item across the whole job, sortable by type and size. Fastest way to eyeball a 100 room hotel.
* **Send to FileMaker** enabled only with nothing open. Marks the survey Checked and sets the revision FileMaker will pull. A later office edit needs another Send, and FileMaker flags the differences.
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
* **Send to Curtainwise for costing** (new script and export, from the quote). One row per curtain: Q- number, room, window, item, quantity, rail width, finished drop, drop reference, heading, pair or single, lining, extras, and the allocated fabric (supplier, range, colour, roll width, pattern repeat). Column layout to match Curtainwise's template (already on the to do list as "Curtainwise column template").
* **Curtainwise return:** fabric metres per curtain and making cost go back onto the fabric option row (`Quantity` in metres, so `TotalCost` = metres x per metre cost works as now) and the line's making cost. Typed by the office for the first version; an import only if Curtainwise can send a file in a fixed layout.
* **Roll width and pattern repeat must come from `FabricDesignRanges`**, not the measurer's notes. Standards does not list these fields on `FabricDesignRanges`; check they exist before scoping, add them if not.

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
| B | Admin review screen, edits with full tracking, warning clearance, Send to FileMaker | Now. No FileMaker dependency, and useful on its own (checked data going to the supplier Excel) |
| C | Worker feed of Checked surveys (survey design stage 1) | When B is stable |
| D | FileMaker survey tables, Survey panel, allocation card, Send to Curtainwise, return entry, Create quote lines | After the Phase 1 gate, Greg's go ahead |
| E | Revision differences flagged on quote lines | After D |
| F | Product and fabric back to the fit job screen | With Phase 2 |

## 8. Decisions

**Agreed (Greg, 2026-10-23)**
* All warnings are reviewed and cleared on the web admin page, not in FileMaker.
* Missing and Check warning kinds as in section 3; each warning has its own Checked, correct.
* Quote lines grouped by identical spec (type, product, fabric, order size) with a quantity and the room numbers listed; headings per floor or room type. The room by room schedule stays in the survey and job sheet.
* Critical choices (blind fit, drop reference, heading, lining) start blank on the phone and count as Missing until chosen.
* Fabric metres come from Curtainwise, not a FileMaker calculation: Curtainwise adds heading and workroom allowances and returns metres and making cost, which go into FileMaker.
* The send to Curtainwise for costing lives in FileMaker, because it needs the fabric allocated to each curtain, with roll width and pattern repeat.
* **Until then, the app's Supplier data (Excel) is the Curtainwise file** (Greg, 2026-10-23): everything they need, no client details or notes, a Location per item. Live in soft furnishings v16 (PR #6, 2026-10-23).

* Only checked surveys go to FileMaker, and the admin team sends them: a **Send to FileMaker** button on the review screen, available once nothing is open. Not automatic on Measured or on Checked. FileMaker picks it up on its next 15 minute pull.
* Grouping happens in FileMaker, at Create quote lines, after fabrics are allocated. Items only group when type, product, fabric and order size all match, so it cannot be decided on the web side.
* Roller blinds are made by Excel ([excelgb.com](https://www.excelgb.com), ordering portal [excel.blindata.co.uk](https://excel.blindata.co.uk)), the blind maker, not the spreadsheet. The admin team prices them there and types the costs into FileMaker per line. No price grid needed. Roman blinds are made by Curtainwise and go in the Curtainwise send like curtains.
* No bulk clear of warnings: each is checked on its own.
* The office can edit measurements on the review screen. Every change is tracked: field, old value, new value, who, when, plus every warning clearance. The measurer's copy locks after Measured.

* **Everything in cm** (Greg, 2026-10-23): the phone app, admin page, feed and Curtainwise file all use cm, matching FileMaker and Curtainwise. Allow one decimal place (for example 122.5) so blind sizes and deductions keep 5 mm precision. Existing surveys in mm are converted once when the change ships.
* **Curtainwise can export CSV or Excel** (Greg, 2026-10-23). FileMaker imports the quotation and fabric summary export, matched by Q- number and Location, and shows it for the office to confirm before anything is written to quote lines. No PDF reading, no typing.

**Still open**
1. **Curtainwise input format.** What they want to receive from us, ideally the same columns as their export so the round trip matches.
2. **Excel (rollers):** keyed by hand in mm (section 8b). Recommended: a printable roller list from FileMaker in portal order, recess size sent as measured.
3. A sample Curtainwise CSV or Excel export, to fix the import mapping.

## 8a. What Curtainwise sends back (from example PDFs, Q17532 and Q17420)

Two PDFs per job, both carrying **Cust Ref = our Q- number**:

**Quotation** (one line per curtain or roman, grouped by room):
* `Line #`, `Location` (for example "Solway W1", "Living Doors"), type (Curtain, Roman), width and drop in **cm**, measure basis (Rail Size, Recess), quantity.
* Fabric (always "Cust Fab", supplied by Elite), number of widths, **fabric amount in metres** (with "Railroad" when turned).
* Single or pair and side, heading (for example Wave Lined), lining (Curtainwise's own, for example "Cw09 Black Out Ivory FR"), labels, lead weights, extras; romans add control, control unit, rail, stack, child safety, stitch, install height and special instructions.
* List price, discount (10%), **net price per line** (making, with lining). Room subtotals and job totals with VAT.

**Fabric summary** (one block per fabric):
* Fabric name, **roll width (mm)** and **pattern repeat (mm)**, then metres per location and type (Main Fabric or Lining), and **Total Quantity Used** per fabric. Lining is listed too, but it is Curtainwise's and inside their price.

What this means for FileMaker:
* Match lines back by Q- number plus Location. So the send to Curtainwise must give every item a Location that is unique on the job, built from room and window (for example "101 W1"), and Curtainwise must keep it. Then returned figures land on the right survey item without guessing.
* Per item, FileMaker stores the making cost (net) and the fabric metres. Main fabric metres go on the `Main Fabric` option row (`Quantity`), so cost = metres x per metre cost as now. The making cost is the line's maker cost; Curtainwise is the supplier for that part.
* The fabric totals per fabric are what Elite orders from the fabric supplier: exactly the Phase 2 PO quantity per fabric supplier. FileMaker can total them from the option rows, so the summary PDF becomes a check, not a source.
* Curtainwise can export CSV or Excel (Greg, 2026-10-23), so FileMaker imports that instead of reading the PDFs.

## 8b. Excel blind portal (excel.blindata.co.uk, read 2026-10-23)

* Orders are keyed in by hand: no import or upload option.
* Order header: account (ELI08), customer and delivery address, order and required dates, Customer Order No, **Customer Reference** (put the Q- number here, as with Curtainwise), Measure Type (Metric).
* Each roller blind (example: Roller, Carnival B/O (2m) Raven): Blind Type, Measurement, Fabric (Excel's own range) and a fabric calculator, Location, Instructions, **Measure To** (for example Recess Size), **Width and Drop in mm**, **Install Height**, **Control Length** (calculated by Excel, can be overridden).
* Options per blind, each with its own price: operation, tube, colour, bracket, covers, control type (left or right hand), control, chain, connector, crank, motor, remote, remote channel, power pack, cable, solar cell, batteries, bottom bar, etching, end caps, and more below. Changing one can swap others automatically (the portal pops up a message listing the swaps).
* Price per blind: Base Price, Options Price, Discount % (20% on the example), Net Price, VAT, Total.

What this means:
* **Units:** Excel works in mm. Everything on our side is cm; the printable roller list for Excel shows mm (cm x 10), clearly labelled.
* **Measure To** (Greg, 2026-10-23) has three options: Recess Size, Blind Size, Cloth Size. Inside the recess: send the measured recess size with **Recess Size**, so Excel applies its own deductions (never our order size, or the deduction is taken twice; the app's roller deduction stays 0). Outside the recess: send our order size (opening plus overlaps) with **Blind Size**. Cloth Size is not used from the survey.
* **Child safety (to do item 4):** Excel already takes Install Height and calculates the Control Length. The survey's hanging height feeds Install Height; our chain length calculation becomes a check against Excel's figure, not a second source.
* **Fabric:** Excel's own range, so in FileMaker a fabric range (or product) under supplier Excel, picked on the allocation card like any other.
* **Options:** too many and too supplier specific to model in FileMaker. The survey captures what matters on site (control side and type, bracket, colour, motor or not, child safety device); the admin team picks the rest in the portal, then copies the Net Price per blind back into FileMaker.

## 9. Not checked

* The current FileMaker product structure for curtain making and blinds (how they are set up as `Products`, unit of measure, whether a curtain is one product with a fabric option). Need a short extract or Greg's description before scoping step D.
* `FabricDesignRanges` field list: roll width and pattern repeat are needed for the Curtainwise file and are not listed in Standards.
