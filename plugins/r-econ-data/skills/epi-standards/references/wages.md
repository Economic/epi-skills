# Wage analysis standards

Source: https://economic.github.io/earn_code_library/standard_wages.html
(EPI data team). Reflects the standard as of 2026-09-10.

## Defining the universe

For the standard definition — no special exclusions or inclusions — filter to
`age >= 16`, `emp == 1`, and `cow1 <= 5`. This is non-self-employed,
non-self-incorporated, employed workers at or over 16 years old.

```r
library(tidyverse)
library(epiextractr)

cps_org <- load_org(2020:2024, "year", "age", "statefips", "wage", "emp",
                    "union", "orgwgt", "cow1") |>
  filter(age >= 16, emp == 1, cow1 <= 5)
```

- Self-employed and self-incorporated workers do not have wages in the CPS ORG,
  so the `cow1` filter is somewhat duplicative for ORG wage data. It is
  necessary for Basic data.
- This is not always the target universe (e.g. EPOPs). Use discretion when
  defining the sample, and state the universe in the output.

## Inflation adjusting

Use **chained** CPI `c_cpi_u`. For analysis extending before 2000, use
**chained extended** `c_cpi_u_extended`. Both are in the `realtalk` package.

## Median wages

Use `averaged_median()` from `epidatatools`, not `binipolate`. `quantiles_n` and
`quantiles_w` take the defaults shown below, so they are not strictly necessary
in the code, though including them documents the process.

```r
wages_gender <- cps_org |>
  summarise(
    wage_median = averaged_median(
      x = realwage,
      w = orgwgt / 12,
      quantiles_n = 9L,
      quantiles_w = c(1:4, 5, 4:1)),
    n = n(),
    .by = c(female, year)
  )
```

The `orgwgt / 12` here assumes a full 12 months. For any series reaching 2025 or
later, use `orgwgt / case_match(year, 2025 ~ 11, .default = 12)` — the October
2025 shutdown left 11 months of data. See
[missing-data.md](missing-data.md).

## Wage premiums and regressions

EPI's standard wage regression uses the log of real wages as the dependent
variable, with race (`wbho`), gender (`female`), education (`educ`), age, and
age squared as regressors, plus fixed effects for marital status (`married`) and
state (`statefips`). **Multi-year data requires year fixed effects as well.**
With year fixed effects, log nominal and log real wages give identical results.

```r
library(fixest)

wage_reg <- cps_org |>
  mutate(
    log_realwage = log(realwage),
    age_2 = age^2
  ) |>
  feols(
    log_realwage ~
      i(female, ref = "0") +
      age + age_2 |
      educ + wbho +
      married + statefips,
    weights = ~orgwgt
  )
```

**Coefficients are in logs and must be converted.** A one-unit change in a
regressor is associated with a `(exp(β) - 1) * 100%` change in wages, holding
other factors constant. Example: in 2025 the coefficient on `female` in the
conventional EPI wage regression is −0.203424, implying women's wages are about
`(exp(-0.203424) - 1) * 100 ≈ -18.4%` different from men's, after controlling
for race, education, age, marital status, and state.

Worked examples:
- https://github.com/Economic/equal_pay_day
- https://github.com/Economic/swa_data_library/blob/main/R/wage_reg.R

## Imputed wage filtering

Sometimes you want to remove wages imputed (allocated) by the BLS. This is
**especially important for union vs. non-union wage comparisons**, because the
BLS does not account for union status when imputing wages — leaving imputations
in compresses the measured union premium.

Do not write `filter(a_earnhour != 1)`. Because of how `a_earnhour` and
`a_weekpay` are coded and because `filter()` drops `NA` by default, that
expression also removes every non-hourly worker.

To flag imputation while preserving salaried workers with self-reported wages:

```r
mutate(
  wage_imputed = case_when(
    paidhre == 1 & a_earnhour == 1 ~ 1,
    paidhre == 0 & a_weekpay == 1 ~ 1,
    .default = 0
  )
)
```

Then `filter(wage_imputed == 0)` when you want them gone.

To filter them out from the start:

```r
filter_out(a_earnhour == 1 & paidhre == 1 | a_weekpay == 1 & paidhre == 0)
```

Both should produce the same non-imputed sample. Verify with
`crosstab(data, paidhre, wage_imputed)` against
`crosstab(data, paidhre, a_earnhour)`.
