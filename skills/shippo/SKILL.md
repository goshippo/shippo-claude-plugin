---
name: shippo
description: >
  Route shipping requests to the appropriate Shippo sub-skill. Covers address validation,
  rate shopping, label purchase, tracking, batch processing, and shipping analysis.
---

# Shippo Shipping Router

This skill routes user requests to the correct sub-skill. Do not execute workflows here -- delegate to the matched sub-skill.

## Prerequisites

- The Shippo MCP server must be connected and providing tools.
- A valid Shippo API token must be configured on the server.
- At least one carrier account must be active in Shippo for rate and label operations. Shippo provides managed carrier accounts by default for major carriers.

See `references/tool-reference.md` for the full list of available MCP tools.

---

## Decision Guide

| User Intent | Sub-Skill |
|---|---|
| Validate, parse, or check an address | `address-validation` |
| Parse a freeform address string | `address-validation` |
| Bulk-validate a list of addresses | `address-validation` |
| Compare shipping rates or find cheapest option | `rate-shopping` |
| Get a shipping recommendation | `rate-shopping` |
| Check rates for a checkout flow | `rate-shopping` |
| Buy a shipping label (domestic) | `label-purchase` |
| Buy a shipping label (international) | `label-purchase` |
| Create a return label | `label-purchase` |
| Add signature, insurance, or Saturday delivery | `label-purchase` |
| Void or refund a label | `label-purchase` |
| Handle customs declarations | `label-purchase` |
| Create an order or get a packing slip | `label-purchase` |
| Track a package | `tracking` |
| Find tracking numbers for past shipments | `tracking` |
| Set up tracking notifications / webhooks | `tracking` |
| Process a CSV of shipments | `batch-shipping` |
| Create labels for many packages at once | `batch-shipping` |
| Generate an end-of-day manifest | `batch-shipping` |
| Analyze costs across routes or carriers | `shipping-analysis` |
| Optimize package dimensions | `shipping-analysis` |
| Review past shipping spend | `shipping-analysis` |
| Compare carriers on a route | `shipping-analysis` |

---

## Error Handling (Global)

- Never guess parcel dimensions, weight, customs values, HS codes, or signer names. Ask the user.
- If any tool returns a transport, auth, or rate-limit error, report the error and do not retry automatically.
- Parcel `length`, `width`, `height`, and `weight` must be strings, not numbers.
- Label URLs are S3 signed URLs. Always display the complete URL without truncation.
