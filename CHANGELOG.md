# Changelog

Tracks notable updates to the payslip parsing logic in `index.html`, especially
changes driven by new `Response` text formats coming from the Google Sheet.
Keeping this updated lets future sessions quickly trace what formats are
already supported instead of re-deriving it from the regexes.

## 2026-06-16 — Earnings breakdown support

Google Sheet started producing `Response` text with an earnings breakdown
before the `GROSS:` total, e.g.:

```
Date:06/15/2026 Toledo, Darwin Gabarda  SALARY:10,691.31 Earnings: 9,870. +1,678.(Fr.+Ml.)= GROSS:11,548. LESS:PREM (05/26) = 856.69,  Total Less= 856.6875   TOTAL PAY:10,691.31     *This is a system-generated msg. Please do not reply
```

Confirmed variants (all handled by the same regex, since the parenthesized
label is treated as an opaque string):

- Single component: `Earnings: 9,730. +550.(Meal)= GROSS:10,280.`
- Combined components: `Earnings: 9,870. +1,678.(Fr.+Ml.)= GROSS:11,548.`
- Multiple combined components: `Earnings: 7,800. +4,565.(Fr.+Ml.+Acmd)= GROSS:12,365.`
- Multiple deduction lines also confirmed working: `LESS:PREM (05/26) = 1,314., St. Peter (1100)= 550.,  Total Less= 1864`

**Change:** `parsePayslip()` in `index.html` now also extracts
`earningsBase`, `earningsExtra`, `earningsExtraLabel` via:

```js
/Earnings:\s*([\d,]+)\.?\s*\+\s*([\d,]+)\.?\(([^)]+)\)\s*=/i
```

`buildPayslipCard()` renders these as two extra rows ("Basic Pay" and
"+ <label>") above "Gross Earnings" whenever `earningsBase` is present.
If the `Response` text has no `Earnings:` breakdown (old format), those
rows are skipped and the card looks the same as before.

Existing fields (`SALARY:`, `GROSS:`, `LESS:` deductions, `Total Less=`,
`TOTAL PAY:`, name/date extraction) required no changes — the new format
is a superset of the old one.
