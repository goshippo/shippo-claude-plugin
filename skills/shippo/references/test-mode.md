# Test Mode Guide

How to work with Shippo's test mode for development and demonstration.

---

## Identifying Test vs Live Mode

The API key prefix determines the mode:

| Prefix | Mode | Description |
|---|---|---|
| `shippo_test_` | Test | Simulated data, no real charges, no real labels |
| `shippo_live_` | Live | Real carrier rates, real charges, real labels |

Test and live mode have **completely separate data**. Object IDs created in test mode do not exist in live mode and vice versa.

---

## What Works in Test Mode

- **Address creation and validation** -- works normally, returns real validation results.
- **Shipment creation** -- works normally, returns simulated rates.
- **Rate retrieval** -- returns simulated rates. Prices may differ from live carrier rates.
- **Label purchase** -- returns a simulated label PDF/PNG with a test tracking number. No real charge.
- **Batch processing** -- full batch workflow works (create, validate, purchase).
- **Customs declarations** -- creation and attachment works normally.
- **Tracking** -- uses mock tracking numbers (see below). Real carrier tracking numbers will not return data in test mode.
- **All CRUD operations** -- creating, reading, updating, and deleting objects works normally.

## What Does Not Work (or Differs)

- **Carrier rates** are simulated and may not match live pricing. Do not use test mode rates for cost estimation.
- **Labels** are not valid for actual shipping. They have test barcodes.
- **Tracking** is simulated only -- you must use mock tracking numbers.
- **Carrier-specific features** (signature confirmation surcharges, residential surcharges, etc.) may not be accurately reflected in test rates.
- **Webhooks** will fire for test events, but tracking webhooks only work with mock tracking numbers.

---

## Mock Tracking Numbers

Use these predefined tracking numbers with `tracking-status-get` to simulate different tracking states. Use `shippo` as the carrier token.

| Tracking Number | Simulated Status |
|---|---|
| `SHIPPO_PRE_TRANSIT` | Package label created, not yet with carrier |
| `SHIPPO_TRANSIT` | Package in transit |
| `SHIPPO_DELIVERED` | Package delivered |
| `SHIPPO_RETURNED` | Package returned to sender |
| `SHIPPO_FAILURE` | Delivery failed |
| `SHIPPO_UNKNOWN` | Status unknown |

### Example

```
tracking-status-get carrier="shippo" tracking_number="SHIPPO_DELIVERED"
```

This returns a full tracking response with simulated events showing the package progressing from pre-transit through delivery.

---

## Test Addresses

These US addresses work well for testing:

**Sender (San Francisco):**
```json
{
  "name": "Test Sender",
  "street1": "731 Market St",
  "street2": "#200",
  "city": "San Francisco",
  "state": "CA",
  "zip": "94103",
  "country": "US",
  "email": "sender@test.com",
  "phone": "+1-555-000-0001"
}
```

**Domestic recipient (New York):**
```json
{
  "name": "Test Recipient",
  "street1": "350 5th Ave",
  "city": "New York",
  "state": "NY",
  "zip": "10118",
  "country": "US"
}
```

**International recipient (London):**
```json
{
  "name": "Test International",
  "street1": "10 Downing St",
  "city": "London",
  "state": "",
  "zip": "SW1A 2AA",
  "country": "GB",
  "email": "intl@test.com",
  "phone": "+44-20-7925-0918"
}
```

---

## Switching Between Modes

The mode is determined solely by the API key set in the `SHIPPO_API_KEY` environment variable. To switch modes:

1. Update `SHIPPO_API_KEY` to the desired key (`shippo_test_*` or `shippo_live_*`).
2. Restart the MCP server.
3. All subsequent operations use the new mode.

There is no way to mix test and live operations in a single session.
