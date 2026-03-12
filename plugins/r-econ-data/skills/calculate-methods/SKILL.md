---
name: calculate-methods
description: Methods for calculating statistics and estimates using economic data. Use when user asks to calculate or estimate quantities like percentiles, means, medians, shares, counts, or regression coefficients or regression-adjusted statistics.
---

## Moving averages and sliding windows

Often data over time is very noisy and moving averages can better extract signals. Use the R package `slider` for calculations over sliding windows like moving averages.

## Weighted means, medians, percentiles

Use `weighted.mean(vector, w = weight)` for weighted means.

Use the `MetricsWeighted` package for weighted medians and percentiles: `weighted_median(vector, w = weight)` and `weighted_quantile(vector, w = weight, probs = c(0.25, 0.50, 0.75))`.

## Percentiles with "bunched" data

When using data that is assumed to be continuous but in practice is "bunched" at discrete points -- like reported hourly wages -- it can be helpful to smooth the estimated percentiles. One technique is to take a weighted average of nearby percentiles. For example, consider using the `epidatatools` package `averaged_median(vector, w = weight)` and `averaged_quantile(vector, w = weight, probs = c(0.25, 0.50, 0.75))`. 