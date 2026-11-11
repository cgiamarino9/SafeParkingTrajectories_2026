## Technical Appendix VII: Doubly Robust Difference-in-Differences (DR DID) Results Matrix
### Sant'Anna & Zhao's (2020) Doubly Robust Difference-in-DIfferences (DR DID) estimator 
This appendix validates the LPM DID findings. It combines inverse probability weighting (IPW) aka propensity scores with outcome regression (OR). It produces consistent, efficient treatment effects even if the propensity score model or the outcome regression model is mis-specified. Here is a [link](https://doi.org/10.1016/j.jeconom.2020.06.003) to the article. 

| Participant Cohort Sample | Econometric Model Specification | Complete Case N | Treatment Estimand ($\hat{\tau}_{\text{DR}}$) | Standard Error (SE) | $z$-Statistic | $p$-Value | 95% Confidence Interval |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| *First-Time Cohort* (`df_first_clean`) | DR DID | 48,690 | **+9.88 pp**** | 1.59 pp | 6.197 | 0.0000 | [6.76pp, 13.01pp] |
| *Repeat Cohort* (`df_repeat_clean`) | DR DID | 49,952 | **+8.15 pp**** | 1.48 pp | 5.524 | 0.0000 | [5.26pp, 11.04pp] |

*Note*: ** p < 0.01, * p < 0.05. Standard errors are analytically computed via the influence function. Propensity scores are estimated via inverse probability tilting trimmed at the 99.5th percentile. `received_service_jobs_employment` was excluded from the first-time cohort due to zero within-cell variance in the post-treatment Safe Parking sub-cell partitions ($T = 1, D = 1$).

### Safe Parking Evaluation: DiD $T \times D$ Sub-Cell Partition Matrix

| Sub-Cell Partition | Time Period Indicator ($T$) | Treatment Indicator ($D$) | Study Sub-Cell Sample | Econometric Function |
| :---: | :---: | :---: | :--- | :--- |
| *Sub-Cell 1* | $T = 0$ (Baseline: 2017–2021) | $D = 0$ (Control) | Pre-expansion Emergency Shelter clients | Pre-treatment control baseline ($\bar{Y}_{0,0}$) |
| *Sub-Cell 2* | $T = 0$ (Baseline: 2017–2021) | $D = 1$ (Treated) | Pre-expansion Safe Parking Program clients | Pre-treatment treatment baseline ($\bar{Y}_{0,1}$) |
| *Sub-Cell 3* | $T = 1$ (Post-Era: 2022–2025) | $D = 0$ (Control) | Post-expansion Emergency Shelter clients | Post-treatment control trajectory ($\bar{Y}_{1,0}$) |
| *Sub-Cell 4* | $T = 1$ (Post-Era: 2022–2025) | $D = 1$ (Treated) | Post-expansion Safe Parking Program clients | Post-treatment treatment outcome ($\bar{Y}_{1,1}$) |

---

### Sub-Cell Descriptions and Econometric Specifications

* *Sub-Cell 1 ($T=0, D=0$): Pre-2022 Control Partition*
  * Description: Emergency Shelter participants enrolled prior to the 2022 policy expansions, shifts, and scandals (2017–2021).
  * Econometric Specification: Creates the baseline exit probability against which the time trends are measured.

* *Sub-Cell 2 ($T=0, D=1$): Pre-2022 Treated Partition*
  * Description: Safe Parking Program (SPP) participants enrolled prior to the 2022 policy expansions, shifts, and scandals (2017–2021).
  * Econometric Specification: Looks at initial program selection bias and pre-2022 demographic differences between SPP clients and emergency shelter clients before expansion.

* *Sub-Cell 3 ($T=1, D=0$): Post-2022 Control Partition*
  * Definition: Emergency Shelter participants enrolled during the post-2022 policy expansions, shifts, and scandals era (2022–2025).
  * Econometric Specification: Explores broader macroeconomic housing market and homelessness trends and dynamics affecting unhoused exits independently of SPP expansion.

* *Sub-Cell 4 ($T=1, D=1$): Post-2022 Treated Partition*
  * Definition: Safe Parking Program (SPP) participants enrolled during the post-2022 policy expansions, shifts, and scandals era (2022–2025).
  * Econometric Specification: Analyzes the post-expansion treatment outcome. In `drdid_rc`, inverse probability weighting and outcome regressions are placed separately into each partition. In the first-time cohort dataset, `received_service_jobs_employment` exhibited zero variation ($SD = 0.00$) in this specific sub-cell, which caused design matrix singularity Python errors across all sub-cell partitions. It was not included in this specification. 