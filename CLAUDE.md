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
- Bank Balance: $516,931.60 | In RM: $512,000.00 static; the TR strip computes live and, with t6
  pre-excluded, shows In RM $511,681.60 / Difference **−$5,250.00** at rest (−4,931.60 is only the
  static `difference` field). Resolving all remaining items still proves to $0.00.
- 8 items whose total net = +$4,931.60: +1,850 (t1 Rivera) +2,100 (t2 Okafor) −1,240 (t3 Rose City)
  +6,939 (t4 Zego batch — carries `depPayments`, 12 seeded resident rows that sum exactly to 6,939.00; select-all balances the split) −2,860 (t5 Cascade HVAC) −318.40 (t6 café, pre-excluded via `SEED_EXCLUDED`)
  −1,150 (t7 NSF Reyes) −389 (t8 Recology dup)
- Review beats: Match (t1), Select Match (t2), Add New (t3), Split (t4), Match (t5 — was the Find
  Matches beat; now pre-suggested to Check 2212: Cascade HVAC), NSF Return (t7), Possible Duplicate (t8);
  t6 starts in the Excluded tab. TR opens with In Review (7) / Matched (0) / Excluded (1);
  `txStatus.count` is 4 (static, drives the register strip only)
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

## Key render functions

`renderBC`, `renderTR`, `renderBR`, `renderSlideout`, `renderFindMatch`, `openAddTx`, `openReconcile`

## Notable CSS patterns

- RMX token pass applied (per the rmx-prototyping skill): page bg `#F3F4F8`, notice amber `#FAA61C`, checked checkboxes `#F79B4D` (blue `#008dd5` for register-header select-alls), btn hovers `#0071AA`/`#EBF1F5`, register headers 12.6px/500/+1.1px on `#737373`, italic `#b3b3b3` placeholders, Orion chat border = Orion_2 gradient (`#008dd5→#6eb744`) with blue glow. Native `<select>` popups are suppressed app-wide by the `.rmx-dd` delegated component — options render in an RMX floating panel while the native element keeps state and fires its own change events (no form logic touched). Statement End Date stays a native date input by explicit request.

- Banking Insights tile REMOVED from the Banking Center (08/28/2026) — the `INSIGHTS` data, `openInsights()` overlay, and `.bc-card.hi::before` gradient-border CSS (`-webkit-mask` composite trick) remain in the file unused, so it can be restored by re-adding the `orionInsights` tile entry
- Background image inlined as base64 data URL on `#screen-bc` to avoid GitHub Pages path issues
- Find Match overlay: grey `#f2f2f2` background via `#find-match-root .at-modal`

## Figma file

`https://www.figma.com/design/xvuCf7Sg3DKmYJjupeEQvm/Banking-Center`
