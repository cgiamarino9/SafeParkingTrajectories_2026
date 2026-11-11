## Technical Appendix VII: Doubly Robust Difference-in-Differences (DR DID) Results Matrix
### Sant'Anna & Zhao's (2020) Doubly Robust Difference-in-DIfferences (DR DID) estimator 
This appendix validates teh LPM findings. It combines inverse probability weighting (IPW) aka propensity scores with outcome regression (OR). It produces consistent, efficient treatment effects even if the propensity score model or the outcome regression model is mis-specified. Here is a [link](https://doi.org/10.1016/j.jeconom.2020.06.003) to the article. 

| Participant Cohort Sample | Econometric Model Specification | Complete Case N | Treatment Estimand ($\hat{\tau}_{\text{DR}}$) | Standard Error (SE) | $z$-Statistic | $p$-Value | 95% Confidence Interval |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| *First-Time Cohort* (`df_first_clean`) | DR DID | 48,690 | **+9.88 pp**** | 1.59 pp | 6.197 | 0.0000 | [6.76pp, 13.01pp] |
| *Repeat Cohort* (`df_repeat_clean`) | DR DID | 49,952 | **+8.15 pp**** | 1.48 pp | 5.524 | 0.0000 | [5.26pp, 11.04pp] |

*Note*: ** p < 0.01, * p < 0.05. Standard errors are analytically computed via the influence function. Propensity scores are estimated via inverse probability tilting trimmed at the 99.5th percentile. `received_service_jobs_employment` was excluded from the first-time cohort due to zero within-cell variance in the Post-2022 Safe Parking interaction (Post2022==1, SPP==1).