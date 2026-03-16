---
name: batch-shipping
description: Process bulk shipments from CSV files, create and purchase batch labels, and generate end-of-day manifests via the Shippo API
---

# Batch Shipping

## Test vs Live Mode

At the start of any batch purchase workflow, check the API key prefix:
- **Test keys** (`shippo_test_*`): Labels are free. No charges are incurred.
- **Live keys** (`shippo_live_*`): Labels incur real charges. Inform the user which mode they are in before proceeding.

---

## Purchase Confirmation Gate

Before every call to `batches-purchase`, summarize the following and ask the user for explicit confirmation:
- Total number of shipments to be purchased
- Carrier and service level (or selection rule if varied)
- Estimated total cost
- Number of domestic vs international shipments

**Do not proceed without explicit user confirmation.**

---

## CSV Batch Processing

See `references/csv-format.md` for the column specification.

1. Read and parse the CSV. Validate required columns are present. Report row count.
2. Validate each row for non-empty required fields. Report invalid rows with reasons.
3. Detect international rows (sender_country != recipient_country). Create customs declarations for those rows. See `references/customs-guide.md`. Use correct customs enum values: `RETURN_MERCHANDISE` (not `RETURN`) for returned goods, `HUMANITARIAN_DONATION` (not `HUMANITARIAN`) for charitable donations.
4. Build the `batch_shipments` array with inline address and parcel objects per row.
5. Call `batches-create` with the array.
6. Poll `batches-get` until status changes from `VALIDATING` to `VALID`. See Polling Intervals below.
7. Review per-shipment validation results. Report failures before proceeding.
8. **Confirm purchase** (see Purchase Confirmation Gate above).
9. Call `batches-purchase` to buy labels for all valid shipments.
10. Poll `batches-get` until status changes from `PURCHASING` to `PURCHASED`. See Polling Intervals below.
11. Report: total attempted, succeeded, failed. For successes: tracking_number and label_url (complete URL). For failures: error messages.

### Batch Size Guidance

For batches over 500 shipments, consider splitting into multiple batches. Large batches take longer to validate and purchase, and a single failure can be harder to diagnose.

---

## Polling Intervals

- For batches under 100 shipments: poll every 3-5 seconds.
- For batches with 100+ shipments: poll every 5-10 seconds.
- Report progress to the user every 30 seconds.
- Stop after 60 retries and suggest the user check back later using `batches-get` with the batch object_id.

---

## Batch with Rate Shopping

1. Call `shipments-create` per shipment to get rate quotes (see `rate-shopping` skill).
2. Present rates. User picks a service level rule (e.g., "cheapest for each" or a specific carrier/service).
3. Build `batch_shipments` with `servicelevel_token` per item.
4. Create, validate, **confirm purchase**, purchase, report as above.

---

## Managing an Existing Batch

- Add shipments: `batches-add-shipments` (before purchase only). Note: adding an invalid shipment will change the entire batch status to `INVALID`. Check per-shipment statuses after adding.
- Remove shipments: `batches-remove-shipments` (before purchase only).

---

## End-of-Day Manifest

1. Collect: `carrier_account` (object_id), `shipment_date` (YYYY-MM-DD, default today), `address_from` (pickup address).
2. Optionally collect specific transaction object_ids to scope the manifest. You must pass specific transaction object_ids -- there is no auto-include for a date range.
3. Call `manifests-create`.
4. Poll `manifests-get` until status is `SUCCESS` or `ERROR`.
5. Return the manifest PDF URL(s) and shipment count.

---

## Quick Reference

**CSV batch:**
Parse CSV -> `customs-declarations-create` (international rows) -> `batches-create` -> poll `batches-get` -> confirm -> `batches-purchase` -> poll `batches-get`

**Manifest:**
`manifests-create` (with transaction object_ids) -> poll `manifests-get`
