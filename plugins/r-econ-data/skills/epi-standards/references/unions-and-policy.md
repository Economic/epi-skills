# Union and policy-area standards

Source: https://economic.github.io/earn_code_library/standard_misc.html
(EPI data team). Reflects the standard as of 2026-09-10.

## Union density vs. union coverage

The universe for both is paid employed workers 16 and older, not self-employed
or self-incorporated: `age >= 16, cow1 < 6, emp == 1`.

The two are different variables on the same universe, and the distinction
matters — coverage exceeds membership:

- **`union`** — represented by a union (coverage)
- **`unmem`** — a union member (density)

Share represented by a union, reproducing
https://data.epi.org/unions/union_members (percent_union_covered):

```r
library(tidyverse)
library(epiextractr)

load_org(2020:2024, year, age, cow1, emp, union, orgwgt) |>
  filter(age >= 16, cow1 < 6, emp == 1) |>
  summarize(union_coverage = weighted.mean(union, w = orgwgt / 12), .by = year)
```

Share in a union, reproducing the same indicator's percent_union_members:

```r
load_org(2020:2024, year, age, cow1, emp, unmem, orgwgt) |>
  filter(age >= 16, cow1 < 6, emp == 1) |>
  summarize(union_coverage = weighted.mean(unmem, w = orgwgt / 12), .by = year)
```

Both examples use `orgwgt / 12`, which assumes a full 12 months. For series
reaching 2025 or later use
`orgwgt / case_match(year, 2025 ~ 11, .default = 12)`; see
[missing-data.md](missing-data.md).

For union **wage** comparisons, also drop BLS-imputed wages — see
[wages.md](wages.md). The BLS does not account for union status when imputing.

Union questions were not asked in the ORG until 1983; earlier years require the
May CPS with adjusted weights. See [missing-data.md](missing-data.md).

## Right to work

Do not hand-code RTW status by state and year. Use the repo that assigns it to
CPS microdata: https://github.com/Economic/right_to_work

- Usage example: https://economic.github.io/earn_code_library/rtw_definitions.html
- Regression repo for Gould and Cohn (2026): https://github.com/Economic/rtw_regression
