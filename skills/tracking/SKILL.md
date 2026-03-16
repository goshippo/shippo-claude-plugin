---
name: tracking
description: Track packages across carriers, view tracking history, and set up tracking webhooks via the Shippo API
---

# Package Tracking

## Track by Number

1. Determine carrier and tracking number. Carrier must be a lowercase Shippo token (e.g., `usps`, `ups`, `fedex`, `dhl_express`). Inference hints:
   - USPS: 20-22 digits or starts with 9 + 20 digits
   - UPS: starts with `1Z` + 16 alphanumeric
   - FedEx: 12 or 15 digits
   - DHL Express: 10 digits
   - If uncertain, ask the user.
2. Call `tracking-status-get` with `carrier` and `tracking_number`.
   - In test mode, use `shippo` as the carrier, not the real carrier name.
3. Key response fields: `tracking_status` (status, status_details, status_date, location), `tracking_history`, `eta`.
4. Each tracking event includes a `substatus` object with `code`, `text`, and `action_required` (boolean). Include substatus details when presenting tracking history -- these provide more specific information about what happened at each step.
5. Present: current status, location, ETA, substatus details, and chronological event history (most recent first).

---

## Test Mode Tracking Numbers

When using a test API key (`shippo_test_*`), use `shippo` as the carrier and one of these mock tracking numbers:

| Tracking Number | Simulated Status |
|---|---|
| `SHIPPO_PRE_TRANSIT` | Label created, not yet with carrier |
| `SHIPPO_TRANSIT` | Package in transit |
| `SHIPPO_DELIVERED` | Package delivered |
| `SHIPPO_RETURNED` | Returned to sender |
| `SHIPPO_FAILURE` | Delivery failed |
| `SHIPPO_UNKNOWN` | No tracking info available |

---

## Status Values

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
