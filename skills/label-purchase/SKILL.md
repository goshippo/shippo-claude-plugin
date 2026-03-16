---
name: label-purchase
description: Purchase domestic and international shipping labels, handle customs declarations, return labels, and void/refund labels via the Shippo API
---

# Label Purchase

## Test vs Live Mode

At the start of any label purchase workflow, check the API key prefix:
- **Test keys** (`shippo_test_*`): Labels are free. No charges are incurred. Use for testing workflows.
- **Live keys** (`shippo_live_*`): Labels incur real charges. Inform the user which mode they are in before proceeding.

---

## Purchase Confirmation Gate

Before every call to `transactions-create`, summarize the following and ask the user for explicit confirmation:
- Carrier and service level
- Estimated cost
- Estimated delivery time
- Origin and destination

**Do not proceed without explicit user confirmation.**

---

## Domestic Label

1. Optionally validate both addresses with `addresses-validate-v2` (see `address-validation` skill).
2. Call `shipments-create` with `address_from`, `address_to` (as inline address objects using v1 field names -- `street1`, `city`, `state`, `zip`, `country`), `parcels`, and `async: false`.
3. Present rates to the user. Let them choose.
4. **Confirm purchase** (see Purchase Confirmation Gate above).
5. Call `transactions-create` with: `rate` (selected rate object_id), `label_file_type` (default `PDF_4x6`), `async: false`.
6. Check response `status`:
   - `SUCCESS`: return `tracking_number`, `label_url` (display the COMPLETE URL -- S3 signed URLs break if truncated), and `tracking_url_provider`.
   - `QUEUED`/`WAITING`: poll `transactions-get` until resolved.
   - `ERROR`: report messages from the `messages` array.

---

## International Label

All domestic steps apply, plus customs handling before shipment creation. See `references/customs-guide.md` for the full customs workflow.

1. Optionally validate addresses with `addresses-validate-v2`. Sender must include `email` and `phone`. Ask if missing.
2. Create customs items: call `customs-items-create` per item (description, quantity, net_weight, mass_unit, value_amount, value_currency, origin_country, tariff_number). Alternatively, you can skip this step and pass inline item objects directly in the declaration (step 3).
3. Create the customs declaration: call `customs-declarations-create` with contents_type, non_delivery_option, certify: true, certify_signer, and the items (either object_ids from step 2, or inline item objects). See `references/customs-guide.md` for field details.
4. Call `shipments-create` with all standard fields plus `customs_declaration` (the declaration object_id).
5. Present rates, **confirm purchase** (see Purchase Confirmation Gate), then purchase label and return results as in the domestic flow.

### Contents Type Decision Tree

Use this to determine the correct `contents_type` value:

| Scenario | Value |
|---|---|
| Selling to the recipient (commercial sale) | `MERCHANDISE` |
| Sending a free gift | `GIFT` |
| Sending a product sample | `SAMPLE` |
| Paper documents only | `DOCUMENTS` |
| Customer returning a purchased item | `RETURN_MERCHANDISE` |
| Charitable donation | `HUMANITARIAN_DONATION` |
| None of the above | `OTHER` (requires `contents_explanation`) |

### Incoterms Decision Logic

The `incoterm` field on the customs declaration controls who pays duties and taxes:

- **B2C / e-commerce (default):** Use `DDU` (Delivered Duty Unpaid) -- recipient pays duties at delivery.
- **Seller prepays duties:** Use `DDP` (Delivered Duty Paid) -- seller covers all duties and taxes.
- **FedEx/DHL only:** `FCA` (Free Carrier) is available for advanced trade scenarios.

If the user does not specify, default to `DDU` for standard e-commerce shipments.

---

## Return Labels

To generate a return label, swap `address_from` and `address_to` so the original recipient becomes the sender and the original sender becomes the recipient. All other steps (shipment creation, rate selection, label purchase) remain the same.

---

## Label Format Options

Default to `PDF_4x6` unless the user specifies otherwise. Supported formats: `PDF_4x6`, `PDF_4x8`, `PDF_A4`, `PDF_A5`, `PDF_A6`, `PDF`, `PDF_2.3x7.5`, `PNG`, `PNG_2.3x7.5`, `ZPLII`.

---

## Label Customization Options

When purchasing a label via `transactions-create`, the following options may be set on the shipment or rate:

- **Signature confirmation**: set `signature_confirmation` on the shipment's `extra` field. Values: `STANDARD`, `ADULT`, `CERTIFIED`, `INDIRECT`, `CARRIER_CONFIRMATION`.
- **Insurance**: set `insurance` on the shipment's `extra` field with `amount`, `currency`, and `provider`.
- **Saturday delivery**: set `saturday_delivery` to `true` in the shipment's `extra` field. Only supported by certain carriers and service levels.
- **Reference fields**: pass `metadata` on the transaction for order numbers or internal references.

---

## Label from Existing Rate

If the user already has a rate object_id: optionally call `rates-get` to confirm details, then **confirm purchase** (see Purchase Confirmation Gate), then call `transactions-create` directly.

---

## Voiding a Label

Call `refunds-create` with the transaction object_id.

**Refund limitations:** Void/refund eligibility depends on carrier and timing. Not all labels can be refunded after purchase. If `refunds-create` fails, advise the user to contact Shippo support.

---

## Quick Reference

**Domestic label:**
(optional) `addresses-validate-v2` (x2) -> `shipments-create` (with inline addresses) -> user picks rate -> confirm -> `transactions-create`

**International label:**
(optional) `addresses-validate-v2` (x2) -> `customs-items-create` (per item) -> `customs-declarations-create` -> `shipments-create` (with inline addresses + customs_declaration) -> user picks rate -> confirm -> `transactions-create`

**Return label:**
Same as domestic/international, but swap `address_from` and `address_to`.

**Order-to-label:**
`orders-create` -> `shipments-create` (using order address/item data) -> user picks rate -> confirm -> `transactions-create` -> `orders-get-packing-slip`

---

## Orders and Packing Slips

Use orders to represent e-commerce fulfillment requests. An order captures the shipping address, line items, and totals -- then feeds into the standard label purchase workflow.

### Tools

- **`orders-create`**: Create an order with line items, shipping address, and order details.
- **`orders-get`**: Retrieve an order by its object_id.
- **`orders-list`**: List all orders.
- **`orders-get-packing-slip`**: Generate a packing slip PDF for an order.

### Workflow

1. Call `orders-create` with the shipping address, line items (title, quantity, sku, total_price, etc.), and order-level fields.
2. Use the order's address and item data to call `shipments-create`, then follow the standard label purchase flow (rate selection, confirmation, `transactions-create`).
3. After purchasing the label, call `orders-get-packing-slip` to generate a PDF packing slip for the order.
