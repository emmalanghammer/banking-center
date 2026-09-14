# Banking Center Prototype

Single-file HTML/CSS/JS prototype for a leadership demo of the Banking Center → Transaction Review → Bank Register → Reconciliation flow.

## Key facts

- **All code lives in one file:** `banking-center/index.html`
- **Dev server:** name `banking-center`, port `3456` — start with `preview_start("banking-center")`
- **GitHub:** `emmalanghammer/banking-center`
- **Two branches:** `main` (source), `mvp` (always FF-merge main → mvp after pushing)

## Git workflow

```bash
git add index.html
git commit -m "..."
git push origin main
git checkout mvp && git merge --ff-only main && git push origin mvp && git checkout main
```

All git commands must be run from inside `banking-center/` or use `git -C banking-center`.

## Demo company: Cascade Property Group

Portland, OR residential + light-commercial property manager. Demo "today" = mid-June 2026.

- **Properties:** Hawthorne Court Apartments, Pearl District Lofts, Alberta Commons, Sellwood Row
- **Owners/LLCs:** Hawthorne Holdings LLC, Pearl District Partners, Alberta Commons Group
- **Vendors:** Portland General Electric, NW Natural, Rose City Plumbing, Cascade HVAC, Stumptown Landscaping, Recology, CenturyLink, Standard Insurance, City of Portland, Multnomah County
- **Banks:** Chase, Umpqua Bank, U.S. Bank, KeyBank, Wells Fargo
- **Tenants:** Rivera, Okafor, Delgado, Nguyen, Park, Reyes, Flores, Walsh, Chen, Henderson, Steinberg, Johnson, Morrison

## Hero account

**1001 Operating (`op3487`)** — Chase ••3847  
- Bank Balance **$516,931.60** | Cleared Balance **$516,931.60** | Difference **$0.00**, consistently
  on the Banking Center and in Transaction Review. `inRm: 512000` is the OPENING book balance;
  `acctLiveInRm()` adds the resolved feed on top, and every surface uses it. Do not raise the static
  `inRm` to match the displayed figure - the live path would double-count.
- **1001 Operating opens reconcile-ready (09/14/2026):** t1–t5, t7, t8 are seeded into `SEED_MATCHED`
  and t6 into `SEED_EXCLUDED`, so TR opens In Review (0) / Matched (7) / Excluded (1) and the Reconcile
  button is enabled. All eight beats still exist in the data — remove their ids from `SEED_MATCHED`
  to put them back into review. `txStatus` is `{type:'matched',count:0}`.
- Beat inventory (now in the Matched/Excluded tabs): Match (t1), Select Match (t2), Add New (t3),
  Split (t4), Match (t5 → Check 2212: Cascade HVAC), NSF Return (t7), Possible Duplicate (t8), t6 excluded
- **Outstanding-bill beat lives in 1007 Maintenance Escrow (`me6`)** — a synced $1,450 Cascade HVAC ACH
  that Orion ties to open Bill B1183. Primary action **Create Bill Payment** (`openBillPayment`) opens the
  Add overlay on the `Bill Payment` type, whose "Bills Paid" grid applies the payment to the bill;
  "Add as Check" is offered as the lesser alternative.
- Suggested-match confidence tiers shown in an unlabeled column of filled lozenges (fixed-position why-tooltip above the row): t1=rule (High, green), t2/t3/t5/t7=ai (Medium, amber), t8=hint (Low, pink)
- `SEED_MATCHED` pre-matched examples (net $0.00 per account, survive account-switch resets) cover every record type: Bill/Deposit/Check/Journal in 1004 Trust Comm + 1007 Maintenance, Charge/Credit in 2001 Mastercard + 2003 Amex

## Key data locations (approx line numbers)

| Data | ~Line |
|------|-------|
| `INSIGHTS` | 937 |
| `ACCOUNTS` array | 976 |
| `TRANSACTIONS` (hero feed) | 992 |
| `TRANSACTIONS_BY_ACCT` | 1012 |
| `BR_TX` (Bank Register rows) | 5162 |
| `recData` (Smart Reconciliation rows: matched bank txs + RM-only Review items) | ~5466 |

## Bulk selection (Transaction Review)

QuickBooks-style multi-select: a checkbox column leads each row plus a header select-all
(scoped to the rows visible in the active tab via `visibleTxIds()`). Selecting any row reveals
the `.bulk-bar` — a floating overlay pinned to the bottom of the table (RMX filled mode-banner
treatment: navy `#13314C`, r4, hugging its content, elevated) with **Match / Edit / Exclude** +
Clear selection. Its tooltips open upward so they clear the table's bottom edge.
A bulk action is offered only when EVERY selected row qualifies (`bulkEligibility()`): Match is
disabled if any selection lacks a suggestion or is already matched, Exclude if any is already
excluded — the disabled button carries a tooltip explaining which rows block it, and the handlers
re-check before acting (no partial application);
**Create** records a new Rent Manager transaction for every selected unmatched item at once
(`bulkCreate()`, the mirror of Match — enabled only when no selected row has a suggestion); it infers
Deposit/Check/Bill from the description and assigns refs from `NEXT_CREATED_REF`.
Bulk Edit opens an overlay applying Property / GL Account / Memo across the selection.
`S.selectedIds` is cleared on account switch so actions can never touch off-screen rows.

## Connect a Bank flow

`openConnect()` → 4-step overlay in `#connect-root`: choose institution (searchable, 8 banks in
`CONNECT_BANKS`) → sign in (visual only, no credentials collected) → select accounts found at that
bank (`CONNECT_FOUND`) → done. Picked accounts are appended to `ACCOUNTS` with `isNew:true`, which
renders a **New** badge in the TR sidebar and the Accounts Overview table. Entry points: Add Account
in the TR sidebar (expanded + collapsed) and the **Connect** link on unlinked accounts in the BC table.

## Banking Center / Reconciliation conventions (09/14/2026)

- BC tiles have no icon badges; the Accounts Overview table dropped its **Difference** column and
  "In Rent Manager" is now **Cleared Balance** (also in the TR summary strip and the BC tiles)
- Connection Status shows an explicit green **Connected** lozenge; accounts with `connection:'none'`
  (Petty Cash) show "Not connected" + a Connect link and **no** match status at all
- TR sidebar rows are quiet by default — a status only appears when the account needs something
  (`N to review` or `Login required`); unlinked accounts read "No account linked"
- The screen is called **Reconciliation**, not Smart Reconciliation
- **No statement concept**: the start overlay shows Bank Account / Last Reconciliation Date /
  Beginning Balance / Bank Balance / Transactions to Reconcile (no upload, no statement date or
  ending balance). `REC_STATE.statementBalance` still exists internally and defaults to the account's
  bank balance, which is what the strip's **Bank Balance** cell and `srDifference()` compare against
- Reconcile rows are **single-line** (bank side); the Rent Manager record behind each row appears in a
  fixed-position hover tooltip (`data-rmtip` → `.rm-tip`). No close-forecast/pace banner

## Video script demo path (Attenborough spot)

1. **Banking Center** — tiles (Bank Balance, Credit Card Balance, Unmatched Transactions, Category
   Breakdown, Connection Health) + Accounts Overview with per-account Connected state. Connection
   Health carries "N of M accounts syncing" and **18,000+ financial institutions**, which the VO names.
2. **Connect a Bank** (optional beat) — search placeholder and helper line both cite 18,000+ institutions.
3. **Review Transactions → 1007 Maintenance Escrow** (6 in review; 1001 Operating is deliberately at 0
   so it can be reconciled). Select the four unassigned expenses (me1/me2/me3/me5) → floating bulk bar →
   **Create** → "4 transactions created in Rent Manager", leaving the bill beat next.
4. **Open bill detection** — `me6` (Cascade HVAC ACH) → Create Bill Payment → applies to Bill B1183.
5. **Reconciliation** — 1001 Operating, Reconcile enabled, single-line rows, proves to $0.00.

## Key render functions

`renderBC`, `renderTR`, `renderBR`, `renderSlideout`, `renderFindMatch`, `openAddTx`, `openReconcile`

## App header

Built from the RMX Components library, `Header` node **18:2314**
(`figma.com/design/YhvzfcXOniQJ7xlC8ONzS4`). One `.top-hdr` in the app shell, shared by every screen.
Three equal `flex:1 0 0` zones so Command Launch centres in the bar: logo lockup (175×32) ·
icon cluster (39/45/38 × 32, 1px gaps, `#425a70`, outer corners r4) + 454px search bar ·
company code + bell + 32px avatar (32px gap, 20px between bell and avatar).
All six glyphs (logo, menu, reports, grade, search, notifications) are the Figma-exported SVGs
inlined verbatim — the reports glyph is an Express custom icon with no Material equivalent.
The search bar is allowed to shrink below 454px so narrow viewports don't push the manage zone off.

## RMX consistency pass (09/14/2026)

Applied with the `rmx-prototyping` skill to every surface **except the Banking Center**, which is
deliberately styled differently (all `.bc-*` / `.ao-*` / `.qa-*` rules are skipped), and except the
header/context bar already built to their own Figma components.

- Controls standardised at **36px / 14px**: inputs, selects, buttons, segmented controls
- Input Field anatomy: `#F5F8FA` fill, `#CEDBE7` border, blue on focus only, italic = placeholder only
- Register anatomy: 28px headers, 36px rows · Tile header rule is **2px `#008DD5`** (was navy)
- Type ramp snapped to 12/14 · stray radii (3/5/6px) → 4px
- **Deliberate divergence:** the Bank Register filter bar stays blue-bordered on white, matching
  Figma 2732-43969 as originally requested
- Custom selects (`.csel-menu`) switch to **fixed positioning while open** (`_cselPlace`) so overlay
  bodies that scroll can't clip them; they flip above the trigger when short on room
- Reconcile row hover uses the RMX hover tint `#EBF1F5`, and the hover card is the RMX
  **Tooltip Text** component (RMX-Components 31:303): 312px white card, 1px `#CEDBE7`, 4px radius,
  16px padding, bordered title row, 14px/20px. The row's insight icon uses the same card
  ("Transaction Insight") and takes priority on hover, so the two never stack
- Reconcile strip: amounts are regular weight; only **Bank Balance** and **Difference** stay bold

## Connect a Bank — Quiltt

The connector is framed as **Quiltt** (the aggregator in use): "Secured by Quiltt" in the modal
header, 18,000+ institutions attributed to Quiltt, and the sign-in step explains that Quiltt passes
credentials to the institution and Rent Manager never sees or stores them. Launched from
**Add Account** in the TR sidebar and **Add Institution** in Settings › Institutions.

## Visible accounts

Only accounts carrying a demo beat are visible (`SETTINGS_STATE.accounts[id].visible`): 1001
Operating, 1004 Trust Comm., 1007 Maintenance Escrow, 1009 Petty Cash, 2001 Mastercard, 2003 Amex.
Payroll / Trust Res. / Owner Disbursement / Visa Maintenance / Visa Vendor are hidden but intact.
Balance counts and the reconcile account picker are scoped to visible accounts.

## Notable CSS patterns

- RMX token pass applied (per the rmx-prototyping skill): page bg `#F3F4F8`, notice amber `#FAA61C`, checked checkboxes `#F79B4D` everywhere (incl. header select-alls, per user preference over the RMX blue-select-all variant), btn hovers `#0071AA`/`#EBF1F5`, register headers 12.6px/500/+1.1px on `#737373`, italic `#b3b3b3` placeholders, Orion chat border = Orion_2 gradient (`#008dd5→#6eb744`) with blue glow. Native `<select>` popups are suppressed app-wide by the `.rmx-dd` delegated component — options render in an RMX floating panel while the native element keeps state and fires its own change events (no form logic touched). Statement End Date stays a native date input by explicit request.

- Banking Insights tile REMOVED from the Banking Center (08/28/2026) — the `INSIGHTS` data, `openInsights()` overlay, and `.bc-card.hi::before` gradient-border CSS (`-webkit-mask` composite trick) remain in the file unused, so it can be restored by re-adding the `orionInsights` tile entry
- Background image inlined as base64 data URL on `#screen-bc` to avoid GitHub Pages path issues
- Find Match overlay: grey `#f2f2f2` background via `#find-match-root .at-modal`

## Figma file

`https://www.figma.com/design/xvuCf7Sg3DKmYJjupeEQvm/Banking-Center`
