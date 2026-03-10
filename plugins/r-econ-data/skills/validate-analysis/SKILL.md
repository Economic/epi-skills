---
name: validate-analysis
description: Common ways to validate or benchmark data analysis estimates using external sources. Use to confirm or test the plausibility of results from an economic analysis.
---

For example, say you wanted to calculate the real median hourly wage for a demographic group not available in SWADL, like foreign-born workers in California. To verify your basic approach, you might first try to replicate something in SWADL approximating the target group, like the real median wages of all workers in California:

```
library(tidyverse)
library(epidatatools)
library(epiextractr)
library(realtalk)
library(swadlr)
library(assertr)

swadl_ca_median = get_swadl(
  "hourly_wage_median",
  measure = "real_wage_median_2025",
  geography = "CA"
) |>
  mutate(year = year(date)) |>
  select(year, swadl_real_median = value)

cpi_data = c_cpi_u_annual

cpi_base = cpi_data |>
  filter(year == 2025) |>
  pull(c_cpi_u)

org_median_wage = load_org(
  2010:2025,
  year,
  statefips,
  wage,
  orgwgt,
  citistat
) |>
  filter(wage > 0, statefips == 6) |>
  summarize(
    org_nominal_median = averaged_median(wage, w = orgwgt),
    .by = year
  ) |>
  left_join(cpi_data, by = "year") |>
  mutate(org_real_median = org_nominal_median * cpi_base / c_cpi_u)

org_median_wage |>
  left_join(swadl_ca_median, by = "year") |>
  verify(swadl_real_median - org_real_median < 1e-8)

```