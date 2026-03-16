---
name: address-validation
description: Validate, parse, and standardize shipping addresses via the Shippo API
---

# Address Validation

## Address Field Format

The Shippo API uses **v1 field names** for address components in most endpoints (including `shipments-create`). Always use:

| Field | Description | Example |
|---|---|---|
| `name` | Full name | `Jane Smith` |
| `street1` | Street address line 1 | `731 Market St` |
| `street2` | Street address line 2 (optional) | `Suite 200` |
| `city` | City | `San Francisco` |
| `state` | State or province | `CA` |
| `zip` | Postal code | `94103` |
| `country` | ISO 3166-1 alpha-2 country code | `US` |
| `email` | Email (required for international senders) | `jane@example.com` |
| `phone` | Phone (required for international senders) | `+1-555-123-4567` |

Note: The v2 address endpoints (`addresses-create-v2`, `addresses-validate-v2`) use different field names (`address_line_1`, `city_locality`, `state_province`, `postal_code`), but when passing addresses inline to `shipments-create`, you must use the v1 names above.

---

## Validate a Structured Address

1. Collect at minimum: `street1`, `city`, `state`, `zip`, `country` (ISO 3166-1 alpha-2).
2. Call `addresses-create-v2` with the address fields. This creates the address and returns an object ID.
3. Call `addresses-validate-v2` with the address fields to get validation results. Note: this endpoint takes address fields as query parameters, not an object ID.
4. Check `analysis.validation_result.value` in the response. Values: `"valid"`, `"invalid"`, or `"partially_valid"` (address found with corrections applied). Check `analysis.validation_result.reasons` for details.
5. Report the standardized address back. Highlight any corrected fields (listed in `changed_attributes`). Note `analysis.address_type` (`"residential"`, `"commercial"`, or `"unknown"`) -- residential classification affects carrier surcharges.
6. If invalid: relay the reason descriptions. If the API returns a `recommended_address`, present it to the user.
7. If `partially_valid`: show what was corrected and ask the user to confirm the corrections are acceptable.

---

## Parse a Freeform Address

1. Call `addresses-parse` with the raw string (e.g., "123 Main St, Springfield IL 62704").
2. Review the structured output for completeness. The parse response uses v2 field names: `address_line_1`, `city_locality`, `state_province`, `postal_code`.
3. Note: the parse response does not include `country`. You must ask the user for the country or infer it, then add it before proceeding.
4. Validate the parsed result by passing the fields to `addresses-create-v2` then `addresses-validate-v2` (follow the structured address workflow above from step 2).

---

## International Addresses

- Always require the `country` field. Do not guess.
- Pass non-Latin characters as-is; the API handles encoding.
- Validation depth varies by country. US, CA, GB, AU, and major EU countries have deep validation. Others may only confirm structural completeness. Inform the user of this limitation.

---

## Bulk Address Validation

There is no batch validation endpoint. Call `addresses-create-v2` per address. Track results (row number, valid/invalid, corrections, errors, residential classification) and report a summary when done. For 50+ addresses, set expectations about processing time and provide progress updates.

---

## Re-validate an Existing Address

Call `addresses-validate-v2` with the address fields. This endpoint validates by address fields, not by object ID.

---

## Duplicate Addresses

If `addresses-create-v2` returns a "Duplicate address" error, the address already exists in the account. Retrieve it via `addresses-list` or proceed directly to validation.

---

## Quick Reference

**Validate an address:**
`addresses-create-v2` (saves address) + `addresses-validate-v2` (validates with same fields)

**Parse then validate:**
`addresses-parse` -> add country -> `addresses-create-v2` + `addresses-validate-v2`
