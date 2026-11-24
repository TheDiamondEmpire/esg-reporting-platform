# Emission-factor provenance

Every number the engine multiplies by comes from one of the files in the `factors/` folder. This page says where
each file came from, how it was extracted, what it covers and what it does not. Nothing in the library was typed
from memory: values are read programmatically from the published files and each factor records the table, row
label and cell addresses it came from.

## Sets in the library

| Set id | File | Source | Edition / date | Factors | Licence as recorded |
|---|---|---|---|---:|---|
| `epa-hub-2025` | `factors/epa_hub_2025.json` | U.S. EPA Center for Corporate Climate Leadership, GHG Emission Factors Hub (workbook) | 2025 edition, last modified 2025-01-15 | 421 | US Government work; the download page states no licence; EPA's data licence statement says EPA-produced data is by default public domain (17 U.S.C. 105) |
| `epa-hub-2024` | `factors/epa_hub_2024.json` | same, 2024 edition (kept so historical inventories can be regenerated) | last modified 2024-06-05 | 421 | same |
| `useeio-v1.3` | `factors/useeio_v1_3.json` | U.S. EPA Office of Research and Development, Supply Chain Greenhouse Gas Emission Factors v1.3.0, NAICS-6, kg CO2e per 2022 USD (purchaser price) | dataset modified 2024-07-05 | 1016 | US Government work; EPA data licence statement: public domain by default, no warranty |
| `test-synthetic-v1` | `factors/test_factors.json` | **Synthetic.** Written by hand so worked examples have easy arithmetic | n/a | 5 | n/a |

All three published sets were retrieved on 2026-10-23.

Source URLs (also stored in every factor record):

- 2025 Hub: `https://www.epa.gov/system/files/other-files/2025-01/ghg-emission-factors-hub-2025.xlsx`
- 2024 Hub: `https://www.epa.gov/system/files/documents/2024-02/ghg-emission-factors-hub-2024.xlsx`
- Supply chain factors: `https://pasteur.epa.gov/uploads/10.23719/1531143/SupplyChainGHGEmissionFactors_v1.3.0_NAICS_CO2e_USD2022.csv` (DOI `https://doi.org/10.23719/1531143`; the dataset "About" document is by Wesley Ingwersen, 2024-07-05)

SHA-256 of the downloaded source files (recorded in each factor as `source_sha256`):

| File | SHA-256 |
|---|---|
| 2025 Hub workbook | `43afb91d79b2ae765b3a447549d9e1021a144a407cec5eb804be9fcf69a668a7` |
| 2024 Hub workbook | `e9eb5b3bf8ee6aae1153444f2d915b72d8c7b408c80269b4434a0422e33299c0` |
| Supply chain factors CSV | `6025a14c4fe16675735efba1683030e4d36011e3973f3c4261f8231aa69c002f` |

## What was extracted

From each Hub edition (all numeric entries of these tables, nothing selected by hand):

| Hub table | Library activity types | Scope / category |
|---|---|---|
| Table 1, stationary combustion | `stationary.<fuel>` (per mmBtu, and per physical unit where the source gives one) | Scope 1 |
| Table 2, mobile combustion CO2 | `mobile.<fuel>` (CO2 only) | Scope 1 |
| Table 6, electricity (eGRID subregions, total output emission rates) | `electricity.grid` by region | Scope 2 location-based |
| Table 7, steam and heat | `steam_heat.purchased` | Scope 2 |
| Table 8, transport | `freight.<mode>` (vehicle-mile and short ton-mile) | Scope 3 cat. 4 |
| Table 9, waste | `waste.<material>.<route>` (metric tons CO2e per short ton) | Scope 3 cat. 5 |
| Table 10, business travel and commuting | `travel.<mode>` | Scope 3 cat. 6 and 7 |
| Tables 11 and 12, GWPs and blended refrigerants | `refrigerant.<gas or blend>` (kg CO2e per kg released = the 100-year GWP) | Scope 1 fugitive |

The set's CO2e conversion uses the Hub's own AR5 100-year GWPs (CH4 28, N2O 265), read from Table 11.

From the supply-chain CSV: all 1,016 NAICS-6 commodities, using the column "Supply Chain Emission Factors with
Margins" (kg CO2e per 2022 USD at purchaser price). The "without margins" and "margins" values are kept in each
factor's notes.

## How values were extracted and checked

- `tools/build_factor_library.py --build` reads the workbook cells and the CSV rows and writes the JSON files. Values
  are copied as stored in the file; the only transformation is stripping binary floating-point noise to 12
  significant digits (for example `0.022000000000000002` becomes `0.022`). Cells containing the text `NA` are
  skipped.
- The 2024 workbook stores several transport and travel factors at full floating-point precision (for example
  `0.129260450161`); those are kept as stored. The 2025 workbook stores rounded values. The displayed PDF value may
  differ from the stored cell value.
- `tools/build_factor_library.py --verify` re-extracts from the downloaded files and compares byte for byte with the
  checked-in JSON. It needs the downloaded sources (`--fetch` downloads them into the gitignored `.tmp/` folder).
- `tests/test_factors.py` spot-checks values that were read directly from the cells (for example 2025 natural gas
  per scf `0.05444 kg CO2`, 2025 RFCW `911.321 lb CO2 / MWh`, 2024 RFCW `1000.053 lb CO2 / MWh`) and asserts that
  every factor has the full provenance record, that TEST factors are segregated, and that a changed factor file is
  detected when an inventory is regenerated.
- Inventories pin each set by the SHA-256 of its JSON file. If a pinned file changes, regenerating the inventory
  fails with a drift error instead of silently giving a different number.

## What the sources do not give, and what this platform assumed

- **Uncertainty.** None of these sources publishes an uncertainty for the factors used here. The relative
  half-widths of the 95% interval stored in each factor (5% fuels, 15% grid, 20% steam, 30% freight, travel and
  refrigerants, 40% waste, 50% spend-based) are **platform default assumptions**, labelled as such in every
  record (`basis`). They are not published figures.
- **Residual mix.** The Hub contains no residual-mix factors. Market-based Scope 2 therefore uses
  organization-supplied contractual-instrument rates, and falls back to the location-based factor (flagged) for
  electricity not covered by an instrument.
- **Mobile CH4 and N2O.** The Hub's vehicle-mile tables for on-road and non-road CH4 and N2O (Tables 3 to 5) are not
  in the library, so mobile combustion is CO2 only. Table 1 CH4 and N2O factors are stationary-source factors and
  are not used for vehicles.
- **Non-baseload grid factors and grid loss rates** (Table 6) are not used. The source says non-baseload factors are
  not for inventories and loss rates are not for end-use Scope 2.
- **Upstream (well-to-tank) emissions** are not included by any Hub factor used here; the factors are combustion
  only. Scope 3 category 3 (fuel- and energy-related activities) is therefore not calculated.
- **Geography.** The factors describe the United States (eGRID subregions, US sector averages). Applying them to
  another country is not meaningful.
- **Spend-based factors** describe average US sectors in 2022 dollars. The dataset documentation says they are
  primarily intended for Scope 3 categories 1 and 2; using them as the spend-based alternative for categories 4, 5,
  6 and 7 is a platform choice. The v1.3 release has no sub-industry detail for waste sectors (for example landfills
  share the generic waste-sector factor).
- **Synthetic TEST factors.** Ids start with `TEST_`, the set is `test-synthetic-v1`, and every report, factor page
  and line that uses one carries a TEST warning. They exist only so the engine can be demonstrated with arithmetic
  that is easy to do by hand.

## Not retrieved

UK government (DESNZ/Defra) conversion factors were not retrieved in this build, so no UK or other non-US factor set
exists in the library.
