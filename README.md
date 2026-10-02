# polish-electricity-tariffs

Static electricity-tariff data for Polish operators, served over **GitHub Pages**
(no server required). Any application can consume this data to compute electricity
cost without each user having to type their per-kWh rates by hand. A Home Assistant
integration is the first consumer, but the format is deliberately generic.

Prices come from the annual tariffs approved by **URE** (the energy regulator).
Because distribution charges are regulated and uniform per operator × tariff group,
and the obligated-seller (sprzedawca zobowiązany) energy prices are public, this is
a small curated dataset that changes roughly **once a year** — no live scraping, no
server, just JSON files updated when new tariffs are approved.

## Data

Served from GitHub Pages:

- `https://davo22.github.io/polish-electricity-tariffs/data/index.json` — operator list
- `https://davo22.github.io/polish-electricity-tariffs/data/tauron.json` — TAURON

## Format

All rates are **net** (without VAT), in PLN. Each tariff carries a list of time
`periods`; for a given date the applicable period is the one where
`valid_from <= date <= valid_to` (a `null` `valid_to` means it is still in force).
Each period lists, per zone:

| Field | Meaning |
| --- | --- |
| `energy_net` | energy (sprzedaż) price per kWh |
| `distribution_variable_net` | variable network charge per kWh |
| `quality_net` | quality charge (stawka jakościowa) per kWh |
| `oze_net` | RES charge (opłata OZE) per kWh |
| `cogeneration_net` | cogeneration charge per kWh |
| `fixed_monthly_net` | monthly fixed fees (do not affect per-kWh cost) |

Derived per-kWh prices (gross):

```
consumption_price  = (energy + distribution_variable + quality + oze + cogeneration) × (1 + vat)
net-metering credit = net_metering_factor × Σ(opust_components) × (1 + vat)
```

`opust_components` lists which components the net-metering credit is based on
(OZE and cogeneration are charged on the full draw, so they are excluded).

Energy prices are the **obligated-seller** (TAURON Sprzedaż) rates. On a market
offer the energy component differs — a consumer of this data should allow an
override. In 2023–2025 the energy component was under a statutory price freeze
(ochrona cenowa), hence a single day/night energy rate in those periods.

## Confidence

Each period has a `confidence` flag (`high` / `medium`) indicating how firmly the
rates are pinned to the published tariff tables for that year.

## Contributing data

Add or correct an operator by editing its JSON under `docs/data/` and updating
`index.json`. Cite the source (URE tariff / operator price list) and the year in
the file's `source` / `source_url` fields.

## License

[MIT](LICENSE) © Dawid Rashid. Not affiliated with TAURON, URE, or any operator.
