# Build Log — Sri Murugan Tools

**Someshwaran Murugan** · Melbourne · March 2026 – present

## About this log

Since March 2026 I have built more than 20 working tools for Sri Murugan's outlets and warehouse. This log records what I built, when, what broke and what I changed. Dates come from my working sessions.

How I work: I find the problem on the shop floor, write the requirements, choose the stack, test with real Cin7 exports and staff, and deploy. I use Claude as my coding assistant for writing and debugging code. The decisions, testing, rollouts and fixes after real-world feedback are mine.

Scope: only the tools for Sri Murugan. Supplier names, customer data and security details are left out of this public version.

The earliest entries here start on 10 March 2026. Entries up to 25 November 2026 were written up together on 26 November 2026 from my session history; from then on, each day is added in its own dated commit.

## Contents

- [Timeline at a glance](#timeline-at-a-glance)
- [Stock-take apps](#stock-take-apps)
- [Price tags and PriceLynk](#price-tags-and-pricelynk)
- [Online store](#online-store-srimuruganonlinecom)
- [Aging, expiry and stock transfers](#aging-expiry-and-stock-transfers)
- [Container unloading and outlet splits](#container-unloading-and-outlet-splits)
- [Ordering, sourcing and range planning](#ordering-sourcing-and-range-planning)
- [Promotions and store marketing](#promotions-and-store-marketing)
- [Automation and AI agents](#automation-and-ai-agents)
- [What I learned](#what-i-learned)
- [Daily log](#daily-log)

## Timeline at a glance

Newest first. Each row is a session where I built or changed something real.

| Date (2026) | Tool / work | What happened |
| --- | --- | --- |
| 25 Nov | Rice container order planner | Shared sheet that sizes the next 20-ft order from sales, incoming stock and months of cover |
| 24 Nov | IGA range list + delivery check | Rebuilt the IGA list by category, brand and pack size; found 40 of 112 frozen lines not yet delivered |
| 24 Nov | Promo videos | Code-rendered ad for the online store and a teaser for the IGA store |
| 23 Nov | PriceLynk on Google Sheets | Rebuilt the database, admin portal and staff printer on Apps Script; added print tracking and live Cin7 option |
| 21 Nov | SMT JARVIS | Pricing rules engine, re-order logic and approval gates with a voice UI; 13 tests passing |
| 18 Nov | Outlet Split Tally + Supabase app | Tap-to-tally unloading tool reloaded for 3 shipments; separate React/Supabase production version |
| 18 Nov | POS + scale research | Compared four ways to run weighing scales with Cin7 Core |
| 13 Nov | Stock & Ordering Portal | Reviewed the existing Singapore tool and planned the Australian portal |
| 10 Nov | Hold inspection matching | Matched handwritten batch codes to a container invoice and listed shortages |
| 9 Nov | IGA range analysis | Ranked ~9,500 products across 4 outlets into a 2,624-line IGA range |
| 1 Nov | PriceLynk dairy module + scanner | Dairy stock take, camera scanner fixed for iPhone, promo and combo tags |
| 1 Nov | Members Club sign-up | Apps Script that builds the sign-up form and member sheet, plus tested QR codes |
| 30 Oct | Shopfront posters | Three A3 vector posters for the Carrum Downs store |
| 28 Oct | Offline transfer PDF app | PWA that reads product photos with OCR and outputs a transfer PDF |
| 18–29 Oct | Transfer and split sheets | Store-wise transfer, removal and carton-split workbooks from photos and invoices |
| 3 Oct | 8-week promotion planner | 10-sheet planner with 154 offers and competitor research |
| 2 Oct | Rice order prediction | First demand-based container order from the movement report |
| 21 Sep | Dairy stock-take artifact | Admin and staff views with variance report |
| 14 Sep | PriceLynk 5th outlet | Added Cranbourne to admin upload and staff printer; built the /connect page |
| 13 Sep | OneMobile sync | Troubleshot the mobile app showing 1–2 of 12,000 products |
| 3 Sep | iMin + thermal printer | Got the Android POS printing to the TM-T82X through RawBT |
| 21 Jul | Epping aging pilot | Rotation model and weekly tracker for expired and slow stock |
| 7 Jul | Unload tally (offline HTML) | Parsed a 333-line invoice (14,080 units) into a tap-to-tally app |
| 5 Jul | Power BI expiry dashboard | Star schema, 6 input tables, 17 DAX measures |
| 1 Jul | Aging Stock Transfer Planner | React tool over a 7,066-SKU aging report with transfer history |
| 26 Jun | Online store | Built the homepage on Dawn and fixed broken search; site went live |
| 14 Jun | Supplier price comparison | Two-supplier lentil price matrix by product and pack size |
| 22 May | Cin7 spend discount | Set up 5% over $50 excluding rice and atta |
| 14 May | Expiry triage + transfer planner | Expiry reports split for promotion vs outlet transfer |
| 29 Apr | Price tag printer + price check site | Python print server to the Epson, Netlify price-check pages per store |
| 26 Apr | Daily price-change report | Found the Cin7 API returns 0 prices; switched to the scheduled email |
| 22 Apr | Cin7 POS + Linkly EFTPOS | Fixed receipt template and terminal errors at Point Cook |
| 1 Apr | Stock Disposable Form | Printable form matching the transfer form |
| 22 Mar | Membership points page | Prototype lookup of Cin7 loyalty by phone number |
| 10–15 Mar | StockTake Pro | First stock-take app: CSV import, camera scanning, reports, then Supabase |

## Stock-take apps

This was my first tool, and it went through four versions in eight months.

**How it started (10 March).** I wanted a free stock-take app using our Cin7 data: barcodes, SKUs, cost with and without GST. Version 1 was a single HTML file with CSV import, camera scanning and a variance report. I also made a Google Sheets template for an AppSheet version.

**What failed and what I changed:**

- Uploading 18,123 products in one go to Supabase failed with an ON CONFLICT error. I split uploads into batches of 200 and paged reads in 1,000s.
- Our extra barcodes sit in Cin7's AdditionalAttribute columns, not barcode columns. I mapped them so each product carries up to 4 barcodes.
- Scanning was fast on Android Chrome but did nothing on iPhone, because iOS has no BarcodeDetector. I switched to ZXing.
- A dropped connection lost scans. I added offline-first saving: scans store on the phone and sync when the internet returns.
- I checked Supabase free-tier limits against 10–20 staff scanning at once and cut admin polling to stay under 50,000 requests a day.
- I tested an Expo / React Native build for a native scanner.

**Dairy stock take (21 Sep → 1 Nov).** Dairy needs batch and expiry counts. I first built it as a shared tool with admin and staff views and a zero-count flag. Then I rebuilt it inside PriceLynk as a module that works offline on the device, makes batch codes per product and expiry, and exports PDF and CSV grouped by expiry.

Live: [somesh142.github.io/Stock-Take](https://somesh142.github.io/Stock-Take)

## Price tags and PriceLynk

This is my biggest system: it started as a printer that wouldn't print and is now a multi-outlet pricing platform used every day.

**Stage 1 — the printer (April).** Printing tags on the Epson TM-T82X kept failing with Error 87 through the Windows spooler. The real cause was the printer set to a COM port in Epson's settings. After switching it to USB (TMUSB001) I wrote ESC/POS bytes straight to the port from a small Python print server. I tuned price size, bold names, a CODE39 barcode and the cut.

**Stage 2 — the price-check site (29 April).** I put the staff printer and four customer price-check pages (/pc, /cn, /cd, /ep) on Netlify from a GitHub repo. The daily routine was: export from Cin7, upload to GitHub, auto-deploy in about 30 seconds.

**Alongside — the daily price report (26 April).** I wanted a report of price changes each day. The Cin7 API returned $0.00 for every tier because our prices are markup-based. I set up Cin7's scheduled Price List email instead, and later automated reading it with Apps Script.

**Stage 4 — PriceLynk on Firebase.** I moved everything to Firebase: admin upload, staff printer, customer checker, price-change report and a central hub. On 14 September I added Cranbourne as the 5th outlet. Existing products weren't getting Cranbourne prices because the skip logic compared undefined with zero; I added a forced full rewrite.

**Stage 5 — scanner and promo tags (November).** I added a phone camera scanner beside search. html5-qrcode, then BarcodeDetector, both failed on iPhone; ZXing worked. I added barcode matching that handles leading zeros and UPC/EAN variants, WAS/NOW promo tags and multi-tier combo tags in our brand colours.

**What failed:** a checksum-based upload to save Firestore reads produced a 1.83 MB cache, over Firestore's 1 MB limit. I reverted to the safe upload rather than risk production. Then the daily free read limit ran out, mostly because the staff printer reloaded all 18,500 products each time.

**Stage 6 — rebuild on Google Sheets (23 November).** To remove the read limit I rebuilt the database, admin portal and staff printer on Apps Script with Sheets as the only database. The live Firebase system kept running untouched. I migrated the price history, kept the same label designs, and added:

- a staff list that shows each product once at its latest price, so a price changed and changed back is not reprinted
- print tracking per outlet ("Tags to print")
- the Cin7 email arriving 3–4 times a day, before the evening pricing meeting and before opening
- an optional live Cin7 connection with a switch back to email
- settings to add other businesses, in case I offer PriceLynk as a subscription

The Netlify drag-and-drop deploy also fixed phones that couldn't reach the API when the page was opened as a downloaded file.

## Online store (srimuruganonline.com)

I built the online store on Shopify's Dawn theme with about 12,000 products synced from Cin7 Core, and it went live by late June.

**How it started (26 June).** My boss assigned the project. I used a leading Indian grocery site as my reference and built the homepage step by step: an announcement bar with all stores, a dark header with our logo, a hero banner, a horizontal category row, a promo banner grid and a store locations section (the last two as custom Liquid sections). I set the palette: saffron #E07B20, deep green #1D6B3A.

**What broke and how I fixed it:**

- A header generated by Shopify Sidekick replaced Dawn's own header and broke predictive search. I went back to Dawn's header and styled it to match.
- Deleted sections left block references in index.json, causing "Invalid value for type in block" errors. I removed and re-added those sections cleanly.
- The category row scrolled vertically. I fixed it with flex, no-wrap and horizontal overflow in the section CSS.
- Product names arrive from Cin7 in ALL CAPS. I planned a Liquid title-case filter instead of editing 12,000 names by hand.

**Mobile app (13 September).** The OneMobile app showed only 1–2 products. I ruled out search apps and permissions and worked through bulk-publishing all 12,000 SKUs to the channel.

**Later work (November).** Homepage redesign on a copy of the theme, image standard of white background at 1000×1000 with an "Image coming soon" fallback, Pooja perishables for pre-order, and the "Sri Murugan Plus" brand with a video ad using our real products and prices pulled from the store.

## Aging, expiry and stock transfers

My brief was clear: clear expired stock out of Cin7, rotate anything not sold in 60–90 days, and clear short-dated lines. Each tool below came from trying to do that across five locations.

- **Stock Disposable Form (1 April).** A printable form in the same style as our transfer form. I cut it down from price and GST columns and four sign-offs to 7 columns, 20 tall rows and a 3-field approval, because staff fill it by hand.
- **Expiry triage (14 May).** Expiry reports for two stores, split by month. My rule: under 10 units, promote in-store; 10 or more, transfer, with the busiest stores first. When our Excel licence lapsed I moved the work to Google Sheets.
- **Aging Stock Transfer Planner (1 July).** I chose a web tool over Excel because I needed "mark as done" across days. It reads the Cin7 aging report (7,066 SKUs, 5 locations), merges batches per SKU, ranks 15–30 products a day by age, idle quantity and expiry, and logs history so moved items drop out.
- **Power BI expiry dashboard (5 July).** I designed the data model: 6 Excel tables for weekly entry, a star schema, 17 DAX measures and 5 report pages.
- **Epping pilot (21 July).** I ran one store first before scaling. I merged aging with the movement summary to work out weeks of stock left and give each line a decision: write off, urgent clearance, stop reorder, promote or monitor.
- **Offline transfer PDF app (28 October).** Staff take photos of products or a handwritten list; OCR reads name and expiry and the app builds a transfer PDF grouped by route and soonest expiry, with barcodes and sign-off lines. All libraries are bundled so it works offline. It worked in tests; iPhone and real handwriting were still untested at handover.
- **Transfer sheets from photos (October).** For real transfers I built workbooks from photos and handwritten counts. When whole cartons went to one outlet, the first split was wrong, so I re-read the handwriting line by line and corrected it.

## Container unloading and outlet splits

Every container from Singapore gets split across the outlets. I went from spreadsheets to a tap-to-tally tool after watching how unloading actually works.

**Version 1 — offline HTML (7 July).** I parsed a 333-line commercial invoice (14,080 units) into a self-contained app with search, +/− per outlet, a progress bar, auto-save and CSV export. After the first use I swapped percentage labels for suggested quantities and added typed inputs.

**Version 2 — spreadsheets, then back to an app (November).** I tried Excel layouts first. The first was too complex, so I simplified it twice. Then I described the real job: three people unload, one records, two sort cartons onto outlet pallets. That made it clear a phone tool beats a sheet.

**Version 3 — Outlet Split Tally.** A shared tool with cloud storage, carton buttons plus editable piece counts, CSV and PDF export, and a "New list" import. I reloaded it for the September (322 products) and October (190 lines) shipments, clearing old data each time. I added manual product entry with duplicate-SKU blocking.

**Version 4 — production app.** A separate React, TypeScript and Supabase system with invoice import, allocation screens, role-based access and invite-only users.

**Mistakes I caught:** at one point product data was filled in from memory instead of the uploaded PDF. I threw that work out and had it redone from the actual file. I also flagged products showing over-allocation even though marked "Done".

**Related checks:**

- hold-list removals when held stock had already gone to stores (October)
- missing-carton report from a receiving sheet (11 October)
- matching handwritten batch codes to a container invoice and listing short and missing lines (10 November)
- snack splits by whole carton, with any opened carton listed separately

## Ordering, sourcing and range planning

I moved ordering from guesswork to numbers: sales from the Cin7 movement report, stock on hand and lead time.

- **Supplier price comparison (14 June).** Two lentil suppliers compared: one row per product, a column per pack size (1, 2, 4, 5 kg), cheapest price highlighted. The first layout was vertical; I changed it to a pivot so it reads at a glance.
- **Average cost fill (18 September).** Matched 789 supplier-price lines to the inventory list by SKU; 557 filled, 232 flagged for checking.
- **Rice container order (2 October → 25 November).** My first model had the method wrong. I corrected it three times: demand is all four outlets, on-hand is the warehouse only, and summing outlet "Out" double-counts transfers. The current version is a shared planner: change the months of cover or type warehouse stock and the order recalculates.
- **Singapore vs local sourcing (November).** Plan to classify fast and slow movers per outlet and bring long-life, steady items from Singapore while keeping short-life lines local.
- **Stock & Ordering Portal (13 November).** Before building, I reviewed the existing Singapore tool and listed what the new one must do better. The Australian portal design uses per-user login, cached stock, and clear "no expiry tracking" labels for non-batch items.
- **IGA range (9 → 24 November).** From fast-moving SKUs across all outlets I built the range for the new Sri Murugan Plus IGA in Cranbourne West. My first pass only covered our own brand, which was wrong; the redo covers all brands, grouped category → brand → pack size. I then compared the delivery against the list: 72 of 112 frozen lines delivered, 40 outstanding.
- **New Year Sale order (November).** Split a 124-line order into Singapore and local lists, and caught one line tagged to the wrong source.

## Promotions and store marketing

Customers told us our prices felt high even when they weren't, so I built tools to change that perception and to run promotions properly.

- **Membership page prototype (22 March).** Looked up Cin7 loyalty and sales by phone number through a small backend, so API keys never sit in the browser.
- **Cin7 deals (May–September).** Set up 5% off over $50 with rice and atta excluded by including only eligible categories, and worked out how to do "2 for $9.99" when Cin7 deals only do % or $ off.
- **Carrum Downs strategy (11 July).** Researched nearby competition and reviews, then built a competitor price sheet for 18 key SKUs, staff standards and a plan for a live entrance price board and green "we're cheaper" shelf tags.
- **Price tag designs (July–October).** Text-only tags for 14 promo items, Weekend Bazaar full-page tags as an editable PowerPoint template, and Diwali cut-apart price strips.
- **8-week promotion planner (3 October).** 10 sheets, 154 offers and 22 combos, using the container load list and public competitor prices. Rules I set: never invent a price, mark unknown competitor prices as not available, and judge each promo on margin, not just sales.
- **Shopfront (30 October).** Assessed the Carrum Downs frontage and made three print-ready A3 vector posters (Fresh Pooja, Weekend Bazaar, combined).
- **Members Club (1 November).** An Apps Script that builds the sign-up form and member list in one run, with Australian mobile validation and QR codes checked by decoding them before printing.
- **Video ads (24 November).** A code-rendered ad for Sri Murugan Plus with real products, prices and voice-over, and an IGA opening teaser that I sent back for a completely different style.

## Automation and AI agents

In November I started building tools that do the manual work themselves, with a person always approving before anything changes in Cin7.

**POS and hardware groundwork.** I fixed Linkly EFTPOS errors in Cin7 POS (April), got the iMin Swan 2 Android POS printing to the Epson through RawBT (3 September), and compared four ways to run weighing scales with Cin7 Core, including NMI rules for trade scales (18 November).

**SMT JARVIS (21 November).** My goal: an assistant I can teach my own pricing and re-ordering judgement. Design decisions:

- pricing rules are fixed code: margins by category, rounding to .49 or .99, auto-approve small changes, review large ones, alert on big cost jumps
- re-ordering tries a stock transfer from the warehouse or an overstocked store before raising a supplier PO
- three brains: Claude online, a local Ollama model offline, and a no-AI command parser that always works
- approvals need a human; the AI cannot approve itself, and bulk approval never includes high-risk items
- every rejection is saved as a lesson for future training

13 automated tests cover rounding, pack-size matching, risk flags and approval guards. When it first failed to start on my PC (port in use, browser opening too early), I fixed the launcher to fall back to another port and show clear errors.

## What I learned

The same few lessons came up again and again, and they now shape how I start every tool.

| Lesson | Where it hit me | What I do now |
| --- | --- | --- |
| iPhone is different | Scanners in StockTake, PriceLynk and the transfer app all failed on iOS | ZXing first, test on an iPhone before rollout |
| Free tiers have hard limits | Firestore 1 MB documents and daily reads; Supabase request caps | Work out reads per day before building; cache on the device |
| Cin7 data isn't what it looks like | API prices of $0, barcodes in attribute columns, outlet "Out" double-counting | Check a real export or API sample before writing logic |
| Simple beats clever | Disposable form, split sheets, unloading layout all cut down | Start with what staff do on the floor, then the screen |
| Never break what's live | PriceLynk upload optimisation, Sheets rebuild | Build the new version beside the old one and switch only when tested |
| Verify against the source | Handwritten counts, invoice data, wrong-brand IGA list | Re-read the original; flag anything doubtful in yellow |
| Keep secrets out of the browser | Reviewing older tools | Keys stay on a backend or in Script Properties; rotate if exposed |
| A human approves | Draft POs and transfers, JARVIS approvals | Tools create drafts; a manager authorises in Cin7 |

## Daily log

One entry per day I work on a Sri Murugan tool, newest first, each added in its own commit.

| Date | Tool | What I did | Problem → fix | Status |
| --- | --- | --- | --- | --- |
