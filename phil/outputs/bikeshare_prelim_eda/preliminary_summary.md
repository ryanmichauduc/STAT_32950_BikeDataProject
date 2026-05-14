# Capital Bikeshare Preliminary EDA

## Cleaning and integrity checks
- Rows/columns: 17,379 rows x 18 raw columns; the working EDA frame adds parsed date/time and readable labels.
- Date range: 2011-01-01 00:00:00 through 2012-12-31 23:00:00.
- Missing cell values: 0.
- Duplicate timestamps: 0.
- Missing hourly timestamps in full calendar range: 165.
- Days with most absent hourly records: 2012-10-29: 23; 2011-01-27: 16; 2012-10-30: 13; 2011-01-18: 12; 2011-01-26: 8; 2011-08-28: 7; 2011-02-22: 6; 2011-08-27: 6; 2011-03-10: 2; 2011-02-11: 2; 2011-02-28: 2; 2011-01-12: 2
- `cnt = casual + registered` for all rows: True; exclude `casual` and `registered` from any model predicting `cnt` or `high_demand`.
- Median `cnt`: 142; `high_demand` counts: Low=8,708, High=8,671
- IQR high-count threshold: 642.5; 505 rows exceed it. These are meaningful peak-demand periods, not automatic data errors.
- Zero humidity rows: 22; zero windspeed rows: 2180. Treat as suspicious/sensor-coded values to check in sensitivity analysis, not immediate deletion.

## Redundancy and scaling decisions
- Correlation between normalized `temp` and `atemp`: 0.988; likely use one, combine them, or let ridge/lasso handle the collinearity.
- Correlation between `cnt` and `registered`: 0.972; between `cnt` and `casual`: 0.695; confirms leakage risk.
- For PCA/clustering/regularized models, standardize continuous predictors and one-hot/cyclically encode calendar variables. Treat `hr`, `weekday`, `mnth`, `season`, and `weathersit` as categorical/cyclical, not ordinary linear quantities.

## Strong exploratory patterns
### Top hours overall
| hr | mean_cnt | high_rate |
| --- | --- | --- |
| 17.000 | 461.452 | 0.916 |
| 18.000 | 425.511 | 0.894 |
| 8.000 | 359.011 | 0.732 |
| 16.000 | 311.984 | 0.837 |
| 19.000 | 311.523 | 0.806 |
| 13.000 | 253.661 | 0.774 |
| 12.000 | 253.316 | 0.791 |
| 15.000 | 251.233 | 0.770 |

### Top hour/day-type combinations
| hr | day_type | mean_cnt | median_cnt | high_rate | n |
| --- | --- | --- | --- | --- | --- |
| 17 | Working day | 525.291 | 539.000 | 0.962 | 499 |
| 18 | Working day | 492.227 | 504.500 | 0.952 | 498 |
| 8 | Working day | 477.006 | 463.000 | 0.964 | 496 |
| 13 | Non-working day | 372.732 | 367.000 | 0.879 | 231 |
| 12 | Non-working day | 366.260 | 367.000 | 0.883 | 231 |
| 14 | Non-working day | 364.645 | 361.000 | 0.892 | 231 |
| 15 | Non-working day | 358.814 | 361.000 | 0.874 | 231 |
| 16 | Non-working day | 352.727 | 356.000 | 0.879 | 231 |
| 19 | Working day | 348.402 | 349.500 | 0.857 | 498 |
| 17 | Non-working day | 323.550 | 322.000 | 0.818 | 231 |

### Season summary
| season_label | mean_cnt | high_rate | n |
| --- | --- | --- | --- |
| Summer | 236.016 | 0.624 | 4496 |
| Spring | 208.344 | 0.548 | 4409 |
| Fall | 198.869 | 0.533 | 4232 |
| Winter | 111.115 | 0.282 | 4242 |

### Weather summary
| weather_label | mean_cnt | high_rate | n |
| --- | --- | --- | --- |
| Clear/Mist | 204.869 | 0.536 | 11413 |
| Cloudy/Mist | 175.165 | 0.478 | 4544 |
| Light Rain/Snow | 111.579 | 0.268 | 1419 |
| Heavy Rain/Snow | 74.333 | 0.333 | 3 |

### Year summary
| year_label | mean_cnt | high_rate | n |
| --- | --- | --- | --- |
| 2012 | 234.666 | 0.587 | 8734 |
| 2011 | 143.794 | 0.410 | 8645 |

### Correlations with cnt, excluding direct components
| index | cnt |
| --- | --- |
| high_demand_flag | 0.759 |
| temp | 0.405 |
| atemp | 0.401 |
| hr | 0.394 |
| yr | 0.250 |
| mnth | 0.121 |
| windspeed | 0.093 |
| workingday | 0.030 |
| weekday | 0.027 |
| holiday | -0.031 |
| weathersit | -0.142 |
| hum | -0.323 |

### Standardized predictor PCA diagnostic
| PC | explained_variance_ratio | corr_with_cnt | top_loading_terms |
| --- | --- | --- | --- |
| 1 | 0.066 | 0.359 | temp (+0.47); atemp (+0.46); season_3 (+0.37); season_1 (-0.33); mnth_7 (+0.22); mnth_1 (-0.21); mnth_8 (+0.20); mnth_2 (-0.17) |
| 2 | 0.052 | -0.056 | workingday_0 (-0.54); workingday_1 (+0.54); weekday_0 (-0.31); weekday_6 (-0.31); holiday_1 (-0.20); holiday_0 (+0.20); weekday_3 (+0.16); weekday_2 (+0.15) |
| 3 | 0.041 | -0.115 | season_4 (+0.47); hum (+0.36); season_1 (-0.30); mnth_10 (+0.28); weathersit_1 (-0.26); mnth_11 (+0.24); windspeed (-0.21); weathersit_2 (+0.21) |
| 4 | 0.038 | 0.044 | season_2 (-0.58); mnth_5 (-0.36); mnth_4 (-0.33); season_3 (+0.31); season_1 (+0.22); mnth_7 (+0.20); weathersit_1 (+0.20); mnth_8 (+0.18) |
| 5 | 0.035 | 0.100 | holiday_1 (+0.58); holiday_0 (-0.58); weekday_1 (+0.36); weekday_6 (-0.21); weekday_0 (-0.20); yr_1 (+0.11); yr_0 (-0.11); workingday_1 (+0.10) |

## Candidate research directions
- Cluster hourly records into demand regimes using standardized weather/calendar predictors, then compare clusters against `cnt` and `high_demand`.
- Predict `high_demand` from weather/calendar variables using logistic regression/LDA/lasso/tree, and compare performance to unsupervised cluster/PCA structure.
- Split analysis by working vs non-working days because the hourly shape changes sharply; this looks like the main interaction in the data.
