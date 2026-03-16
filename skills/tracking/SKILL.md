---
name: tracking
description: Track packages across carriers, view tracking history, and set up tracking webhooks via the Shippo API
---

# Package Tracking

## Track by Number

1. Determine carrier and tracking number. Carrier must be a lowercase Shippo token (e.g., `usps`, `ups`, `fedex`, `dhl_express`). See `references/carrier-guide.md` for tracking number format hints per carrier. If uncertain, ask the user.
2. Call `tracking-status-get` with `carrier` and `tracking_number`.
   - In test mode, use `shippo` as the carrier — see `references/test-mode.md` for details.
3. Key response fields: `tracking_status` (status, status_details, status_date, location), `tracking_history`, `eta`.
4. Each tracking event includes a `substatus` object with `code`, `text`, and `action_required` (boolean). Include substatus details when presenting tracking history -- these provide more specific information about what happened at each step.
5. Present: current status, location, ETA, substatus details, and chronological event history (most recent first).

---

## Test Mode

See `references/test-mode.md` for mock tracking numbers and test mode behavior. Key points: use `shippo` as the carrier token and one of the `SHIPPO_*` mock tracking numbers (e.g., `SHIPPO_TRANSIT`, `SHIPPO_DELIVERED`).

---

## Status Values

See `references/carrier-guide.md` for carrier-specific status nuances. Standard values:

| Status | Meaning |
|---|---|
| PRE_TRANSIT | Label created, carrier has not received the package |
| TRANSIT | Package is in transit |
| DELIVERED | Delivered |
| RETURNED | Being returned or returned to sender |
| FAILURE | Delivery failed |
| UNKNOWN | No tracking information from carrier |

---

## Find Trackable Packages

Call `transactions-list`. Filter for `object_status: SUCCESS`. Each successful transaction has `tracking_number` and carrier info. Then call `tracking-status-get` for selected items.

---

## Register a Tracking Webhook

1. Get the user's HTTPS webhook URL.
2. Call `webhooks-create` with `url` and `event: track_updated`.
3. Optionally call `tracking-status-create` with carrier and tracking number to register a specific shipment for push updates.

---

## Quick Reference

**Track a package:**
`tracking-status-get` with carrier + tracking number

**Find past shipment tracking:**
`transactions-list` -> filter SUCCESS -> `tracking-status-get`
