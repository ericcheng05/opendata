# Hong Kong Restaurants

Branch listings, ordering systems, and dine-in table QR mappings for Hong Kong restaurant chains.

> Sourced from third-party online ordering platforms, not government open data — see [Ordering Systems](#ordering-systems).

---

## Endpoint
https://opendata.myexample.work/restaurants/index.json

---

## Files

| File | Description |
|---|---|
| `restaurants/index.json` | Registry of all restaurants in the dataset |
| `restaurants/{slug}.json` | Branches for one restaurant (e.g. `restaurants/sukiya.json`) |

---

## `restaurants/index.json` — Index

```json
{
  "last_updated": "YYYY-MM-DD",
  "restaurants": [
    { "slug": "sukiya", "name_en": "Sukiya", "name_zh": "すき家" }
  ]
}
```

| Property | Type | Description |
|---|---|---|
| `last_updated` | `string` | ISO date of last index update (`YYYY-MM-DD`) |
| `restaurants` | `Restaurant[]` | List of restaurant entries |

### `Restaurant`

| Property | Type | Description |
|---|---|---|
| `slug` | `string` | URL-safe unique identifier, used as the filename (e.g. `jong-ro-korean-restaurant` → `restaurants/jong-ro-korean-restaurant.json`) |
| `name_en` | `string` | English name |
| `name_zh` | `string` | Traditional Chinese name |

---

## `restaurants/{slug}.json` — Restaurant

```json
{
  "slug": "...",
  "last_updated": "YYYY-MM-DD",
  "branches": [ ... ]
}
```

| Property | Type | Description |
|---|---|---|
| `slug` | `string` | Matches the filename and the index entry |
| `last_updated` | `string` | ISO date of last update for this file |
| `branches` | `Branch[]` | One entry per physical location |

### `Branch`

| Property | Type | Description |
|---|---|---|
| `branch` | `string` | Human-readable branch name (e.g. `"Jordan"`, `"Mong Kok"`) |
| `system` | `string` | Name of the online ordering platform used at this branch |
| `system_url` | `string` | Homepage of the ordering platform provider |
| `urls` | `BranchUrls` | Named URL shortcuts for this branch |
| `dinein_tables` | `DineInTable[]` | Per-table ordering URLs for dine-in QR scanning. `[]` when the branch has no per-table URLs |

### `BranchUrls`

All fields are optional — only present when the branch supports that flow.

| Property | Type | Description |
|---|---|---|
| `home` | `string` | Landing page for the branch's online menu |
| `dinein` | `string` | Direct URL of the dine-in order mode |
| `takeaway` | `string` | Direct URL of the takeaway order mode |

### `DineInTable`

| Property | Type | Description |
|---|---|---|
| `id` | `number` | Table identifier as used by the ordering system |
| `url` | `string` | Full URL that would be embedded in the table's QR code |

---

## Ordering Systems

| `system` | Provider | Used by | Table URL format |
|---|---|---|---|
| `aigens` | https://www.aigens.com/ | Sukiya | `?mode=dinein&spot={id}` query parameter |
| `PinMe` | https://www.pin2eat.com/ | Jong Ro Korean Restaurant | `table_name` query parameter |
| `UCR Technology` | https://www.ucr.com.hk/ | Cafe 100% | Table URL embedded as base64 in the `data` parameter |
| `鴻智互動` | https://www.cashier.tw/ | Matsuya, Matsunoya | No per-table URLs — branch-level URLs only |

---

## Notes

- **Sushiro is a stub.** `restaurants/sushiro.json` lists branches as `{ "id": number }` only — Sushiro's own store IDs — with none of the `Branch` fields above.
- A slug can never be `index`, since that name is taken by the registry file.

---

## Example: Full Branch Entry

```json
{
  "branch": "Jordan",
  "system": "aigens",
  "system_url": "https://www.aigens.com/zh-hant/",
  "urls": {
    "home":     "https://food.order.place/home/store/444394119229440",
    "dinein":   "https://food.order.place/order/store/444394119229440/mode/dinein",
    "takeaway": "https://food.order.place/order/store/444394119229440/mode/takeaway"
  },
  "dinein_tables": [
    { "id": 1, "url": "https://food.order.place/home/store/444394119229440?mode=dinein&spot=1" },
    { "id": 2, "url": "https://food.order.place/home/store/444394119229440?mode=dinein&spot=2" }
  ]
}
```

---

## Usage Example

```js
const BASE = 'https://opendata.myexample.work';
const { restaurants } = await fetch(`${BASE}/restaurants/index.json`).then(r => r.json());

const sukiya = await fetch(`${BASE}/restaurants/sukiya.json`).then(r => r.json());
console.log(sukiya.branches.map(b => `${b.branch}: ${b.urls.dinein ?? b.urls.home ?? b.urls.takeaway}`));
```
