# Amazon Order Exporter (chmsant fork)

A Tampermonkey/Violentmonkey userscript that exports your Amazon order history to CSV or JSON.

This is a personal fork of [IeuanK/AmazonExporter](https://github.com/IeuanK/AmazonExporter) with per-item pricing and security improvements. The upstream script exports order totals; this fork enriches each order with per-item unit prices and a structured `Items` column, enabling automated split categorization when used with personal finance tools.

---

## What's different from upstream

### Per-item pricing (new in this fork)

The upstream script exports one row per order with the total amount. This fork adds an `Items` column containing a JSON array of every item in the order:

```json
[
  {"name": "Keurig K-Cup Pods Variety Pack", "qty": 2, "unitPrice": 34.99, "status": "Delivered"},
  {"name": "USB-C Hub 7-in-1", "qty": 1, "unitPrice": 29.95, "status": "Delivered"}
]
```

`unitPrice` is the pre-tax per-unit price pulled from the Amazon order detail page. It is `null` for marketplace/third-party items where Amazon doesn't expose the unit price.

**How price extraction works:**

1. During capture, the script fetches each order's detail page
2. It attempts a direct `fetch()` of the `/order-details` URL and parses item pricing from the response HTML
3. If that fails (JS-rendered pages, redirects), it opens a popup window to the detail page, runs the same DOM extraction there, and writes results to `localStorage`
4. The opener polls `localStorage` until the popup writes the price map, then closes the popup
5. Results are cached by order ID in `localStorage` (evicts oldest entries beyond 500 orders)
6. Name matching uses fuzzy normalization (lowercased, punctuation stripped, common suffixes removed) with a positional fallback for single unmatched items

### Security fixes (v0.4.12)

Upstream issues addressed in this fork:

- **XSS**: `createButton()` used `innerHTML` to set button labels — replaced with `textContent`
- **CSV injection**: order data wasn't sanitized before CSV export — formula characters (`=`, `+`, `-`, `@`) are now escaped
- **SRI integrity**: the `moment.js` CDN `@require` now pins a SHA-256 hash
- **Input validation**: `loadState()` validates the shape of data read from `localStorage` before applying it to avoid corrupted-state crashes
- **Cache eviction**: `savePriceCache()` evicts the oldest entries when the price cache exceeds 500 orders, preventing `QuotaExceededError`
- **Year bug**: `formatDateFromParts()` hardcoded the year as 2024 — fixed to use the current year

---

## Installation

- Install [Tampermonkey](https://www.tampermonkey.net/) (Chrome/Firefox/Safari/Edge)
- Open the raw script URL from **this fork**: `https://github.com/chmsant/AmazonExporter/raw/main/AmazonExporter.user.js`
- Tampermonkey will prompt to install it

> If you want the upstream version (no per-item pricing), use `https://github.com/IeuanK/AmazonExporter/raw/main/AmazonExporter.user.js` instead.

---

## Usage

1. Log in to Amazon
2. Go to your orders page (`amazon.com/gp/css/order-history`)
3. Select a year from the dropdown
4. Click **Start Capturing** — captures the current page and fetches per-item prices
5. Click **Next Page** to advance (or use the ➡️ button)
6. Repeat for all pages
7. Click **Export CSV** to download

**Allow popups from amazon.com** — the popup-based price fallback requires popups to be allowed. If popup extraction isn't working, check your browser's popup blocker.

---

## CSV format

| Column | Description |
|--------|-------------|
| `OrderId` | Amazon order ID (e.g. `114-1234567-8901234`) |
| `Date` | Order date (YYYY-MM-DD) |
| `Payee` | Always `"Amazon"` |
| `Notes` | Comma-separated item names |
| `Total` | Order total in local currency |
| `Currency` | e.g. `USD` |
| `ItemCount` | Number of items |
| `Items` | JSON array: `[{name, qty, unitPrice, status}]` |

The `Items` column is what differentiates this fork. `unitPrice` is `null` when the order detail page doesn't expose pricing (marketplace sellers, digital items).

---

## Upstream

Original script by [IeuanK](https://github.com/IeuanK/AmazonExporter). Bug fixes and UK/DE locale support contributed upstream via PRs. Per-item pricing and security fixes are specific to this fork.

![image](https://github.com/user-attachments/assets/e3d306bb-7cac-4c49-a492-1fbab0209e11)
![image](https://github.com/user-attachments/assets/e3c84085-199a-4b91-956f-064bb0076e81)
![image](https://github.com/user-attachments/assets/771ab79c-68cc-4e38-bdba-f5c17152f792)
