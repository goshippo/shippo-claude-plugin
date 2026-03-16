# Shippo MCP Tool Reference

Complete list of MCP tools provided by the Shippo server, grouped by category.

---

## Addresses

| Tool | Description |
|---|---|
| `addresses-create-v2` | Create and validate a new address (v2, preferred) |
| `addresses-validate-v2` | Validate an existing address by object ID (v2) |
| `addresses-parse` | Parse a freeform address string into structured components |
| `addresses-create-v1` | Create an address without automatic validation (legacy) |
| `addresses-validate-existing` | Validate an existing address by object ID (legacy) |
| `addresses-get` | Retrieve a previously created address by ID |
| `addresses-list` | List all stored addresses |

## Shipments

| Tool | Description |
|---|---|
| `shipments-create` | Create a new shipment and retrieve available rates |
| `shipments-get` | Retrieve a shipment by ID |
| `shipments-list` | List all shipments |
| `shipments-list-rates` | Retrieve rates for an existing shipment |

## Rates

| Tool | Description |
|---|---|
| `rates-get` | Retrieve a specific rate by ID |
| `rates-list-by-currency-code` | Retrieve shipment rates in a specific currency |
| `rates-at-checkout-create` | Generate live rates for a checkout flow with line items |
| `rates-at-checkout-get-default-parcel-template` | Show the current default parcel template for checkout rates |
| `rates-at-checkout-delete-default-parcel-template` | Clear the current default parcel template for checkout rates |
| `rates-at-checkout-update-parcel-template` | Update the default parcel template for checkout rates |

## Transactions (Labels)

| Tool | Description |
|---|---|
| `transactions-create` | Purchase a shipping label from an existing rate |
| `transactions-get` | Retrieve a transaction (label) by ID |
| `transactions-list` | List all transactions (purchased labels) |

## Tracking

| Tool | Description |
|---|---|
| `tracking-status-get` | Get current tracking status for a carrier + tracking number |
| `tracking-status-create` | Register a shipment for tracking webhook notifications |

## Batches

| Tool | Description |
|---|---|
| `batches-create` | Create a new batch of shipments |
| `batches-get` | Retrieve a batch by ID (includes status and per-shipment results) |
| `batches-purchase` | Purchase labels for all valid shipments in a batch |
| `batches-add-shipments` | Add shipments to an existing batch (before purchase) |
| `batches-remove-shipments` | Remove shipments from an existing batch (before purchase) |

## Customs

| Tool | Description |
|---|---|
| `customs-declarations-create` | Create a customs declaration for international shipments |
| `customs-declarations-get` | Retrieve a customs declaration by ID |
| `customs-declarations-list` | List all customs declarations |
| `customs-items-create` | Create a customs item (individual line item within a declaration) |
| `customs-items-get` | Retrieve a customs item by ID |
| `customs-items-list` | List all customs items |

## Manifests

| Tool | Description |
|---|---|
| `manifests-create` | Create an end-of-day manifest (SCAN form) for carrier pickup |
| `manifests-get` | Retrieve a manifest by ID |
| `manifests-list` | List all manifests |

## Parcels

| Tool | Description |
|---|---|
| `parcels-create` | Create a new parcel object |
| `parcels-get` | Retrieve an existing parcel by ID |
| `parcels-list` | List all parcels |

## Parcel Templates

| Tool | Description |
|---|---|
| `carrier-parcel-templates-list` | List all carrier-provided parcel templates (e.g., USPS Flat Rate) |
| `carrier-parcel-templates-get` | Retrieve a specific carrier parcel template |
| `user-parcel-templates-list` | List all user-created parcel templates |
| `user-parcel-templates-create` | Create a new user parcel template |
| `user-parcel-templates-get` | Retrieve a user parcel template by ID |
| `user-parcel-templates-update` | Update an existing user parcel template |
| `user-parcel-templates-delete` | Delete a user parcel template |

## Carrier Accounts

| Tool | Description |
|---|---|
| `carrier-accounts-list` | List all carrier accounts |
| `carrier-accounts-create` | Create a new carrier account |
| `carrier-accounts-get` | Retrieve a carrier account by ID |
| `carrier-accounts-update` | Update a carrier account |
| `carrier-accounts-register` | Add a Shippo carrier account |
| `carrier-accounts-get-registration-status` | Get carrier registration status |
| `carrier-accounts-initiate-oauth2-signin` | Connect a carrier account using OAuth 2.0 |

## Orders

| Tool | Description |
|---|---|
| `orders-create` | Create a new order |
| `orders-get` | Retrieve an order by ID |
| `orders-list` | List all orders |
| `orders-get-packing-slip` | Get a packing slip for an order |

## Refunds

| Tool | Description |
|---|---|
| `refunds-create` | Create a refund (void a label) |
| `refunds-get` | Retrieve a refund by ID |
| `refunds-list` | List all refunds |

## Pickups

| Tool | Description |
|---|---|
| `pickups-create` | Schedule a carrier pickup |

## Service Groups

| Tool | Description |
|---|---|
| `service-groups-list` | List all service groups |
| `service-groups-create` | Create a new service group |
| `service-groups-update` | Update an existing service group |
| `service-groups-delete` | Delete a service group |

## Webhooks

| Tool | Description |
|---|---|
| `webhooks-create` | Create a new webhook subscription |
| `webhooks-get` | Retrieve a specific webhook |
| `webhooks-list` | List all webhooks |
| `webhooks-update` | Update an existing webhook |
| `webhooks-delete` | Delete a webhook |

## Shippo Accounts

| Tool | Description |
|---|---|
| `shippo-accounts-list` | List all Shippo accounts |
| `shippo-accounts-create` | Create a Shippo account |
| `shippo-accounts-get` | Retrieve a Shippo account |
| `shippo-accounts-update` | Update a Shippo account |
