# AmazonExporter (chmsant fork)

A Tampermonkey/Violentmonkey userscript that exports Amazon order history to CSV or JSON. This is a personal fork of [IeuanK/AmazonExporter](https://github.com/IeuanK/AmazonExporter) extended with per-item pricing and security hardening.

## What it does

Adds a floating control panel to Amazon's orders page. You page through your order history clicking "Capture", then export everything as a CSV. The CSV feeds into the Actual Budget `/budget-review amazon` skill to categorize Amazon transactions.

## Key file

`AmazonExporter.user.js` — the entire script is one userscript file. Install via Tampermonkey by opening the raw URL from GitHub.

## Version history

Current version is tracked in the `@version` header at the top of the script. Use semantic versioning; bump before committing feature or fix changes.

## How to test

1. Install the script in Tampermonkey (or reload after edits via the Tampermonkey dashboard)
2. Go to amazon.com → Orders
3. Select a year from the dropdown
4. Click "Start Capturing", then page through orders
5. Click "Export CSV" and verify the output includes `Items` column with JSON-encoded item arrays

The `Items` column should contain arrays like:
```json
[{"name":"Product Name","qty":1,"unitPrice":12.99,"status":"Delivered"}]
```

`unitPrice` will be `null` for marketplace/third-party items that don't expose pricing in the order detail page.

## Upstream sync

Upstream is `IeuanK/AmazonExporter`. To check for upstream changes:
```bash
git fetch upstream
git log upstream/main..HEAD --oneline   # commits unique to this fork
git log HEAD..upstream/main --oneline   # upstream commits we don't have
```

Do not blindly merge upstream — upstream does not have the Items/unitPrice feature. Cherry-pick upstream bug fixes as needed, then verify the Items column still works.

## Integration with Actual Budget

The CSV output is consumed by the `/budget-review amazon` skill in `Code/actualbudget/`. Drop the exported file at `reference/amazon-export.csv` before running the skill. The skill uses the `Items` JSON column for split categorization of mixed-category orders.
