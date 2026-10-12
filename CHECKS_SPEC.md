# MBM Cheques — Product Spec

Product name: "MBM Cheques" (Arabic UI name: "MBM للشيكات"). Part of the MBM product family. Use this name in the app title, PDFs, Excel exports, README and package name (`mbm-cheques`). Never use any company name of a specific client.

A standalone, general-purpose cheque printing and cheque register web app, usable by any business (any industry, any bank, any currency). Runs fully in the browser with local storage. No backend.

## 0. How to work on this repo

- Build in the phases in section 10, one phase per session, in order. At the start of each session read this file and `PROGRESS.md`; at the end update `PROGRESS.md` (what's done, what's left, known issues) and open a PR.
- Do not start a phase until the previous one is merged.
- Every phase must leave the app building, passing all tests, and usable.
- Ask nothing that this spec answers. Where this spec is silent, choose the most robust option and record the decision in `DECISIONS.md`.

## 1. Tech stack

- Vite + TypeScript (strict) + React.
- IndexedDB via Dexie for storage. Call `navigator.storage.persist()` on first run.
- ExcelJS for real .xlsx export/import (styled, not CSV).
- PDF: real vector PDFs with selectable text. Arabic must render correctly (connected letters, RTL, correct bidi with numbers). Choose a library/approach that achieves this with an embedded Arabic font (e.g. Noto Naskh Arabic / Cairo / IBM Plex Sans Arabic) and prove it with a test. Rasterised screenshots (html2canvas-style) are NOT acceptable.
- Vitest for unit tests, Playwright for end-to-end tests.
- Output is a static site deployable to GitHub Pages or Netlify. Include deploy instructions in README.

## 2. Language and direction

- Full Arabic and English UI with a toggle; Arabic is default. All strings in translation dictionaries, no hard-coded text.
- Layout switches RTL/LTR correctly, including tables, forms, dialogs, PDFs and Excel sheets (Excel sheet view RTL when Arabic).
- Setting for numerals: Western (123) or Arabic-Indic (١٢٣) in printed/exported output.
- Dates: Gregorian; format configurable per bank template.

## 3. Data model (minimum)

- Company profile: name (ar/en), logo, default currency.
- Bank: name (ar/en), logo optional.
- Cheque template (per bank, can have several): cheque size in mm, background scan image (for alignment only, never printed), field positions in mm (x, y, width, font, size, alignment, direction), amount-in-words language (ar / en), date format, currency wording, prefix/suffix wording (e.g. "فقط ... لا غير" / "Only"), cross-line option ("يصرف للمستفيد الأول" / "A/C Payee Only"), per-printer calibration offsets (x/y mm).
- Account: bank, account name/number, currency, linked template.
- Cheque book: account, first/last serial, next serial.
- Payee: name (ar/en), optional notes. Auto-suggest from history.
- Cheque: account, serial number, payee, amount, currency, issue date, due date, memo/reference, status, status history (date, user note), printed flag, print count.
- Audit log: every create/edit/status change/print/void/import/restore with timestamp.
- Settings, schema version.

## 4. Business rules

- Serial numbers unique per account; never reuse.
- A printed cheque is locked. To change it: void it (with reason) and issue a replacement linked to it.
- Statuses: Draft, Printed, Delivered, Cleared, Bounced, Cancelled/Void. Allowed transitions enforced; each transition logged.
- Post-dated cheques supported; due date drives follow-up.
- Amount: positive, max 2 decimals (or per-currency minor units).

## 5. Amount in words (tafqeet)

- Arabic: grammatically correct, including dual forms, gender agreement, numbers 1–999,999,999,999 with minor units ("خمسة آلاف ومائتان وخمسون جنيهاً مصرياً وخمسة وعشرون قرشاً فقط لا غير").
- English: "Five Thousand Two Hundred Fifty Egyptian Pounds and Twenty-Five Piasters Only".
- Currencies configurable (EGP, USD, EUR, SAR, AED, KWD with 3 decimals, etc.) with singular/dual/plural names and minor unit names in both languages.
- At least 60 unit tests covering edge cases (0 minor units, 1, 2, 3–10, 11, 100, 200, 1000, 2000, 1,000,000, large numbers, KWD 3 decimals).

## 6. Screens

1. Banks & templates: visual designer — drag fields over the scanned cheque image, fine-tune in mm, live preview, test print on plain paper with alignment guides, calibration wizard.
2. Accounts & cheque books.
3. Issue cheques: single entry with live preview; batch entry; Excel import (download a ready template; map columns; validate with row-by-row errors before importing; nothing saved until confirmed).
4. Print: single or batch, print queue, preview, exact mm positioning via @page and CSS; reprint requires confirmation and is logged.
5. Register: searchable/filterable table of all cheques; status changes in bulk.
6. Follow-up: due today, this week, overdue, bounced; dashboard counters.
7. Reports: by bank/account, payee, status, period; totals by currency.
8. Backup & restore (section 8).
9. Settings: language, numerals, company profile, currencies, backup preferences.

## 7. Exports — must be real and ready to send

- Excel (.xlsx): branded header (company name/logo, report title, period, generated date), styled header row, frozen panes, autofilter, correct number/date formats, currency per column, totals row using real formulas, column widths fitted, RTL sheet when Arabic, print setup (A4, fit to width, repeat header rows, page numbers).
- PDF: same reports as Excel with header, footer, page numbers, totals, table that breaks across pages cleanly, A4 portrait/landscape automatically by column count.
- Exportable: register (filtered view), follow-up lists, every report, single cheque voucher/receipt (payment voucher with signature boxes), import error report.
- Tests verify generated files open, contain expected values, and Arabic text is correct in PDF text extraction.

## 8. Backup and restore (critical — data lives only in the browser)

- Auto-backup folder: in Chrome/Edge use the File System Access API — user picks a folder once; app writes a backup after each significant change (debounced) and keeps the last N files there. Re-request permission gracefully when needed.
- Fallback for other browsers: download backup file + reminder.
- Internal snapshot history: last 10 snapshots stored inside the app; restore any.
- Automatic snapshot before any risky operation: import, restore, bulk status change, delete, schema migration.
- Reminder banner when last external backup is older than X days or after Y new cheques (configurable).
- Backup file: single JSON with schema version, app version, created date, record counts, SHA-256 checksum. Optional password encryption (Web Crypto AES-GCM, PBKDF2).
- Restore: preview (date, counts) before applying; choose full replace or merge (dedupe by stable IDs); reject corrupted/tampered files; migrate older schema versions forward.
- Same mechanism used to move data between devices.
- Warn on first run if in private/incognito mode or if persistent storage was denied.
- Round-trip tests: backup → wipe → restore gives identical data; encrypted and unencrypted; old version migration.

## 9. Quality bar

- Clean, modern, calm UI; responsive (desktop first, usable on tablet).
- Keyboard-friendly data entry.
- No data loss paths; every destructive action confirmed.
- Accessible labels and contrast.
- README in Arabic and English: what it is, how to use, how to deploy, how backups work.

## 10. Phases (one cloud session each)

1. Foundation: project setup, i18n + RTL, data layer with schema versioning, company profile, banks, accounts, cheque books, tafqeet engine + full tests.
2. Templates & printing: visual template designer, calibration, single and batch printing with exact positioning, test print.
3. Issuing & import: cheque entry, payees, business rules, status workflow, audit log, Excel import with mapping and validation.
4. Register, follow-up, reports, and all Excel/PDF exports (section 7) with tests.
5. Backup & restore layer (section 8) with full tests.
6. Hardening: full Playwright suite of main flows, bug fixes, performance with 20,000 cheques, bilingual review of every screen and export, docs, deploy config.
