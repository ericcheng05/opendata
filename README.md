# Open Data

Documentation for public Hong Kong open datasets, republished as plain JSON.

## Endpoint
[opendata.myexample.work](https://opendata.myexample.work)

## Datasets

| Dataset | Entry point | Description |
|---|---|---|
| [holidays](docs/hksar/holidays.md) | `/hksar/holidays/index.json` | Hong Kong general holidays |
| [swimming-pools](docs/hksar/lcsd/swimming-pools.md) | `/hksar/lcsd/swimming-pools/index.json` | LCSD public swimming pools — identity, facilities, and schedules |
| [swimming-pool-admission](docs/hksar/lcsd/swimming-pool-admission.md) | `/hksar/lcsd/swimming-pool-admission/latest.json` | LCSD public swimming pools — live and historical admission figures |
| [restaurants](docs/restaurants.md) | `/restaurants/index.json` | Restaurant branch listings, ordering systems, and dine-in table QR mappings |

Each dataset's doc describes its JSON file layout, field reference, and usage examples.

## Layout

Every dataset is a folder. Its entry point is `index.json` for reference data or `latest.json` for time series, and its records sit beside it, keyed by id or date:

```
hksar/                               Published by the HKSAR Government
  holidays/index.json
  lcsd/                              Leisure and Cultural Services Department
    swimming-pools/index.json
    swimming-pools/{swpId}.json
    swimming-pool-admission/latest.json
    swimming-pool-admission/{YYYY-MM-DD}.json
restaurants/                         Third-party ordering platforms
  index.json
  {slug}.json
```

Government datasets live under `hksar/`, grouped by department. The docs follow the same tree under [`docs/`](docs).

## License
[CC BY 4.0](LICENSE) — reuse freely with attribution. `restaurants` is sourced from third-party ordering platforms rather than government open data; see its [doc](docs/restaurants.md) for the platforms involved before redistributing that dataset.
