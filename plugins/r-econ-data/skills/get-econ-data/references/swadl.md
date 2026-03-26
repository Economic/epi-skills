# Data from EPI State of Working America Data Library (SWADL)

## Background on SWADL

SWADL contains time series of important US labor market data, particularly data on wages and the labor force, often with detailed demographic or geographic cuts. EPI compiles and calculates these data from public sources.

The time series in SWADL are very convenient because they are often calculated from public microdata, using defensible conventions that take pains to make series comparable over time. 

## Access methods

User-facing website at https://data.epi.org.

API access with R package `swadlr`. 

If asked to make dozens of calls to the API, do not do this in order to avoid overburdening the API. Instead direct the user to download a .csv file of all the data in bulk first: https://economic.github.io/data/.

To install `swadlr`, use `install.packages("swadlr", repos = c("https://economic.r-universe.dev", getOption("repos")))`

## General guidance using `swadlr`

SWADL is organized by *indicators*, that group different data series, like "CEO pay ratio" or "Employment by demographics". Each indicator has an *id* and a human-readable *name*. 

Each indicator is associated with a set of *measures*, *dimensions*, *dates*, and *geographies* for which the indicator is available.

*Measures* can be thought of as how the indicator may be measured in the database. Measures also have an "id" and a human-readable "name" like "Nominal minimum wage" or "Number of union members". 

*Dimensions* are different "cuts" of the data like "education" or "race". These have an id and human-readable name and are associated with specific dimension values, which have their own value ids and names. For example, as of this writing the dimension id "age group" has dimension name "Age", and it is associated with dimension value ids "age_16_24" and "age_25_54" and "age_55_plus", which each have their own human readable names.

Most indicators in SWADL are available at the national (US)-level. But some are available for specific regions or states.

Available time frequencies depend on the indicator, but SWADL contains monthly, quarterly, and annual data.

## Recommended workflow for using `swadlr`

### Determining what is available

Start with `swadl_availability()` for a quick overview across all indicators, then use `swadl_indicator()` for detailed metadata on a specific indicator.

`swadl_availability()` will provide a tibble containing every combination of indicators, measures, dimensions, and geographies, as well as date ranges. Columns returned are `indicator_id`, `indicator_name`, `date_interval`, `measure_id`, `geo_level`, `dimensions`, `date_start`, `date_end`.

A more complete list for a given indicator id can be retrieved with `swadl_indicator(indicator_id)`. This returns a detailed nested list and a pretty print method summary, and it also provides information on sources and when the indicator was last updated.

`swadl_id_names()` also provides quick lookups of ids and human-readable names for indicators, measures, dimensions, and geographies. For example,

```
swadl_id_names("indicators")
swadl_id_names("measures")
swadl_id_names("dimensions")
swadl_id_names("geographies")
```

### Retrieving the data

Once you have an indicator id and a desired measure, dimension, and geography, you can retrieve the actual time series of data using

```
get_swadl(
  indicator,
  measure,
  date_interval = c("year", "month", "quarter"),
  geography = "national",
  dimension = "overall",
  date = NULL
)
```

which returns a tibble with columns `date`, `value`, `geography`, and columns named after the requested dimension. Default `date_interval` is "year" (requesting annual data), the default dimension id is "overall", and the default geography is "national".

For example, this is the US overall employment-to-population ratio each month as a 12-month average:

```
get_swadl(
  indicator = "labor_force_emp", 
  measure = "percent_emp_12_month", 
  date_interval = "month"
)
```

That is a single time series because the implicit "overall" dimension requested has one dimension value. You can retrieve repeated cross sections by specifying a dimension that has multiple dimension values:

```
get_swadl(
  indicator = "labor_force_emp", 
  measure = "percent_emp_12_month", 
  date_interval = "month",
  dimension = "age_group"
)
```

To retrieve a single time series within a given dimension, specify a dimension value within a dimension:

```
get_swadl(
  indicator = "labor_force_emp", 
  measure = "percent_emp_12_month", 
  date_interval = "month",
  dimension = list("age_group" = "age_25_54")
)
```

Allowed dimension specifications are

- "overall": Returns aggregate data without demographic breakdown
- A dimension ID (e.g., "wage_percentile"): Returns all values for that dimension
- A list with named and/or unnamed elements:
  - Named elements filter to specific values: list("wage_percentile" = "wage_p50")
  - Unnamed elements return all values: list("wage_percentile")
  - Multiple dimensions can be cross-tabulated, but due to API constraints only one dimension can return all values; the others must specify values: list("gender" = "gender_male", "age_group")

To retrieve specific geographies, specify the `geography` argument, which accepts a single state name (e.g., "California"), abbreviation (e.g., "CA"), region name (e.g., "Midwest"), division name (e.g., "Pacific"), or API ID (e.g., "state06"). It defaults to "national". You cannot request multiple geographies at once.

In the data returned by `get_swadl()`, `date` is a standard date column in R, regardless of whether the series is monthly, quarterly, or annual. The convention is that monthly date value January 1978, quarterly value 1978q1, and annual value 1978 will all be coded as the YMD date "1978-01-01". 

When requesting data with `get_swadl()` allowed values for the `date` argument are

- NULL (default): All available dates
- A single date (YYYY-MM-DD character or Date): Returns only that date
- A vector of two dates (YYYY-MM-DD characters or Dates): Returns dates in that range (inclusive)

Here is an example with state and date filtering:

```
get_swadl(
  indicator = "hourly_wage_percentiles",
  measure = "nominal_wage",
  geography = "CA",
  dimension = list("wage_percentile" = "wage_p50"),
  date = c("2000-01-01", "2023-01-01")
)
```

## Additional documentation for `swadlr`

The R package documentation and vignettes provide additional examples.
