# Employment statistics standards

Source: https://economic.github.io/earn_code_library/standard_employment.html
(EPI data team). Reflects the standard as of 2026-09-10.

Each of these reproduces a published SWADL series, so they double as benchmarks.
All three take `age >= 16` and adjust the weight for the number of months in the
sample.

Missing observations return missing. Handling them with `filter()` gives the
most control over which observations are dropped; `na.rm = TRUE` gives less.

## Weighting annual estimates

The denominator is the number of months actually included in the sample, which
is **not always 12**. The October 2025 government shutdown left the basic
monthly CPS with 11 months, so a flat `basicwgt / 12` undercounts 2025.

```r
adj_wgt = basicwgt / case_match(year, 2025 ~ 11, .default = 12)
```

Use this form rather than `/ 12` in any series that reaches 2025 or later, and
add a case for any future year with a collection gap — the `.default` will
otherwise absorb it silently. The same applies to `orgwgt` for ORG data; see
[missing-data.md](missing-data.md) for the full weight-adjustment standard,
including the May CPS 1/4-sample correction for 1981.

## Employment-to-population ratio (EPOP)

Reproduces https://data.epi.org/labor_force/labor_force_emp

```r
library(tidyverse)
library(epiextractr)

load_basic(2020:2025, year, age, emp, basicwgt) |>
  filter(age >= 16, !is.na(emp)) |>
  mutate(adj_wgt = basicwgt / case_match(year, 2025 ~ 11, .default = 12)) |>
  summarise(
    epop = weighted.mean(emp, w = adj_wgt),
    .by = year)
```

## Labor force participation rate

Reproduces https://data.epi.org/labor_force/labor_force_lf

In the labor force is `lfstat` of 1 (employed) or 2 (unemployed).

```r
load_basic(2020:2025, year, age, lfstat, basicwgt) |>
  filter(age >= 16) |>
  mutate(
    adj_wgt = basicwgt / case_match(year, 2025 ~ 11, .default = 12),
    lfp = if_else(lfstat == 1 | lfstat == 2, 1, 0)) |>
  summarise(
    lfpr = weighted.mean(lfp, w = adj_wgt, na.rm = TRUE),
    .by = year)
```

## Unemployment rate

Reproduces https://data.epi.org/labor_force/labor_force_unemp

The denominator is the labor force, so those not in it (`lfstat == 3`) are
excluded rather than counted as employed.

```r
load_basic(2020:2025, year, age, unemp, lfstat, basicwgt) |>
  filter(age >= 16, lfstat != 3) |>
  mutate(adj_wgt = basicwgt / case_match(year, 2025 ~ 11, .default = 12)) |>
  summarise(
    urate = weighted.mean(unemp, w = adj_wgt),
    .by = year)
```

Note that a rate is far less sensitive to the weight denominator than a count is
— numerator and denominator scale together — but weighted **counts** and any
series pooling months across years will be wrong without the adjustment.
