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

## Where to get the official numbers (update checklist)

New tariffs are approved by **URE** around **mid-December** and take effect **1 January**.
To refresh this dataset for a new year, pull the numbers from the primary sources below
(not from price-comparison sites — their figures are inconsistent and often mix market
offers with regulated tariffs).

**Primary source — URE Biuletyn Branżowy (authoritative for everything):**
- Index: <https://bip.ure.gov.pl/bip/taryfy-i-inne-decyzje-b/energia-elektryczna> —
  contains the full approving decisions **and** the tariff attachments (rate tables) for
  every OSD (distribution) and every sprzedawca z urzędu (energy). This is the ground truth.
- Example (TAURON Dystrybucja 2024): <https://bip.ure.gov.pl/download/3/17802/TauronDystrybucja.pdf>
  — the rate tables are in the section **"STAWKI OPŁAT ZA USŁUGI DYSTRYBUCJI… DLA
  POSZCZEGÓLNYCH GRUP TARYFOWYCH"** (near the end of the document).

**Distribution (stawki sieciowe) — the regulated, operator-specific part:**
- TAURON Dystrybucja: <https://www.tauron-dystrybucja.pl/uslugi-dystrybucyjne/stawki-oplat-dystrybucyjnych>
  (full tariff + a short "Wyciąg … dla odbiorców grup G" with just the G11/G12/G13 tables).
- Other OSDs publish the same on their own sites: PGE Dystrybucja, Enea Operator,
  Energa-Operator, Stoen Operator.

**Energy (cena energii / sprzedaż) — the seller-specific part:**
- Sprzedawca z urzędu (regulated default price): the seller's URE-approved tariff,
  e.g. TAURON Sprzedaż at <https://www.tauron.pl/dla-domu/prad/taryfy-cennik> and in the
  URE Biuletyn. Market offers (gwarancja ceny, eko, z serwisem, …) differ — consumers of
  this dataset override the energy component with their own contract price.

**What to copy into each `*.json` period (all values *net*, zł/kWh unless noted):**
- `distribution_variable_net` — "składnik zmienny stawki sieciowej" (per zone for G12).
- `quality_net` — "stawka jakościowa" (system-wide; same for all G groups).
- `oze_net` — "stawka opłaty OZE" (given in zł/MWh → divide by 1000).
- `cogeneration_net` — "stawka opłaty kogeneracyjnej" (zł/MWh → /1000).
- `energy_net` — sprzedawca-z-urzędu price (per zone for G12).
- `fixed_monthly_net` — składnik stały, abonament, opłata przejściowa, opłata mocowa tiers.

**Facts that save time (verified Dec 2025 for the 2024 & 2026 tariffs):**
- TAURON Dystrybucja G-group network rates are **uniform across its whole area** — one
  table, no regional split (the old Enion/EnergiaPro regional rates were unified by ~2022).
- `quality`, `oze`, `cogeneration` and `opłata mocowa` are **system-wide** — identical for
  G11 and G12 in a given year, so you can read them once.
- Energy was under a **statutory freeze** until 31.12.2025 (≈0,4140 net to 30.06.2024,
  then cena maksymalna 0,5050 to 31.12.2025, flat day=night). The freeze ended 01.01.2026,
  so from 2026 the energy component is offer-dependent again.
- Distribution was **also frozen in H1 2024** (billed at the 2023 level ≈0,1824), with the
  full 2024 tariff (0,2573 for G11) applying from 01.07.2024 — hence the split 2024 periods.

## Contributing data

Add or correct an operator by editing its JSON under `docs/data/` and updating
`index.json`. Cite the source (URE tariff / operator price list) and the year in
the file's `source` / `source_url` fields.

## License

[MIT](LICENSE) © Dawid Rashid. Not affiliated with TAURON, URE, or any operator.
