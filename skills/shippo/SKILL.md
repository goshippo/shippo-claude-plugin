---
name: shippo
description: >
  Ship packages using the Shippo MCP server. Covers address validation and parsing,
  multi-carrier rate shopping, domestic and international label generation (including
  return labels and customs declarations), package tracking across carriers, shipping
  cost analysis and optimization, and batch processing from CSV files with end-of-day
  manifest creation. Requires a configured Shippo API token and at least one carrier account.
---

# Shippo Shipping Skill

## Prerequisites

- The Shippo MCP server must be connected and providing tools.
- A valid Shippo API token must be configured on the server.
- At least one carrier account must be active in Shippo for rate and label operations.

See `references/tool-reference.md` for the full list of available MCP tools.

---

## 1. Address Validation and Parsing

### Validate a Structured Address

1. Collect at minimum: `street1`, `city`, `state`, `zip`, `country` (ISO 3166-1 alpha-2).
2. Call `addresses-create-v2` with the address fields. This creates the address and returns an object ID.
3. Call `addresses-validate-v2` with the address fields to get validation results. Note: this endpoint takes address fields as query parameters, not an object ID.
4. Check `analysis.validation_result.value` in the response. Values: `"valid"`, `"invalid"`, or `"partially_valid"` (address found with corrections applied). Check `analysis.validation_result.reasons` for details.
5. Report the standardized address back. Highlight any corrected fields (listed in `changed_attributes`). Note `analysis.address_type` (`"residential"`, `"commercial"`, or `"unknown"`) — residential classification affects carrier surcharges.
6. If invalid: relay the reason descriptions. If the API returns a `recommended_address`, present it to the user.
7. If `partially_valid`: show what was corrected and ask the user to confirm the corrections are acceptable.

### Parse a Freeform Address

1. Call `addresses-parse` with the raw string (e.g., "123 Main St, Springfield IL 62704").
2. Review the structured output for completeness. The response uses V2 field names: `address_line_1`, `city_locality`, `state_province`, `postal_code`.
3. Note: the parse response does not include `country`. You must ask the user for the country or infer it, then add it before proceeding.
4. Validate the parsed result by passing the fields to `addresses-create-v2` then `addresses-validate-v2` (follow the structured address workflow above from step 2).

### International Addresses

- Always require the `country` field. Do not guess.
- Pass non-Latin characters as-is; the API handles encoding.
- Validation depth varies by country. US, CA, GB, AU, and major EU countries have deep validation. Others may only confirm structural completeness. Inform the user of this limitation.

### Bulk Address Validation

There is no batch validation endpoint. Call `addresses-create-v2` per address. Track results (row number, valid/invalid, corrections, errors, residential classification) and report a summary when done. For 50+ addresses, set expectations about processing time and provide progress updates.

### Re-validate an Existing Address

Call `addresses-validate-v2` with the address fields. This endpoint validates by address fields, not by object ID.

### Duplicate Addresses

If `addresses-create-v2` returns a "Duplicate address" error, the address already exists in the account. Retrieve it via `addresses-list` or proceed directly to validation.

---

## 2. Rate Shopping and Comparison

### Get Rates for a Shipment

1. Collect: origin address, destination address, parcel (length, width, height, distance_unit, weight, mass_unit). All dimension and weight values must be **strings** (e.g., `"10"` not `10`).
2. Optionally validate both addresses with `addresses-validate-v2`.
3. Call `shipments-create` with `address_from`, `address_to` (as inline address objects, not object IDs — the shipments endpoint does not accept v2 address IDs), and `parcels`.
4. The response `rates` array contains available options. Present a table: carrier, service level, price, estimated days, attributes (CHEAPEST, BESTVALUE, FASTEST).
5. Note: the same carrier may return duplicate rates from multiple carrier accounts. Present the best rate per carrier/service combination.

### Filter by Speed

Map user requests: "overnight" = estimated_days 1, "2-day" = estimated_days <= 2, "within N days" = estimated_days <= N. Filter the rates array accordingly. If nothing matches, show the fastest available option.

### International Rates

Some carriers may return international rates without a customs declaration, but others will not. If no rates are returned, try attaching a customs declaration to the shipment. Some carriers also require a phone number on the destination address for international rate retrieval. Inform the user that customs will be required at label purchase time regardless.

### Checkout Rates (Line Items)

Call `rates-at-checkout-create` instead of `shipments-create`. Accepts `address_from`, `address_to`, and `line_items` (each with title, quantity, total_price, currency, weight, weight_unit).

### Rates in a Specific Currency

Call `rates-list-by-currency-code` with the preferred ISO currency code (USD, EUR, GBP, CAD, etc.).

### Recommendation

Identify the cheapest, fastest, and best-value options. State the trade-off: "Option A is $X cheaper but takes Y more days than Option B."

### Troubleshooting: No Rates

- Verify both addresses passed validation (most common cause).
- Check that the user has carrier accounts configured (`carrier-accounts-list`).
- Confirm parcel dimensions are reasonable (not zero, not exceeding carrier limits).
- Rates expire. If stale, create a new shipment to get fresh rates.

---

## 3. Label Generation

### Domestic Label

1. Optionally validate both addresses with `addresses-validate-v2`.
2. Call `shipments-create` with `address_from`, `address_to` (as inline address objects), `parcels`, and `async: false`.
3. Present rates to the user. Let them choose.
4. Call `transactions-create` with: `rate` (selected rate object_id), `label_file_type` (default `PDF_4x6`), `async: false`.
5. Check response `status`:
   - `SUCCESS`: return `tracking_number`, `label_url` (display the COMPLETE URL -- S3 signed URLs break if truncated), and `tracking_url_provider`.
   - `QUEUED`/`WAITING`: poll `transactions-get` until resolved.
   - `ERROR`: report messages from the `messages` array.

### International Label

All domestic steps apply, plus customs handling before shipment creation. See `references/customs-guide.md` for the full customs workflow.

1. Optionally validate addresses with `addresses-validate-v2`. Sender must include `email` and `phone`. Ask if missing.
2. Create customs items: call `customs-items-create` per item (description, quantity, net_weight, mass_unit, value_amount, value_currency, origin_country, tariff_number). Alternatively, you can skip this step and pass inline item objects directly in the declaration (step 3).
3. Create the customs declaration: call `customs-declarations-create` with contents_type, non_delivery_option, certify: true, certify_signer, and the items (either object_ids from step 2, or inline item objects). See `references/customs-guide.md` for field details.
4. Call `shipments-create` with all standard fields plus `customs_declaration` (the declaration object_id).
5. Present rates, purchase label, return results as in the domestic flow.

### Return Labels

To generate a return label, swap `address_from` and `address_to` so the original recipient becomes the sender and the original sender becomes the recipient. All other steps (shipment creation, rate selection, label purchase) remain the same.

### Label Format Options

Default to `PDF_4x6` unless the user specifies otherwise. Supported formats: `PDF_4x6`, `PDF_4x8`, `PDF_A4`, `PDF_A5`, `PDF_A6`, `PDF`, `PDF_2.3x7.5`, `PNG`, `PNG_2.3x7.5`, `ZPLII`.

### Label Customization Options

When purchasing a label via `transactions-create`, the following options may be set on the shipment or rate:

- **Signature confirmation**: set `signature_confirmation` on the shipment's `extra` field. Values: `STANDARD`, `ADULT`, `CERTIFIED`, `INDIRECT`, `CARRIER_CONFIRMATION`.
- **Insurance**: set `insurance` on the shipment's `extra` field with `amount`, `currency`, and `provider`.
- **Saturday delivery**: set `saturday_delivery` to `true` in the shipment's `extra` field. Only supported by certain carriers and service levels.
- **Reference fields**: pass `metadata` on the transaction for order numbers or internal references.

### Label from Existing Rate

If the user already has a rate object_id: optionally call `rates-get` to confirm details, then call `transactions-create` directly.

### Voiding a Label

Call `refunds-create` with the transaction object_id. Eligibility depends on carrier and timing.

---

## 4. Package Tracking

### Track by Number

1. Determine carrier and tracking number. Carrier must be a lowercase Shippo token (e.g., `usps`, `ups`, `fedex`, `dhl_express`). Inference hints:
   - USPS: 20-22 digits or starts with 9 + 20 digits
   - UPS: starts with `1Z` + 16 alphanumeric
   - FedEx: 12 or 15 digits
   - DHL Express: 10 digits
   - If uncertain, ask the user.
2. Call `tracking-status-get` with `carrier` and `tracking_number`.
   - In test mode, use `shippo` as the carrier, not the real carrier name.
3. Key response fields: `tracking_status` (status, status_details, status_date, location), `tracking_history`, `eta`.
4. Present: current status, location, ETA, and chronological event history (most recent first).

### Status Values

| Status | Meaning |
|---|---|
| PRE_TRANSIT | Label created, carrier has not received the package |
| TRANSIT | Package is in transit |
| DELIVERED | Delivered |
| RETURNED | Being returned or returned to sender |
| FAILURE | Delivery failed |
| UNKNOWN | No tracking information from carrier |

### Find Trackable Packages

Call `transactions-list`. Filter for `object_status: SUCCESS`. Each successful transaction has `tracking_number` and carrier info. Then call `tracking-status-get` for selected items.

### Register a Tracking Webhook

1. Get the user's HTTPS webhook URL.
2. Call `webhooks-create` with `url` and `event: track_updated`.
3. Optionally call `tracking-status-create` with carrier and tracking number to register a specific shipment for push updates.

---

## 5. Shipping Analysis and Optimization

### Geographic Cost Analysis

1. Confirm origin address, destination list (or use representative cities), and parcel details.
2. Call `carrier-accounts-list` to see configured carriers.
3. Call `shipments-create` per destination to collect rates. Creating shipments is free; only `transactions-create` costs money.
4. Write results to `analysis/` directory (markdown report + CSV). Columns: Route, Destination, Carrier, Service, Cost, Currency, EstimatedDays, Zone.

### Package Optimization

1. Confirm the route.
2. Define dimension profiles to test (or use user-provided ones).
3. Check `carrier-parcel-templates-list` and `user-parcel-templates-list` for flat-rate and saved templates.
4. Call `shipments-create` per profile on the same route.
5. Compare: cheapest rate, carrier options, fastest option per profile. Note where flat-rate templates beat custom dimensions and where dimensional weight causes price jumps.

### Carrier Comparison

1. Call `shipments-create` for the route.
2. Group the `rates` array by `provider`.
3. Per carrier: cheapest service, fastest service, number of service levels, price range.

### Historical Cost Optimization

1. Call `shipments-list` and `transactions-list` to get past activity.
2. Cross-reference: what the user paid vs. what alternatives were available.
3. Identify patterns: carrier concentration, service-level mismatch, consistent overpayment.
4. For a sample of shipments with tracking numbers, call `tracking-status-get` to check actual vs. estimated delivery times.
5. If fewer than 5 successful transactions exist (not just shipments — shipments are rate quotes, transactions represent actual spend), redirect to forward-looking analysis.

### Output Conventions

Write reports to the `analysis/` directory. Create it if it does not exist. Include both markdown and CSV. CSV must have a header row. Markdown must include a timestamp and input parameters.

---

## 6. Batch Processing

### CSV Batch Processing

See `references/csv-format.md` for the column specification.

1. Read and parse the CSV. Validate required columns are present. Report row count.
2. Validate each row for non-empty required fields. Report invalid rows with reasons.
3. Detect international rows (sender_country != recipient_country). Create customs declarations for those rows (see `references/customs-guide.md`).
4. Build the `batch_shipments` array with inline address and parcel objects per row.
5. Call `batches-create` with the array.
6. Poll `batches-get` until status changes from `VALIDATING` to `VALID`. For 100+ rows, warn that validation may take minutes.
7. Review per-shipment validation results. Report failures before proceeding.
8. Call `batches-purchase` to buy labels for all valid shipments.
9. Poll `batches-get` until status changes from `PURCHASING` to `PURCHASED`.
10. Report: total attempted, succeeded, failed. For successes: tracking_number and label_url (complete URL). For failures: error messages.

### Batch with Rate Shopping

1. Call `shipments-create` per shipment to get rate quotes.
2. Present rates. User picks a service level rule (e.g., "cheapest for each" or a specific carrier/service).
3. Build `batch_shipments` with `servicelevel_token` per item.
4. Create, validate, purchase, report as above.

### Managing an Existing Batch

- Add shipments: `batches-add-shipments` (before purchase only). Note: adding an invalid shipment will change the entire batch status to `INVALID`. Check per-shipment statuses after adding.
- Remove shipments: `batches-remove-shipments` (before purchase only).

### End-of-Day Manifest

1. Collect: `carrier_account` (object_id), `shipment_date` (YYYY-MM-DD, default today), `address_from` (pickup address).
2. Optionally collect specific transaction object_ids to scope the manifest.
3. Call `manifests-create`.
4. Poll `manifests-get` until status is `SUCCESS` or `ERROR`.
5. Return the manifest PDF URL(s) and shipment count.

---

## Quick Reference: Common Tool Sequences

**Validate an address:**
`addresses-create-v2` (saves address) + `addresses-validate-v2` (validates with same fields)

**Parse then validate:**
`addresses-parse` -> add country -> `addresses-create-v2` + `addresses-validate-v2`

**Get rates:**
(optional) `addresses-validate-v2` (x2) -> `shipments-create` (with inline addresses) -> read `rates` array

**Domestic label:**
(optional) `addresses-validate-v2` (x2) -> `shipments-create` (with inline addresses) -> user picks rate -> `transactions-create`

**International label:**
(optional) `addresses-validate-v2` (x2) -> `customs-items-create` (per item) -> `customs-declarations-create` -> `shipments-create` (with inline addresses + customs_declaration) -> user picks rate -> `transactions-create`

**Return label:**
Same as domestic/international, but swap `address_from` and `address_to`.

**Track a package:**
`tracking-status-get`

**CSV batch:**
Parse CSV -> `customs-declarations-create` (international rows) -> `batches-create` -> poll `batches-get` -> `batches-purchase` -> poll `batches-get`

**Manifest:**
`manifests-create` -> poll `manifests-get`

---

## Decision Guide

Use this to route the user's request to the right workflow.

| User Intent | Section |
|---|---|
| Validate or check an address | 1. Address Validation |
| Parse a freeform address string | 1. Address Validation (Parse) |
| Compare shipping rates or find cheapest option | 2. Rate Shopping |
| Get a shipping recommendation | 2. Rate Shopping (Recommendation) |
| Buy a shipping label (domestic) | 3. Label Generation (Domestic) |
| Buy a shipping label (international) | 3. Label Generation (International) |
| Create a return label | 3. Label Generation (Return Labels) |
| Add signature, insurance, or Saturday delivery | 3. Label Generation (Customization Options) |
| Track a package | 4. Package Tracking |
| Find tracking numbers for past shipments | 4. Package Tracking (Find Trackable) |
| Set up tracking notifications | 4. Package Tracking (Webhook) |
| Analyze costs across routes or carriers | 5. Shipping Analysis |
| Optimize package dimensions | 5. Shipping Analysis (Package Optimization) |
| Review past shipping spend | 5. Shipping Analysis (Historical) |
| Process a CSV of shipments | 6. Batch Processing (CSV) |
| Create labels for many packages at once | 6. Batch Processing |
| Generate an end-of-day manifest | 6. Batch Processing (Manifest) |
| Void or refund a label | 3. Label Generation (Voiding) |

---

## Error Handling Notes

- Never guess parcel dimensions, weight, customs values, HS codes, or signer names. Ask the user.
- If any tool returns a transport, auth, or rate-limit error, report the error and do not retry automatically.
- Parcel `length`, `width`, `height`, and `weight` must be strings, not numbers.
- Label URLs are S3 signed URLs. Always display the complete URL without truncation.
- In test mode, use `shippo` as the carrier for `tracking-status-get`, not real carrier names.
