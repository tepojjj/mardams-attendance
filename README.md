# Mardams Attendance & Payroll

A standalone companion app to the **Mardams Apparel — Job Order Form**
app. It holds everything that used to live on the Job Orders app's
Attendance and Payroll tabs:

- **Clock In / Clock Out** — every account (Staff, Admin, Accounting,
  Super Admin) punches their own attendance here, in four sequential
  steps each day: Morning In → Lunch Out → Afternoon In → End-of-Shift
  Out. A separate, optional Overtime In/Out pair unlocks once the
  regular shift is finished. Every timestamp is always taken from the
  server clock, never the browser's.
- **Attendance Report** — Accounting and Super Admin only. The full
  punch log across every employee (all six timestamps per day), with
  Late/Undertime badges based on each employee's regular shift times
  and an OT badge for days with overtime.
- **Payroll** — Accounting and Super Admin only. For each configured
  employee, computes: Base Pay (Daily rate × days present, or flat
  Fixed Monthly), + Overtime Pay (OT hours × that employee's OT rate),
  + Allowance, − SSS/PhilHealth/Pag-IBIG/Cash Advance deductions, =
  Net Pay. Click a row for the full breakdown plus that employee's
  day-by-day attendance for the period. OT Rate, Allowance, and the
  four deductions are set right here, per employee, via the Edit
  button on each payroll row — by Accounting or the Super Admin.

Job Orders itself (New/Browse/Monitoring Sheet/Analytics/Users/Account
Log) stays in the other app — this one is purely the workforce/payroll
side, so Sign Ads and Accounting accounts (who don't touch Job Orders at
all) have a place to log in.

## This app shares accounts with the Job Orders app

Both apps read and write the **same** account list, attendance log, and
payroll settings — there's only ever one set of usernames/passwords.
For that to work when they're deployed as two separate Vercel projects,
**both projects must be pointed at the same storage and use the same
session secret**:

1. Deploy the Job Orders app first (if you haven't already) and create
   its Vercel KV database, `AUTH_SECRET`, `SUPERADMIN_USERNAME`, and
   `SUPERADMIN_PASSWORD` env vars, per that app's own README.
2. Deploy **this** app as its own Vercel project (see below).
3. In this project's **Settings → Storage**, connect the **same** KV
   database you created for the Job Orders app (don't create a new
   one) — this is what makes "one account, works in both apps" true.
4. In this project's **Settings → Environment Variables**, set
   `AUTH_SECRET` to the **exact same value** as the Job Orders app.
   This is what lets a session created by logging into one app also be
   understood as valid by the other (though in practice, since these
   are two different domains, everyone still logs in separately on
   each one — this just means they use the same username/password).
5. Redeploy so the env vars and storage connection are picked up.

No `SUPERADMIN_USERNAME` / `SUPERADMIN_PASSWORD` are needed here —
the Super Admin account is bootstrapped once, from the Job Orders app.

## Deploying to Vercel

Same as the Job Orders app — this needs a Git-based deploy or the
Vercel CLI, because it includes backend functions.

### Option A — Vercel CLI

```
npm install -g vercel
vercel --prod
```

### Option B — Connect a Git repo

Push this folder to its own GitHub repo, then in the Vercel dashboard:
**Add New… → Project**, import the repo, framework preset **Other**,
Deploy.

## Accounts & access

Accounts themselves are still created and managed from the **Job
Orders app's Users tab** (Admin/Super Admin only) — this app doesn't
have a Users tab of its own. What matters here is role:

- **Everyone** can clock in/out on the Attendance tab.
- **Accounting** and **Super Admin** additionally see the Attendance
  Report and Payroll tabs, and can edit OT Rate, Allowance, and the
  SSS/PhilHealth/Pag-IBIG/Cash Advance deductions from the Payroll
  tab's per-row Edit button.
- **Staff, Admin, and Sign Ads-department accounts** only see the
  clock in/out widget — no report, no payroll figures.

Pay Type, Rate, and regular shift times are still configured from the
**Job Orders app's Users tab** (Super Admin only) — this app only reads
them. OT Rate, Allowance, and the SSS/PhilHealth/Pag-IBIG/Cash Advance
deductions are configured **here instead**, from the Payroll tab, by
Accounting or the Super Admin — they were moved off the Job Orders
Users tab so Accounting (who has no access to Job Orders at all) can
maintain payroll figures without needing Super Admin.

## Files

- `index.html` — login, account bar, settings, the Attendance tab
  (clock widget + report), and the Payroll tab
- `api/login.js` — verifies login and issues a session token (same
  account store as the Job Orders app)
- `api/attendance.js` — clock in/out, and the attendance report
  (`?report=1`, Accounting/Super Admin only)
- `api/users.js` — lists accounts (Admin/Super Admin/Accounting) so
  Payroll can pull each employee's Pay Type/Rate/OT Rate/Allowance/
  deductions; also handles saving OT Rate, Allowance, and the SSS/
  PhilHealth/Pag-IBIG/Cash Advance deductions (Accounting or Super
  Admin). Account create/delete, password resets, and Pay Type/Rate/
  Shift stay Admin/Super-Admin-only, same rules as the Job Orders app
- `api/profile.js` — self-service display name / avatar / password
  change, same as the Job Orders app's Settings modal
- `api/_auth.js`, `api/_account-log.js` — shared auth/session and
  account-change-log helpers (identical to the Job Orders app)
- `package.json` — declares the `@vercel/kv` dependency
