# LCSD Public Swimming Pools

Identity, facilities, and schedules for Hong Kong's public swimming pools.

Data source: [Leisure and Cultural Services Department (LCSD)](https://www.lcsd.gov.hk/clpss/en/webApp/Swimming.do)
Coverage: 46 public swimming pools across Hong Kong
All times are **HKT (UTC+8)**. All distances are in **metres**.

Live occupancy for the same pools is in the companion dataset [`swimming-pool-admission`](./swimming-pool-admission.md), joined on `swpId`.

---

## Endpoint
https://opendata.myexample.work/hksar/lcsd/swimming-pools/index.json

---

## Files

| File | Description |
|------|-------------|
| `hksar/lcsd/swimming-pools/index.json` | Index — region/district tree with `swpId` lists and facility class reference |
| `hksar/lcsd/swimming-pools/{swpId}.json` | Full detail for a single pool (e.g. `hksar/lcsd/swimming-pools/8.json`) |

---

## `index.json` — Index

Top-level index used to browse pools by region and district, and to look up facility class names.

```json
{
  "last_updated": "YYYY-MM-DD",
  "total": 46,
  "region": [ ... ],
  "facility_class": [ ... ]
}
```

| Property | Type | Description |
|----------|------|-------------|
| `last_updated` | `string` | ISO date of the last index update (`YYYY-MM-DD`) |
| `total` | `integer` | Total number of pools in the dataset |
| `region` | `Region[]` | Pools grouped by region, then district — see below |
| `facility_class` | `FacilityClass[]` | Reference table for facility type codes — see below |

---

### `region[]`

| Property | Type | Description |
|----------|------|-------------|
| `name_en` | `string` | Region name in English |
| `name_zh` | `string` | Region name in Traditional Chinese |
| `district` | `District[]` | List of districts within this region |

**Regions**

| `name_en` | `name_zh` |
|-----------|-----------|
| Hong Kong | 香港島 |
| Kowloon | 九龍區 |
| New Territories | 新界區 |

#### `region[].district[]`

| Property | Type | Description |
|----------|------|-------------|
| `dist` | `string` | District code used in the LCSD URL query parameter (e.g. `loc3`) |
| `name_en` | `string` | District name in English |
| `name_zh` | `string` | District name in Traditional Chinese |
| `swpId_list` | `integer[]` | List of `swpId` values belonging to this district |

**District codes**

| `dist` | `name_en` | `name_zh` | Region |
|--------|-----------|-----------|--------|
| `loc1` | Eastern | 東區 | Hong Kong |
| `loc2` | Wan Chai | 灣仔 | Hong Kong |
| `loc3` | Central and Western | 中西區 | Hong Kong |
| `loc4` | Southern | 南區 | Hong Kong |
| `loc5` | Yau Tsim Mong | 油尖旺 | Kowloon |
| `loc6` | Sham Shui Po | 深水埗 | Kowloon |
| `loc7` | Kowloon City | 九龍城 | Kowloon |
| `loc8` | Wong Tai Sin | 黃大仙 | Kowloon |
| `loc9` | Kwun Tong | 觀塘 | Kowloon |
| `loc10` | Sha Tin | 沙田 | New Territories |
| `loc11` | Sai Kung | 西貢 | New Territories |
| `loc12` | Tuen Mun | 屯門 | New Territories |
| `loc13` | Kwai Tsing | 葵青 | New Territories |
| `loc14` | Yuen Long | 元朗 | New Territories |
| `loc15` | North | 北區 | New Territories |
| `loc16` | Tai Po | 大埔 | New Territories |
| `loc17` | Tsuen Wan | 荃灣 | New Territories |
| `loc18` | Islands | 離島 | New Territories |

---

### `facility_class[]`

Reference table for the facility type codes used in every pool's `facilities[].class` field.

| Property | Type | Description |
|----------|------|-------------|
| `class` | `integer` | Facility class ID (1–8) |
| `name_en` | `string` | Facility type in English |
| `name_zh` | `string` | Facility type in Traditional Chinese |

**Class values**

| `class` | `name_en` | `name_zh` |
|---------|-----------|-----------|
| `1` | Main Pool | 主池 |
| `2` | Secondary Pool | 副池 |
| `3` | Diving Pool | 跳水池 |
| `4` | Training Pool | 訓練池 |
| `5` | Teaching Pool | 習泳池 |
| `6` | Leisure Pool | 嬉水池 |
| `7` | Toddlers Pool | 戲水池 |
| `8` | Other | 其他 |

Class `8` covers anything else, such as jacuzzis and multi-purpose pools.

---

## `{swpId}.json` — Pool Detail

One file per pool, named by its `swpId`. Contains identity, address, facilities, cleansing schedule, and opening/closure schedule.

```
GET /hksar/lcsd/swimming-pools/8.json
```

---

### Top-level fields

| Property | Type | Description |
|----------|------|-------------|
| `swpId` | `integer` | Official LCSD swimming pool ID. Matches the filename and the LCSD website URL parameter `?swpId=N` |
| `name_en` | `string` | Pool name in English |
| `name_zh` | `string` | Pool name in Traditional Chinese |
| `address_en` | `string` | Street address in English |
| `address_zh` | `string` | Street address in Traditional Chinese |
| `telephone` | `string` | Contact phone number (8 digits, no spaces) |
| `url` | `string` | Direct URL to the pool's page on the LCSD website |
| `url_maps` | `string` (optional) | Google Maps link for the pool. Present on only a few pools |
| `coordinates` | `object` | GPS location — see below |
| `weekly_cleansing_day` | `object` | Weekly pool cleaning schedule — see below |
| `facilities` | `Facility[]` | List of individual pools / water features — see below |
| `schedule` | `object` | Opening sessions, annual maintenance, and temporary closures — see below |
| `last_updated` | `string` (optional) | ISO date (`YYYY-MM-DD`) the temporary closures were last refreshed. Absent on pools that have never been refreshed |

---

### `coordinates`

| Property | Type | Description |
|----------|------|-------------|
| `latitude` | `number` | WGS84 latitude (decimal degrees) |
| `longitude` | `number` | WGS84 longitude (decimal degrees) |

---

### `weekly_cleansing_day`

All LCSD pools undergo a weekly cleansing operation from 10:00 to the end of the 2nd session. The pool reopens for the 3rd session on the same day. If the primary day falls on a public holiday, cleansing is rescheduled to the alternative day.

| Property | Type | Description |
|----------|------|-------------|
| `primary` | `integer` | Primary cleansing weekday as a JavaScript `getDay()` integer (0 = Sunday, 1 = Monday … 6 = Saturday) |
| `alternative` | `integer` | Fallback cleansing weekday if primary falls on a public holiday, same integer encoding |

---

### `facilities[]`

Each entry describes one distinct water feature within the pool complex.

| Property | Type | Description |
|----------|------|-------------|
| `fid` | `integer` | Local facility ID, scoped to this pool (1-based). Used as the reference key in `schedule.sessions[].facility_ids`, `schedule.annual_maintenance[].facility_ids`, and `schedule.temporary_closures[].facility_ids` |
| `class` | `integer` | Facility type code (1–8). Matches `facility_class[].class` in `index.json` |
| `name_en` | `string` | Facility name in English (e.g. `"Training pool"`, `"Teaching pool 2"`). There is no Chinese facility name — use `facility_class[].name_zh` for the type |
| `dimensions` | `object` | Physical dimensions — see below |
| `indoor` | `boolean` | `true` if this facility is housed in an indoor hall |
| `heated` | `boolean` | `true` if heated water is provided (typically November to mid-April) |

#### `facilities[].dimensions`

| Property | Type | Description |
|----------|------|-------------|
| `length` | `number \| null` | Pool length in metres. `null` for irregular or circular shapes |
| `width` | `number \| null` | Pool width in metres. `null` for irregular or circular shapes |
| `depth_min` | `number \| null` | Minimum water depth in metres. `null` if not specified |
| `depth_max` | `number \| null` | Maximum water depth in metres. `null` if not specified |

> Leisure pools, toddler pools, and jacuzzis are typically irregular-shaped and have `null` for `length` and `width`. Where only one depth value is given in the source, `depth_min` and `depth_max` are set to the same value.

---

### `schedule`

Describes when the pool is open and what is closed for maintenance.

```json
"schedule": {
  "sessions": [ ... ],
  "annual_maintenance": [ ... ],
  "temporary_closures": [ ... ]
}
```

---

#### `schedule.sessions[]`

An array of session schedule entries. Each entry defines the opening hours for a given combination of days, facility subset, and date range. **Entries are ordered from most specific (winter restrictions) to least specific (full-season default).** To resolve the current schedule, find the first entry whose `days`, `valid_from`, and `valid_until` match today.

| Property | Type | Description |
|----------|------|-------------|
| `days` | `integer[]` | Days of the week this entry applies to, as JavaScript `getDay()` integers. Usually `[0,1,2,3,4,5,6]` (all days); weekday-specific entries list a subset |
| `facility_ids` | `integer[] \| null` | Which facilities are open during this session entry. `null` = all facilities open. A specific list means only those `fid` values are open |
| `valid_from` | `object` | Start of the date range this entry applies to (month + day, no year — recurs annually) |
| `valid_until` | `object` | End of the date range this entry applies to (month + day, no year) |
| `slots` | `Slot[]` | Opening time slots within the day |
| `exclude_public_holidays` | `boolean` (optional) | When `true`, this entry does not apply on public holidays. Present only on a few weekday-specific entries |
| `remark_en` | `string` (optional) | Free-text note for special conditions such as lane reservations or trial schemes |

##### `valid_from` / `valid_until`

| Property | Type | Description |
|----------|------|-------------|
| `month` | `integer` | Month number (1 = January … 12 = December) |
| `day` | `integer` | Day of the month |

> These dates are **year-agnostic** — the same bounds recur every year. Both bounds are always present, and a pool's schedule is split into seasonal entries (e.g. 1 Apr → 31 Oct, 1 Nov → 31 Mar). When `valid_until` falls earlier in the year than `valid_from`, the range wraps across the new year.

##### `sessions[].slots[]`

| Property | Type | Description |
|----------|------|-------------|
| `session` | `integer` | Session number: `1` = morning, `2` = afternoon, `3` = evening. Some pools have only 1 or 2 sessions |
| `open` | `string` | Session open time in 24-hour `HH:MM` format (HKT) |
| `close` | `string` | Session close time in 24-hour `HH:MM` format (HKT) |

> Standard breaks between sessions are not explicitly represented. The gap between `slots[n].close` and `slots[n+1].open` is the session break.

**Common session patterns**

| Pattern | Pools |
|---------|-------|
| 3 sessions (06:30–12:00 / 13:00–18:00 / 19:00–22:00) | Most outdoor pools, summer season |
| 3 sessions (06:30–12:00 / 13:00–17:00 / 18:00–22:00) | Indoor-only pools (5 pm break) |
| 3 sessions with 3rd ending at 21:00 or 21:30 | Winter-restricted pools |
| 1 continuous slot (07:00–22:00) | Wan Chai (swpId 3) — no session breaks |

---

#### `schedule.annual_maintenance[]`

Recurring annual closures. Dates are **year-agnostic** (`month`/`day` only) because the same maintenance window repeats every year.

| Property | Type | Description |
|----------|------|-------------|
| `facility_ids` | `integer[] \| null` | Which facilities are closed. `null` = entire pool complex closed. A specific list means only those `fid` values are closed; others remain open |
| `start_month` | `integer` | Month the closure begins (1–12) |
| `start_day` | `integer` | Day of the month the closure begins |
| `end_month` | `integer` | Month the closure ends (1–12) |
| `end_day` | `integer` | Day of the month the closure ends (inclusive) |

> A pool may have multiple entries with different `facility_ids` and date ranges — for example, outdoor pools may close November–April while indoor pools close April–June as separate entries.

**Common annual maintenance patterns**

| Typical date range | Usually applies to |
|-------------------|-------------------|
| 1 Nov → 15 Apr | Outdoor facilities |
| 1 Nov → 31 Mar | Whole complex |
| 16 Apr → 5 Jun | Varies by pool — indoor, outdoor, or whole complex |
| 11 Sep → 31 Oct | Whole complex or indoor facilities |

---

#### `schedule.temporary_closures[]`

One-off closures published in the LCSD "Notice of Temporary Closure" table. These are specific dated events, not recurring patterns.

| Property | Type | Description |
|----------|------|-------------|
| `start` | `string` | Closure start as ISO 8601 datetime `YYYY-MM-DDTHH:MM` (HKT) |
| `end` | `string` | Closure end as ISO 8601 datetime `YYYY-MM-DDTHH:MM` (HKT) |
| `facility_ids` | `integer[] \| null` | Which facilities are affected. `null` = all facilities. A specific list means only those `fid` values are closed |
| `reason_en` | `string \| null` | Reason for closure as listed by LCSD (e.g. `"School Swimming Gala"`, `"Insufficient Lifeguard"`) |
| `remark_en` | `string \| null` | Additional remarks from the LCSD notice, if any |

Entries are sorted by `start`.

---

## Usage Examples

```js
const BASE = 'https://opendata.myexample.work/hksar/lcsd/swimming-pools';
```

### Get all pools in a district

```js
const index = await fetch(`${BASE}/index.json`).then(r => r.json());

const centralWestern = index.region
  .flatMap(r => r.district)
  .find(d => d.dist === 'loc3');

// centralWestern.swpId_list → [2, 40]
```

### Fetch a pool and check if open right now

```js
const pool = await fetch(`${BASE}/8.json`).then(r => r.json());
const now = new Date();
const currentMonth = now.getMonth() + 1; // JS months are 0-based
const currentDay   = now.getDate();
const currentHour  = now.getHours() + now.getMinutes() / 60;
const currentDow   = now.getDay();

function inRange(from, until) {
  const start = from.month  * 100 + from.day;
  const end   = until.month * 100 + until.day;
  const today = currentMonth * 100 + currentDay;
  return start <= end
    ? today >= start && today <= end
    : today >= start || today <= end; // wraps across Jan
}

function toDecimalHour(hhmm) {
  const [h, m] = hhmm.split(':').map(Number);
  return h + m / 60;
}

const activeSession = pool.schedule.sessions.find(sess =>
  sess.days.includes(currentDow) && inRange(sess.valid_from, sess.valid_until)
);

const openSlot = activeSession?.slots.find(slot =>
  currentHour >= toDecimalHour(slot.open) &&
  currentHour <  toDecimalHour(slot.close)
);

console.log(openSlot ? `Open — Session ${openSlot.session}` : 'Closed');
```

### Check which facilities are open today

```js
const pool = await fetch(`${BASE}/40.json`).then(r => r.json());

// Find the active session entry for today
const activeSession = /* ... same logic as above ... */;

const openFids = activeSession?.facility_ids ?? pool.facilities.map(f => f.fid);

const openFacilities = pool.facilities.filter(f => openFids.includes(f.fid));
console.log(openFacilities.map(f => f.name_en));
```

### Check if a facility is under annual maintenance today

```js
function isUnderAnnualMaintenance(pool, fid) {
  const now = new Date();
  const m = now.getMonth() + 1;
  const d = now.getDate();

  return pool.schedule.annual_maintenance.some(entry => {
    // Check this entry applies to the facility
    const appliesToFid = entry.facility_ids === null || entry.facility_ids.includes(fid);
    if (!appliesToFid) return false;

    // Check date range (handles year wrap-around)
    const start = entry.start_month * 100 + entry.start_day;
    const end   = entry.end_month   * 100 + entry.end_day;
    const today = m * 100 + d;
    return start <= end
      ? today >= start && today <= end
      : today >= start || today <= end;
  });
}
```

### List all heated indoor pools

```js
const index = await fetch(`${BASE}/index.json`).then(r => r.json());
const allSwpIds = index.region.flatMap(r => r.district).flatMap(d => d.swpId_list);

const results = await Promise.all(
  allSwpIds.map(id => fetch(`${BASE}/${id}.json`).then(r => r.json()))
);

const heatedIndoor = results.flatMap(pool =>
  pool.facilities
    .filter(f => f.indoor && f.heated)
    .map(f => ({ pool: pool.name_en, facility: f.name_en, swpId: pool.swpId }))
);
```

---

## Notes

- **Session breaks** are implicit — the gap between `slots[n].close` and `slots[n+1].open` is the break period. Pools do not admit new swimmers during breaks but may allow current swimmers to continue.
- **Weekly cleansing** is always 10:00 to the end of the 2nd session. The pool reopens for the 3rd session. Check `weekly_cleansing_day` for which day applies.
- **Lunar New Year** — all pools are closed for the first three days regardless of the schedule data.
- **Annual maintenance dates** are year-agnostic ranges. When the `end_month` is earlier in the calendar year than `start_month` (e.g. start = November, end = March), the closure wraps across the year boundary.
- **`temporary_closures`** reflect the LCSD notice board as of the pool's `last_updated` date. Check the LCSD page linked in `url` for anything newer.
- **`facility_ids: null`** in any schedule field always means "applies to all facilities at this pool."
