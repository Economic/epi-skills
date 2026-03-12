---
name: get-econ-data
description: Provides useful economic data sources and ways to retrieve the data. Use when user asks for "prices", "inflation", "employment", "unemployment", "inequality", "productivity", "poverty", "CPS", "Census", "BLS", "BEA", "FRED", "IPUMS", "SWADL", "State of Working America Data Library".
---

# economics-data

Useful economic data sources and ways to retrieve the data.

## Aggregated time series and panels

### Bureau of Economic Analysis (BEA)

Outcomes: NIPA, GDP, PCE, consumption, investment, spending, trade

Geography: national, state, local areas

Access: API access with R package `epidatatools`. User-facing website at https://bea.gov.

### Economic Policy Institute (EPI) State of Working America Data Library (SWADL)

Outcomes: wages, wage percentiles, pay versus productivity, wage disparities, prices, minimum wages, population shares, unions, unemployment, employment.

Geography: national, state

Data sources: ACS, BLS, CPS, BLS, BEA, EPI, NIPA, SSA

Access: API access with R package `swadlr`. User-facing website at https://data.epi.org.

More information: [references/swadl.md](references/swadl.md).

### Bureau of Labor Statistics (BLS)

Outcomes: Employment, unemployment, labor market participation, wages, productivity

Geography: national, state, local areas

Data sources: BLS, CPI, CPS, LAUS, OEWS

Access: API access with `get_bls()` and `find_bls()` from the R package `epidatatools`. User-facing website at https://bls.gov.

More information: [references/bls.md](references/bls.md).

## Prices and inflation
Common CPI- and PCE-based indexes are available in the R package `realtalk`.

More information: [references/inflation.md](references/inflation.md).

## Other important US economic data at the national or regional level

The BLS, FRED, and BEA provide very extensive time series data on the US economy. Most of these data are available via public APIs, and the R package `epidatatools` has functions to help you find and retrieve the data:

- BLS: `get_bls`, `find_bls`
- BEA: `get_bea_nipa`, `get_bea_regional`
- FRED: `get_fred`, `find_fred`

The Federal Reserve Bank of St. Louis maintains FRED: an enormous 

Use `epidatatools` or `fredr` R packages to access FRED: https://fred.stlouisfed.org/.

Use the `tidycensus` R package to access the Census API.

## Microdata
One can often use the aggregated data from the sources above, for say an analysis of how Black unemployment rates vary over time nationally or by state. But sometimes aggregated information is unavailable or inadequate. In that case, consider individual-level data, sometimes called microdata.

### EPI CPS microdata extracts
This should be your first stop for analysis using Current Population Survey microdata, especially using data on hourly wages. Try local copies of the EPI CPS extracts, using the R package `epiextractr` and its functions `load_basic` and `load_org`. 

The EPI CPS variables are documented at https://microdata.epi.org.

More information: [references/epiextracts.md](references/epiextracts.md).

### IPUMS microdata extracts

The IPUMS microdata extracts are a useful secondary source, and contain more variables and data sources than the EPI CPS extracts, but be aware that IPUMS contains fewer consistent codes across time. You can download them via their microdata API with the associated functions in the `epidatatools` package or the `ipumsr` package.

More information: [references/ipums.md](references/ipums.md).
