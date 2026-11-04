# Admin personal expense reimbursements

## Change record

This change adds an automatic Admin-only marker for project vouchers paid
personally by the Admin. A voucher is marked when its `Done by` value matches
the Admin's name (or `Admin`/`Administrator`). Marked vouchers remain ordinary
project expenses. Project spend totals include them; Budget vs Actual excludes
them. The tracking start date is 3 November 2026 (midnight IST), excluding
earlier vouchers from personal-spend and reimbursement calculations.

The Admin's project release can optionally include a personal reimbursement.
That amount is paid out of the release total, the remainder is credited to the
project account, and the selected Admin's outstanding personal spend decreases
by the amount reimbursed. Partial reimbursements are supported. The Admin can
open their own marked voucher list from each project row to see the amount paid
back and the remaining amount. Other project spend views include these expenses.

## Files changed

- `src/db.js`: creates `admin_personal_expenses`, tracks reimbursed amounts,
  and adds project/personal destination details to funds movements.
- `src/api.js`: records Admin personal expenses, applies reimbursements, and
  excludes personal expenses only from Budget vs Actual.
- `public/app.js`: displays personal spend by project and the live release split.

## Data and rollback

No existing voucher rows are rewritten or backfilled. Schema initialization
sets the fixed tracking start date. Only vouchers created from that point
onward are added to personal-spend and reimbursement calculations.

To roll back the feature, restore the three code files above to their prior
versions. Before dropping the table, back up any `admin_personal_expenses`
rows because those rows record which vouchers were personally funded. After
backup, the table and index can be removed
with:

```sql
DROP TABLE IF EXISTS admin_personal_expenses;
```

The rollback does not require changing vouchers, budgets, existing fund
requests, or existing funds ledger rows. Before removing the table, preserve
its reimbursement amounts if you need to retain partial payment history.
Releases that included personal payments deduct the full amount from
`ADMIN-FUND` and credit the split amounts to the respective destinations; review
those releases before rolling back the split-payment behavior. To remove the
additional movement metadata after backing up, drop the `target_project_id` and
`personal_amount` columns from `funds`.
