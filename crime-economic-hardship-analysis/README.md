# Does Economic Hardship Drive Crime?

### A Hypothesis Testing Analysis of U.S. State Crime and Poverty Data

**By a data science student, about 6 weeks in**

This is my second statistics project. The first one tested home advantage in football using descriptive stats and basic hypothesis testing. This one goes deeper: multiple hypothesis tests, a real methodological decision framework, and a finding that surprised me.

All analysis was done in Excel. Two government datasets, manually merged, manually cleaned, every formula written and understood before entering.

---

## The Question

Does poverty actually drive crime, or is that just something people say?

I wanted to test it properly. Not with a chart and a vague conclusion, but with formal hypothesis tests that either confirm or reject the claim with a stated probability.

---

## Datasets

**Crime Data:** [CORGIS State Crime Dataset](https://corgis-edu.github.io/corgis/csv/state_crime/)
Source: FBI Unified Crime Reporting Statistics, U.S. Department of Justice
Coverage: All 50 U.S. states + D.C., 1960–2019

**Demographics Data:** [CORGIS State Demographics Dataset](https://corgis-edu.github.io/corgis/csv/state_demographics/)
Source: U.S. Census Bureau
Coverage: All 50 U.S. states + D.C., snapshot circa 2015–2019

I used violent crime rates (per 100,000 people), poverty levels (% below poverty line), and per capita income.

### Data Cleaning

The crime dataset had 3,116 rows on download. 60 of them were "United States" aggregate rows: national totals, not state observations. Left in, they'd contaminate every state-level calculation. Removed them, leaving 3,056 rows.

The two datasets had different structures: crime data was time series (one row per state per year), demographics was a single snapshot per state. Merging them required AVERAGEIFS to compute each state's average crime rate over 2010–2019, then VLOOKUP to attach poverty and income figures. After merging, I spot-checked California manually against the raw demographics sheet. Values matched.

One thing worth noting: I initially considered extracting data by row position, assuming the file was sorted by year. It wasn't. It was sorted alphabetically by state. Catching that before writing formulas saved a lot of broken results.

---

## What I Applied

This project was built specifically around hypothesis testing with dependent and independent samples. Every test I cover here was chosen deliberately, not randomly applied.

The full decision framework:

1. Run an F-test to check variance equality
2. If variances are equal → pooled t-test
3. If variances are unequal → Welch's t-test
4. For same-subjects measured twice → paired t-test

I didn't assume equal variance anywhere. I checked first.

---

## Key Findings

### 1. Descriptive Overview

| Statistic | Violent Crime Rate | Property Crime Rate |
|---|---|---|
| Mean | 378.00 | 2580.23 |
| Median | 352.81 | 2624.99 |
| Std Dev | 177.06 | 626.66 |
| Skewness | 2.09 | 0.69 |

Violent crime is positively skewed at 2.09. A few states pull the distribution hard to the right. The range across states is 1,061 per 100,000; the most dangerous state has roughly 10× the violent crime rate of the safest. That's not noise. That's structural variation worth testing.

Mode wasn't calculated for crime rates or poverty levels. These are continuous variables; the chance of two states sharing identical values to multiple decimal places is negligible. Reporting N/A is more honest than reporting a meaningless figure.

Washington D.C. is a visible outlier in the scatter plot at ~1,180 violent crimes per 100,000. It's not a data error. D.C. is an entirely urban jurisdiction with no rural areas to moderate its crime rate, unlike every actual state in the dataset. I left it in but flagged it.

| Statistic | Poverty Level (%) | Per Capita Income ($) |
|---|---|---|
| Mean | 12.17 | 33,743 |
| Median | 11.80 | 32,176 |
| Std Dev | 2.68 | 5,689 |
| Range | 12.30 | 31,778 |

Poverty ranges from 7.3% to 19.6% across states. The scatter plot of poverty vs violent crime shows a clear upward trend left to right; states on the high-poverty end cluster noticeably higher in violent crime. Maine sits at rank 1 for lowest violent crime despite being in the middle of the poverty distribution, which I'll come back to.

---

### 2. F-Test: Equality of Variances

Before running any t-test comparing high vs low poverty states, I tested whether their variances were equal.

| Metric | Value |
|---|---|
| Variance: High Poverty Group | 35,529.82 |
| Variance: Low Poverty Group | 16,499.28 |
| F Statistic | 2.15 |
| P-Value (two-tailed) | 0.031 |
| Decision | Reject H₀: use Welch's t-test |

The high poverty group's variance is more than double the low poverty group's. P = 0.031 < 0.05. Equal variance assumption rejected. This ruled out the pooled t-test for this comparison; using it anyway would have produced a wrong standard error and a wrong p-value.

---

### 3. Welch's T-Test: High vs Low Poverty States

H₀: No difference in violent crime between high and low poverty states (μ₁ = μ₂)
H₁: High poverty states have significantly higher violent crime (μ₁ > μ₂)
α = 0.05, one-tailed

States were split at the median poverty level (11.80%): 25 high poverty states, 26 low poverty states.

| Metric | Value |
|---|---|
| Mean: High Poverty Group | 456.51 |
| Mean: Low Poverty Group | 302.51 |
| T Statistic | 3.40 |
| Degrees of Freedom (Welch) | 42.15 |
| Critical Value | 1.68 |
| P-Value | 0.00075 |
| Decision | Reject H₀ |

High poverty states average 154 more violent crimes per 100,000 than low poverty states. Across a state of 1 million people, that's roughly 1,540 additional violent crimes per year. The t-statistic of 3.40 is more than double the critical value.

The Welch-Satterthwaite equation produced a non-integer df of 42.15. This is expected and correct. Welch's adjusts degrees of freedom to account for the variance imbalance between groups; the pooled t-test would have used a flat df of 49.

---

### 4. Pooled T-Test: Indiana vs Ohio

To demonstrate the pooled t-test correctly, I needed two states where equal variance was actually defensible.

I checked variance across several neighbouring states for the 2010–2019 window:

| State | Variance |
|---|---|
| Indiana | 12,580 |
| Ohio | 13,807 |
| Kentucky | 8,266 |
| Tennessee | 46,312 |
| Illinois | 41,276 |

Indiana and Ohio had the closest variances, less than 10% apart, versus Tennessee and Illinois at 3-4× higher. Similar geography, similar economic profiles, genuinely comparable. The pair was chosen on contextual grounds first, then confirmed by the variance check.

H₀: No significant difference in violent crime between Indiana and Ohio (μ₁ = μ₂)
H₁: Indiana has significantly higher violent crime than Ohio (μ₁ > μ₂)
α = 0.05, one-tailed

| Metric | Value |
|---|---|
| Mean: Indiana | 366.78 |
| Mean: Ohio | 295.35 |
| Pooled Variance | 432.15 |
| T Statistic | 7.68 |
| Degrees of Freedom | 18 |
| P-Value | ≈ 0 |
| Decision | Reject H₀ |

Indiana averaged 71 more violent crimes per 100,000 than Ohio across 2010–2019. The difference is significant.

One honest limitation: the within-period variance for Indiana (756.78) turned out to be ~7× Ohio's (107.52) for 2010–2019 specifically, even though their full-period variances were close. This suggests Welch's may have been more appropriate even here. I'm documenting it rather than hiding it.

---

### 5. Paired T-Test: Before vs After the 2008 Recession

This was the most interesting test. Same 51 states, measured twice: average violent crime 2004–2007 (pre-recession) vs 2009–2012 (post-recession).

H₀: No significant change in violent crime before vs after the recession (μd = 0)
H₁: Violent crime changed significantly after the recession (μd ≠ 0)
α = 0.05, two-tailed

Two-tailed because economic theory genuinely predicts both directions. Hardship could increase crime through desperation; reduced economic activity could decrease opportunity-based property crime. I didn't assume a direction.

| Metric | Value |
|---|---|
| Mean Difference (Post − Pre) | -46.00 |
| Std Dev of Differences | 53.24 |
| Standard Error | 7.46 |
| T Statistic | -6.17 |
| Degrees of Freedom | 50 |
| Critical Value (two-tailed) | 1.68 |
| P-Value | ≈ 0 |
| Decision | Reject H₀ |

42 out of 51 states saw violent crime decrease after the recession. The mean drop was 46 crimes per 100,000. That's statistically significant and in the opposite direction from what most people would predict.

The recession didn't drive crime up. It coincided with a significant, widespread decline. This is consistent with the broader long-term crime drop in the U.S. that had been running since the early 1990s. The recession hit during a trend already in motion. Short-term economic shocks and long-term structural poverty appear to operate through different mechanisms.

---

## Limitations

A few things I'm not satisfied with:

The pooled t-test state selection (Indiana vs Ohio) revealed a within-period variance mismatch that I only caught after running the test. Rechecking variance assumptions on the specific time window, not just the full dataset, should happen before selecting the test.

The demographics dataset is a single snapshot, not time series. Poverty levels in 2015–2019 were matched to crime averages from 2010–2019. This introduces a temporal mismatch; poverty in 2010 and poverty in 2019 aren't the same figure, and I'm treating them as if they are.

Maine ranks 1st for lowest violent crime but sits at median poverty. Utah has the 2nd highest poverty rank but moderate crime. These anomalies (and there are probably more) suggest poverty explains part of the crime variation across states, not all of it. Urbanisation, policing policy, demographics, and social infrastructure all matter.

---

## Tools Used

- Microsoft Excel
- Crime data: CORGIS / FBI UCR Statistics
- Demographics data: CORGIS / U.S. Census Bureau

---

*Second project in a self-taught data science track. Feedback welcome.*
