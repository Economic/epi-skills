---
name: epi-standards
description: EPI's required conventions for computing labor market statistics from CPS microdata. Use BEFORE writing code that computes EPOPs, labor force participation, unemployment rates, median or mean wages, wage regressions, wage premiums, union density or union coverage, or that inflation-adjusts a dollar figure. Also covers CPS weight adjustments for annual estimates and for years with missing months.
---

# epi-standards

EPI data team standards for labor market statistics. These are conventions, not
suggestions: an estimate that ignores them is usually wrong in a way that looks
plausible. Apply them by default and say so when you deviate.

## Non-negotiables

| Situation | Requirement |
|---|---|
| Wage analysis universe | `age >= 16, emp == 1, cow1 <= 5` |
| Annual estimate from monthly CPS | Divide the weight by the number of months present — `/12`, but `/11` for 2025 |
| Inflation adjustment | Chained CPI `c_cpi_u`; `c_cpi_u_extended` for pre-2000 |
| Median wages | `epidatatools::averaged_median()`, not `binipolate` |
| Union vs. non-union wage comparison | Drop BLS-imputed wages (see wages reference) |
| Race/ethnicity breakouts | The `wbhao` standard — see the `code-race-ethnicity` skill |

Two traps worth naming up front, because both fail silently:

- **Missing data returns missing.** Prefer explicit `filter(!is.na(x))` over
  `na.rm = TRUE`; filtering gives control over which observations are dropped.
- **`filter(a_earnhour != 1)` also drops every non-hourly worker,** because
  `filter()` drops `NA`. Never write that. Use the pattern in the wages reference.

## References

Read the relevant file before writing code — each holds the standard's own code,
which is more specific than the table above.

- [references/wages.md](references/wages.md) — universe, inflation adjustment,
  median wages, EPI's standard wage regression, imputed-wage handling
- [references/employment.md](references/employment.md) — EPOPs, labor force
  participation, unemployment rate
- [references/missing-data.md](references/missing-data.md) — weight adjustments
  for May CPS and short years, rolling 12-month averages with missing months
- [references/unions-and-policy.md](references/unions-and-policy.md) — union
  density vs. coverage, right-to-work status

## Validating against the standard

Encode the universe as an assertion rather than trusting the filter, and
benchmark against SWADL where a comparable series exists (see the
`validate-analysis` skill). The standards' employment, union, and wage examples
are written to reproduce published SWADL series, so a mismatch is a real signal.

```r
library(assertr)

cps_org |>
  verify(age >= 16) |>
  verify(emp == 1) |>
  verify(cow1 <= 5)
```
