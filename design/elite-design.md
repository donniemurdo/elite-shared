# Elite shared design file

Source: Elite CF, Elite Beds and Elite FF&E brand guidelines (PDFs uploaded 23 Oct 2026).
Tokens for code: [elite-brand.css](elite-brand.css). Logos: [logos/](logos/).

**Source of truth (Greg, 23 Oct 2026):** the brand PDFs, as set out in this file, override any older colours or fonts in the apps or earlier handover notes (navy #012547, cream backgrounds, Questa Grande, Louis George Cafe).

Every app takes its colours, fonts, buttons and forms from this file. If something is not covered here, add it here first rather than styling it in one app.

## 1. Which brand for which app

| App | Brand | Reason |
|---|---|---|
| Soft furnishings survey (phone) | **Elite CF** | Contract work for hotels, care homes, restaurants. The app already uses a near CF palette. |
| Office admin page | **Elite CF** | Internal tool, same company. |
| FileMaker (EliteFM) | **Elite CF** | House brand for internal systems. Quote and invoice outputs could switch per brand later. |
| Bed survey | **To confirm** (on hold) | Beds if surveys are for retail home customers, CF if for hotel beds. |
| FF&E | Customer facing outputs only | No app for FF&E work yet. |

Rule: internal tools use CF. Beds and FF&E styling applies only to things a Beds or FF&E customer sees (PDFs, emails, quotes).

## 2. Common to all three brands

* Fonts: **Poppins** for headings (SemiBold 600 in the PDFs, Bold for big titles), **Lato** for body text.
* Type scale: small heading 21px, medium 28px, large 37px, page title or hero 49px, body 16px.
* Charcoal **#3A3A3A** is in every palette and is the body text colour.
* Logo: the "Elite" wordmark with the division name underneath. Primary is white on the brand colour, secondary is the brand colour on white.
* Icons: solid filled shapes with no background (pin, globe, envelope, phone, truck and so on), in one colour on a high contrast backdrop.
* Voice: friendly, family business, experienced.

## 3. What differs

| | Elite CF | Elite Beds | Elite FF&E |
|---|---|---|---|
| Primary | Navy **#0A1F4A** | Sage **#8B9D83** | Chestnut Rust **#994636** |
| Buttons | Olive **#777D23** | Not stated, use Dark Olive **#51564D** | Not stated, use Rust |
| Highlight | Kiwi **#C3CB4D** (highlights only) | Sand **#DED1BE** | Wet Asphalt **#2C3E50** |
| Light | White **#FCFCFC** | Ivory **#F2F2E7** | Platinum **#F2F2F2**, Concrete **#E7E7E7** |
| Neutral | Charcoal **#3A3A3A** | Charcoal **#3A3A3A** | Charcoal **#3A3A3A** |
| Icon colour | White or Olive | Ivory or Dark Olive | Platinum or Charcoal |
| Imagery | Casual but clean, people included | Beds only, light tones, pencil sketch style | Large projects, artistic, black and white environments |

Note: the Beds and FF&E palette pages still carry template placeholder text ("Presentations are communication tools...") instead of usage rules, so button and highlight roles for those two are my reading, not stated.

## 4. Elite CF rules (used by all current apps)

**Colour roles**
* Navy: headings, header band, primary backdrops, logo background.
* Olive: buttons.
* Kiwi: highlights only (accent stripe under the header, left edge of tiles, "to do" chips, fitting notes). Never as text on a light background (1.7:1).
* Charcoal: body text.
* White #FCFCFC: page background. Cards are pure white with a navy tint border.

**Contrast check**

| Pair | Ratio | Result |
|---|---|---|
| White on Navy | 15.7:1 | Pass |
| Navy on Kiwi | 9.2:1 | Pass |
| Charcoal on White | 11.1:1 | Pass |
| White on Olive #777D23 | 4.3:1 | **Fails** AA for 16px text (needs 4.5) |
| White on darker Olive #686D1E | 5.5:1 | Pass |
| White on Sage (Beds) | 2.9:1 | Fails, logo and large text only |
| White on Rust (FF&E) | 6.4:1 | Pass |

The brand Olive with white text is just under the accessibility minimum, which matters outdoors on a phone. The CSS uses a slightly darker Olive **#686D1E** for button fills. It is visually very close. **Needs Greg's OK**; the alternative is brand Olive with 19px bold labels.

**Logo**
* Header band: primary logo (white on Navy). Light screens and printouts: secondary (Navy on white).
* Use the image, do not retype "Elite" in a font. The wordmark is not Poppins.
* Minimum height on screen: 32px for the wordmark.
* Current files in [logos/](logos/) are 1100px PNGs taken from the PDFs. **Ask the designer for SVG versions** for sharp rendering and small file size.

**Type**
* Phones: page title 28px, section heading 21px, card title 18px, body 16px.
* Office browsers (900px and wider): page title 37px, section 28px, card title 21px.
* 49px is for marketing and print only.
* Inputs stay at 16px so iPhones do not zoom.

**Buttons**
* Primary: Olive fill, white Poppins SemiBold 16px, 8px corners, 48px tall minimum.
* Secondary: white fill, Navy border and text.
* Danger: white fill, red border and text.
* Small: 40px tall, 14px text. Office screens only.
* One primary button per screen or bar.

**Forms**
* Inputs: white fill, 1.5px navy tint border, 8px corners, 48px tall, Lato 16px, Charcoal text.
* Labels: Poppins SemiBold 13px, muted Charcoal, above the field.
* Focus: 3px blue ring. Functional colour, not brand, kept for accessibility.
* Status colours (green ok, amber warning, red error) are functional and kept from the current apps. They are not in the brand guidelines.

## 5. Fonts offline

Poppins and Lato are free (Open Font License). Do not load them from Google Fonts at run time, because the apps must work offline. Bundle the woff2 files with each app and cache them in the service worker:
* Poppins 600 (and 700 if large titles need it)
* Lato 400 and 700

Roughly 100 KB in total.

## 6. Soft furnishings app: current mismatches

Checked against `donniemurdo/elite-softfurnishing-survey` main (app/_head.html, public/admin/index.html, manifest, icons).

| # | Where | Now | Brand | Fix |
|---|---|---|---|---|
| 1 | Survey and admin, theme colour, app icons | Navy #012547 | Navy #0A1F4A | Swap value |
| 2 | Admin | Olive #777C22 | Olive #777D23 | Swap value |
| 3 | Survey and admin backgrounds, manifest | Cream #F9F5EA, #F1ECDB, #FBFCEB, lines #DAD3BD | Not in CF palette (cream sits closer to Beds Ivory) | White #FCFCFC with navy tint lines |
| 4 | Survey fonts | 'Questa Grande' and 'Louis George Cafe', never loaded, so phones show Georgia and system font | Poppins and Lato | Bundle fonts, swap variables |
| 5 | Admin fonts | System font, Georgia for headings | Poppins and Lato | As above |
| 6 | Primary buttons (survey and admin) | Navy fill | Olive fill | Use --brand-action-text |
| 7 | Admin "alt" buttons | Kiwi fill | Kiwi is highlights only | Secondary style |
| 8 | Body text | Navy | Charcoal, Navy for headings | Swap |
| 9 | Header wordmark | "Elite" typed in Georgia | Logo image | Use primary logo PNG, SVG later |
| 10 | Heading sizes | h1 30px, h2 20px, card titles 18 to 26px | 21 / 28 / 37 | Use scale in section 4 |
| 11 | Back link in header | Kiwi text on Navy | Highlights only | Passes contrast, minor. Change to white |

Already matching: Kiwi accent stripe and tile edges, 48px touch targets, 8px input corners, status colours, focus ring.

## 7. Bed survey (on hold)

Uses an unrelated palette (grey blue with yellow #F2B705) and Arial Narrow. Restyle once its brand is confirmed and work restarts.

## 8. Open points

1. Darker Olive #686D1E for buttons, or brand Olive with larger bold labels.
2. Bed survey brand: Beds or CF.
3. SVG logo files from the designer.
4. Home for this file: a shared project repo (expected `donniemurdo/elite-shared`). Each app keeps its own copy of elite-brand.css and the fonts so it still works offline.
