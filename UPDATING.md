# How to update this tracker

> Looking for the Cape Town trip planner? See the "Cape Town trip" section
> near the bottom of this file — `capetown.html` / `capetown.json` are a
> separate page from the Remitly tracker below.

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
same "static page + JSON" pattern so it's easy to keep updating over time.

`capetown.json` has three parts:

- `trip` — start/end dates and destination, drives the calendar's default
  month and the highlighted trip range.
- `items` — the planning checklist (Costs, Gifts, People, Car, Fun, etc).
  Each has a `status` (`todo` / `planning` / `holding` / `done`), a
  `summary`, a `details` list, and a free-text `notes` field. Update these
  as things get resolved — e.g. flip `status` to `"done"` once Kwakhanya's
  been messaged, or add a line to `details` when a new fact comes in.
- `calendar` — dated events (`{date, title, type, notes}`) plotted on the
  calendar grid. `type` is one of `travel` / `car` / `fun` / `meeting` and
  controls the event's color. Add entries here as plans get pinned to
  specific days.

To verify, open `capetown.html` locally the same way as above and check the
list and calendar render as expected, then commit and push
`capetown.json`.
