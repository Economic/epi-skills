# Dealing with missing data

Source: https://economic.github.io/earn_code_library/standard_missing.html
(EPI data team). Reflects the standard as of 2026-09-10.

## Adjusting weights in years with irregular samples

Some years have discrepancies in sample size and data collection, and weights
must be adjusted to get overall counts right.

1. **Union numbers before 1983 require the May CPS** — union questions were not
   asked in the ORG until 1983. May weights must be adjusted for 1981. Other
   calculations, such as wages, can use ORG data back to 1979 with unadjusted
   `orgwgt`.
2. **October 2025 needs adjustment across all surveys** due to missing data.

The script below accounts for:

- May 1981 asked only a 1/4 sample
- May 1973–1980 asked the full sample
- ORG 1983–present asked the full ORG sample
- ORG 2025 has only 11 months of data due to the 2025 government shutdown

```r
library(tidyverse)
library(epiextractr)

may_data = load_may(may_years, all_of(cps_vars)) |>
  rename(weight = finalwgt) |>
  mutate(month = NA)

output = load_org(org_years, all_of(cps_vars)) |>
  rename(weight = orgwgt) |>
  bind_rows(may_data) |>
  filter(age >= 16, weight > 0, selfemp == 0) |>
  mutate(weight = case_match(
    year,
    1973:1980 ~ weight,
    1981 ~ weight * 4,
    2025 ~ weight / 11,
    .default = weight / 12
  ))
```

The general rule the `.default` encodes: divide by the number of months actually
present, not always 12. A year with a collection gap needs its own case.

> **Check currency before extending a series.** The `case_match()` above is
> exhaustive only through the years the standard was written for. A later year
> with a collection gap needs its own case, and will otherwise silently take the
> `/12` default and undercount. Confirm with the data team when adding years.

## Rolling 12-month averages with a missing month

Use `slider::slide_index_dbl()` with `.before = months(11)`, not a fixed row
count. `slide_index` takes a span of calendar months rather than a number of
rows, so it averages over a consistent time period and adjusts the denominator
when months inside the window are missing. A fixed-row window instead reaches
farther back to find rows, silently changing the period covered.

A November 2025 rolling average built this way goes back to October 2024 and
counts the blank October 2025 as a month, averaging over 11 months of data.
**This matches the SWADL methodology.**

Step 1 — monthly weighted counts, universe, and sample size:

```r
library(tidyverse)
library(epiextractr)
library(slider)

cps_data <- load_basic(2023:2026, year, month, basicwgt, emp) |>
  mutate(weight = basicwgt)

monthly_data = cps_data |>
  mutate(universe = if_else(!is.na(emp), 1, 0)) |>
  summarize(
    # count of population
    count = sum(emp * weight, na.rm = TRUE),
    # count of universe
    universe_total = sum(universe * weight, na.rm = TRUE),
    # sample size of universe
    sample_size = sum(universe, na.rm = TRUE),
    .by = c(year, month)
  ) |>
  mutate(percent = count / universe_total)
```

Step 2 — the rolling window itself. Note `sample_size` sums while rates and
counts average.

```r
smoothed_monthly_data = monthly_data |>
  mutate(date = as.Date(paste(year, month, "01", sep = "-"))) |>
  arrange(date) |>
  mutate(
    percent = slide_index_dbl(percent, date, mean, .before = months(11), .complete = TRUE),
    count = slide_index_dbl(count, date, mean, .before = months(11), .complete = TRUE),
    sample_size = slide_index_dbl(sample_size, date, sum, .before = months(11), .complete = TRUE)
  )
```

The variable names are deliberately generic — the same pattern applies to
`unemp`, labor force status, and other rates.
