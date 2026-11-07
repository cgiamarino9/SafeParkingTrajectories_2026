# Technical Appendix IV: Continuous Average Macro Trends

**Specification:** Binomial Logit | **Trend Variable:** Year - 2017 | **Clustering:** `ProjectID`

| Cohort            | Variable / Predictor                  |         Coef |   Robust_SE | aOR [95% CI]         | p_value   | Significance   |
|:------------------|:--------------------------------------|-------------:|------------:|:---------------------|:----------|:-------|
| First-Time Cohort | Constant / Intercept                  | -0.145581    |  0.17437    | 0.865 [0.614, 1.217] | 0.4038    |        |
| First-Time Cohort | Annual Linear Trend (Years Post-2017) | -0.151644    |  0.0192754  | 0.859 [0.827, 0.892] | < 0.001   | **     |
| First-Time Cohort | Safe Parking Program (SPP Baseline)   | -1.21335     |  0.330027   | 0.297 [0.156, 0.568] | < 0.001   | **     |
| First-Time Cohort | Annual Trend × SPP Interaction        |  0.171227    |  0.0658432  | 1.187 [1.043, 1.350] | 0.0093    | **     |
| First-Time Cohort | Woman                                 |  0.205646    |  0.0481552  | 1.228 [1.118, 1.350] | < 0.001   | **     |
| First-Time Cohort | LGBTQ+                                | -0.0483148   |  0.144479   | 0.953 [0.718, 1.265] | 0.7381    |        |
| First-Time Cohort | Family Household                      |  0.758383    |  0.107304   | 2.135 [1.730, 2.635] | < 0.001   | **     |
| First-Time Cohort | Veteran                               |  0.0456367   |  0.030889   | 1.047 [0.985, 1.112] | 0.1396    |        |
| First-Time Cohort | Black / African American              |  0.321173    |  0.041278   | 1.379 [1.272, 1.495] | < 0.001   | **     |
| First-Time Cohort | Age (Years)                           |  0.00587029  |  0.00274806 | 1.006 [1.000, 1.011] | 0.0327    | *      |
| First-Time Cohort | Employed at Entry                     |  0.441711    |  0.0656781  | 1.555 [1.367, 1.769] | < 0.001   | **     |
| First-Time Cohort | Chronically Unhoused                  |  0.0203133   |  0.0474312  | 1.021 [0.930, 1.120] | 0.6685    |        |
| First-Time Cohort | Domestic Violence Survivor            |  0.0545098   |  0.0354755  | 1.056 [0.985, 1.132] | 0.1244    |        |
| First-Time Cohort | Disabling Condition                   | -0.000359735 |  0.0445095  | 1.000 [0.916, 1.091] | 0.9936    |        |
| First-Time Cohort | Unable to Work                        |  0.0781035   |  0.0387282  | 1.081 [1.002, 1.167] | 0.0437    | *      |
| First-Time Cohort | Prior Location: Unsheltered           | -0.26461     |  0.0548815  | 0.768 [0.689, 0.855] | < 0.001   | **     |
| First-Time Cohort | Received Housing & Financial Services |  0.674004    |  0.114119   | 1.962 [1.569, 2.454] | < 0.001   | **     |
| First-Time Cohort | Received Jobs & Employment Services   | -0.218713    |  0.1494     | 0.804 [0.600, 1.077] | 0.1432    |        |
| First-Time Cohort | Days Enrolled in Program (100s)       |  0.240517    |  0.0168772  | 1.272 [1.231, 1.315] | < 0.001   | **     |
| First-Time Cohort | Earned Monthly Income ($1,000s)       |  0.0759732   |  0.0386216  | 1.079 [1.000, 1.164] | 0.0492    | *      |
| First-Time Cohort | Total Lifetime Enrollments            | -0.850435    |  0.0665802  | 0.427 [0.375, 0.487] | < 0.001   | **     |
| Repeat Cohort     | Constant / Intercept                  | -1.7417      |  0.220394   | 0.175 [0.114, 0.270] | < 0.001   | **     |
| Repeat Cohort     | Annual Linear Trend (Years Post-2017) | -0.0347333   |  0.0200855  | 0.966 [0.929, 1.005] | 0.0838    |        |
| Repeat Cohort     | Safe Parking Program (SPP Baseline)   | -1.55937     |  0.360908   | 0.210 [0.104, 0.427] | < 0.001   | **     |
| Repeat Cohort     | Annual Trend × SPP Interaction        |  0.222968    |  0.0735563  | 1.250 [1.082, 1.444] | 0.0024    | **     |
| Repeat Cohort     | Woman                                 |  0.159037    |  0.0416943  | 1.172 [1.080, 1.272] | < 0.001   | **     |
| Repeat Cohort     | LGBTQ+                                |  0.054904    |  0.0898606  | 1.056 [0.886, 1.260] | 0.5412    |        |
| Repeat Cohort     | Family Household                      |  0.563652    |  0.154559   | 1.757 [1.298, 2.379] | < 0.001   | **     |
| Repeat Cohort     | Veteran                               |  0.0487288   |  0.0282516  | 1.050 [0.993, 1.110] | 0.0846    |        |
| Repeat Cohort     | Black / African American              |  0.316575    |  0.0390628  | 1.372 [1.271, 1.482] | < 0.001   | **     |
| Repeat Cohort     | Age (Years)                           |  0.00818972  |  0.00264554 | 1.008 [1.003, 1.013] | 0.0020    | **     |
| Repeat Cohort     | Employed at Entry                     |  0.280491    |  0.0619827  | 1.324 [1.172, 1.495] | < 0.001   | **     |
| Repeat Cohort     | Chronically Unhoused                  |  0.0465889   |  0.0450369  | 1.048 [0.959, 1.144] | 0.3009    |        |
| Repeat Cohort     | Domestic Violence Survivor            |  0.112462    |  0.0319112  | 1.119 [1.051, 1.191] | < 0.001   | **     |
| Repeat Cohort     | Disabling Condition                   |  0.133831    |  0.0465415  | 1.143 [1.044, 1.252] | 0.0040    | **     |
| Repeat Cohort     | Unable to Work                        |  0.0478427   |  0.0352303  | 1.049 [0.979, 1.124] | 0.1745    |        |
| Repeat Cohort     | Prior Location: Unsheltered           | -0.43642     |  0.0511567  | 0.646 [0.585, 0.715] | < 0.001   | **     |
| Repeat Cohort     | Received Housing & Financial Services |  0.504893    |  0.142885   | 1.657 [1.252, 2.192] | < 0.001   | **     |
| Repeat Cohort     | Received Jobs & Employment Services   |  0.0639022   |  0.196493   | 1.066 [0.725, 1.567] | 0.7450    |        |
| Repeat Cohort     | Days Enrolled in Program (100s)       |  0.25837     |  0.0158295  | 1.295 [1.255, 1.336] | < 0.001   | **     |
| Repeat Cohort     | Earned Monthly Income (\\$1,000s)       |  0.190059    |  0.030762   | 1.209 [1.139, 1.284] | < 0.001   | **     |
| Repeat Cohort     | Total Lifetime Enrollments            | -0.142232    |  0.0101659  | 0.867 [0.850, 0.885] | < 0.001   | **     |

*`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*
