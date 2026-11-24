# ESG & Carbon Reporting Platform: Research Report

> **Where the code is.** The experiment scripts that produced these results are part of the source package, which is distributed separately (see [SOURCE-ACCESS.md](../SOURCE-ACCESS.md)). The raw results they wrote are included in this repository under [`results/`](../results/).

> **Status: protocol v0.2, results recorded.** All four experiments were run once, in a single uninterrupted invocation, on **synthetic organizations**; every number below is copied from files in [`results/`](../results/tables.md). The generator defines the ground truth, so the experiments measure the platform's behaviour under the generator's assumptions, not real-world accuracy. Nothing here states that any inventory satisfies a regulation or disclosure framework.

## Abstract

The study asks how much the choice of method (spend-based versus activity-based) and the quality of input data change reported emissions, and how well the platform's Monte-Carlo uncertainty intervals reflect error. On synthetic organizations with known truth: (E1) the two methods differ by more than 30% (median absolute difference) in 2 of 4 tested Scope 3 categories at the assumed unit prices, which meets the pre-stated "at least half" rule only at the boundary and flips with a factor-of-two change in the assumed prices; the assumption-free output is the breakeven unit price implied by the two published EPA factors. (E2) When the generator matches the engine's assumptions, the nominal 90% interval covered the truth in 89.5% of 1,000 organizations (95% CI 87.4% to 91.3%); when assumptions are wrong coverage collapses (1.0% under a systematic 20% under-reporting bias, 58.5% when true factor errors are twice the assumed size). (E3) Upgrading the inputs that contribute most to total variance reduced the 90% interval width by 36.6% to 65.5% (budgets of 1 to 5 inputs) against 2.0% to 9.8% for random inputs; picking the inputs with the largest relative uncertainty did worse than random. (E4) The engine reproduced 21 of 21 hand-derived worked examples to a maximum relative error of 4.7e-16, and passed 16 of 16 unit-safety checks. The emission factors are real, sourced U.S. EPA publications; the activity data, prices, errors and uncertainty parameters are synthetic or assumed.

## 1. Research question

How much does the choice of method (spend-based vs. activity-based) and the quality of input data change reported emissions, and how well do the platform's uncertainty estimates reflect that error?

## 2. Hypotheses

Protocol v0.1 stated four hypotheses. v0.2 keeps them and fixes how each is decided (before the final run; see the change log for what was reworded and why).

- **H1.** Spend-based Scope 3 estimates differ from activity-based estimates by more than 30% for at least half of the tested categories. *Decision rule:* at the assumed median prices, the median absolute relative difference exceeds 30% in at least half of the four categories. The outcome depends on the generator's price assumptions and says nothing about real-world differences.
- **H2.** Monte-Carlo uncertainty intervals at 90% nominal level contain the reference value in 85 to 95% of synthetic test cases. *Decision rule:* confirmatory only for condition A (generator equal to the engine's assumptions): supported iff the Wilson 95% interval of the coverage lies entirely within [85%, 95%]. Conditions B to G are exploratory sensitivity runs.
- **H3.** Targeted improvement of the highest-uncertainty inputs reduces total inventory uncertainty more than improving the same number of random inputs. *Decision rule:* "targeted" means the single-input variance ranking (`top_variance`); supported iff for every budget k in {1, 2, 3, 5} it beats random with a Holm-adjusted one-sided Wilcoxon p below 0.05 and a positive mean difference. A second heuristic (`top_relative`, the largest relative uncertainty) is reported as exploratory.
- **H4.** The calculation engine reproduces worked examples to within rounding tolerance. *v0.2 wording:* **hand-derived** worked examples written by the repository author (not published examples; none was retrieved), compared at a relative tolerance of 1e-9 (1e-8 where a unit conversion is involved).

## 3. Background

The scope structure, the Scope 2 location- and market-based methods and the 15 Scope 3 categories used here follow the GHG Protocol documents ([1], [2], [3]); ISO 14064-1 ([4]) is the corresponding organization-level standard, and the 2006 IPCC Guidelines ([5]) include a general-guidance chapter on uncertainties. None of these texts was re-read in full for this build; they are cited for context, and no claim in this report depends on their exact wording.

The emission factors used are U.S. EPA publications: the GHG Emission Factors Hub ([6]) for Scope 1, Scope 2 and activity-based Scope 3 factors, and the Supply Chain Greenhouse Gas Emission Factors v1.3 ([7]) for spend-based factors. The dataset's own documentation (read for this build) says the supply-chain factors are primarily intended for Scope 3 categories 1 and 2 and complement the Hub for Scopes 1 and 2; using them as the spend-based alternative for categories 4, 5, 6 and 7 is this platform's choice and is outside that primary intended use.

The study is deliberately small. It tests whether this implementation behaves as designed (correct arithmetic, honest unit handling, an uncertainty layer that is calibrated when its assumptions hold and informative about where to improve data) rather than claiming a new estimate of any real organization's emissions. Statistical methods: Wilson intervals for proportions ([8]), Holm's sequentially rejective procedure for multiple comparisons ([9]) and Cliff's delta as an ordinal effect size ([10]).

## 4. Method

### 4.1 System under test

A carbon-accounting platform (Python 3.12) that gathers activity data from CSV or JSON files (fuel, utilities, travel, waste, procurement, supplier replies), converts it to emissions with versioned and sourced factors, scores data quality and uncertainty, and keeps an evidence trail behind every number. Calculated: Scope 1 (stationary, mobile CO2 only, refrigerants), Scope 2 (location- and market-based, steam and heat), Scope 3 categories 1, 4, 5, 6 and 7. See the [README](../README.md) for what is not built.

### 4.2 Environment

- **Organizations:** synthetic, generated by `experiments/synth.py` (a menu of 12 activity types plus 3 to 8 spend lines; quantities lognormal across organizations; about 17 calculated lines per organization). Seeds are derived from a master seed (20261023) and a per-unit key.
- **Factors:** real, sourced EPA factors (see [factor provenance](factor-provenance.md)): `epa-hub-2025` and `useeio-v1.3`. No TEST factor is used in any experiment except E4 cases C01 and C02, whose purpose is easy hand arithmetic.
- **Uncertainty parameters:** factor spreads (5% fuels to 50% spend-based, relative half-width of the 95% interval) and the data-quality-to-spread table (2%, 5%, 10%, 25%, 50% for scores 1 to 5) are **platform assumptions, not published figures**.
- **Machine:** AMD EPYC 7543P (8 logical cores visible), Windows 11, Python 3.12.13, numpy 2.5.4, scipy 1.18.1. At most 2 worker processes with one BLAS thread each. The machine was shared with other builds and measured CPU load was 100% before and after the run (see `results/run_metadata.json`). No timing is reported as a result.

### 4.3 Baselines

- **Spend-based only:** the comparator in E1 (sector-average factor times spend).
- **Single-factor lookup without uncertainty:** the central value of every line is exactly this; E2 and E3 evaluate what the uncertainty layer adds. It is not run as a separate comparator.
- **Spreadsheet calculation of the same inputs:** not run as a separate comparator. The hand-derived arithmetic in E4 plays that role for the central values.

### 4.4 Independent variables

Calculation method (E1); the generator's error model (E2 conditions A to G); which inputs are upgraded and how many (E3); factor edition (a pinned edition changes the result in E4 case C07 versus C06; not otherwise varied).

### 4.5 Metrics

Relative difference between methods; coverage of the 90% interval (and 50%, 80%, 95%); reduction of the relative 90% interval width `(p95 - p5) / p50` of the location-basis total; relative error against the hand-derived values.

## 5. Experiment matrix

| ID | Experiment | Manipulation | Measured | Command |
|---|---|---|---|---|
| E1 | Method comparison | Spend-based vs. activity-based by category; price multiplier 0.25 to 4 on the assumed median unit price | Relative difference | `uv run python -m experiments.e1` |
| E2 | Uncertainty calibration | Monte-Carlo intervals vs known synthetic truth under 6 error models | Interval coverage | `uv run python -m experiments.e2` |
| E3 | Where to improve data | Random vs relative-uncertainty vs variance-ranked vs greedy upgrades, budgets 1 to 5 | Uncertainty reduction | `uv run python -m experiments.e3` |
| E4 | Correctness | Hand-derived worked examples and unit-safety checks | Reproduction error | `uv run python -m experiments.e4` |

All four run with `uv run python -m experiments.run_all`, which is resumable (finished work units are skipped after an interruption or a crashed worker).

## 6. Statistical plan (as executed)

- **Replication:** run counts fixed before the final run: E1 2,000 organizations per category and price multiplier; E2 1,000 organizations per condition with 4,000 Monte-Carlo draws each; E3 300 organizations with 3,000 draws and 20 random subsets per budget.
- **Summaries:** E1 medians with percentile-bootstrap 95% intervals (2,000 resamples); E3 means with percentile-bootstrap 95% intervals. Proportions (E1 share above 30%, E2 coverage) use Wilson 95% intervals. v0.1 said bootstrap for everything; Wilson is the standard choice for a binomial proportion and was fixed before the final run.
- **Comparisons:** E3 paired one-sided Wilcoxon signed-rank tests (scipy), across organizations, with Cliff's delta.
- **Multiple comparisons:** Holm adjustment across the four budgets in E3 (separately for each strategy comparison).
- **Exclusions:** none. Every organization generated was analysed.
- **Negative results are results:** see the Discussion.

## 7. Results

Raw per-unit results are in [`results/units/`](../results/tables.md); the summary is `results/summary.json` and the tables below are copied from `results/tables.md`. Run metadata: invocation `1bf793ec-2bf0-4e09-972e-9b6ea9ab23fb`, 93 work units executed in one invocation (none resumed, no worker crash, no stale unit), 259.7 s wall clock, started 2026-10-23T16:12:41Z, finished 2026-10-23T16:17:08Z, two worker processes, none left running afterwards.

### E1. Method comparison

**Manipulation:** spend-based vs. activity-based by category. **Measured:** relative difference `(spend - activity) / activity`.

**Read this first.** The generator draws a true activity and a unit price and sets spend = activity x price, so the relative difference equals `price / breakeven price - 1`. The result therefore measures the generator's price assumption against the breakeven price implied by two published factors. It does **not** say which method is closer to reality, and it says nothing about real-world differences between the methods. The only assumption-free output is the breakeven price (computed from the sourced factors alone). The assumed median prices are illustrative guesses written down by the author before the run, with the factor magnitudes already in view; they are not market data.

| Category | Activity factor (kg CO2e/unit) | Spend factor (kg CO2e/USD) | Breakeven price | Assumed median price |
|---|---:|---:|---:|---:|
| Cat. 4 truck freight | 0.18748 | 0.595 | 0.3151 USD per short ton-mile | 0.2 |
| Cat. 4 rail freight | 0.021177 | 0.466 | 0.04544 USD per short ton-mile | 0.05 |
| Cat. 5 landfilled waste | 580 | 0.988 | 587 USD per short ton | 90 |
| Cat. 6 air travel (medium haul) | 0.1301 | 0.644 | 0.202 USD per passenger-mile | 0.15 |

| Condition (price multiplier on the assumed median) | n | Result: median (spend - activity) / activity | 95% CI (bootstrap) | Notes: share with absolute difference above 30% (Wilson 95% CI) |
|---|---:|---:|---|---|
| Cat. 4 truck freight, x0.25 | 2000 | -83.9% | [-84.2%, -83.6%] | 100.0% [99.8%, 100.0%] |
| Cat. 4 truck freight, x0.5 | 2000 | -68.4% | [-69.1%, -67.8%] | 98.9% [98.3%, 99.3%] |
| **Cat. 4 truck freight, x1** | 2000 | -35.7% | [-36.8%, -34.5%] | 60.4% [58.2%, 62.5%] |
| Cat. 4 truck freight, x2 | 2000 | 26.3% | [24.4%, 28.3%] | 49.9% [47.7%, 52.1%] |
| Cat. 4 truck freight, x4 | 2000 | 152.0% | [147.3%, 157.2%] | 97.0% [96.2%, 97.7%] |
| Cat. 4 rail freight, x0.25 | 2000 | -72.6% | [-73.0%, -72.1%] | 99.7% [99.3%, 99.9%] |
| Cat. 4 rail freight, x0.5 | 2000 | -45.7% | [-46.8%, -44.8%] | 77.0% [75.2%, 78.8%] |
| **Cat. 4 rail freight, x1** | 2000 | 10.4% | [8.7%, 12.6%] | 41.9% [39.8%, 44.1%] |
| Cat. 4 rail freight, x2 | 2000 | 118.2% | [113.9%, 122.5%] | 93.3% [92.2%, 94.4%] |
| Cat. 4 rail freight, x4 | 2000 | 341.7% | [332.2%, 348.4%] | 100.0% [99.7%, 100.0%] |
| Cat. 5 landfilled waste, x0.25 | 2000 | -96.1% | [-96.2%, -96.1%] | 100.0% [99.8%, 100.0%] |
| Cat. 5 landfilled waste, x0.5 | 2000 | -92.2% | [-92.3%, -92.0%] | 100.0% [99.8%, 100.0%] |
| **Cat. 5 landfilled waste, x1** | 2000 | -84.4% | [-84.7%, -84.2%] | 100.0% [99.8%, 100.0%] |
| Cat. 5 landfilled waste, x2 | 2000 | -69.2% | [-69.7%, -68.5%] | 99.0% [98.5%, 99.4%] |
| Cat. 5 landfilled waste, x4 | 2000 | -40.2% | [-41.6%, -39.1%] | 67.8% [65.7%, 69.8%] |
| Cat. 6 air travel (medium haul), x0.25 | 2000 | -81.7% | [-82.0%, -81.3%] | 100.0% [99.8%, 100.0%] |
| Cat. 6 air travel (medium haul), x0.5 | 2000 | -62.9% | [-63.4%, -61.9%] | 96.5% [95.6%, 97.2%] |
| **Cat. 6 air travel (medium haul), x1** | 2000 | -25.0% | [-26.3%, -23.4%] | 47.9% [45.8%, 50.1%] |
| Cat. 6 air travel (medium haul), x2 | 2000 | 46.6% | [43.8%, 48.7%] | 65.1% [63.0%, 67.2%] |
| Cat. 6 air travel (medium haul), x4 | 2000 | 194.9% | [188.1%, 200.4%] | 99.1% [98.5%, 99.4%] |

Bold rows are the assumed prices (multiplier 1) on which H1 is decided. **H1 at the assumed prices:** 2 of 4 categories (truck freight and landfilled waste) have a median absolute difference above 30%; rail (+10.4%) and air (-25.0%) do not. By the pre-stated rule ("at least half") H1 is supported, **but only at the boundary**: a different assumed price in any one category flips the verdict (air at x0.5 or x2, rail at x0.5 or x2), so the honest summary is that the verdict is set by the price assumptions.

### E2. Uncertainty calibration

**Manipulation:** Monte-Carlo propagation on synthetic organizations, truth drawn under six error models. **Measured:** coverage of the nominal 90% central interval, the share of organizations whose true total lies inside it. Condition A is the only confirmatory condition.

| Condition | n | Result: coverage of nominal 90% interval | 95% CI (Wilson) | Notes: coverage at 50% / 80% / 95% nominal; truth below / above the 90% interval; median relative width |
|---|---:|---:|---|---|
| A_well_specified (confirmatory) | 1000 | 89.5% | [87.4%, 91.3%] | 52% / 82% / 95%; 4.9% / 5.6%; 0.145 |
| B_activity_errors_2x | 1000 | 86.0% | [83.7%, 88.0%] | 44% / 74% / 92%; 8.5% / 5.5%; 0.145 |
| C_underreporting_bias (reported = 0.8 x true) | 1000 | 1.0% | [0.5%, 1.8%] | 0% / 0% / 2%; 0.0% / 99.0%; 0.145 |
| D_heavy_tails (Student-t(3) activity errors) | 1000 | 89.4% | [87.3%, 91.2%] | 50% / 80% / 95%; 5.3% / 5.3%; 0.146 |
| E_factor_errors_independent | 1000 | 89.3% | [87.2%, 91.1%] | 50% / 78% / 94%; 5.5% / 5.2%; 0.145 |
| G_factor_errors_2x | 1000 | 58.5% | [55.4%, 61.5%] | 27% / 46% / 68%; 17.3% / 24.2%; 0.145 |

**H2 (condition A only):** the Wilson 95% interval [87.4%, 91.3%] lies within [85%, 95%]: supported. This is the expected outcome of a correct implementation when the generator equals the engine's assumptions; it is a calibration check of the code, not evidence that real data errors follow these distributions.

### E3. Where to improve data

**Manipulation:** targeted vs. random input upgrades (an upgrade shrinks one input's 95% half-width to a quarter). **Measured:** reduction of the 90% interval width `(p95 - p5) / p50` of the location-basis total. 300 synthetic organizations, median 30 uncertain inputs each, median baseline relative width 0.147.

| Budget k | Strategy | n | Result: mean width reduction | 95% CI (bootstrap) | Notes: mean difference vs random [95% CI]; Holm-adjusted p; Cliff's delta |
|---:|---|---:|---:|---|---|
| 1 | random | 300 | 2.0% | [1.8%, 2.2%] | reference |
| 1 | top_relative | 300 | 1.0% | [0.5%, 1.5%] | -1.1% [-1.6%, -0.4%]; p = 1; delta -0.71 |
| 1 | top_variance | 300 | 36.6% | [35.0%, 38.3%] | +34.6% [33.0%, 36.2%]; p = 1.22e-50; delta 1.00 |
| 1 | greedy | 300 | 36.6% | [35.0%, 38.2%] | not tested (ceiling) |
| 2 | random | 300 | 3.8% | [3.5%, 4.1%] | reference |
| 2 | top_relative | 300 | 1.7% | [1.2%, 2.4%] | -2.1% [-2.8%, -1.3%]; p = 1; delta -0.73 |
| 2 | top_variance | 300 | 50.6% | [49.3%, 51.9%] | +46.7% [45.5%, 48.1%]; p = 1.22e-50; delta 1.00 |
| 2 | greedy | 300 | 50.6% | [49.3%, 52.0%] | not tested (ceiling) |
| 3 | random | 300 | 5.9% | [5.6%, 6.3%] | reference |
| 3 | top_relative | 300 | 2.3% | [1.6%, 3.1%] | -3.6% [-4.4%, -2.7%]; p = 1; delta -0.76 |
| 3 | top_variance | 300 | 58.1% | [56.9%, 59.2%] | +52.2% [51.1%, 53.3%]; p = 1.22e-50; delta 1.00 |
| 3 | greedy | 300 | 58.1% | [57.0%, 59.2%] | not tested (ceiling) |
| 5 | random | 300 | 9.8% | [9.4%, 10.3%] | reference |
| 5 | top_relative | 300 | 3.9% | [3.1%, 5.0%] | -5.9% [-7.0%, -4.9%]; p = 1; delta -0.77 |
| 5 | top_variance | 300 | 65.5% | [64.8%, 66.1%] | +55.6% [54.8%, 56.4%]; p = 1.22e-50; delta 1.00 |
| 5 | greedy | 300 | 65.6% | [64.9%, 66.3%] | not tested (ceiling) |

**H3 (top_variance vs random, every budget):** supported. `top_variance` beat random in all 300 organizations at every budget (Cliff's delta 1.00); the four identical Holm-adjusted p-values (1.22e-50) come from scipy's normal approximation and only mean "extremely small". `top_relative` is **not** better than random: it is worse on average at every budget (Cliff's delta -0.71 to -0.77) and beat random in only 11% to 13% of organizations (`share_orgs_better` in `results/summary.json`).

### E4. Correctness

**Manipulation:** hand-derived worked examples (not published examples). **Measured:** relative reproduction error. Deterministic, so no confidence interval applies.

| Condition | n | Result: maximum relative error | 95% CI | Notes |
|---|---:|---:|---|---|
| Engine vs hand-derived worked examples (C01 to C16) | 21 comparisons | 4.69e-16 | n/a (deterministic) | 21 of 21 within tolerance (1e-9; 1e-8 for two unit-conversion cases). Includes both factor editions, location and market basis, biogenic CO2 memo, deflated spend |
| Unit-safety checks | 16 checks | 16 of 16 passed | n/a | kWh + MWh raises; kWh + gallon raises; unknown unit, wrong price year, non-USD spend, method mismatch, TEST-id in a published set, changed factor file all fail loudly; kWh/MWh inputs agree to 1e-12 on 500 random quantities |

The full list of comparisons and checks is in `results/tables.md`; the arithmetic for each case is in [worked examples](worked-examples.md).

## 8. Discussion

**What the results support.** The code does what it was designed to do: its arithmetic matches independent hand derivations (H4) and its uncertainty layer is calibrated when the world matches its assumptions (H2, condition A). Among the inputs that drive an inventory's uncertainty a few dominate: ranking inputs by their contribution to total variance recovers almost all of the reduction an adaptive greedy search finds (36.6% vs 36.6% at k = 1, 65.5% vs 65.6% at k = 5), so the ranking is cheap and nearly as good as an expensive search (H3).

**What surprised me.** (1) The naive rule "improve the inputs with the largest relative uncertainty" did worse than random at every budget. A likely reason, not tested here, is that inputs with a wide relative spread are often small lines, so shrinking their spread barely moves the total; a spreadsheet user following that rule would spend effort where it does not matter. (2) Misspecification hurts far more for some assumptions than others. A systematic 20% under-report took coverage from 89.5% to 1.0%: an interval built from random-error assumptions cannot detect a bias it does not model. Doubling the true factor errors cut coverage to 58.5% while doubling activity errors cut it only to 86.0%. A plausible reason, also not tested, is that the assumed factor spreads (5% to 50%) are wider than most data-quality-driven activity spreads (2% to 25%), so factor error carries more of the total variance. Heavy-tailed activity errors (condition D) did not change coverage at the 90% level (89.4%) or at 95% (95%). (3) Condition E (independent rather than shared factor errors) showed no difference (89.3%). That should not be read as "the correlation assumption does not matter": a factor is rarely used on several lines in a generated organization, so there was little shared error to mis-model. The experiment could not test that assumption.

**H1 and what E1 can and cannot say.** By the pre-stated rule H1 is supported (2 of 4 categories), but the verdict is a knife edge set by assumed prices. With the other prices unchanged, an assumed truck price about 10% higher (0.22 instead of 0.20 USD per short ton-mile) would put truck freight inside the 30% band and leave 1 of 4 categories, so H1 would be unsupported; an assumed air price about 6% lower (0.14 instead of 0.15) would push air outside the band. The robust, assumption-free finding is the breakeven prices. At the sourced factors the methods agree only if freight costs about 0.32 USD per short ton-mile (truck), 0.045 (rail), air travel 0.20 USD per passenger-mile, and landfilled waste costs about 587 USD per short ton. The landfill figure shows the structural mismatch most clearly: the activity factor (580 kg CO2e per short ton) is tied to the mass and composition of waste, while the spend factor is a generic waste-sector average per dollar of service (the dataset's documentation says v1.3 has no landfill-specific sector detail), so a spend-based estimate of landfilled waste will be small at any plausible disposal price. Practical guidance: do not treat a spend-based Scope 3 figure as an approximation of the activity-based one; use the breakeven price to judge how far apart they will be for your own prices, and prefer activity data where it exists.

**Practical guidance from E2 and E3.** Report ranges, but say what they cover: they describe random error under assumed distributions and say nothing about bias, boundary or completeness errors. Spend improvement effort on the few inputs the variance ranking names rather than on the inputs with the widest relative spread. E3 did not record whether the selected inputs were emission factors or activity quantities, so this report does not claim which kind dominates; the platform's inventory report lists the ranking for each inventory.

**Limits of what was shown.** Every experiment is internally consistent by construction: the truth, the errors and the engine's assumptions come from the same author. E2's calibration for condition A is close to tautological; E3 ranks and evaluates inputs inside the same model, so "targeted beats random" is nearly guaranteed and only the size of the gap is informative; E1's outcome is a function of assumed prices. None of this tests how the platform would perform on real, messy company data.

## 9. Threats to validity

- **Synthetic organizations** may not reflect real data messiness (missing months, misclassified spend, unit mistakes, double counting). Nothing here measures real-world accuracy.
- **The generator defines the ground truth.** Method-comparison results (E1) measure sensitivity to the generator's price assumptions, not real-world differences between methods. The assumed median prices were chosen by the author with the factor magnitudes in view and are not market data.
- **Uncertainty distributions are assumed.** Factor spreads and the data-quality-to-spread table are platform assumptions, not published values. E2 shows what happens when they are wrong (conditions B, C, G) but cannot say which condition real data resembles. Component-level (CO2, CH4, N2O) uncertainty is not modelled; a single relative spread applies to each factor.
- **Emission factors change between editions.** Results are tied to the pinned sets `epa-hub-2025` and `useeio-v1.3` (case C07 versus C06: for grid region RFCW the 2024 edition gives a CO2e per MWh 9.8% higher than the 2025 edition). The factors are U.S.-specific.
- **Spend factors used outside their primary intended use** (categories 4 to 7) and without landfill-specific detail for waste.
- **Same-author derivations.** The worked examples were hand-derived by the same author who wrote the engine; independent arithmetic reduces but does not remove shared misunderstandings of the factors' meaning. E4 validates the implementation, not the factors or the methodology.
- **Shared-factor correlation** is modelled but not meaningfully tested (condition E, see Discussion).
- **Contention.** The machine was at 100% CPU load throughout and shared with other builds. No timing result is reported, results do not depend on timing, and the run was seeded and resumable.
- **Researcher degrees of freedom.** Analysis choices were fixed in this protocol before the final run; deviations from v0.1 are in the change log.
- **Framework mappings** in the reports are unverified aids (the framework texts were not retrieved).

## 10. Reproducibility checklist

- [ ] Code tagged at the commit used for the reported results. *(No version-control repository exists in this build; the SHA-256 of the source files that influence results is recorded in `results/run_metadata.json` instead.)*
- [x] Environment pinned (`uv.lock`; its SHA-256 is recorded in the run metadata; no container image is used).
- [x] Random seeds and run counts recorded (master seed 20261023; per-unit seeds derived from it; counts in section 6 and `results/units/`).
- [x] Raw results published (JSON per work unit under `results/units/`, summary in `results/summary.json`); there is no analysis notebook, the analysis is `experiments/*.py`.
- [x] Hardware or service configuration described (CPU model, logical cores, OS, library versions, CPU load before and after) in `results/run_metadata.json`.
- [x] One command reproduces every table: `uv run python -m experiments.run_all` (there are no figures; `results/tables.md` is generated, and the tables in this report were copied from it).
- [x] Data sources, licences and versions listed ([factor provenance](factor-provenance.md)).

## 11. References

Verification status is stated for each: "verified" means the bibliographic details were confirmed against the publisher's or an authoritative listing during this build; the full text was not read unless stated.

1. WRI & WBCSD. *The Greenhouse Gas Protocol: A Corporate Accounting and Reporting Standard, Revised Edition.* GHG Protocol. Verified: the GHG Protocol page offers the "Revised" edition and refers to a 2004 edition; the page shows no single publication date for the revised edition (appendices dated March 2004, gases and GWP amendment February 2013).
2. WRI & WBCSD. *GHG Protocol Scope 2 Guidance* (2015). Verified: GHG Protocol page, title and year (supporting documents dated January 2015). Whether the guidance requires dual reporting was not confirmed from the page and is not claimed here.
3. WRI & WBCSD. *Corporate Value Chain (Scope 3) Accounting and Reporting Standard* (2011). Verified: GHG Protocol page (released 2011; 15 categories).
4. ISO 14064-1:2018, *Greenhouse gases, Part 1: Specification with guidance at the organization level for quantification and reporting of greenhouse gas emissions and removals.* Verified through standards-body listings (national standards bodies' catalogues); the ISO website returned an access error. The v0.1 title was missing the words "of greenhouse gas emissions and removals" and is corrected here.
5. IPCC (2006). *2006 IPCC Guidelines for National Greenhouse Gas Inventories* (five volumes). Verified: IPCC NGGIP page; Volume 1, chapter 3 is titled "Uncertainties". Chapter text not read.
6. U.S. EPA, Center for Corporate Climate Leadership. *GHG Emission Factors Hub*, 2025 edition (workbook last modified 2025-01-15) and 2024 edition (last modified 2024-06-05). Verified: downloaded and read for this build (2026-10-23); URLs and file digests in [factor provenance](factor-provenance.md).
7. Ingwersen, W. (2024). *About the Supply Chain Greenhouse Gas Emission Factors v1.3 NAICS-6 Datasets*, 2024-07-05, and the dataset *Supply Chain Greenhouse Gas Emission Factors v1.3 by NAICS-6*, U.S. EPA, DOI 10.23719/1531143. Verified: both downloaded and read for this build.
8. Wilson, E. B. (1927). Probable inference, the law of succession, and statistical inference. *Journal of the American Statistical Association* 22(158), 209-212. Verified: bibliographic details via search (DOI 10.1080/01621459.1927.10502953); full text not read.
9. Holm, S. (1979). A simple sequentially rejective multiple test procedure. *Scandinavian Journal of Statistics* 6(2), 65-70. Verified: bibliographic details via search; full text not read.
10. Cliff, N. (1993). Dominance statistics: Ordinal analyses to answer ordinal questions. *Psychological Bulletin* 114(3), 494-509. Verified: bibliographic details via secondary listings; full text not read.

## 12. Change log

| Date | Version | Change |
|---|---|---|
| (v0.1 draft) | v0.1 | Protocol drafted; no results yet. |
| 2026-10-23 | v0.2 | Protocol finalized before the final run and results recorded. **Changes from v0.1, all made before the final run:** (1) H4 reworded from "published worked examples" to "hand-derived worked examples" because no published example was retrieved. (2) Decision rules written down for H1 (median absolute difference above 30% at assumed prices in at least half of the categories), H2 (confirmatory only for condition A; Wilson interval inside [85%, 95%]) and H3 (variance ranking vs random, Holm-adjusted, all budgets). (3) Wilson intervals instead of bootstrap for proportions; bootstrap (2,000 resamples) for medians and means. (4) E3 strategies defined (random, top_relative, top_variance, greedy); the v0.1 phrase "highest-uncertainty inputs" is operationalised as the variance ranking, with the naive relative-uncertainty reading reported as an exploratory comparison. (5) Baselines 2 and 3 are not run as separate comparators (section 4.3). (6) ISO 14064-1 title corrected in the references; references marked verified or not. (7) The assumed prices in E1 were fixed by the author with the factor magnitudes already seen (disclosed above). |
| 2026-10-23 | v0.2 | **Development runs, outputs discarded and not reported:** (a) tiny smoke runs (`--smoke`, 10 to 50 units of work per experiment) of all four experiments to check the pipeline; (b) one 150-organization check of E2 condition A on organization indexes 5000 to 5149 (outside the reported indexes 0 to 999) to look for implementation bugs; it gave about 89% coverage and led to no code change; (c) tests of the runner's crash recovery on a fake experiment. The factor library files were rebuilt once after the smoke runs (a documentation note was added to the spend-factor records), and the golden regression fixtures were regenerated at the same time. |
| 2026-10-23 | v0.2 | **Final run:** all four experiments in one uninterrupted invocation (`uv run python -m experiments.run_all`), invocation id `1bf793ec-2bf0-4e09-972e-9b6ea9ab23fb`, 93 units, no resume, no crash, no rerun. No hypothesis was re-tested, tuned or re-run after seeing results. |

---

[README](../README.md) · [Client case study](case-study.md) · [Research report](research-report.md)
