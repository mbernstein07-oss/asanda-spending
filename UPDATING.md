# How to update this tracker

> Looking for the Cape Town trip planner? See the "Cape Town trip" section
> near the bottom of this file — `capetown.html` / `capetown.json` are a
> separate page from the Remitly tracker below.
>
> Looking for the medical claims page? See "Medical claims" further down —
> `claims.html` / `claims.json` are also separate from the Remitly tracker.

This site tracks Remitly transfers to Asanda Mkiva. `index.html` and
`budget.html` are static pages that read `data.json` — updating means
adding new transfer records to that file.

## 1. Find new receipts in Gmail

Remitly sends two emails per transfer, both from `no-reply@remitly.com`
with subject `Status Update: The latest on your transfer to Asanda`:

- **"In progress"** — sent right after you submit the transfer. This one
  has the full receipt: amount sent, fee, exchange rate, reference number.
- **"Delivered"** — sent once funds land in the recipient's account.
  Confirms the transfer completed but doesn't repeat the USD/fee details.

Search Gmail for everything since the last entry in `data.json`:

```
from:remitly.com after:YYYY/MM/DD
```

(use the most recent `date` already in `data.json` as the cutoff). Ignore
Remitly's marketing emails (roundups, promos, "you're pre-approved", etc.)
— only the "Status Update" ones are transfers. For each transfer, confirm
a matching "Delivered" email exists before adding it (a transfer with only
an "In progress" email hasn't completed yet — don't add it, or add it with
`"status": "In progress"` and revisit next update).

## 2. Pull the details from the "In progress" email

Open the "In progress" email's plain text body — it contains a full
receipt table:

- **Reference No.** — e.g. `R21 898 804 689 2`. Strip all spaces to get
  the `ref` value: `R218988046892`.
- **Amount Sent** — the `usd` value (e.g. `100.00 USD` → `100.0`).
- **Fee** — the `fee` value (e.g. `1.49 USD` → `1.49`).
- **Total to Recipient** — the `zar` value (e.g. `R 1,617.00 ZAR` →
  `1617.0`, strip the comma).
- **Submitted** date — convert to `YYYY-MM-DD` for the `date` field.

## 3. Add a record to data.json

Each entry follows this shape:

```json
{
  "ref": "R218988046892",
  "date": "2026-04-18",
  "usd": 100.0,
  "zar": 1617.0,
  "fee": 1.49,
  "status": "Delivered"
}
```

Append new records anywhere in the array — the page sorts by date at
render time, so exact position in the file doesn't matter. Just make sure
the JSON stays valid (comma-separated objects inside the `[...]`).

Notes:
- `usd <= 200` renders as a "Top Up" (weekly-ish small transfers, ~$100).
  `usd > 200` renders as "Monthly Medical Aid" (the larger, roughly
  monthly transfers, ~$370–390).
- Every entry so far has `"status": "Delivered"` — only "Delivered"
  transfers count toward the stats and charts (see `activeTransfers()` in
  `index.html`).

## 4. Verify

Open `index.html` locally (e.g. `python3 -m http.server` from the repo
root, then visit `/index.html`) and confirm the new transfer count, total,
and latest date look right. Then commit and push `data.json`.

---

## Cape Town trip (Nov 7–21, 2026)

`capetown.html` reads `capetown.json` the same way `index.html` reads
`data.json` — it's a planning page, not a spending tracker, but follows the
same "static page + JSON" pattern.

### Editing in the browser (easiest)

The page has an **Edit** button. In edit mode you can rewrite any item,
change its status, reorder or delete items, add new ones, and add/edit/
delete calendar events (click a day's `+`, an existing event, or the
"Add event" button).

GitHub Pages is static, so the page **cannot write back to the repo**.
Edits are saved to that browser's `localStorage` and survive reloads, and
an amber banner reminds you they're local-only. To publish them:

1. Hit **⧉ Copy JSON** (or **⬇ Download**) in the header.
2. Paste the result over `capetown.json` in this repo.
3. Commit and push.

Once the published file matches, hit **Discard** on the banner to drop the
local copy and go back to reading straight from the repo. If `capetown.json`
changes upstream while you still have local edits, the banner switches to a
"published plan changed" warning and offers **Keep mine** / **Use
published**.

### Editing the JSON directly

`capetown.json` has four parts:

- `trip` — start/end dates and destination. Drives the countdown, the
  calendar's default month, and the highlighted trip range.
- `logistics` — the booked-and-done facts (flights, stay, car, baggage)
  rendered as cards at the top. Each is
  `{id, kind, label, headline, lines[], ref}`; `kind` is one of `flight` /
  `stay` / `car` / `bags` and sets the card's accent color. These are
  reference data pulled from confirmation emails, so they're not editable
  in the browser — change them here.
- `items` — the planning checklist. Each has a `status` (`todo` /
  `planning` / `holding` / `done`), a `summary`, a `details` list, and a
  free-text `notes` field. The progress bar counts `done` over total.
- `calendar` — dated events (`{date, title, type, notes}`) plotted on the
  grid and listed in the agenda. `type` is one of `travel` / `stay` /
  `car` / `fun` / `meeting` and sets the color.

To verify, open `capetown.html` locally the same way as above and check the
list and calendar render as expected, then commit and push
`capetown.json`.

---

## Medical claims

`claims.html` reads `claims.json` — one record per claim line item from
Momentum's claims history, same "static page + JSON" pattern as the rest of
the site.

`claims.json` was built by merging two raw Momentum CSV exports (one in a
wide one-hot-encoded format with hundreds of `detail_<tariff code>` columns,
one already normalized to two explanation columns) and de-duplicating exact
matches. Each record has: `date`, `provider` (+ `provider_number` /
`provider_contact` / `provider_address`), `claim_amount`, `amount_to_you`,
`amount_to_provider`, `pay_from` (benefit type), `status`, `tariff_code` +
`tariff_explanation` (what the treatment was), `pay_code` +
`pay_code_explanation` (why it was paid/rejected the way it was),
`expected_pay_date`, and `source` (`clean` or `partial`, i.e. which export
it came from).

To add a new export: parse it into the same record shape and append to the
JSON array (dedupe against existing records on
`date+provider+claim_amount+amount_to_you+amount_to_provider+pay_from+status+tariff_code+pay_code`
before appending, since re-downloaded date ranges tend to overlap). The page
computes every stat, chart, and the rejected-by-provider/rejected-by-reason
breakdowns client-side from the raw array — no precomputed aggregates to
keep in sync.

Note: a claim with `status` starting with `"Rejected"` isn't always money
owed — some are the scheme fully declining a claim, others are the supplier
reversing/resubmitting it. Check the reason text before treating the
rejected total as an out-of-pocket figure.

To verify, open `claims.html` locally the same way as above and confirm the
stats, table, and filters look right, then commit and push `claims.json`
(and `claims.html` if it changed).
