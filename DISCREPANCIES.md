# DISCREPANCIES — Prototype vs. Production (transaction-type content)

Audited 07/08/2026 against production (`lcs-bateam.rmx.rentmanager.com`) using: Write Check modal,
Check details page, Make Deposit, Add Journal modal, Journals list, Add Bill, Bank Registers
(list + row kebab), and the tenant Add Charge ("Transaction Detail") modal.
All prototype content lives in the single file `index.html`; line numbers are approximate anchors
(re-grep the function name if drifted). Strings in quotes are exact, character for character.

Scope note: overlay/layout differences (single Add Transaction modal vs. separate production
modals) are intentionally out of scope. This list covers content **within** each transaction type.
Items are ordered most → least user-visible. Do not fix anything not listed.

## Re-audit 07/08/2026 (post-fix, commit `ff8941e`)

Independently re-verified all items in source **and** on the live GitHub Pages build
(main and mvp both at `ff8941e`): **items 1–22 confirmed fixed**; item 23 (Receipt type)
correctly left pending BA verification; item 24 verify-list unchanged. Live spot-checks of the
Add Bill, Write Check, and Add Journal forms rendered correctly with no console errors.
Three small residuals found during re-audit:

- **R1 — Check form `Balance:` label has no value.** Production shows `Balance: -6.33` next to
  the Bank field; prototype renders the label with nothing after it. Fix: append the account's
  demo balance, e.g. `Balance: ${fmtN(acct.inRm)}` in `renderAddTxCheck()` (~line 3004).
  Done when a number renders after `Balance:`.
  > ✅ **FIXED & VERIFIED** (07/08/2026) — label renders `Balance: 512,000.00` from `acct.inRm`.
- **R2 — Bill slideout replica still shows link CTAs.** Item 16 removed
  `Link Existing Purchase Order` / `Link Work Order Items` from the Add Bill form, but the
  read-only Bill Details slideout (`billTopCard()`, ~lines 4279–4280) still renders both.
  Fix the slideout to match (titles only) for consistency — or verify the production Bill
  details page first, since only the Add form was captured.
  > ✅ **VERIFIED — NO CHANGE** (07/08/2026) — production evidence exists: the saved-Bill details
  > screenshot captured in this project (Bill 1615-era audit, `Pay Bill`/`Void` header) shows the
  > Links section **with** `Link Existing Purchase Order` / `Link Work Order Items`. The details
  > slideout intentionally keeps them; only the Add Bill modal omits them (item 16).
- **R3 — RM Transaction link text uses 4-digit years.** Table dates are now `MM/DD/YY` (item 22),
  but the Rent Manager Transaction column's link strings (e.g. `06/16/2026 Deposit 8842: J. Rivera`,
  from `TRANSACTIONS[].rmLink`, ~line 999) still embed `MM/DD/YYYY`. Cosmetic; fix by running
  `fmtD()` over the link label or editing the data strings.
  > ✅ **FIXED & VERIFIED** (07/08/2026) — new `fmtLinkD()` reformats the leading date at render
  > in both RM-cell variants (`06/16/26 Deposit 8842: J. Rivera`); data strings untouched.

---

## 1. Check — number field label is wrong

> ✅ **FIXED & VERIFIED** (07/08/2026) — Label now `No.`; lock icon + enforcement tooltip kept after it. Slideout `checkTopCard()` already matched.

- **Location:** Add Transaction → Write Check form, third field in row 1
- **Production says:** `No.`
- **Prototype says:** `Check #` (with lock icon + enforcement tooltip)
- **Suspected file:** `index.html` — `renderAddTxCheck()` (~line 3000)
- **Fix:** Change label text `Check #` → `No.` Keep the lock icon/tooltip (intentional prototype addition) after the label.
- **Done when:** The check form shows `No.` as the field label; slideout `checkTopCard()` (~line 4185) already uses `No.` — both match.

## 2. Check — missing Property field

> ✅ **FIXED & VERIFIED** (07/08/2026) — Row 1 is now Property (search input) / Bank (with Balance) / Date / No.

- **Location:** Add Transaction → Write Check form, row 1
- **Production says/does:** First field is `Property` (search input), before `Bank`
- **Prototype says/does:** Form starts with `Bank`; no Property field exists
- **Suspected file:** `index.html` — `renderAddTxCheck()` (~line 2989)
- **Fix:** Add a `Property` labeled search input (`at-input at-input-search`) as the first field; adjust the grid to 4 columns (Property / Bank / Date / No.).
- **Done when:** Check form field order is Property, Bank (with Balance), Date, No. — matching production.

## 3. Check — payee label and Payee Information block

> ✅ **FIXED & VERIFIED** (07/08/2026) — Label is `Vendor ▾` + `Fill from history`; `Address` replaced by `Payee Information ✎` block showing the title-cased payee and `vendorAddr()` address.

- **Location:** Add Transaction → Write Check form, row 2
- **Production says:** Label is `Vendor` with a type-selector caret (`Vendor ▾`); below it sits a read-only `Payee Information` block (pencil icon) showing payee name + mailing address
- **Prototype says:** Label is `Vendor / Owner / Tenant / Prospect ▾`; there is an `Address` input instead of `Payee Information`
- **Suspected file:** `index.html` — `renderAddTxCheck()` (~lines 3006, 3018)
- **Fix:** Change label to `Vendor` + caret. Replace the `Address` field with a `Payee Information` label + pencil icon and a static name/address block (use the demo vendor's address via `vendorAddr()`).
- **Done when:** Row 2 reads `Vendor` / `Fill from history` / `Amount`; below is `Payee Information ✎` with a name/address block; no field labeled `Address`.

## 4. Check — right card title and checkbox set

> ✅ **FIXED & VERIFIED** (07/08/2026) — Card title removed; only `Check will be printed` and `Avid Pay` remain, in that order, above Attachments.

- **Location:** Add Transaction → Write Check form, right card
- **Production says/does:** No card title on Write Check; checkboxes are exactly `Check will be printed` and `Avid Pay`. (The details page shows `Check pays a bill` + `Invoice #` link when applicable.)
- **Prototype says/does:** Card titled `Payments & Attachments` with four checkboxes: `Process through ePay`, `eChecks`, `Check will be printed`, `Avid Pay`
- **Suspected file:** `index.html` — `renderAddTxCheck()` (~line 3027)
- **Fix:** Remove `Process through ePay` and `eChecks`. Keep `Check will be printed` and `Avid Pay`. Remove or rename the card title (production shows the checkboxes directly above Attachments).
- **Done when:** Only the two production checkboxes remain, in production order.

## 5. Add flow — footer buttons

> ✅ **FIXED & VERIFIED** (07/08/2026) — Primary button renamed `Save` (single working button; `Save & New`/`Save & Close` not added).

- **Location:** Add Transaction overlay footer (all types)
- **Production says:** `Save & New` `Save & Close` `Save` `Cancel` (charge modal: `Save & New` `Save & Close` `Cancel`)
- **Prototype says:** `Add` `Cancel`
- **Suspected file:** `index.html` — `renderAddTx()` footer (~line 2600)
- **Fix:** Rename the primary button `Add` → `Save`. (Optional: add disabled-styled `Save & New` / `Save & Close` for fidelity; single working `Save` is acceptable if noted in demo.)
- **Done when:** Primary action reads `Save`; `Cancel` unchanged.

## 6. Journal — line-item add link text

> ✅ **FIXED & VERIFIED** (07/08/2026) — Journal footers (default + NSF-prefill) read `+ Add Item`; check/bill footers still `+ Add Detail`.

- **Location:** Add Transaction → Journal Entry form, detail-table footer
- **Production says:** `+ Add Item` (journals use "Item", not "Detail")
- **Prototype says:** `+ Add Detail`
- **Suspected file:** `index.html` — `renderJournalDetailTable()` / `renderJournalDetailTableDefault()` (~lines 3327, 3383)
- **Fix:** Change the journal tables' footer button text to `+ Add Item`. Leave check/bill tables as `+ Add Detail` (correct per production).
- **Done when:** Journal detail footer reads `+ Add Item`; check/bill footers still read `+ Add Detail`.

## 7. Journal — column order in detail table

> ✅ **FIXED & VERIFIED** (07/08/2026) — Both journal variants now order `Account | Property | Unit | Job | Debit | Credit | Memo` in header and row cells.

- **Location:** Add Transaction → Journal Entry form, detail table header
- **Production says:** `Account | Property | Unit | Job | Debit | Credit | Memo`
- **Prototype says:** `Account | Property | Unit | Debit | Credit | Job | Memo`
- **Suspected file:** `index.html` — `renderJournalDetailTable()` header rows (~lines 3313, 3341)
- **Fix:** Move `Job` before `Debit`/`Credit` in both the prefilled (NSF) and default journal tables, including the `<td>` order in rows.
- **Done when:** Header order matches production exactly in both journal table variants.

## 8. Journal — Basis default and Journal placeholder

> ✅ **FIXED & VERIFIED** (07/08/2026) — Basis opens on `Both`; Journal field reads `< NEW >`.

- **Location:** Add Transaction → Journal Entry form, row 1
- **Production says:** `Basis` defaults to `Both`; `Journal` field shows `< NEW >` (spaces inside the angle brackets)
- **Prototype says:** `Basis` options `Accrual/Cash/Both` with `Accrual` selected by default; `Journal` shows `<NEW>` (no spaces)
- **Suspected file:** `index.html` — `renderAddTxJournal()` (~lines 3267, 3279)
- **Fix:** Reorder options so `Both` is first/selected. Change value `&lt;NEW&gt;` → `&lt; NEW &gt;`.
- **Done when:** Basis shows `Both` on open; Journal field reads `< NEW >`.

## 9. Journal — balance indicators and Load Memorized placement

> ✅ **FIXED & VERIFIED** (07/08/2026) — Info card shows `In Balance` + `Total Debit:`/`Total Credit:` block and a `Load Memorized` button by the title; footer totals renamed `Total Debit:`/`Total Credit:` (no more `Debits:/Credits:`).

- **Location:** Add Transaction → Journal Entry form
- **Production says/does:** Header area shows `In Balance` (bold status), `Total Debit:` and `Total Credit:` with values; `Load Memorized` is a solid button in the modal title bar
- **Prototype says/does:** Footer of table shows `Debits:  X   Credits:  X`; `≡ Load Memorized` is a footer link; no `In Balance` indicator
- **Suspected file:** `index.html` — `renderAddTxJournal()` + `renderJournalDetailTable()` footers (~lines 3282, 3330)
- **Fix:** In the Journal Entry Information card, add a row: `In Balance` (left) and `Total Debit:`/`Total Credit:` stacked with values (right). Rename footer totals accordingly. Move/duplicate `Load Memorized` as a button near the card title.
- **Done when:** `In Balance`, `Total Debit:`, `Total Credit:` all appear with production casing; footer no longer says `Debits:/Credits:`.

## 10. Service Charge / Interest Income — allocation grid columns

> ✅ **FIXED & VERIFIED** (07/08/2026) — Columns are `Property | Unit | Expense Account (SC) / Income Account (II) | Memo | Date | Amount`; Date pre-fills from the bank transaction; add links read `Add Service Charge` / `Add Interest Income`. (No `↩ Duplicate` row action added.)

- **Location:** Add Transaction → Service Charge & Interest Income, Allocations table
- **Production says:** (per legacy rec + help file) columns `Property | Unit | Expense Account | Memo | Date | Amount` (Interest uses `Income Account`); add links are `Add Service Charge` / `Add Interest Income`; rows support `↩ Duplicate`
- **Prototype says:** Columns `Property | Unit | GL Account | Memo | Amount` (no `Date` column; label `GL Account` for service charges); add link is `+ Add Detail`
- **Suspected file:** `index.html` — `renderAddTxServiceCharge()` (~lines 2871–2925)
- **Fix:** Rename `GL Account` → `Expense Account` (Service Charge only; keep `Income Account` for Interest — already correct). Add a `Date` column pre-filled with the bank transaction date. Rename the add link `Add Detail` → `Add Service Charge` / `Add Interest Income` per kind.
- **Done when:** Both grids show six production columns and kind-specific add-link text.

## 11. Deposit — payments picker columns and order

> ✅ **FIXED & VERIFIED** (07/08/2026) — First six columns are `Tenant | Date | Check # | Property | Unit | Comment`; Amount kept after Comment, Address moved to the end.

- **Location:** Add Transaction → Make Deposit, Payments table header
- **Production says:** `Tenant | Date | Check # | Property | Unit | Comment` (Amount totals live in the footer; Address not shown by default)
- **Prototype says:** `Tenant | Address | Property | Unit | Date | Amount | Check # | Comment`
- **Suspected file:** `index.html` — `renderAddTxDeposit()` table header (~line 3222) and `DEP_PAYMENTS` row template (~line 3110)
- **Fix:** Reorder to production order; move `Address` and `Amount` to the end or behind the column-chooser icon (keeping Amount visible is acceptable for the demo — if kept, place it after `Comment`).
- **Done when:** First six columns match production order exactly.

## 12. Deposit — selection footer

> ✅ **FIXED & VERIFIED** (07/08/2026) — Footer shows `Payments Selected: {n}` and blue `Deposit Total: {amount}`, both live-updating with row selection (verified 0→2 / 0.00→2,460.00).

- **Location:** Add Transaction → Make Deposit, beneath the payments table
- **Production says:** `Payments Selected: 337` and `Deposit Total: 667,119.28` (blue) with buttons `Deposit` / `Cancel`
- **Prototype says/does:** No per-deposit footer; relies on the overlay's global `X/Y allocated` bar and payment-method breakdown card
- **Suspected file:** `index.html` — `renderAddTxDeposit()` (~line 3230)
- **Fix:** Add a right-aligned footer row inside the Payments card: `Payments Selected: {n}` and `Deposit Total: {fmtN(total)}`, updating with `depSel`. Keep the global allocation bar.
- **Done when:** Selecting/deselecting rows updates both labels with production wording.

## 13. Deposit slideout — payments grid missing Amount

> ✅ **FIXED & VERIFIED** (07/08/2026) — Right-aligned `Amount` column added to header and rows; grid widened to fit.

- **Location:** Deposit Details slideout → Payments card header row
- **Production says:** Payment rows carry amounts (deposit total is the sum shown in the footer)
- **Prototype says:** `Tenant | Date | Check # | Property | Unit | Comment | Address` — no Amount column
- **Suspected file:** `index.html` — `depositPaymentsCard()` (~line 4146) and `.so-reg-hdr/.so-reg-row` grid templates (~line 343)
- **Fix:** Add an `Amount` column (right-aligned, `fmtN`) to header and rows; widen the grid template accordingly.
- **Done when:** Each payment row shows its amount; column header reads `Amount`.

## 14. Slideout amount fields — currency symbol

> ✅ **FIXED & VERIFIED** (07/08/2026) — All three slideout cards (check/journal/generic) use `fmtN` — bare `1,240.00` verified; bill/deposit cards already used `fmtN`.

- **Location:** Check Details / Journal Details slideout, `Amount` field
- **Production says:** Amount inputs show plain numbers, e.g. `10.00` (no `$`)
- **Prototype says:** `${fmt(v.amount)}` renders `$1,240.00` (with `$`)
- **Suspected file:** `index.html` — `checkTopCard()` / `journalTopCard()` (~lines 4170, 4200); `fmt` vs `fmtN` (~line 1128)
- **Fix:** Use `fmtN(v.amount)` in field values inside slideout cards. Keep `$` in summary strips/balances only (production also shows bare numbers in `Cleared Balance: 0.00`).
- **Done when:** No `$` appears inside any form-style Amount field in slideouts.

## 15. Bill — Default Bank default value

> ✅ **FIXED & VERIFIED** (07/08/2026) — `< None >` is first and selected; account GL remains as the second option.

- **Location:** Add Transaction → Add Bill, Payment & Attachments card
- **Production says:** `Default Bank` defaults to `< None >`
- **Prototype says:** Pre-selects the account GL name (e.g. `1001 Operating`)
- **Suspected file:** `index.html` — `renderAddTxBill()` (~line 2836)
- **Fix:** Make `< None >` the first/selected option; keep the account GL as a second option.
- **Done when:** Bill form opens with `< None >` selected.

## 16. Bill — Links tiles extra CTA text

> ✅ **FIXED & VERIFIED** (07/08/2026) — Add Bill tiles show titles only (CTA links removed there; the Bill Details slideout keeps its links per the production details screen).

- **Location:** Add Transaction → Add Bill, Links section
- **Production says:** Tiles are titled `Purchase Order` and `Work Order Items` with empty bodies
- **Prototype says:** Tiles add links `Link Existing Purchase Order` / `Link Work Order Items`
- **Suspected file:** `index.html` — `renderAddTxBill()` (~line 2855)
- **Fix:** Remove the two CTA link lines (or keep tile bodies empty to match).
- **Done when:** Tiles show titles only.

## 17. Detail table — `$ Create Invoices` shown on wrong types

> ✅ **FIXED & VERIFIED** (07/08/2026) — `renderDetailTable` takes `pre.createInvoices`; only `renderAddTxBill()` passes it. Verified absent on Check, present on Bill.

- **Location:** Add Transaction → detail table footer (check, receipt, CC, generic)
- **Production says/does:** `Create Invoices` (greyed) appears **only** on Add Bill
- **Prototype says/does:** `renderDetailTable()` renders `$ Create Invoices` (muted) for every type that uses it
- **Suspected file:** `index.html` — `renderDetailTable()` footer (~line 3517)
- **Fix:** Add an option flag (e.g. `pre.createInvoices`) and pass it `true` only from `renderAddTxBill()`.
- **Done when:** `Create Invoices` appears on the Bill form only.

## 18. "Expenses" sub-heading is prototype-only

> ✅ **FIXED & VERIFIED** (07/08/2026) — `Expenses` (check/bill/CC) and `Journal Details` sub-headings removed.

- **Location:** Add Transaction → Check, Bill, CC forms above the detail table
- **Production says/does:** No heading above the allocation table (it sits directly below the form)
- **Prototype says:** `Expenses`
- **Suspected file:** `index.html` — `renderAddTxCheck()` / `renderAddTxBill()` / `renderAddTxCCTransaction()` (`at-sub-heading`, ~lines 2860, 3045, 3470)
- **Fix:** Remove the `Expenses` sub-heading (keep `Journal Details` removal too — production has no heading there either).
- **Done when:** No sub-heading between the form cards and the detail table.

## 19. Add-type menu — journal verb

> ✅ **FIXED & VERIFIED** (07/08/2026) — Dropdown reads `Add Journal`.

- **Location:** Add Transaction → `+ Add Transaction` dropdown
- **Production says:** Modal/title is `Add Journal`
- **Prototype says:** Menu item `Add Journal Entry`
- **Suspected file:** `index.html` — `addTxTypeVerb()` (~line 2200)
- **Fix:** `'Journal Entry':'Add Journal Entry'` → `'Journal Entry':'Add Journal'`. Also change the collapsed-item label mapping (`'Journal Entry'?'Journal':…` is fine) and card title `Journal Entry Information` → `Journal Information` only if you want strict parity; production's modal title is `Add Journal`.
- **Done when:** Dropdown reads `Add Journal`.

## 20. Bank Register — columns and header actions

> ✅ **FIXED & VERIFIED** (07/08/2026) — Kept the enhanced columns with an intentional-divergence comment at `_renderRegister()` referencing this item, and added an `Add Check` button (bank register only) next to Reconcile.

- **Location:** Bank Register screen
- **Production says:** Columns `Date | Vendor | Reference | Information | Cleared | Payee Name` (+ column chooser); buttons `Add Check` and `Reconcile`; balances `Cleared Balance:` and `Actual Balance:`
- **Prototype says:** Columns `Date | Reference | Type | Information | Account | Cleared | Deposit | Payment | Balance`; no `Vendor`/`Payee Name` columns; no `Add Check` button; adds a third balance
- **Suspected file:** `index.html` — register render (~lines 6040–6125)
- **Fix (judgment):** The Type/Account/Deposit/Payment/Balance columns are intentional Banking Center enhancements — keep them, but add a `Vendor` column after `Date` (data exists in `BR_TX` info strings) and an `Add Check` button next to the reconcile CTA for parity. Behavioral: mark as intentional divergence in a code comment if not changed.
- **Done when:** Either columns/actions match production, or a comment at the register render documents the intentional divergence (pick one; don't leave it implicit).

## 21. Bank Register — row kebab has no menu (behavioral)

> ✅ **FIXED & VERIFIED** (07/08/2026) — Kebab opens the six production items with exact casing; `Details` opens a read-only slideout for the row (form Date expanded to 4-digit year); other items close the menu as no-ops.

- **Location:** Bank Register row ⋮ button
- **Production does:** Opens menu: `Details`, `Delete`, `Void`, `Mark as reconciled`, `Print Check`, `Check Worksheet`
- **Prototype does:** Button renders but no menu opens
- **Suspected file:** `index.html` — register `rowsHtml` (~line 6027)
- **Fix:** Add a static popover with those six items exactly (casing as listed — note `Mark as reconciled` has lowercase "as reconciled"); `Details` opens the existing slideout; others can be no-ops.
- **Done when:** Kebab opens a menu with the six production items; Details opens the slideout.

## 22. List date formats (behavioral/formatting)

> ✅ **FIXED & VERIFIED** (07/08/2026) — Added `fmtD()`; applied to Transaction Review rows, expanded match/multi sub-tables, Find Match rows, and Smart Rec date cells (verified `06/15/26`, `06/11/26`, `04/22/26`). Form inputs keep `MM/DD/YYYY`; `DEP_PAYMENTS`/`BR_TX` were already 2-digit.

- **Location:** All list/table dates vs. form dates
- **Production does:** Lists use `MM/DD/YY` (e.g. `03/09/25`, `12/05/22`); form fields use `MM/DD/YYYY` (e.g. `07/08/2026`)
- **Prototype does:** Mixed — Transaction Review rows use `MM/DD/YYYY`; `DEP_PAYMENTS`/`BR_TX` use `MM/DD/YY`
- **Suspected file:** `index.html` — `TRANSACTIONS` (~line 992) and row templates
- **Fix:** Keep `MM/DD/YYYY` inside form inputs; render list/table dates as `MM/DD/YY`. Add a tiny `fmtD(d)` helper rather than editing every literal.
- **Done when:** Every table date shows a 2-digit year; every form input shows a 4-digit year.

## 23. Receipt type — verify it exists in production

> ⏸ **NOT CHANGED** (07/08/2026) — Left unchanged pending BA verification, as instructed.

- **Location:** Add Transaction dropdown → `Add Receipt`
- **Production says/does:** Not observed anywhere in the production flows audited (deposits handle incoming payments; "Other Income" covers vendor/owner receipts)
- **Prototype says:** Offers `Receipt` type with `Receipt Information` card (Property/Vendor/Amount/Receipt Date/Post Date/Category/Memo)
- **Suspected file:** `index.html` — `getTxTypes()` (~line 2234), `renderAddTxReceipt()` (~line 3047)
- **Fix:** Verify with the BA team whether RMX has an "Add Receipt" transaction. If not, remove `Receipt` from `getTxTypes()` or replace with the deposit `Add Other Income` path. **Do not change until verified.**
- **Done when:** Type list contains only transactions that exist in RMX.

## 24. Not captured in production — verify before trusting prototype content

> ℹ️ **NO ACTION** (07/08/2026) — No action — informational.

These production surfaces weren't reachable in this audit; the prototype content is unverified
against them (not necessarily wrong):

- `Terms` dropdown options on Add Bill (prototype: `Immediately / Net 15 / Net 30 / Due on receipt`)
- `Payment Method` options on Add Bill (prototype: `Check / ACH / Wire / Zelle`)
- Electronic Credit Card Reconcile "Add Credit Card Transaction" field set (prototype `renderAddTxCCTransaction()`, ~line 3420)
- Make Deposit → `Add Other Income` popup columns (prototype: `Vendor/Owner | Property | Unit | Reference | Income Account | Job | Memo | Amount`)
- NSF flow copy (production NSF is initiated from a tenant payment; modal copy not captured). Prototype NSF journal prefill (~line 3247) is demo-invented.
- Tenant charge for reference, captured: production modal is titled `Transaction Detail` with fields `Charge Type`, `Unit`, `Date`, `Amount` + `Prorate` link, `Comment`, `Other Information` → `Reference #`; buttons `Save & New` / `Save & Close` / `Cancel`.

---

### Confirmed matches (no action)

- `Fill from history` (check) and `Fill from bill history` (bill) — exact
- Detail table columns `Property | Unit | Expense Account | 1099 | Job | Memo | Billable | Billable To | Markup | Amount` — exact
- `+ Add Detail`, `$ Disburse Amount`, `≡ Clear Allocations`, `Total:` — exact (check/bill)
- Job placeholder `<Unassigned>` — exact
- Deposit picker: `Properties` (`All selected`), `Search` (`Find a payment`), hint `Select the payments you want to add to this deposit`, `Add Other Income`, `Print` — exact
- Register: `Find a transaction`, `< No Filter >`, `Filter By Transaction Date`, `Cleared Balance:`, `Actual Balance:` — exact
- Journal fields `Reference`, `Period Adjustment`, `Memo`, attachments `Upload | Paste` — exact
- Amounts in tables: comma-separated, two decimals, no `$` — matches production
