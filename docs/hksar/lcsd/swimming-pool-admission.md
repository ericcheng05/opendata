# LCSD Public Swimming Pools — Live Admission Figures

How busy each public swimming pool is, as a live snapshot plus a 15-minute daily history.

Data source: [LCSD Real-time Swimming Pool Admission Figures](https://lcsd-swimming.xports.cn/swimming-pool-page?language=en)
Coverage: the 45 pools listed by the source — every pool in [`swimming-pools`](./swimming-pools.md) except Wan Chai Swimming Pool (`swpId` 3)
All times are **HKT (UTC+8)**. The source table self-refreshes about every 60 seconds.

This dataset is the **time-series companion** to [`swimming-pools`](./swimming-pools.md). It captures how busy each pool is *right now*; the static dataset holds identity, facilities, and schedules. Records join on `swpId`.

> **Not the same as the DATA.GOV.HK feed.** The official "Swimming Pool Attendance by Venue" dataset reports **yearly** attendance totals, not live occupancy. This dataset is scraped from the LCSD real-time page instead.

---

## Endpoint
https://opendata.myexample.work/hksar/lcsd/swimming-pool-admission/latest.json

---

## Files

| File | Description |
|------|-------------|
| `hksar/lcsd/swimming-pool-admission/latest.json` | Latest snapshot for every pool. Overwritten on every capture |
| `hksar/lcsd/swimming-pool-admission/{YYYY-MM-DD}.json` | History for one day — a compact columnar time series, one sample every 15 minutes |

---

## Update frequency

Figures are captured roughly **every 15 minutes during pool opening hours** (~06:00–22:00 HKT). Outside that window, `latest.json` simply reflects the last capture of the day.

---

## `latest.json` — Latest snapshot

```json
{
  "captured_at": "2026-07-10T14:15:00+08:00",
  "source": "lcsd-swimming.xports.cn",
  "total": 45,
  "pools": [
    {
      "swpId": 2,
      "status": "open",
      "max_capacity": 781,
      "current_admission": 378
    }
  ]
}
```

> Neither the remaining headroom nor the occupancy ratio is stored — derive them from `max_capacity` and `current_admission` (`allowed = max_capacity − current_admission`, `ratio = current_admission / max_capacity`) when both are non-`null`.

> Pool name, district, address, and coordinates are **not** duplicated here — look them up in [`swimming-pools/{swpId}.json`](./swimming-pools.md) using `swpId`.

### Top-level fields

| Property | Type | Description |
|----------|------|-------------|
| `captured_at` | `string` | Capture time as ISO 8601 with `+08:00` offset (HKT) |
| `source` | `string` | Origin host of the figures |
| `total` | `integer` | Number of pool records in this snapshot |
| `pools` | `AdmissionRecord[]` | One entry per pool — see below |

---

### `pools[]` — `AdmissionRecord`

| Property | Type | Description |
|----------|------|-------------|
| `swpId` | `integer \| null` | Official LCSD pool ID, resolved by name against the static dataset. Use it to look up name, district, and location in `swimming-pools/{swpId}.json`. `null` only if the source name could not be matched |
| `status` | `string` | Operational state — see values below |
| `max_capacity` | `integer \| null` | Maximum simultaneous swimmers allowed. `null` when the pool is closed |
| `current_admission` | `integer \| null` | Swimmers currently admitted. `null` when the pool is closed |
| `name_en` | `string` (optional) | **Fallback only.** Present *only* when `swpId` is `null`, so an unmatched pool stays identifiable |

Records are ordered by `swpId` ascending (unresolved pools, if any, sort last).

#### `status` values

| Value | Meaning |
|-------|---------|
| `open` | Pool is open and reporting figures |
| `closed` | Pool is closed (e.g. weekly cleansing, temporary closure). Figures are `null` |
| `annual_maintenance` | Pool is under annual maintenance. Figures are `null` |
| `unknown` | Source status text did not match a known state |

> When a pool is closed, the source replaces its figures with a "Please contact the officer-in-charge of the swimming pool" message. Those records keep the pool present with `null` figures so the history has no gaps.

---

## `{YYYY-MM-DD}.json` — Daily history

One file per calendar day (HKT), storing every 15-minute sample that day in a
compact **columnar** layout. `swpId` defines a fixed column order for the file;
each sample appends one time to `sampling_at` and one aligned row to each figure
matrix. Minified (no whitespace) to keep the files small.

```json
{
  "date": "2026-07-14",
  "sampling_at": ["06:30", "06:45"],
  "swpId": [1, 2, 4, 5],
  "max_capacity":      [[1838, 901, 700, 1037], [1838, 901, 700, 1037]],
  "current_admission": [[12,   45,  30,  60],   [18,   52,  33,  71]]
}
```

| Property | Type | Description |
|----------|------|-------------|
| `date` | `string` | Calendar day in HKT, `YYYY-MM-DD` |
| `sampling_at` | `string[]` | Sample times as `HH:MM` (HKT), in chronological order. Index `i` selects sample `i` across the matrices |
| `swpId` | `integer[]` | Column order — the pools, ascending. Index `j` selects a pool across every matrix row |
| `max_capacity` | `(integer \| null)[][]` | `max_capacity[i][j]` = capacity of pool `swpId[j]` at time `sampling_at[i]`. `null` when the pool was not reporting |
| `current_admission` | `(integer \| null)[][]` | `current_admission[i][j]` = swimmers admitted, same indexing. `null` when not reporting |

> **No `status` field.** A `null` in both matrices means the pool was not reporting figures at that time (closed for cleansing, temporary closure, or annual maintenance). Annual-maintenance windows are documented in [`swimming-pools/{swpId}.json`](./swimming-pools.md); `max_capacity` is stored per sample because LCSD publishes no guarantee that it is constant.

History starts on 2026-07-10. A day with no captures has no file.

---

## Usage Examples

```js
const BASE = 'https://opendata.myexample.work/hksar/lcsd';
```

### Show how busy every open pool is right now

```js
const snap = await fetch(`${BASE}/swimming-pool-admission/latest.json`).then(r => r.json());

const busiest = snap.pools
  .filter(p => p.status === 'open' && p.max_capacity)
  .map(p => ({ swpId: p.swpId, ratio: p.current_admission / p.max_capacity }))
  .sort((a, b) => b.ratio - a.ratio);

// Records carry swpId, not names — fetch the name from the static dataset if needed.
console.log(busiest.map(p => `pool ${p.swpId}: ${Math.round(p.ratio * 100)}%`));
```

### Join live figures to static pool details

```js
const [snap, pool] = await Promise.all([
  fetch(`${BASE}/swimming-pool-admission/latest.json`).then(r => r.json()),
  fetch(`${BASE}/swimming-pools/2.json`).then(r => r.json())
]);

const live = snap.pools.find(p => p.swpId === pool.swpId);
console.log(`${pool.name_en} — ${live.current_admission}/${live.max_capacity}`);
```

### Plot a pool's occupancy over a day

```js
const day = await fetch(`${BASE}/swimming-pool-admission/2026-07-14.json`).then(r => r.json());

// Find the pool's column, then read that column down every sample.
const j = day.swpId.indexOf(2);

const series = day.sampling_at.map((t, i) => ({
  t,
  current: day.current_admission[i][j],
  ratio:   day.max_capacity[i][j] ? day.current_admission[i][j] / day.max_capacity[i][j] : null
}));
```

---

## Notes

- **`swpId` is resolved by name.** The source table only prints pool names, so figures are matched to `swpId` against the static dataset by name. If LCSD renames a pool so it no longer matches, `swpId` comes back `null`; use the fallback `name_en` in that case.
- **Closed pools stay in the data** with `null` figures — they are never dropped, so the columnar rows stay aligned to `swpId` across every sample.
- **Adverse weather.** The source disclaims that under the Black Rainstorm Warning, "extreme conditions", or Tropical Cyclone Signal No. 8+, the displayed status may not reflect reality in real time. Treat figures during those signals with caution.
- **Derived figures.** Remaining headroom (`max_capacity − current_admission`) and occupancy ratio (`current_admission / max_capacity`) are intentionally not stored — compute them on read.
