# Hong Kong General Holidays

Hong Kong general holidays, mirrored from the government's official calendar.

Data source: [1823.gov.hk general holidays iCal feed](https://www.1823.gov.hk/common/ical/en.json)

---

## Endpoint
https://opendata.myexample.work/hksar/holidays/index.json

---

## `hksar/holidays/index.json`

```json
{
  "holidays": [
    { "name": "The first day of January", "year": 2026, "month": 0, "date": 1 },
    { "name": "Lunar New Year's Day", "year": 2026, "month": 1, "date": 17 }
  ]
}
```

| Property | Type | Description |
|---|---|---|
| `holidays` | `Holiday[]` | Every holiday in the source feed, in feed order (currently 2024–2026) |

### `Holiday`

| Property | Type | Description |
|---|---|---|
| `name` | `string` | Holiday name, as published by the source |
| `year` | `integer` | Calendar year |
| `month` | `integer` | Month, **0-indexed** (`0` = January … `11` = December), matching JavaScript `Date` conventions |
| `date` | `integer` | Day of the month |

---

## Notes

- The file is a straight re-shaping of the source iCal feed's `vevent` entries — no filtering or deduplication is applied.
- `month` is 0-indexed to match `Date.getMonth()`/`new Date(year, month, date)` in JavaScript. Add 1 when formatting for display or other date libraries.

---

## Usage Example

```js
const BASE = 'https://opendata.myexample.work';
const { holidays } = await fetch(`${BASE}/hksar/holidays/index.json`).then(r => r.json());

const isHoliday = (d) => holidays.some(h =>
  h.year === d.getFullYear() && h.month === d.getMonth() && h.date === d.getDate()
);

console.log(isHoliday(new Date()) ? 'Public holiday today' : 'Not a public holiday');
```
