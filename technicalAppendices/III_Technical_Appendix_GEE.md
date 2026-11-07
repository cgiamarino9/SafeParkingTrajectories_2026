# Technical Appendix III: Generalized Estimating Equations (GEE) Exit Odds

**Specification:** Binomial GEE Logit | **Working Correlation:** Robust Cluster-Sandwich | **Reference Year:** 2017 | **Clustering:** `ProjectID`

| Cohort            | Variable / Predictor                  |        Coef |   Robust SE | aOR [95% CI]         | p-value    | Significance |
|:------------------|:--------------------------------------|------------:|------------:|:---------------------|:-----------|:-------------|
| First-Time Cohort | Constant / Intercept                  | -0.237104   |  0.216579   | 0.789 [0.516, 1.206] | 0.2736     |              |
| First-Time Cohort | Calendar Year 2018 (vs. 2017)         | -0.0715866  |  0.146576   | 0.931 [0.698, 1.241] | 0.6253     |              |
| First-Time Cohort | Calendar Year 2019 (vs. 2017)         | -0.295884   |  0.167596   | 0.744 [0.536, 1.033] | 0.0775     |              |
| First-Time Cohort | Calendar Year 2020 (vs. 2017)         | -0.556508   |  0.175068   | 0.573 [0.407, 0.808] | 0.0015     | **           |
| First-Time Cohort | Calendar Year 2021 (vs. 2017)         | -0.475592   |  0.18036    | 0.622 [0.436, 0.885] | 0.0084     | **           |
| First-Time Cohort | Calendar Year 2022 (vs. 2017)         | -0.67786    |  0.173984   | 0.508 [0.361, 0.714] | < 0.001    | **           |
| First-Time Cohort | Calendar Year 2023 (vs. 2017)         | -0.620169   |  0.179558   | 0.538 [0.378, 0.765] | < 0.001    | **           |
| First-Time Cohort | Calendar Year 2024 (vs. 2017)         | -1.12219    |  0.189519   | 0.326 [0.225, 0.472] | < 0.001    | **           |
| First-Time Cohort | Calendar Year 2025 (vs. 2017)         | -2.29554    |  0.345181   | 0.101 [0.051, 0.198] | < 0.001    | **           |
| First-Time Cohort | Safe Parking Program (SPP Baseline)   | -0.409362   |  0.141442   | 0.664 [0.503, 0.876] | 0.0038     | **           |
| First-Time Cohort | Woman                                 |  0.20575    |  0.0484397  | 1.228 [1.117, 1.351] | < 0.001    | **           |
| First-Time Cohort | LGBTQ+                                | -0.0482494  |  0.148774   | 0.953 [0.712, 1.276] | 0.7457     |              |
| First-Time Cohort | Family Household                      |  0.76175    |  0.109562   | 2.142 [1.728, 2.655] | < 0.001    | **           |
| First-Time Cohort | Veteran                               |  0.0466058  |  0.0313383  | 1.048 [0.985, 1.114] | 0.1370     |              |
| First-Time Cohort | Black / African American              |  0.330337   |  0.0417998  | 1.391 [1.282, 1.510] | < 0.001    | **           |
| First-Time Cohort | Age (Years)                           |  0.00649114 |  0.00274621 | 1.007 [1.001, 1.012] | 0.0181     | *            |
| First-Time Cohort | Employed at Entry                     |  0.442386   |  0.0668462  | 1.556 [1.365, 1.774] | < 0.001    | **           |
| First-Time Cohort | Chronically Unhoused                  |  0.026404   |  0.0474863  | 1.027 [0.936, 1.127] | 0.5782     |              |
| First-Time Cohort | Domestic Violence Survivor            |  0.0610089  |  0.0355009  | 1.063 [0.991, 1.139] | 0.0857     |              |
| First-Time Cohort | Disabling Condition                   |  0.00388688 |  0.0450901  | 1.004 [0.919, 1.097] | 0.9313     |              |
| First-Time Cohort | Unable to Work                        |  0.0830464  |  0.0386888  | 1.087 [1.007, 1.172] | 0.0318     | *            |
| First-Time Cohort | Prior Location: Unsheltered           | -0.272187   |  0.0555389  | 0.762 [0.683, 0.849] | < 0.001    | **           |
| First-Time Cohort | Received Housing & Financial Services |  0.687805   |  0.116821   | 1.989 [1.582, 2.501] | < 0.001    | **           |
| First-Time Cohort | Received Jobs & Employment Services   | -0.228701   |  0.155334   | 0.796 [0.587, 1.079] | 0.1409     |              |
| First-Time Cohort | Days Enrolled in Program (100s)       |  0.231105   |  0.0169791  | 1.260 [1.219, 1.303] | < 0.001    | **           |
| First-Time Cohort | Earned Monthly Income (\\$1,000s)       |  0.0779676  |  0.0395124  | 1.081 [1.001, 1.168] | 0.0485     | *            |
| First-Time Cohort | Total Lifetime Enrollments            | -0.845399   |  0.0669716  | 0.429 [0.377, 0.490] | < 0.001    | **           |
| Repeat Cohort     | Constant / Intercept                  | -1.93925    |  0.254217   | 0.144 [0.087, 0.237] | < 0.001    | **           |
| Repeat Cohort     | Calendar Year 2018 (vs. 2017)         |  0.221084   |  0.20862    | 1.247 [0.829, 1.878] | 0.2893     |              |
| Repeat Cohort     | Calendar Year 2019 (vs. 2017)         |  0.028226   |  0.19994    | 1.029 [0.695, 1.522] | 0.8877     |              |
| Repeat Cohort     | Calendar Year 2020 (vs. 2017)         | -0.147227   |  0.207063   | 0.863 [0.575, 1.295] | 0.4771     |              |
| Repeat Cohort     | Calendar Year 2021 (vs. 2017)         |  0.231018   |  0.207308   | 1.260 [0.839, 1.891] | 0.2651     |              |
| Repeat Cohort     | Calendar Year 2022 (vs. 2017)         |  0.0210477  |  0.19603    | 1.021 [0.695, 1.500] | 0.9145     |              |
| Repeat Cohort     | Calendar Year 2023 (vs. 2017)         |  0.125276   |  0.197128   | 1.133 [0.770, 1.668] | 0.5251     |              |
| Repeat Cohort     | Calendar Year 2024 (vs. 2017)         | -0.254027   |  0.201696   | 0.776 [0.522, 1.152] | 0.2079     |              |
| Repeat Cohort     | Calendar Year 2025 (vs. 2017)         | -0.90439    |  0.430629   | 0.405 [0.174, 0.941] | 0.0357     | *            |
| Repeat Cohort     | Safe Parking Program (SPP Baseline)   | -0.543888   |  0.131859   | 0.580 [0.448, 0.752] | < 0.001    | **           |
| Repeat Cohort     | Woman                                 |  0.161225   |  0.0416054  | 1.175 [1.083, 1.275] | < 0.001    | **           |
| Repeat Cohort     | LGBTQ+                                |  0.0534199  |  0.0885708  | 1.055 [0.887, 1.255] | 0.5464     |              |
| Repeat Cohort     | Family Household                      |  0.564875   |  0.154112   | 1.759 [1.301, 2.380] | < 0.001    | **           |
| Repeat Cohort     | Veteran                               |  0.0534402  |  0.028293   | 1.055 [0.998, 1.115] | 0.0589     |              |
| Repeat Cohort     | Black / African American              |  0.325735   |  0.0386279  | 1.385 [1.284, 1.494] | < 0.001    | **           |
| Repeat Cohort     | Age (Years)                           |  0.00893337 |  0.00250528 | 1.009 [1.004, 1.014] | < 0.001    | **           |
| Repeat Cohort     | Employed at Entry                     |  0.274047   |  0.0625782  | 1.315 [1.163, 1.487] | < 0.001    | **           |
| Repeat Cohort     | Chronically Unhoused                  |  0.0475371  |  0.0452684  | 1.049 [0.960, 1.146] | 0.2937     |              |
| Repeat Cohort     | Domestic Violence Survivor            |  0.111566   |  0.0319983  | 1.118 [1.050, 1.190] | < 0.001    | **           |
| Repeat Cohort     | Disabling Condition                   |  0.129499   |  0.0461888  | 1.138 [1.040, 1.246] | 0.0051     | **           |
| Repeat Cohort     | Unable to Work                        |  0.045646   |  0.0349571  | 1.047 [0.977, 1.121] | 0.1916     |              |
| Repeat Cohort     | Prior Location: Unsheltered           | -0.43944    |  0.0517362  | 0.644 [0.582, 0.713] | < 0.001    | **           |
| Repeat Cohort     | Received Housing & Financial Services |  0.515907   |  0.143655   | 1.675 [1.264, 2.220] | < 0.001    | **           |
| Repeat Cohort     | Received Jobs & Employment Services   |  0.0697748  |  0.190852   | 1.072 [0.738, 1.559] | 0.7147     |              |
| Repeat Cohort     | Days Enrolled in Program (100s)       |  0.251779   |  0.0163028  | 1.286 [1.246, 1.328] | < 0.001    | **           |
| Repeat Cohort     | Earned Monthly Income (\\$1,000s)       |  0.195663   |  0.0307754  | 1.216 [1.145, 1.292] | < 0.001    | **           |
| Repeat Cohort     | Total Lifetime Enrollments            | -0.14559    |  0.0103867  | 0.865 [0.847, 0.882] | < 0.001    | **           |

*`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*
