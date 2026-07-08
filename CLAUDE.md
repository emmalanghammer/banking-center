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
- Bank Balance: $516,931.60 | In RM: $512,000.00 | Difference: −$4,931.60  
- 8 review items whose net = +$4,931.60 (drives difference to $0 when all cleared):
  +1,850 (t1 Rivera) +2,100 (t2 Okafor) −1,240 (t3 Rose City) +6,939 (t4 Zego batch)
  −2,860 (t5 Cascade HVAC) −318.40 (t6 café) −1,150 (t7 NSF Reyes) −389 (t8 Recology dup)
- Review beats (order fixed for the Useberry study): Match, Select Match (multi), Add New,
  Split (inside Add), Find Matches, Exclude — then NSF Return (t7), Possible Duplicate (t8)
- Suggested-match confidence tiers shown in an unlabeled column of filled lozenges (fixed-position why-tooltip above the row): t1=rule (High, green), t2/t3/t7=ai (Medium, amber), t8=hint (Low, pink)

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

- Banking Insights gradient border: `.bc-card.hi::before` with `-webkit-mask` composite trick (native `border-image` doesn't work with `border-radius`)
- Background image inlined as base64 data URL on `#screen-bc` to avoid GitHub Pages path issues
- Find Match overlay: grey `#f2f2f2` background via `#find-match-root .at-modal`

## Figma file

`https://www.figma.com/design/xvuCf7Sg3DKmYJjupeEQvm/Banking-Center`
