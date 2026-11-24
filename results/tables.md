## E1: method comparison (synthetic; conditional on the generator's price assumptions)

Breakeven price = activity factor / spend factor (kg CO2e per unit over kg CO2e per USD): the unit price at which the two methods agree. Computed from the sourced factors alone.

| Category | Activity factor (kg CO2e/unit) | Spend factor (kg CO2e/USD) | Breakeven price | Assumed median price |
|---|---:|---:|---:|---:|
| Cat. 4 truck freight | 0.18748 | 0.595 | 0.3151 USD per short ton-mile | 0.2 |
| Cat. 4 rail freight | 0.021177 | 0.466 | 0.04544 USD per short ton-mile | 0.05 |
| Cat. 5 landfilled waste | 580 | 0.988 | 587 USD per short ton | 90 |
| Cat. 6 air travel (medium haul) | 0.1301 | 0.644 | 0.202 USD per passenger-mile | 0.15 |

| Condition (price multiplier on the assumed median) | n | Median (spend - activity) / activity | 95% CI (bootstrap) | Share with abs diff > 30% (Wilson 95% CI) |
|---|---:|---:|---|---|
| Cat. 4 truck freight, x0.25 | 2000 | -83.9% | [-84.2%, -83.6%] | 100.0% [99.8%, 100.0%] |
| Cat. 4 truck freight, x0.5 | 2000 | -68.4% | [-69.1%, -67.8%] | 98.9% [98.3%, 99.3%] |
| Cat. 4 truck freight, x1 | 2000 | -35.7% | [-36.8%, -34.5%] | 60.4% [58.2%, 62.5%] |
| Cat. 4 truck freight, x2 | 2000 | 26.3% | [24.4%, 28.3%] | 49.9% [47.7%, 52.1%] |
| Cat. 4 truck freight, x4 | 2000 | 152.0% | [147.3%, 157.2%] | 97.0% [96.2%, 97.7%] |
| Cat. 4 rail freight, x0.25 | 2000 | -72.6% | [-73.0%, -72.1%] | 99.7% [99.3%, 99.9%] |
| Cat. 4 rail freight, x0.5 | 2000 | -45.7% | [-46.8%, -44.8%] | 77.0% [75.2%, 78.8%] |
| Cat. 4 rail freight, x1 | 2000 | 10.4% | [8.7%, 12.6%] | 41.9% [39.8%, 44.1%] |
| Cat. 4 rail freight, x2 | 2000 | 118.2% | [113.9%, 122.5%] | 93.3% [92.2%, 94.4%] |
| Cat. 4 rail freight, x4 | 2000 | 341.7% | [332.2%, 348.4%] | 100.0% [99.7%, 100.0%] |
| Cat. 5 landfilled waste, x0.25 | 2000 | -96.1% | [-96.2%, -96.1%] | 100.0% [99.8%, 100.0%] |
| Cat. 5 landfilled waste, x0.5 | 2000 | -92.2% | [-92.3%, -92.0%] | 100.0% [99.8%, 100.0%] |
| Cat. 5 landfilled waste, x1 | 2000 | -84.4% | [-84.7%, -84.2%] | 100.0% [99.8%, 100.0%] |
| Cat. 5 landfilled waste, x2 | 2000 | -69.2% | [-69.7%, -68.5%] | 99.0% [98.5%, 99.4%] |
| Cat. 5 landfilled waste, x4 | 2000 | -40.2% | [-41.6%, -39.1%] | 67.8% [65.7%, 69.8%] |
| Cat. 6 air travel (medium haul), x0.25 | 2000 | -81.7% | [-82.0%, -81.3%] | 100.0% [99.8%, 100.0%] |
| Cat. 6 air travel (medium haul), x0.5 | 2000 | -62.9% | [-63.4%, -61.9%] | 96.5% [95.6%, 97.2%] |
| Cat. 6 air travel (medium haul), x1 | 2000 | -25.0% | [-26.3%, -23.4%] | 47.9% [45.8%, 50.1%] |
| Cat. 6 air travel (medium haul), x2 | 2000 | 46.6% | [43.8%, 48.7%] | 65.1% [63.0%, 67.2%] |
| Cat. 6 air travel (medium haul), x4 | 2000 | 194.9% | [188.1%, 200.4%] | 99.1% [98.5%, 99.4%] |

H1 at the assumed prices: 2 of 4 categories have median |difference| > 30% (c4_truck, c5_landfill). Rule: supported iff the median |relative difference| exceeds 30% in at least half of the categories, at the assumed median prices (multiplier 1). Supported: True.

## E2: uncertainty calibration (synthetic truth)

| Condition | n organizations | Coverage of nominal 90% interval | 95% CI (Wilson) | Coverage 50% / 80% / 95% | Truth below / above the 90% interval | Median relative width (p95-p5)/p50 |
|---|---:|---:|---|---|---|---:|
| A_well_specified | 1000 | 89.5% | [87.4%, 91.3%] | 52% / 82% / 95% | 4.9% / 5.6% | 0.145 |
| B_activity_errors_2x | 1000 | 86.0% | [83.7%, 88.0%] | 44% / 74% / 92% | 8.5% / 5.5% | 0.145 |
| C_underreporting_bias | 1000 | 1.0% | [0.5%, 1.8%] | 0% / 0% / 2% | 0.0% / 99.0% | 0.145 |
| D_heavy_tails | 1000 | 89.4% | [87.3%, 91.2%] | 50% / 80% / 95% | 5.3% / 5.3% | 0.146 |
| E_factor_errors_independent | 1000 | 89.3% | [87.2%, 91.1%] | 50% / 78% / 94% | 5.5% / 5.2% | 0.145 |
| G_factor_errors_2x | 1000 | 58.5% | [55.4%, 61.5%] | 27% / 46% / 68% | 17.3% / 24.2% | 0.145 |

H2 (condition A only): supported iff the Wilson 95% CI of the 90% interval's coverage lies entirely within [0.85, 0.95]. Supported: True.

## E3: where to improve data

300 synthetic organizations; median 30 uncertain inputs each; median baseline relative 90% width 0.147. Upgrade = 95% half-width x 0.25.

| Budget k | Strategy | n | Mean reduction of 90% interval width | 95% CI (bootstrap) | Mean difference vs random | 95% CI | Holm-adjusted p (one-sided Wilcoxon) | Cliff's delta |
|---:|---|---:|---:|---|---:|---|---:|---:|
| 1 | random | 300 | 2.0% | [1.8%, 2.2%] | n/a | n/a | n/a | n/a |
| 1 | top_relative | 300 | 1.0% | [0.5%, 1.5%] | -1.1% | [-1.6%, -0.4%] | 1 | -0.71 |
| 1 | top_variance | 300 | 36.6% | [35.0%, 38.3%] | 34.6% | [33.0%, 36.2%] | 1.22e-50 | 1.00 |
| 1 | greedy | 300 | 36.6% | [35.0%, 38.2%] | n/a | n/a | n/a | n/a |
| 2 | random | 300 | 3.8% | [3.5%, 4.1%] | n/a | n/a | n/a | n/a |
| 2 | top_relative | 300 | 1.7% | [1.2%, 2.4%] | -2.1% | [-2.8%, -1.3%] | 1 | -0.73 |
| 2 | top_variance | 300 | 50.6% | [49.3%, 51.9%] | 46.7% | [45.5%, 48.1%] | 1.22e-50 | 1.00 |
| 2 | greedy | 300 | 50.6% | [49.3%, 52.0%] | n/a | n/a | n/a | n/a |
| 3 | random | 300 | 5.9% | [5.6%, 6.3%] | n/a | n/a | n/a | n/a |
| 3 | top_relative | 300 | 2.3% | [1.6%, 3.1%] | -3.6% | [-4.4%, -2.7%] | 1 | -0.76 |
| 3 | top_variance | 300 | 58.1% | [56.9%, 59.2%] | 52.2% | [51.1%, 53.3%] | 1.22e-50 | 1.00 |
| 3 | greedy | 300 | 58.1% | [57.0%, 59.2%] | n/a | n/a | n/a | n/a |
| 5 | random | 300 | 9.8% | [9.4%, 10.3%] | n/a | n/a | n/a | n/a |
| 5 | top_relative | 300 | 3.9% | [3.1%, 5.0%] | -5.9% | [-7.0%, -4.9%] | 1 | -0.77 |
| 5 | top_variance | 300 | 65.5% | [64.8%, 66.1%] | 55.6% | [54.8%, 56.4%] | 1.22e-50 | 1.00 |
| 5 | greedy | 300 | 65.6% | [64.9%, 66.3%] | n/a | n/a | n/a | n/a |

H3 (top_variance vs random, all budgets): supported iff, for every budget k, top_variance exceeds random with Holm-adjusted one-sided Wilcoxon p < 0.05 and a positive mean difference. Supported: True.

## E4: correctness

| Comparison | Basis | Hand-derived (kg CO2e) | Engine (kg CO2e) | Relative error | Tolerance | OK |
|---|---|---:|---:|---:|---:|---|
| C01 TEST fuel A, 1000 gallon | scope1 | 15500 | 15500 | 0.00e+00 | 1e-09 | yes |
| C02 TEST fuel A, 3785.411784 litres (= 1000 US gallons) | scope1 | 15500 | 15500 | 4.69e-16 | 1e-08 | yes |
| C03 EPA Hub 2025 natural gas, 10000 scf (per-scf factor) | scope1 | 544.9534 | 544.9534 | 0.00e+00 | 1e-09 | yes |
| C04 EPA Hub 2025 natural gas, 1000 therm (per-mmBtu factor after conversion) | scope1 | 5311.45 | 5311.45 | 1.71e-16 | 1e-09 | yes |
| C05 EPA Hub 2025 diesel (mobile, CO2 only), 500 gallon | scope1 | 5105 | 5105 | 0.00e+00 | 1e-09 | yes |
| C06 EPA Hub 2025 eGRID RFCW, 250000 kWh, location-based | scope2_location | 103868.0034 | 103868.0034 | 2.80e-16 | 1e-09 | yes |
| C06 EPA Hub 2025 eGRID RFCW, 250000 kWh, location-based | scope2_market | 103868.0034 | 103868.0034 | 2.80e-16 | 1e-09 | yes |
| C07 EPA Hub 2024 eGRID RFCW, 250000 kWh (earlier pinned edition) | scope2_location | 114040.9463 | 114040.9463 | 1.28e-16 | 1e-09 | yes |
| C07 EPA Hub 2024 eGRID RFCW, 250000 kWh (earlier pinned edition) | scope2_market | 114040.9463 | 114040.9463 | 1.28e-16 | 1e-09 | yes |
| C08 Market-based, 250000 kWh RFCW, 60% covered by a zero-emission instrument | scope2_location | 103868.0034 | 103868.0034 | 2.80e-16 | 1e-09 | yes |
| C08 Market-based, 250000 kWh RFCW, 60% covered by a zero-emission instrument | scope2_market | 41547.20136 | 41547.20136 | 3.50e-16 | 1e-09 | yes |
| C09 EPA Hub 2025 purchased steam, 400 mmBtu | scope2_location | 26559.25 | 26559.25 | 1.37e-16 | 1e-09 | yes |
| C09 EPA Hub 2025 purchased steam, 400 mmBtu | scope2_market | 26559.25 | 26559.25 | 1.37e-16 | 1e-09 | yes |
| C10 EPA Hub 2025 R-410A blend, 12 kg released | scope1 | 23088 | 23088 | 0.00e+00 | 1e-09 | yes |
| C11 EPA Hub 2025 truck freight, 120000 short ton-miles (Scope 3 cat. 4) | scope3 | 22497.096 | 22497.096 | 0.00e+00 | 1e-09 | yes |
| C12 EPA Hub 2025 medium-haul air, 16093.44 passenger-km (= 10000 passenger-miles; cat. 6) | scope3 | 1301.033 | 1301.033 | 1.75e-16 | 1e-08 | yes |
| C13 EPA Hub 2025 mixed MSW landfilled, 7000 lb (cat. 5) | scope3 | 2030 | 2030 | 0.00e+00 | 1e-09 | yes |
| C14 EPA Hub 2025 wood and wood residuals, 100 mmBtu (biogenic CO2 outside the scope total) | scope1 | 115.56 | 115.56 | 1.23e-16 | 1e-09 | yes |
| C14 EPA Hub 2025 wood and wood residuals, 100 mmBtu (biogenic CO2 outside the scope total) (biogenic CO2 memo) | biogenic | 9380 | 9380 | 0.00e+00 | 1e-09 | yes |
| C15 USEEIO v1.3 spend, NAICS 484121 truckload freight, 25000 USD (2022 USD) | scope3 | 14875 | 14875 | 0.00e+00 | 1e-09 | yes |
| C16 USEEIO v1.3 spend with explicit price-year deflator 0.92, 25000 USD | scope3 | 13685 | 13685 | 0.00e+00 | 1e-09 | yes |

21 of 21 comparisons within tolerance; maximum relative error 4.69e-16. Unit-safety checks passed: 16 of 16.

| Unit-safety check | Passed |
|---|---|
| kWh + MWh raises UnitMismatchError | yes |
| kWh + gallon raises DimensionError | yes |
| bare number + Qty raises TypeError | yes |
| 1 MWh converts to exactly 1000 kWh | yes |
| gallon to kWh conversion raises DimensionError | yes |
| unknown unit raises UnknownUnitError | yes |
| passenger-mile and vehicle-mile are not interchangeable | yes |
| electricity activity in gallons raises CalcError | yes |
| negative quantity raises RecordError | yes |
| spend in a different price year without deflator raises PriceYearError | yes |
| spend in a non-USD currency raises an error | yes |
| declared activity method with USD quantity raises CalcError | yes |
| factor with TEST_ id inside a published set is rejected | yes |
| changed factor file is detected as drift | yes |
| kWh and MWh inputs agree to 1e-12 relative (500 random quantities) | yes |
| MWh -> kWh -> MWh round trip is exact to 1e-12 (500 random quantities) | yes |

