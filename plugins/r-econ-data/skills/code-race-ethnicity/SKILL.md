---
name: code-race-ethnicity
description: EPI's standard wbhao race and ethnicity coding (White, Black, Hispanic, AAPI, Other) for CPS, ASEC, and ACS/IPUMS microdata. Use whenever grouping, breaking out, or filtering results by race or ethnicity, or when raw race/hispan variables need collapsing into EPI categories.
---

# code-race-ethnicity

EPI collapses race and ethnicity into a five-category `wbhao` variable:

| Value | Label |
|---|---|
| 1 | White |
| 2 | Black |
| 3 | Hispanic |
| 4 | AAPI |
| 5 | Other |

**Hispanic identification takes priority over any race response.** In every
recode below, the `hispan` condition comes first in the `case_when()`, so
Hispanic respondents are coded 3 regardless of race. Reordering these clauses
silently changes the estimates.

## Check for a built-in variable first

The EPI CPS extracts already ship recoded race/ethnicity variables — do not
hand-roll one for `load_basic()` or `load_org()` data. Available variables
include `wbhao` (incl. Asian), `wbho`, `wbhaom` and `wbhom` (incl. multiple
race), `wbho_only` and `wbo_only` (single-race), plus the more detailed
`raceorig` and `hispanic`. See https://microdata.epi.org.

Note that EPI's standard wage regression uses `wbho`, not `wbhao` — see the
`epi-standards` skill.

The recodes below are for **ASEC and ACS/IPUMS**, where no such variable exists.

## ASEC

Uses `hispan` and `race`.

```r
mutate(
  wbhao = case_when(
    hispan >= 100 & hispan <= 612 ~ 3, # Hispanic or Latino
    race == 100 ~ 1, # White
    race %in% c(200, 801, 805, 806, 807, 810, 811, 814, 816, 818) ~ 2, # Black
    race %in% c(650, 651, 652, 803, 804, 808, 809, 812, 813, 817, 819) ~ 4, # AAPI
    TRUE ~ 5
  ),
  wbhao = haven::labelled(wbhao, c(
    "White" = 1,
    "Black" = 2,
    "Hispanic" = 3,
    "AAPI" = 4,
    "Other" = 5
  ))
)
```

## ACS (via IPUMS)

Two approaches. Prefer the detailed `raced` version when multiple-race detail
matters; the `rac*` indicator version is simpler but coarser.

### Using `race` and `raced`

`raced` downloads automatically when `race` is included in the sample.

```r
mutate(
  wbhao = case_when(
    hispan %in% c(1, 2, 3, 4) ~ 3,
    race == 1 ~ 1,
    race == 2 ~ 2,
    raced >= 830 & raced <= 845 ~ 2,
    raced >= 901 & raced <= 904 ~ 2,
    raced >= 930 & raced <= 936 ~ 2,
    raced >= 950 & raced <= 955 ~ 2,
    raced >= 970 & raced <= 973 ~ 2,
    raced >= 980 & raced <= 983 ~ 2,
    raced %in% c(917, 985, 986, 990, 991) ~ 2,
    race %in% c(4, 5, 6) ~ 4,
    raced >= 810 & raced <= 825 ~ 4,
    raced >= 850 & raced <= 855 ~ 4,
    raced >= 860 & raced <= 899 ~ 4,
    raced >= 910 & raced <= 915 ~ 4,
    raced >= 920 & raced <= 927 ~ 4,
    raced >= 940 & raced <= 944 ~ 4,
    raced >= 960 & raced <= 964 ~ 4,
    raced %in% c(905, 974, 975, 976, 984) ~ 4,
    TRUE ~ 5
  ),
  wbhao = labelled(wbhao, c(
    "White" = 1,
    "Black" = 2,
    "Hispanic" = 3,
    "AAPI" = 4,
    "Other" = 5
  ))
)
```

### Using `race` and the `rac*` indicator series

`racblk`, `racasian`, `racpacis` (and `racamind`) are indicators where a value
of 2 means membership in that category.

```r
mutate(
  wbhao = case_when(
    hispan %in% c(1, 2, 3, 4) ~ 3,
    race == 1 ~ 1,
    racblk == 2 ~ 2,
    racasian == 2 ~ 4,
    racpacis == 2 ~ 4,
    TRUE ~ 5
  ),
  wbhao = labelled(wbhao, c(
    "White" = 1,
    "Black" = 2,
    "Hispanic" = 3,
    "AAPI" = 4,
    "Other" = 5
  ))
)
```

## Validating a recode

Category 5 ("Other") is the catch-all, so a miscoded range lands there
silently rather than erroring. Crosstab the result against the source variables
and check that the Other share is plausible before using the recode.

```r
library(epidatatools)

crosstab(data, race, wbhao)
crosstab(data, hispan, wbhao)
```

Source: https://economic.github.io/earn_code_library/standard_wbhao.html
(EPI data team). Reflects the standard as of 2026-09-10.
