# Build Log — Sri Murugan Tools

**Someshwaran Murugan** · Melbourne · March 2026 – present

## About this log

Since March 2026 I have designed and built working software tools for Sri Murugan's outlets and warehouse. This log records each tool: how it started, what broke and what I changed. Dates come from my working sessions.

How I work: I find the problem on the shop floor, write the requirements, choose the stack, test with real Cin7 exports and staff, and deploy. I use Claude as my coding assistant for writing and debugging code. The decisions, testing, rollouts and fixes after real-world feedback are mine.

Scope: only tools and systems I built or implemented. Day-to-day operations work, supplier names, customer data and security details are left out.

The earliest entries start on 10 March 2026. Entries up to 25 November 2026 were written up together on 26 November 2026 from my session history; from then on, each day is added in its own dated commit.

## Contents

- [Timeline at a glance](#timeline-at-a-glance)
- [Stock-take apps](#stock-take-apps)
- [Price tags and PriceLynk](#price-tags-and-pricelynk)
- [Online store](#online-store-srimuruganonlinecom)
- [Aging, expiry and stock transfer tools](#aging-expiry-and-stock-transfer-tools)
- [Container unloading and outlet split tool](#container-unloading-and-outlet-split-tool)
- [Membership tools](#membership-tools)
- [Automation and AI agents](#automation-and-ai-agents)
- [What I learned](#what-i-learned)
- [Daily log](#daily-log)

## Timeline at a glance

Newest first.

| Date (2026) | Tool | What happened |
| --- | --- | --- |
| 23 Nov | PriceLynk on Google Sheets | Rebuilt the database, admin portal and staff printer on Apps Script; added print tracking and live Cin7 option |
| 21 Nov | SMT JARVIS | Pricing rules engine, re-order logic and approval gates with a voice UI; 13 tests passing |
| 18 Nov | Outlet Split Tally + Supabase app | Tap-to-tally unloading tool reloaded for new shipments; separate React/Supabase production version |
| 13 Nov | Stock & Ordering Portal | Reviewed the existing Singapore tool and designed the Australian portal |
| 1 Nov | PriceLynk dairy module + scanner | Dairy stock take, camera scanner fixed for iPhone, promo and combo tags |
| 1 Nov | Members Club sign-up | Apps Script that builds the sign-up form and member sheet, plus tested QR codes |
| 28 Oct | Offline transfer PDF app | PWA that reads product photos with OCR and outputs a transfer PDF |
| 21 Sep | Dairy stock-take tool | Admin and staff views with variance report |
| 14 Sep | PriceLynk 5th outlet | Added Cranbourne to admin upload and staff printer; built the /connect page |
| 13 Sep | Online store mobile app | Fixed the mobile app showing 1–2 of 12,000 products |
| 3 Sep | iMin POS + thermal printer | Got the Android POS printing to the TM-T82X through RawBT |
| 7 Jul | Unload tally (offline HTML) | Turned a 333-line invoice into a tap-to-tally app |
| 5 Jul | Power BI expiry dashboard | Star schema, 6 input tables, 17 DAX measures |
| 1 Jul | Aging Stock Transfer Planner | React tool over a 7,066-SKU aging report with transfer history |
| 26 Jun | Online store | Built the homepage on Dawn and fixed broken search; site went live |
| 29 Apr | Price tag printer + price check site | Python print server to the Epson, Netlify price-check pages per store |
| 26 Apr | Daily price-change report | Found the Cin7 API returns 0 prices; switched to the scheduled email |
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

**Stage 3 — PriceLynk on Firebase.** I moved everything to Firebase: admin upload, staff printer, customer checker, price-change report and a central hub. On 14 September I added Cranbourne as the 5th outlet. Existing products weren't getting Cranbourne prices because the skip logic compared undefined with zero; I added a forced full rewrite.

**Stage 4 — scanner and promo tags (November).** I added a phone camera scanner beside search. html5-qrcode, then BarcodeDetector, both failed on iPhone; ZXing worked. I added barcode matching that handles leading zeros and UPC/EAN variants, WAS/NOW promo tags and multi-tier combo tags in our brand colours.

**What failed:** a checksum-based upload to save Firestore reads produced a 1.83 MB cache, over Firestore's 1 MB limit. I reverted to the safe upload rather than risk production. Then the daily free read limit ran out, mostly because the staff printer reloaded all 18,500 products each time.

**Stage 5 — rebuild on Google Sheets (23 November).** To remove the read limit I rebuilt the database, admin portal and staff printer on Apps Script with Sheets as the only database. The live Firebase system kept running untouched. I migrated the price history, kept the same label designs, and added:

- a staff list that shows each product once at its latest price, so a price changed and changed back is not reprinted
- print tracking per outlet ("Tags to print")
- the Cin7 email arriving 3–4 times a day, before the evening pricing meeting and before opening
- an optional live Cin7 connection with a switch back to email
- settings to add other businesses, in case I offer PriceLynk as a subscription

The Netlify drag-and-drop deploy also fixed phones that couldn't reach the API when the page was opened as a downloaded file.

## Online store (srimuruganonline.com)

I built the online store on Shopify's Dawn theme with about 12,000 products synced from Cin7 Core, and it went live by late June.

**How it started (26 June).** I built the homepage step by step: an announcement bar with all stores, a dark header with our logo, a hero banner, a horizontal category row, a promo banner grid and a store locations section (the last two as custom Liquid sections). I set the palette: saffron #E07B20, deep green #1D6B3A.

**What broke and how I fixed it:**

- A header generated by Shopify Sidekick replaced Dawn's own header and broke predictive search. I went back to Dawn's header and styled it to match.
- Deleted sections left block references in index.json, causing "Invalid value for type in block" errors. I removed and re-added those sections cleanly.
- The category row scrolled vertically. I fixed it with flex, no-wrap and horizontal overflow in the section CSS.
- Product names arrive from Cin7 in ALL CAPS. I planned a Liquid title-case filter instead of editing 12,000 names by hand.

**Mobile app (13 September).** The OneMobile app showed only 1–2 products. I ruled out search apps and permissions and worked through bulk-publishing all 12,000 SKUs to the channel.

**Later work (November).** Homepage redesign on a copy of the theme, an image standard of white background at 1000×1000 with an "Image coming soon" fallback, and Pooja perishables for pre-order.

## Aging, expiry and stock transfer tools

These tools came from one goal: clear expired stock, rotate slow stock and move short-dated lines before they're lost, across five locations.

- **Aging Stock Transfer Planner (1 July).** I chose a web tool over Excel because I needed "mark as done" across days. It reads the Cin7 aging report (7,066 SKUs, 5 locations), merges batches per SKU, ranks 15–30 products a day by age, idle quantity and expiry, and logs history so moved items drop out.
- **Power BI expiry dashboard (5 July).** I designed the data model: 6 Excel tables for weekly entry, a star schema, 17 DAX measures and 5 report pages.
- **Offline transfer PDF app (28 October).** Staff take photos of products or a handwritten list; OCR reads name and expiry and the app builds a transfer PDF grouped by route and soonest expiry, with barcodes and sign-off lines. All libraries are bundled so it works offline. It worked in tests; iPhone and real handwriting were still untested at handover.
- **Stock & Ordering Portal (13 November).** Before building, I reviewed the existing Singapore tool and listed what the new one must do better. The Australian design uses per-user login, cached stock from Cin7, transfers before new orders, and clear "no expiry tracking" labels for non-batch items.

## Container unloading and outlet split tool

Every container gets split across the outlets. I went from spreadsheets to a tap-to-tally tool after watching how unloading actually works.

**Version 1 — offline HTML (7 July).** I turned a 333-line commercial invoice into a self-contained app with search, +/− per outlet, a progress bar, auto-save and CSV export. After the first use I swapped percentage labels for suggested quantities and added typed inputs.

**Version 2 — spreadsheets, then back to an app (November).** I tried Excel layouts first. The first was too complex, so I simplified it twice. Then I described the real job: three people unload, one records, two sort cartons onto outlet pallets. That made it clear a phone tool beats a sheet.

**Version 3 — Outlet Split Tally.** A shared tool with cloud storage, carton buttons plus editable piece counts, CSV and PDF export, and a "New list" import for each new shipment. I added manual product entry with duplicate-SKU blocking.

**Version 4 — production app.** A separate React, TypeScript and Supabase system with invoice import, allocation screens, role-based access and invite-only users.

**Mistakes I caught:** at one point product data was filled in from memory instead of the uploaded PDF. I threw that work out and had it redone from the actual file. I also flagged products showing over-allocation even though marked "Done".

## Membership tools

- **Membership points page (22 March).** A prototype that looks up a customer's Cin7 loyalty and purchases by phone number, through a small backend so API keys never sit in the browser.
- **Members Club sign-up (1 November).** An Apps Script that builds the sign-up form and member list in one run, with Australian mobile validation and QR codes checked by decoding them before printing.

## Automation and AI agents

In November I started building tools that do the manual work themselves, with a person always approving before anything changes in Cin7.

**POS printing (3 September).** The iMin Swan 2 Android POS wouldn't work with Epson's own apps. I got it printing to the TM-T82X through RawBT, and worked out that direct ESC/POS commands are the fast path for PriceLynk.

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
| Cin7 data isn't what it looks like | API prices of $0, barcodes in attribute columns | Check a real export or API sample before writing logic |
| Simple beats clever | Unloading tool, stock-take forms | Start with what staff do on the floor, then the screen |
| Never break what's live | PriceLynk upload optimisation, Sheets rebuild | Build the new version beside the old one and switch only when tested |
| Verify against the source | Invoice data in the unloading tool | Re-read the original; flag anything doubtful |
| Keep secrets out of the browser | Membership page, reviewing older tools | Keys stay on a backend or in Script Properties; rotate if exposed |
| A human approves | JARVIS approvals, draft records in Cin7 | Tools create drafts; a manager authorises in Cin7 |

## Daily log

One entry per day I build or change a tool, newest first, each added in its own commit.

| Date | Tool | What I did | Problem → fix | Status |
| --- | --- | --- | --- | --- |
