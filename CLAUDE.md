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
- The Bank/Credit Card Register strip tracks the same resolved feed (`resolvedDelta`), so Actual
  Balance always equals the Banking Center's Cleared Balance while each account's static
  Actual-vs-Cleared gap is preserved.
- **1001 Operating opens reconcile-ready (09/14/2026):** t1–t5, t7, t8 are seeded into `SEED_MATCHED`
  and t6 into `SEED_EXCLUDED`, so TR opens In Review (0) / Matched (7) / Excluded (1) and the Reconcile
  button is enabled. All eight beats still exist in the data — remove their ids from `SEED_MATCHED`
  to put them back into review. `txStatus` is `{type:'matched',count:0}`.
- Beat inventory (now in the Matched/Excluded tabs): Match (t1), Select Match (t2), Add New (t3),
  Split (t4), Match (t5 → Check 2212: Cascade HVAC), NSF Return (t7), Possible Duplicate (t8), t6 excluded
- **Outstanding-bill beat lives in 1007 Maintenance Escrow (`me6`)** — a synced $1,450 Cascade HVAC ACH
  that Orion ties to open Bill B1183. The row names the bill in the Rent Manager column and its action reads **Match**, which opens the
  Add overlay on the `Bill Payment` type, whose "Bills Paid" grid applies the payment to the bill;
  "Add as Check" is offered as the lesser alternative.
- Suggested-match confidence tiers shown in an unlabeled column of filled lozenges (fixed-position why-tooltip above the row): t1=rule (High, green), t2/t3/t5/t7=ai (Medium, amber), t8=hint (Low, pink)
- `SEED_MATCHED` pre-matched examples (survive account-switch resets) cover every record type:
  Bill/Deposit/Check/Journal in 1004 Trust Comm + 1007 Maintenance, Charge/Credit in 2001 Mastercard
  + 2003 Amex
- **1004 and 1007 open nearly clean (09/15/2026):** 1004 Trust Comm. shows Matched (12) / In Review (1)
  - the one left is `tc1`, a 4,200.00 ACH debit with no suggestion, so its action is **Add**, and it is
  exactly the account's Bank-vs-Cleared difference. 1007 Maintenance Escrow shows Matched (8) / In
  Review (1) - only the open-bill beat `me6`. Their `inRm`/`cleared` opening balances were lowered by
  the newly seeded net (1004 by 4,987.50, 1007 raised by 2,300) so every displayed balance is
  unchanged from before the reseed. Each account then gained 8 more already-matched items (`tcv1-8`,
  `mev1-8`) that net **exactly $0.00**, so history looks worked without touching any balance - 1004
  reads Matched (20), 1007 Matched (16). Each also carries two In Review rows that DO have a
  suggestion (`tcr1`/`tcr2`, `mer1`/`mer2` - one High/rule, one Medium/ai), so **In Review is 3 on
  both**: the beat row plus two one-click Matches. Unresolved rows do not move a balance, so the
  displayed figures are still untouched. `acctTxs()` sorts every feed
  newest-first, so inserted rows cannot break a feed's chronology. **The bulk-Create beat no longer
  has 4 creatable rows anywhere** - restore it by removing `me1`,`me2`,`me3`,`me5` from `SEED_MATCHED`,
  clearing their `rmLink`/`matchStatus`, and re-lowering 1007's `inRm`

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

`openConnect()` → overlay in `#connect-root`, styled as the **Quiltt Connector** from the reference
screenshots in `~/Downloads/Quiltt` (403px modal, `#faf9fd` ground, purple `#6d28d9` accents, back/X
chevrons, institution card grid, "powered by Quiltt" footer). Deliberately does NOT follow RMX - it
is a third-party surface. Steps: choose institution (searchable, 9 banks in `CONNECT_BANKS`, including fictional **Legacy Bank** whose single find is a plain Business Checking ••3092) → sign
in (visual only, no credentials collected) → a `connecting` step with the wire/sync graphic →
select accounts found at that bank (`CONNECT_FOUND`) → done. Picked accounts are appended to `ACCOUNTS` with `isNew:true`, which
carries no badge any more (the **New** lozenge was removed 09/15/2026 from the TR sidebar, the
Accounts Overview and the connector's done step; `isNew` and the `.sb-new`/`.ao-new` CSS remain, so
re-adding one span restores it), and they are registered
visible, which is all Settings › Institutions needs - that list is **derived from `ACCOUNTS`**
(`institutions()`), so it always matches the Accounts Overview: one card per `bankName`, one row per
linked account, status `action-required` when any account at that bank is `connection:'login'` and
`connected` otherwise. Unlinked accounts (Petty Cash) belong to no institution. Collapsed state lives
in `INST_COLLAPSED`, descriptions in `INST_DESC`. Entry points: Add Account
in the TR sidebar (expanded + collapsed) and the **Connect** link on unlinked accounts in the BC table.

## Banking Center / Reconciliation conventions (09/14/2026)

- BC tiles have no icon badges; the Accounts Overview table dropped its **Difference** column and
  "In Rent Manager" is now **Cleared Balance** (also in the TR summary strip and the BC tiles)
- The Accounts Overview carries a **Last Reconciled** column between Cleared Balance and
  Transactions, from a per-account `lastRec` field (the Start Reconciliation overlay reads the same
  field, falling back to `REC_LAST_REC_DATE`). Most accounts closed 05/31/2026; 1007 Maintenance
  Escrow sits at 04/30/2026 and 2001 Mastercard at 03/31/2026, so the column shows real variance.
  An account with no `lastRec` renders a grey "Never"
- Connection Status shows an explicit green **Connected** lozenge; accounts with `connection:'none'`
  (Petty Cash) show "Not connected" + a Link Account link and **no** match status at all. Every cell in a
  row is 14px - no per-cell size overrides
- **Connection Health tile (Option A, 09/15/2026):** the summary line stays ("4 of 6 accounts syncing",
  "All 6" when clean), and under a hairline every account that needs something is NAMED with the action
  that fixes it - lapsed accounts first (`Citi****2247 - last synced 5 days ago` / **Reconnect**), then
  unlinked (`No bank account linked` / **Link Account**). Both run the Quiltt connector. With nothing to
  act on the tile collapses to the one summary line
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
4. **Open bill detection** - `me6` (Cascade HVAC ACH) names Bill B1183 and reads **Match**; matching opens the Bill Payment form, applying the payment to the bill.
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
  -- enforced globally by `::placeholder{font-style:italic!important}` (09/15/2026); several inputs had
  been italicising the TYPED value, which is now upright everywhere. The Quiltt connector opts out
  (`.qc-search input` / `.qc-input` placeholders stay upright) since it is a third-party surface
- **Transaction Date filter** is the RMX **Input Field / Date Range** (RMX-Components 1106:12535): one
  36px `#cedbe7` r4 control holding From input, calendar button, To input, calendar button and the
  relative-date button, all on `#f5f8fa`, placeholders `mm/dd/yyyy`. The calendar buttons open the RMX
  **Date Picker** (5350:5786) - `dpOpen`/`dpRender`, a fixed-position white popover with month chevrons,
  a 32px 7-column grid, striped alternate rows, `#008dd5` selected day and Today / Clear links. Picking
  a date fills the field and filters the rows (`S.dateFrom` / `S.dateTo`); the third button clears both.
  All four glyphs are the Figma exports inlined verbatim
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
credentials to the institution and Rent Manager never sees or stores them. Three entry paths, all through the same connector:
- **Connect** - `openConnect()` from Add Account (TR sidebar) or Add Institution (Settings). Full flow;
  picked accounts become new `ACCOUNTS` entries.
- **Reconnect** - `openConnect(bankName)` from the Reconnect link in the Accounts Overview table or on
  a login-required account header. Skips institution selection, opens on "Reconnect your account" for
  that bank, and on success restores the existing accounts (`connection:'fresh'`) and flips the
  institution back to Connected rather than linking new ones.
- **Link Account** - `openConnect(null, acctId)` from an unlinked account's Link Account link. Runs the
  full flow but the account step is single-select and reads "Pick the one to link to <GL account>";
  finishing attaches the chosen bank account to that existing GL account instead of creating one.

## Visible accounts

Only accounts carrying a demo beat are visible (`SETTINGS_STATE.accounts[id].visible`): 1001
Operating, 1004 Trust Comm., 1007 Maintenance Escrow, 1009 Petty Cash, 2001 Mastercard, 2003 Amex.
Payroll / Trust Res. / Owner Disbursement / Visa Maintenance / Visa Vendor are hidden but intact.
Balance counts and the reconcile account picker are scoped to visible accounts.

## Notable CSS patterns

- RMX token pass applied (per the rmx-prototyping skill): page bg `#F3F4F8`, notice amber `#FAA61C`, checked checkboxes `#F79B4D` everywhere (incl. header select-alls, per user preference over the RMX blue-select-all variant), btn hovers `#0071AA`/`#EBF1F5`, register headers 12.6px/500/+1.1px on `#737373`, italic `#b3b3b3` placeholders, Orion chat border = Orion_2 gradient (`#008dd5→#6eb744`) with blue glow. Native `<select>` popups are suppressed app-wide by the `.rmx-dd` delegated component — options render in an RMX floating panel while the native element keeps state and fires its own change events (no form logic touched). Statement End Date stays a native date input by explicit request.

- Banking Insights tile REMOVED from the Banking Center (08/28/2026) — the `INSIGHTS` data, `openInsights()` overlay, and `.bc-card.hi::before` gradient-border CSS (`-webkit-mask` composite trick) remain in the file unused, so it can be restored by re-adding the `orionInsights` tile entry
- Background image inlined as base64 data URL on `#screen-bc` to avoid GitHub Pages path issues
- **Favicon:** the RMX brand house (RMX-Iconography `doVDYRtepBULKGZntmmS5B` node **68:2992**),
  recoloured from `#008DD5` to **`#3776BC`**. Kept at `assets/favicon.svg` and inlined in the head as a
  percent-encoded `data:image/svg+xml` URI, so it resolves on GitHub Pages, the dev server and file://
- Find Match overlay: grey `#f2f2f2` background via `#find-match-root .at-modal`

## Figma file

`https://www.figma.com/design/xvuCf7Sg3DKmYJjupeEQvm/Banking-Center`
