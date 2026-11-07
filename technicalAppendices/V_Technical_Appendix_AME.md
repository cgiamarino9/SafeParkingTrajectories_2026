# Technical Appendix V: Binomial Logit Average Marginal Effects (AME)

**Analysis**: Overall Average Marginal Effects Expressed in Percentage Points Change (Δpp) | **Clustering:** `ProjectID`

| Cohort            | Variable / Predictor                  | AME (percentage points / Δpp) | [95% CI]          | SE       | p-value   | Significance |
|:------------------|:--------------------------------------|-----------------------------:|:------------------|---------:|:----------|:-------------|
| First-Time Cohort | Safe Parking Program (SPP Baseline)   | -12.73                        | [-18.86, -6.60]   | 3.12819  | < 0.001   | **           |
| First-Time Cohort | Enrolled Post-2022 Era                | -7.76                         | [-9.97, -5.56]    | 1.1252   | < 0.001   | **           |
| First-Time Cohort | SPP × Post-2022 Era (DiD Estimator)   | +10.14                        | [+2.37, +17.91]   | 3.96301  | 0.0105    | *            |
| First-Time Cohort | Woman                                 | +3.45                         | [+1.95, +4.95]    | 0.76301  | < 0.001   | **           |
| First-Time Cohort | LGBTQ+                                | -0.72                         | [-5.28, +3.85]    | 2.33042  | 0.7588    |              |
| First-Time Cohort | Family Household                      | +13.22                        | [+9.99, +16.46]   | 1.65045  | < 0.001   | **           |
| First-Time Cohort | Veteran                               | +0.80                         | [-0.22, +1.82]    | 0.521962 | 0.1247    |              |
| First-Time Cohort | Black / African American              | +5.76                         | [+4.35, +7.17]    | 0.718868 | < 0.001   | **           |
| First-Time Cohort | Age (Years)                           | +0.11                         | [+0.02, +0.20]    | 0.0439932| 0.0119    | *            |
| First-Time Cohort | Employed at Entry                     | +7.43                         | [+5.44, +9.42]    | 1.01484  | < 0.001   | **           |
| First-Time Cohort | Chronically Unhoused                  | +0.28                         | [-1.25, +1.81]    | 0.780759 | 0.7155    |              |
| First-Time Cohort | Domestic Violence Survivor            | +0.89                         | [-0.26, +2.03]    | 0.582933 | 0.1289    |              |
| First-Time Cohort | Disabling Condition                   | -0.17                         | [-1.61, +1.26]    | 0.731918 | 0.8111    |              |
| First-Time Cohort | Unable to Work                        | +1.34                         | [+0.10, +2.57]    | 0.630691 | 0.0338    | *            |
| First-Time Cohort | Prior Location: Unsheltered           | -4.71                         | [-6.51, -2.91]    | 0.918562 | < 0.001   | **           |
| First-Time Cohort | Received Housing & Financial Services | +11.50                        | [+7.50, +15.50]   | 2.04139  | < 0.001   | **           |
| First-Time Cohort | Received Jobs & Employment Services   | -3.47                         | [-8.31, +1.37]    | 2.47007  | 0.1602    |              |
| First-Time Cohort | Days Enrolled in Program (100s)       | +4.06                         | [+3.56, +4.55]    | 0.253703 | < 0.001   | **           |
| First-Time Cohort | Earned Monthly Income ($1,000s)       | +1.21                         | [-0.05, +2.46]    | 0.638661 | 0.0589    |              |
| First-Time Cohort | Total Lifetime Enrollments            | -13.75                        | [-15.61, -11.89]  | 0.947782 | < 0.001   | **           |
| Repeat Cohort     | Safe Parking Program (SPP Baseline)   | -13.34                        | [-18.73, -7.94]   | 2.75164  | < 0.001   | **           |
| Repeat Cohort     | Enrolled Post-2022 Era                | -1.77                         | [-3.84, +0.31]    | 1.05878  | 0.0951    |              |
| Repeat Cohort     | SPP × Post-2022 Era (DiD Estimator)   | +9.93                         | [+2.85, +17.02]   | 3.61472  | 0.0060    | **           |
| Repeat Cohort     | Woman                                 | +2.40                         | [+1.19, +3.61]    | 0.616833 | < 0.001   | **           |
| Repeat Cohort     | LGBTQ+                                | +0.82                         | [-1.81, +3.46]    | 1.34646  | 0.5401    |              |
| Repeat Cohort     | Family Household                      | +8.55                         | [+4.11, +12.99]   | 2.26497  | < 0.001   | **           |
| Repeat Cohort     | Veteran                               | +0.74                         | [-0.09, +1.57]    | 0.42439  | 0.0816    |              |
| Repeat Cohort     | Black / African American              | +4.79                         | [+3.58, +6.00]    | 0.618738 | < 0.001   | **           |
| Repeat Cohort     | Age (Years)                           | +0.12                         | [+0.05, +0.20]    | 0.0382835| 0.0016    | **           |
| Repeat Cohort     | Employed at Entry                     | +4.22                         | [+2.38, +6.06]    | 0.939289 | < 0.001   | **           |
| Repeat Cohort     | Chronically Unhoused                  | +0.68                         | [-0.64, +2.01]    | 0.675239 | 0.3109    |              |
| Repeat Cohort     | Domestic Violence Survivor            | +1.68                         | [+0.75, +2.60]    | 0.471586 | < 0.001   | **           |
| Repeat Cohort     | Disabling Condition                   | +1.99                         | [+0.62, +3.36]    | 0.700718 | 0.0045    | **           |
| Repeat Cohort     | Unable to Work                        | +0.72                         | [-0.32, +1.75]    | 0.528992 | 0.1750    |              |
| Repeat Cohort     | Prior Location: Unsheltered           | -6.55                         | [-8.18, -4.93]    | 0.82948  | < 0.001   | **           |
| Repeat Cohort     | Received Housing & Financial Services | +7.62                         | [+3.33, +11.92]   | 2.19139  | < 0.001   | **           |
| Repeat Cohort     | Received Jobs & Employment Services   | +0.94                         | [-4.86, +6.74]    | 2.95904  | 0.7503    |              |
| Repeat Cohort     | Days Enrolled in Program (100s)       | +3.89                         | [+3.51, +4.28]    | 0.197148 | < 0.001   | **           |
| Repeat Cohort     | Earned Monthly Income ($1,000s)       | +2.84                         | [+1.92, +3.76]    | 0.469983 | < 0.001   | **           |
| Repeat Cohort     | Total Lifetime Enrollments            | -2.14                         | [-2.47, -1.82]    | 0.165966 | < 0.001   | **           |

*`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*
