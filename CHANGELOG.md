# Changelog

All notable changes to the Shippo Claude Code Plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2026-03-18

### Removed

- Router skill (`/shippo:shippo`) — Claude auto-routes to the correct skill based on context; no dispatcher needed

### Changed

- Migrated global error-handling rules from the router into `references/error-reference.md`

## [1.0.0] - 2026-03-16

### Added

- Initial plugin release with 6 task-specific skills: address-validation, rate-shopping, label-purchase, tracking, batch-shipping, shipping-analysis
- 10 reference documents: tool-reference, customs-guide, csv-format, carrier-guide, error-reference, test-mode, address-formats, label-formats, rate-shopping-guide, international-shipping
- Example files: batch-domestic.csv, batch-international.csv, sample-addresses.json, sample-parcels.json
- MCP server connection to Shippo API via Speakeasy (Gram)
- Purchase confirmation gates for label and batch purchase workflows
- Test vs live mode detection
- Privacy notice in README
