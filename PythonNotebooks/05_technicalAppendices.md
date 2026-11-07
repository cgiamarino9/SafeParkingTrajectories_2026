# Technical Appendices
## Tables for Supplementary Data

## Technical Appendix I: Full diagnostics 



```python
# variable mapping
variable_mapping = {
    "Woman": "Woman",
    "LGBTQ": "LGBTQ",
    "Family": "Family",
    # "SingleParentWChild": "Single Parent Household with Child",
    "VeteranStatus": "Veteran",
    # "White": "White",
    "BlackAfAmerican": "Black/African American",
    "Employed_Entry": "Employed",
    "ChronicHomelessness": "Chronically Unhoused",
    "DVSurvivor_Entry": "Domestic Violence Survivor",
    "DisablingCondition": "Disabling Condition",
    "UnableToWork_Entry": "Unable to Work",
    "Prior_Unsheltered": "Prior Location: Unsheltered",
    "received_service_housing_financial": (
        "Received Housing and Financial Services"
    ),
    "received_service_jobs_employment": (
        "Received Jobs and Employment Services"
    ),
    "Post2022": "Enrolled Post-2022 Era",
    "Age": "Age",
    "DaysEnrolled": "Days Enrolled (100s)",
    "Earned_Entry": "Earned Income ($1,000s)",
    "Total_Lifetime_Enrollments": "Total Lifetime Enrollments",
}

predictors = list(variable_mapping.keys())
formula_multivariable = "SPP ~ " + " + ".join(predictors)

# fit model
model_multi = smf.logit(formula_multivariable, data=df_first_all).fit()

# calculate VIF
df_vif_clean = df_first_all[["SPP"] + predictors].dropna()
y_matrix, X_matrix = dmatrices(
    formula_multivariable, data=df_vif_clean, return_type="dataframe"
)

vif_series = pd.Series(
    [
        variance_inflation_factor(X_matrix.values, i)
        for i in range(X_matrix.shape[1])
    ],
    index=X_matrix.columns,
)

# get metrics for technical appendix
appendix_rows = []
conf_int = model_multi.conf_int()

for var in predictors:
  b = model_multi.params[var]
  aor = np.exp(b)
  ci_low = np.exp(conf_int.loc[var, 0])
  ci_high = np.exp(conf_int.loc[var, 1])
  p_val = model_multi.pvalues[var]
  vif_val = vif_series.get(var, np.nan)

  # Significance formatting
  if p_val < 0.01:
    sig_star = "**"
    p_str = f"{p_val:.4f}"
  elif p_val < 0.05:
    sig_star = "*"
    p_str = f"{p_val:.4f}"
  else:
    sig_star = " "
    p_str = f"{p_val:.4f}"

  appendix_rows.append({
      "Variable Label": variable_mapping[var],
      "Coeff (β)": f"{b:+.4f}",
      "aOR": f"{aor:.3f}",
      "95% CI": f"[{ci_low:.2f}, {ci_high:.2f}]",
      "p-value": p_str,
      "Sig": sig_star,
      "VIF": f"{vif_val:.2f}" if not np.isnan(vif_val) else "N/A",
  })

df_appendix_i = pd.DataFrame(appendix_rows)

# export to markdown
markdown_output = f"""# Technical Appendix I: Multivariable Logistic Regression
### Determinants of Safe Parking Program (SPP) Entry (All First-Time Enrollees, 2017-2025)

{df_appendix_i.to_markdown(index=False)}

---
*Notes: Sample Size N = {int(model_multi.nobs):,}. Log-Likelihood = {model_multi.llf:.2f}. Pseudo R² = {model_multi.prsquared:.4f}.*
*Significance levels: *** ** p < 0.01, * p < 0.05.*
"""

output_path = output_dir / "I_Technical_Appendix_Logit.md"
with open(output_path, "w", encoding="utf-8") as f:
  f.write(markdown_output)

print(
    f"Success! Technical Appendix I exported successfully to: {output_path}"
)
```

# Technical Appendix I: Multivariable Logistic Regression
### Determinants of Safe Parking Program (SPP) Entry (All First-Time Enrollees, 2017-2025)

| Variable Label                          |   Coeff (β) |    aOR | 95% CI         |   p-value | Sig   |   VIF |
|:----------------------------------------|------------:|-------:|:---------------|----------:|:------|------:|
| Woman                                   |      0.2647 |  1.303 | [1.21, 1.40]   |    0      | **    |  1.22 |
| LGBTQ                                   |     -0.066  |  0.936 | [0.68, 1.29]   |    0.6892 |       |  1.02 |
| Family                                  |     -0.1323 |  0.876 | [0.76, 1.01]   |    0.0654 |       |  1.15 |
| Veteran                                 |      0.1146 |  1.121 | [1.07, 1.17]   |    0      | **    |  1.01 |
| Black/African American                  |      0.056  |  1.058 | [0.99, 1.13]   |    0.1057 |       |  1.02 |
| Employed                                |      1.156  |  3.177 | [2.89, 3.50]   |    0      | **    |  1.18 |
| Chronically Unhoused                    |     -0.907  |  0.404 | [0.36, 0.45]   |    0      | **    |  1.73 |
| Domestic Violence Survivor              |     -0.0215 |  0.979 | [0.90, 1.07]   |    0.6294 |       |  1.19 |
| Disabling Condition                     |      0.3272 |  1.387 | [1.27, 1.51]   |    0      | **    |  1.61 |
| Unable to Work                          |     -0.2979 |  0.742 | [0.68, 0.81]   |    0      | **    |  1.32 |
| Prior Location: Unsheltered             |      2.5565 | 12.89  | [11.25, 14.76] |    0      | **    |  1.25 |
| Received Housing and Financial Services |      0.2736 |  1.315 | [1.06, 1.63]   |    0.0135 | *     |  1.03 |
| Received Jobs and Employment Services   |      0.2619 |  1.299 | [0.87, 1.94]   |    0.1974 |       |  1.02 |
| Enrolled Post-2022 Era                  |      0.1188 |  1.126 | [1.05, 1.21]   |    0.001  | **    |  1.1  |
| Age                                     |      0.0273 |  1.028 | [1.03, 1.03]   |    0      | **    |  1.24 |
| Days Enrolled (100s)                    |     -0.1991 |  0.819 | [0.80, 0.84]   |    0      | **    |  1.04 |
| Earned Income ($1,000s)                 |      0.121  |  1.129 | [1.08, 1.18]   |    0      | **    |  1.08 |
| Total Lifetime Enrollments              |      0.0355 |  1.036 | [1.01, 1.06]   |    0.0094 | **    |  1.08 |

---
*Notes: Sample Size N = 81,078. Log-Likelihood = -13425.49. Pseudo R² = 0.1402.*
*`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*


## Technical Appendix II prior and exit for all 


```python
# cross tabs and chi-square values 
def compute_crosstab_row(df, col_name, cat_label):
  # Split by Cohort and SPP status
  df_non_repeat = df[df["True_Repeat_Client"] == 0]
  df_repeat = df[df["True_Repeat_Client"] == 1]

  # Non-Repeat Metrics
  spp_nr = (df_non_repeat[df_non_repeat["SPP"] == 1][col_name] == 1).mean() * 100
  nonspp_nr = (
      df_non_repeat[df_non_repeat["SPP"] == 0][col_name] == 1
  ).mean() * 100
  ct_nr = pd.crosstab(df_non_repeat[col_name], df_non_repeat["SPP"])
  p_nr = chi2_contingency(ct_nr)[1] if ct_nr.shape == (2, 2) else np.nan

  # Repeat Metrics
  spp_r = (df_repeat[df_repeat["SPP"] == 1][col_name] == 1).mean() * 100
  nonspp_r = (df_repeat[df_repeat["SPP"] == 0][col_name] == 1).mean() * 100
  ct_r = pd.crosstab(df_repeat[col_name], df_repeat["SPP"])
  p_r = chi2_contingency(ct_r)[1] if ct_r.shape == (2, 2) else np.nan

  p_nr_str = "<0.001**" if p_nr < 0.001 else f"{p_nr:.3f}"
  p_r_str = "<0.001**" if p_r < 0.001 else f"{p_r:.3f}"

  return {
      "Category / Location": cat_label,
      "SPP (Non-Repeat)": f"{spp_nr:.1f}%",
      "Non-SPP (Non-Repeat)": f"{nonspp_nr:.1f}%",
      "p-value (Non-Repeat)": p_nr_str,
      "SPP (Repeat)": f"{spp_r:.1f}%",
      "Non-SPP (Repeat)": f"{nonspp_r:.1f}%",
      "p-value (Repeat)": p_r_str,
  }


# exit and prior variables 
exit_vars = [
    ("Exit_Unsheltered", "Exit: Unsheltered"),
    ("Exit_TemporaryStable", "Exit: Temporary Stable"),
    ("Exit_TemporaryUnstable", "Exit: Temporary Unstable"),
    ("Exit_Permanent", "Exit: Permanent"),
    ("Exit_Institutional", "Exit: Institutional"),
]

prior_vars = [
    ("Prior_Unsheltered", "Prior: Unsheltered"),
    ("Prior_TemporaryStable", "Prior: Temporary Stable"),
    ("Prior_TemporaryUnstable", "Prior: Temporary Unstable"),
    ("Prior_Permanent", "Prior: Permanent"),
    ("Prior_Institutional", "Prior: Institutional"),
]

# create rows
rows = []
rows.append({
    "Category / Location": "**Prior Locations at Entry**",
    "SPP (Non-Repeat)": "",
    "Non-SPP (Non-Repeat)": "",
    "p-value (Non-Repeat)": "",
    "SPP (Repeat)": "",
    "Non-SPP (Repeat)": "",
    "p-value (Repeat)": "",
})

for var_col, var_label in prior_vars:
  rows.append(compute_crosstab_row(df_clean, var_col, var_label))



rows.append({
    "Category / Location": "**Exit Destinations**",
    "SPP (Non-Repeat)": "",
    "Non-SPP (Non-Repeat)": "",
    "p-value (Non-Repeat)": "",
    "SPP (Repeat)": "",
    "Non-SPP (Repeat)": "",
    "p-value (Repeat)": "",
})


for var_col, var_label in exit_vars:
  rows.append(compute_crosstab_row(df_clean, var_col, var_label))

appendix_ii_df = pd.DataFrame(rows)

# markdown 
markdown_content = f"""# Technical Appendix II: Prior & Exit Location Distributions
### Unadjusted Baseline Comparisons Across First-Time and Repeat Client Cohorts

{appendix_ii_df.to_markdown(index=False)}

---
*Notes: Evaluated on sample with known exits (`Exit_MissingOther == 0`). Statistical significance evaluated via Chi-Square (χ²) test of independence (* p < 0.05, ** p < 0.01).*
"""

output_path = output_dir / "II_Technical_Appendix_Crosstabs.md"
with open(output_path, "w", encoding="utf-8") as f:
  f.write(markdown_content)

print(
    f"Success! Technical Appendix II exported successfully to: {output_path}"
)
```

# Technical Appendix II: Prior & Exit Location Distributions
### Unadjusted Baseline Comparisons Across First-Time and Repeat Client Cohorts

| Category / Location          | SPP (Non-Repeat)   | Non-SPP (Non-Repeat)   | p-value (Non-Repeat)   | SPP (Repeat)   | Non-SPP (Repeat)   | p-value (Repeat)   |
|:-----------------------------|:-------------------|:-----------------------|:-----------------------|:---------------|:-------------------|:-------------------|
| **Prior Locations at Entry** |                    |                        |                        |                |                    |                    |
| Prior: Unsheltered           | 94.5%              | 58.6%                  | <0.001**               | 93.0%          | 56.5%              | <0.001**           |
| Prior: Temporary Stable      | 1.8%               | 8.0%                   | <0.001**               | 1.2%           | 5.3%               | <0.001**           |
| Prior: Temporary Unstable    | 2.7%               | 20.4%                  | <0.001**               | 4.5%           | 27.1%              | <0.001**           |
| Prior: Permanent             | 0.6%               | 1.2%                   | 0.075                  | 0.7%           | 0.6%               | 0.604              |
| Prior: Institutional         | 0.3%               | 6.3%                   | <0.001**               | 0.4%           | 5.5%               | <0.001**           |
| **Exit Destinations**        |                    |                        |                        |                |                    |                    |
| Exit: Unsheltered            | 50.3%              | 19.0%                  | <0.001**               | 48.3%          | 23.2%              | <0.001**           |
| Exit: Temporary Stable       | 14.2%              | 12.4%                  | 0.039                  | 12.2%          | 10.4%              | 0.003              |
| Exit: Temporary Unstable     | 10.0%              | 25.9%                  | <0.001**               | 25.7%          | 37.8%              | <0.001**           |
| Exit: Permanent              | 24.0%              | 35.9%                  | <0.001**               | 12.0%          | 21.2%              | <0.001**           |
| Exit: Institutional          | 1.5%               | 6.7%                   | <0.001**               | 1.7%           | 7.4%               | <0.001**           |

---
*Notes: Evaluated on sample with known exits (`Exit_MissingOther == 0`). Statistical significance evaluated via Chi-Square (χ²) test of independence: `*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence). Within SPP programs, overall permanent exits for first-time enrollments are 24%. For repeat enrollments, the second exit is 26.2% and the overall permanent exit rate is 12%. Within Non-SPP programs, overall permanent exits for firs-time enrollments is 36%. For repeat enrollments, the second exit is 24% and the overall permanent exit rate is 21%. These are statisticallay significant drops and suggest that SPPs act as stabilization harbors.*



```python
# get counts of repeats

# cross tab 
counts = pd.crosstab(
    df_clean["True_Repeat_Client"],
    df_clean["SPP"],
    margins=True,
    margins_name="Total",
)

# rename for legibility 
counts.index = ["Non-Repeat (0)", "True Repeat (1)", "Total"]
counts.columns = ["Non-SPP (0", "SPP (1)", "Total"]

print(counts)
```

# Bivariate Chi-Square Comparisons: Prior Locations & Exit Outcomes

### Between Programs - Non-Repeat: Prior Locations

| Feature | Non-SPP (N=31,648) | SPP (N=1,575) | p-value (Chi2) | Result |
| :--- | :---: | :---: | :---: | :---: |
| Prior: Unsheltered | 58.6% | 94.5% | < 0.001 | ** |
| Prior: Temporary Stable | 8.0% | 1.8% | < 0.001 | ** |
| Prior: Temporary Unstable | 20.4% | 2.7% | < 0.001 | ** |
| Prior: Permanent | 1.2% | 0.6% | 0.07532 |  |
| Prior: Institutional | 6.3% | 0.3% | < 0.001 | ** |

---

### Between Programs - Non-Repeat: Exit Outcomes

| Feature | Non-SPP (N=31,648) | SPP (N=1,575) | p-value (Chi2) | Result |
| :--- | :---: | :---: | :---: | :---: |
| Exit: Unsheltered | 19.0% | 50.3% | < 0.001 | ** |
| Exit: Temporary Stable | 12.4% | 14.2% | 0.03881 | ** |
| Exit: Temporary Unstable | 25.9% | 10.0% | < 0.001 | ** |
| Exit: Permanent | 36.0% | 24.0% | < 0.001 | ** |
| Exit: Institutional | 6.7% | 1.5% | < 0.001 | ** |

---

### Between Programs - Repeat: Prior Locations

| Feature | Non-SPP (N=47,773) | SPP (N=2,651) | p-value (Chi2) | Result |
| :--- | :---: | :---: | :---: | :---: |
| Prior: Unsheltered | 56.5% | 93.0% | < 0.001 | ** |
| Prior: Temporary Stable | 5.3% | 1.2% | < 0.001 | ** |
| Prior: Temporary Unstable | 27.1% | 4.5% | < 0.001 | ** |
| Prior: Permanent | 0.6% | 0.7% | 0.60351 |  |
| Prior: Institutional | 5.5% | 0.4% | < 0.001 | ** |

---

### Between Programs - Repeat: Exit Outcomes

| Feature | Non-SPP (N=47,773) | SPP (N=2,651) | p-value (Chi2) | Result |
| :--- | :---: | :---: | :---: | :---: |
| Exit: Unsheltered | 23.2% | 48.3% | < 0.001 | ** |
| Exit: Temporary Stable | 10.4% | 12.2% | 0.00343 | ** |
| Exit: Temporary Unstable | 37.8% | 25.7% | < 0.001 | ** |
| Exit: Permanent Next | 24.1% | 26.2% | 0.03550 | ** |
| Exit: Institutional | 7.4% | 1.7% | < 0.001 | ** |

---

### Within Emergency Shelter (SPP=0): Prior Locations

| Feature | Non-Repeat (0) (N=31,648) | Repeat (1) (N=47,773) | p-value (Chi2) | Result |
| :--- | :---: | :---: | :---: | :---: |
| Prior: Unsheltered | 58.6% | 56.5% | < 0.001 | ** |
| Prior: Temporary Stable | 8.0% | 5.3% | < 0.001 | ** |
| Prior: Temporary Unstable | 20.4% | 27.1% | < 0.001 | ** |
| Prior: Permanent | 1.2% | 0.6% | < 0.001 | ** |
| Prior: Institutional | 6.3% | 5.5% | < 0.001 | ** |

---

### Within Emergency Shelter (SPP=0): Exit Outcomes

| Feature | Non-Repeat (0) (N=31,648) | Repeat (1) (N=47,773) | p-value (Chi2) | Result |
| :--- | :---: | :---: | :---: | :---: |
| Exit: Unsheltered | 19.0% | 23.2% | < 0.001 | ** |
| Exit: Temporary Stable | 12.4% | 10.4% | < 0.001 | ** |
| Exit: Temporary Unstable | 25.9% | 37.8% | < 0.001 | ** |
| Exit: Permanent | 36.0% | 21.2% | < 0.001 | ** |
| Exit: Institutional | 6.7% | 7.4% | < 0.001 | ** |
| Exit: Permanent Next | 0.0% | 24.1% | N/A | |

---

### Within Safe Parking (SPP=1): Prior Locations

| Feature | Non-Repeat (0) (N=1,575) | Repeat (1) (N=2,651) | p-value (Chi2) | Result |
| :--- | :---: | :---: | :---: | :---: |
| Prior: Unsheltered | 94.5% | 93.0% | 0.07210 |  |
| Prior: Temporary Stable | 1.8% | 1.2% | 0.20367 |  |
| Prior: Temporary Unstable | 2.7% | 4.5% | 0.00363 | ** |
| Prior: Permanent | 0.6% | 0.7% | 0.90549 |  |
| Prior: Institutional | 0.3% | 0.4% | 0.55969 |  |

---

### Within Safe Parking (SPP=1): Exit Outcomes

| Feature | Non-Repeat (0) (N=1,575) | Repeat (1) (N=2,651) | p-value (Chi2) | Result |
| :--- | :---: | :---: | :---: | :---: |
| Exit: Unsheltered | 50.3% | 48.3% | 0.21982 |  |
| Exit: Temporary Stable | 14.2% | 12.2% | 0.06273 |  |
| Exit: Temporary Unstable | 10.0% | 25.7% | < 0.001 | ** |
| Exit: Permanent | 24.0% | 12.0% | < 0.001 | ** |
| Exit: Institutional | 1.5% | 1.7% | 0.69214 |  |
| Exit: Permanent Next | 0.0% | 26.2% | N/A |  |

---

*`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*

### Takeaways from above charts
Within SPP programs, overall permanent exits for first-time enrollments are 24%. For repeat enrollments, the second exit is 26.2% and the overall permanent exit rate is 12%
Within Non-SPP programs, overall permanent exits for firs-time enrollments is 36%. For repeat enrollments, the second exit is 24% and the overall permanent exit rate is 21%. 
These are statistically significant drops and suggest that SPPs act as stabilization harbors. 

## Technical Appendix III
### Generalized Estimating Equations (GEE)


```python
# re for remapping
import re


# Directory setup
OUTPUT_DIR = "FinalHousingStudies"
os.makedirs(OUTPUT_DIR, exist_ok=True)

# publication styling 
plt.rcParams.update({
    "font.family": "serif",
    "font.serif": ["Times New Roman", "DejaVu Serif"],
    "font.size": 10,
    "axes.labelsize": 10.5,
    "axes.titlesize": 11.5,
})

# CPP Brand Color Palette
COLOR_PRIMARY_GREEN = "#00843D"  # First-Time Cohort
COLOR_CPP_GOLD = "#FFB500"  # Repeat Cohort

# Master Variable Label Mapping Dictionary (Aligned with Main Body LPM DiD)
VAR_LABEL_MAP = {
    "Intercept": "Constant / Intercept",
    "SPP": "Safe Parking Program (SPP Baseline)",
    "Post2022": "Enrolled Post-2022 Era",
    "SPP:Post2022": "SPP × Post-2022 Era (DiD Estimator)",
    "Post2022:SPP": "SPP × Post-2022 Era (DiD Estimator)",
    "Woman": "Woman",
    "LGBTQ": "LGBTQ+",
    "Family": "Family Household",
    "SingleParentWChild": "Single Parent Household with Child",
    "VeteranStatus": "Veteran",
    "White": "White",
    "BlackAfAmerican": "Black / African American",
    "Age": "Age (Years)",
    "Employed_Entry": "Employed at Entry",
    "ChronicHomelessness": "Chronically Unhoused",
    "DVSurvivor_Entry": "Domestic Violence Survivor",
    "DisablingCondition": "Disabling Condition",
    "UnableToWork_Entry": "Unable to Work",
    "Prior_Unsheltered": "Prior Location: Unsheltered",
    "received_service_housing_financial": (
        "Received Housing & Financial Services"
    ),
    "received_service_jobs_employment": "Received Jobs & Employment Services",
    "DaysEnrolled": "Days Enrolled in Program (100s)",
    "Earned_Entry": "Earned Monthly Income ($1,000s)",
    "Total_Lifetime_Enrollments": "Total Lifetime Enrollments",
    "Year_Trend": "Annual Linear Trend (Years Post-2017)",
    "Year_Trend:SPP": "Annual Trend × SPP Interaction",
    "SPP:Year_Trend": "Annual Trend × SPP Interaction",
}


def clean_term_label(term):
  """Remaps raw GEE model terms into publishable labels."""
  if term in VAR_LABEL_MAP:
    return VAR_LABEL_MAP[term]
  gee_match = re.search(
      r"C\(CalendarYear,\s*Treatment\(reference=2017\)\)\[T\.(\d+)\]", term
  )
  if gee_match:
    return f"Calendar Year {gee_match.group(1)} (vs. 2017)"
  return term


def get_significance_asterisks(p):
  """Returns significance asterisks: ** p < 0.01, * p < 0.05."""
  if np.isnan(p):
    return ""
  if p < 0.01:
    return "**"
  elif p < 0.05:
    return "*"
  return ""


# Controls list for dynamic variance checking
CONTROL_LIST = [
    "Woman",
    "LGBTQ",
    "Family",
    "VeteranStatus",
    "BlackAfAmerican",
    "Age",
    "Employed_Entry",
    "ChronicHomelessness",
    "DVSurvivor_Entry",
    "DisablingCondition",
    "UnableToWork_Entry",
    "Prior_Unsheltered",
    "received_service_housing_financial",
    "received_service_jobs_employment",
    "DaysEnrolled",
    "Earned_Entry",
    "Total_Lifetime_Enrollments",
]


def run_appendix_iii_gee(df_first_clean, df_repeat_clean):
  print(
      "\n>>> Running Technical Appendix III: Generalized Estimating Equations"
      " (GEE)..."
  )

  results = []
  for df, label in [
      (df_first_clean, "First-Time Cohort"),
      (df_repeat_clean, "Repeat Cohort"),
  ]:
    print(f"Fitting model for {label}...")
    df_gee = df.copy()
    df_gee["CalendarYear"] = df_gee["CalendarYear"].astype(int)

    # Filter out invariant variables dynamically
    valid_controls = [
        c
        for c in CONTROL_LIST
        if c in df_gee.columns and df_gee[c].nunique() > 1
    ]
    controls_str = " + ".join(valid_controls)

    gee_formula = f"Exit_Permanent ~ C(CalendarYear, Treatment(reference=2017)) + SPP + {controls_str}"

    # Cluster-Robust Logit Estimation (Independence GEE with Sandwich Covariance)
    model = smf.logit(formula=gee_formula, data=df_gee).fit(
        cov_type="cluster",
        cov_kwds={"groups": df_gee["ProjectID"]},
        disp=False,
    )

    conf = model.conf_int()
    res_df = pd.DataFrame({
        "Raw_Term": model.params.index,
        "Coef": model.params.values,
        "Robust_SE": model.bse.values,
        "aOR": np.exp(model.params.values),
        "CI_Lower": np.exp(conf[0].values),
        "CI_Upper": np.exp(conf[1].values),
        "p_value": model.pvalues.values,
        "Cohort": label,
    })
    results.append(res_df)

  gee_all = pd.concat(results, ignore_index=True)
  gee_all["Variable / Predictor"] = gee_all["Raw_Term"].apply(clean_term_label)
  gee_all["Asterisks"] = gee_all["p_value"].apply(get_significance_asterisks)

  # Format Markdown Table Output
  md_df = gee_all[[
      "Cohort",
      "Variable / Predictor",
      "Coef",
      "Robust_SE",
      "aOR",
      "CI_Lower",
      "CI_Upper",
      "p_value",
      "Asterisks",
  ]].copy()

  def format_p_cell(row):
    p = row["p_value"]
    ast = row["Asterisks"]
    if np.isnan(p):
      return "nan"
    if p < 0.001:
      return f"< 0.001 {ast}".strip()
    return f"{p:.4f} {ast}".strip()

  md_df["p_value_formatted"] = md_df.apply(format_p_cell, axis=1)
  md_df["aOR [95% CI]"] = md_df.apply(
      lambda r: f"{r['aOR']:.3f} [{r['CI_Lower']:.3f}, {r['CI_Upper']:.3f}]",
      axis=1,
  )

  export_table = md_df[[
      "Cohort",
      "Variable / Predictor",
      "Coef",
      "Robust_SE",
      "aOR [95% CI]",
      "p_value_formatted",
  ]].rename(columns={"p_value_formatted": "p_value"})

  md_path = os.path.join(OUTPUT_DIR, "III_Technical_Appendix_GEE.md")
  with open(md_path, "w") as f:
    f.write(
        "# Technical Appendix III: Generalized Estimating Equations (GEE) Exit"
        " Odds\n\n"
    )
    f.write(
        "**Specification:** Binomial GEE Logit | **Working Correlation:**"
        " Robust Cluster-Sandwich | **Reference Year:** 2017 | **Clustering:**"
        " `ProjectID`\n\n"
    )
    f.write(export_table.to_markdown(index=False))
    f.write(
        "\n\n*Significance levels: ** 99% (p < 0.01) & * 95% (p < 0.05)*\n"
    )

  print(f"[Appendix III Markdown Exported] {md_path}")

  # Forest Plot Generation
  plot_df = (
      gee_all[~gee_all["Raw_Term"].isin(["Intercept"])]
      .copy()
      .iloc[::-1]
      .reset_index(drop=True)
  )

  fig, ax = plt.subplots(figsize=(9, 10), dpi=600)

  first_sub = plot_df[plot_df["Cohort"] == "First-Time Cohort"].reset_index(
      drop=True
  )
  repeat_sub = plot_df[plot_df["Cohort"] == "Repeat Cohort"].reset_index(
      drop=True
  )

  merged_plot = pd.merge(
      repeat_sub[[
          "Variable / Predictor",
          "aOR",
          "CI_Lower",
          "CI_Upper",
          "Asterisks",
      ]],
      first_sub[[
          "Variable / Predictor",
          "aOR",
          "CI_Lower",
          "CI_Upper",
          "Asterisks",
      ]],
      on="Variable / Predictor",
      how="left",
      suffixes=("_repeat", "_first"),
  )

  y_pos = np.arange(len(merged_plot))

  # Null Effect Benchmark Line
  ax.axvline(
      1.0, color="#D9381E", linestyle="--", linewidth=1.2, label="Null Effect (aOR = 1.0)"
  )

  # First-Time Cohort: CPP PRIMARY GREEN (#00843D)
  valid_first = merged_plot.dropna(subset=["aOR_first"])
  y_pos_first = y_pos[merged_plot["aOR_first"].notna()] + 0.15

  ax.errorbar(
      valid_first["aOR_first"],
      y_pos_first,
      xerr=[
          valid_first["aOR_first"] - valid_first["CI_Lower_first"],
          valid_first["CI_Upper_first"] - valid_first["aOR_first"],
      ],
      fmt="o",
      color=COLOR_PRIMARY_GREEN,
      ecolor=COLOR_PRIMARY_GREEN,
      capsize=3,
      markersize=5,
      label="First-Time Cohort (df_first)",
  )

  # Place asterisks next to First-Time lines on plot
  for _, row in valid_first.iterrows():
    if row["Asterisks_first"]:
      idx = merged_plot[
          merged_plot["Variable / Predictor"] == row["Variable / Predictor"]
      ].index[0]
      x_val = row["CI_Upper_first"] * 1.04
      y_val = y_pos[idx] + 0.15
      ax.text(
          x_val,
          y_val,
          row["Asterisks_first"],
          color=COLOR_PRIMARY_GREEN,
          va="center",
          ha="left",
          fontsize=8,
          fontweight="bold",
      )

  # Repeat Cohort: CPP GOLD (#FFB500)
  y_pos_repeat = y_pos - 0.15
  ax.errorbar(
      merged_plot["aOR_repeat"],
      y_pos_repeat,
      xerr=[
          merged_plot["aOR_repeat"] - merged_plot["CI_Lower_repeat"],
          merged_plot["CI_Upper_repeat"] - merged_plot["aOR_repeat"],
      ],
      fmt="s",
      color=COLOR_CPP_GOLD,
      ecolor=COLOR_CPP_GOLD,
      capsize=3,
      markersize=5,
      label="Repeat Cohort (df_repeat)",
  )

  # Place asterisks next to Repeat lines on plot
  for idx, row in merged_plot.iterrows():
    if row["Asterisks_repeat"]:
      x_val = row["CI_Upper_repeat"] * 1.04
      y_val = y_pos[idx] - 0.15
      ax.text(
          x_val,
          y_val,
          row["Asterisks_repeat"],
          color="#B38000",
          va="center",
          ha="left",
          fontsize=8,
          fontweight="bold",
      )

  ax.set_yticks(y_pos)
  ax.set_yticklabels(
      merged_plot["Variable / Predictor"], fontweight="bold", fontsize=9
  )
  ax.set_xlabel("Adjusted Odds Ratio (aOR, Log Scale)", fontweight="bold")
  ax.set_xscale("log")
  ax.set_title(
      "Technical Appendix III: Generalized Estimating Equations (GEE)\nAnnual"
      " Exit Odds Relative to 2017 Baseline (Clustered by ProjectID)",
      fontweight="bold",
      pad=12,
  )
  ax.grid(True, linestyle=":", alpha=0.5, which="both")

  # Place legend in lower left to avoid data overlap
  ax.legend(loc="lower left", frameon=True, facecolor="white", framealpha=0.9)

  plt.tight_layout()
  fig_path = os.path.join(OUTPUT_DIR, "III_Appendix_GEE_Forest_Plot.jpg")
  plt.savefig(fig_path, dpi=600, format="jpg", bbox_inches="tight")
  plt.close()

  print(f"[Appendix III Forest Plot Exported] {fig_path}")
  return gee_all


# Execute
gee_results = run_appendix_iii_gee(df_first_clean, df_repeat_clean)
```


```python
# checking variance to fix GEE errors
print(f"total rows:{len(df_first_clean):,}")
print(f"years persent: {sorted(df_first_clean['CalendarYear'].unique())}")
print(f"2017 exit_permanent count:\n{df_first_clean[df_first_clean['CalendarYear']==2017]['Exit_Permanent'].value_counts()}\n")

for col in CONTROL_LIST + ["SPP", "Exit_Permanent"]:
    if col in df_first_clean.columns:
        n_unq = df_first_clean[col].nunique()
        if n_unq <= 1:
            print(f"Zero variance detected: '{col}' has only {n_unq} value(s)!")
```

# Technical Appendix III: Generalized Estimating Equations (GEE) Exit Odds

**Specification:** Binomial GEE Logit | **Working Correlation:** Robust Cluster-Sandwich | **Reference Year:** 2017 | **Clustering:** `ProjectID`

| Cohort            | Variable / Predictor                  |        Coef |   Robust SE | aOR [95% CI]         | p-value    |
|:------------------|:--------------------------------------|------------:|------------:|:---------------------|:-----------|
| First-Time Cohort | Constant / Intercept                  | -0.237104   |  0.216579   | 0.789 [0.516, 1.206] | 0.2736     |
| First-Time Cohort | Calendar Year 2018 (vs. 2017)         | -0.0715866  |  0.146576   | 0.931 [0.698, 1.241] | 0.6253     |
| First-Time Cohort | Calendar Year 2019 (vs. 2017)         | -0.295884   |  0.167596   | 0.744 [0.536, 1.033] | 0.0775     |
| First-Time Cohort | Calendar Year 2020 (vs. 2017)         | -0.556508   |  0.175068   | 0.573 [0.407, 0.808] | 0.0015 **  |
| First-Time Cohort | Calendar Year 2021 (vs. 2017)         | -0.475592   |  0.18036    | 0.622 [0.436, 0.885] | 0.0084 **  |
| First-Time Cohort | Calendar Year 2022 (vs. 2017)         | -0.67786    |  0.173984   | 0.508 [0.361, 0.714] | < 0.001 ** |
| First-Time Cohort | Calendar Year 2023 (vs. 2017)         | -0.620169   |  0.179558   | 0.538 [0.378, 0.765] | < 0.001 ** |
| First-Time Cohort | Calendar Year 2024 (vs. 2017)         | -1.12219    |  0.189519   | 0.326 [0.225, 0.472] | < 0.001 ** |
| First-Time Cohort | Calendar Year 2025 (vs. 2017)         | -2.29554    |  0.345181   | 0.101 [0.051, 0.198] | < 0.001 ** |
| First-Time Cohort | Safe Parking Program (SPP Baseline)   | -0.409362   |  0.141442   | 0.664 [0.503, 0.876] | 0.0038 **  |
| First-Time Cohort | Woman                                 |  0.20575    |  0.0484397  | 1.228 [1.117, 1.351] | < 0.001 ** |
| First-Time Cohort | LGBTQ+                                | -0.0482494  |  0.148774   | 0.953 [0.712, 1.276] | 0.7457     |
| First-Time Cohort | Family Household                      |  0.76175    |  0.109562   | 2.142 [1.728, 2.655] | < 0.001 ** |
| First-Time Cohort | Veteran                               |  0.0466058  |  0.0313383  | 1.048 [0.985, 1.114] | 0.1370     |
| First-Time Cohort | Black / African American              |  0.330337   |  0.0417998  | 1.391 [1.282, 1.510] | < 0.001 ** |
| First-Time Cohort | Age (Years)                           |  0.00649114 |  0.00274621 | 1.007 [1.001, 1.012] | 0.0181 *   |
| First-Time Cohort | Employed at Entry                     |  0.442386   |  0.0668462  | 1.556 [1.365, 1.774] | < 0.001 ** |
| First-Time Cohort | Chronically Unhoused                  |  0.026404   |  0.0474863  | 1.027 [0.936, 1.127] | 0.5782     |
| First-Time Cohort | Domestic Violence Survivor            |  0.0610089  |  0.0355009  | 1.063 [0.991, 1.139] | 0.0857     |
| First-Time Cohort | Disabling Condition                   |  0.00388688 |  0.0450901  | 1.004 [0.919, 1.097] | 0.9313     |
| First-Time Cohort | Unable to Work                        |  0.0830464  |  0.0386888  | 1.087 [1.007, 1.172] | 0.0318 *   |
| First-Time Cohort | Prior Location: Unsheltered           | -0.272187   |  0.0555389  | 0.762 [0.683, 0.849] | < 0.001 ** |
| First-Time Cohort | Received Housing & Financial Services |  0.687805   |  0.116821   | 1.989 [1.582, 2.501] | < 0.001 ** |
| First-Time Cohort | Received Jobs & Employment Services   | -0.228701   |  0.155334   | 0.796 [0.587, 1.079] | 0.1409     |
| First-Time Cohort | Days Enrolled in Program (100s)       |  0.231105   |  0.0169791  | 1.260 [1.219, 1.303] | < 0.001 ** |
| First-Time Cohort | Earned Monthly Income (\\$1,000s)       |  0.0779676  |  0.0395124  | 1.081 [1.001, 1.168] | 0.0485 *   |
| First-Time Cohort | Total Lifetime Enrollments            | -0.845399   |  0.0669716  | 0.429 [0.377, 0.490] | < 0.001 ** |
| Repeat Cohort     | Constant / Intercept                  | -1.93925    |  0.254217   | 0.144 [0.087, 0.237] | < 0.001 ** |
| Repeat Cohort     | Calendar Year 2018 (vs. 2017)         |  0.221084   |  0.20862    | 1.247 [0.829, 1.878] | 0.2893     |
| Repeat Cohort     | Calendar Year 2019 (vs. 2017)         |  0.028226   |  0.19994    | 1.029 [0.695, 1.522] | 0.8877     |
| Repeat Cohort     | Calendar Year 2020 (vs. 2017)         | -0.147227   |  0.207063   | 0.863 [0.575, 1.295] | 0.4771     |
| Repeat Cohort     | Calendar Year 2021 (vs. 2017)         |  0.231018   |  0.207308   | 1.260 [0.839, 1.891] | 0.2651     |
| Repeat Cohort     | Calendar Year 2022 (vs. 2017)         |  0.0210477  |  0.19603    | 1.021 [0.695, 1.500] | 0.9145     |
| Repeat Cohort     | Calendar Year 2023 (vs. 2017)         |  0.125276   |  0.197128   | 1.133 [0.770, 1.668] | 0.5251     |
| Repeat Cohort     | Calendar Year 2024 (vs. 2017)         | -0.254027   |  0.201696   | 0.776 [0.522, 1.152] | 0.2079     |
| Repeat Cohort     | Calendar Year 2025 (vs. 2017)         | -0.90439    |  0.430629   | 0.405 [0.174, 0.941] | 0.0357 *   |
| Repeat Cohort     | Safe Parking Program (SPP Baseline)   | -0.543888   |  0.131859   | 0.580 [0.448, 0.752] | < 0.001 ** |
| Repeat Cohort     | Woman                                 |  0.161225   |  0.0416054  | 1.175 [1.083, 1.275] | < 0.001 ** |
| Repeat Cohort     | LGBTQ+                                |  0.0534199  |  0.0885708  | 1.055 [0.887, 1.255] | 0.5464     |
| Repeat Cohort     | Family Household                      |  0.564875   |  0.154112   | 1.759 [1.301, 2.380] | < 0.001 ** |
| Repeat Cohort     | Veteran                               |  0.0534402  |  0.028293   | 1.055 [0.998, 1.115] | 0.0589     |
| Repeat Cohort     | Black / African American              |  0.325735   |  0.0386279  | 1.385 [1.284, 1.494] | < 0.001 ** |
| Repeat Cohort     | Age (Years)                           |  0.00893337 |  0.00250528 | 1.009 [1.004, 1.014] | < 0.001 ** |
| Repeat Cohort     | Employed at Entry                     |  0.274047   |  0.0625782  | 1.315 [1.163, 1.487] | < 0.001 ** |
| Repeat Cohort     | Chronically Unhoused                  |  0.0475371  |  0.0452684  | 1.049 [0.960, 1.146] | 0.2937     |
| Repeat Cohort     | Domestic Violence Survivor            |  0.111566   |  0.0319983  | 1.118 [1.050, 1.190] | < 0.001 ** |
| Repeat Cohort     | Disabling Condition                   |  0.129499   |  0.0461888  | 1.138 [1.040, 1.246] | 0.0051 **  |
| Repeat Cohort     | Unable to Work                        |  0.045646   |  0.0349571  | 1.047 [0.977, 1.121] | 0.1916     |
| Repeat Cohort     | Prior Location: Unsheltered           | -0.43944    |  0.0517362  | 0.644 [0.582, 0.713] | < 0.001 ** |
| Repeat Cohort     | Received Housing & Financial Services |  0.515907   |  0.143655   | 1.675 [1.264, 2.220] | < 0.001 ** |
| Repeat Cohort     | Received Jobs & Employment Services   |  0.0697748  |  0.190852   | 1.072 [0.738, 1.559] | 0.7147     |
| Repeat Cohort     | Days Enrolled in Program (100s)       |  0.251779   |  0.0163028  | 1.286 [1.246, 1.328] | < 0.001 ** |
| Repeat Cohort     | Earned Monthly Income (\\$1,000s)       |  0.195663   |  0.0307754  | 1.216 [1.145, 1.292] | < 0.001 ** |
| Repeat Cohort     | Total Lifetime Enrollments            | -0.14559    |  0.0103867  | 0.865 [0.847, 0.882] | < 0.001 ** |

*`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*


## Technical Appendix IV
### Continuous Average Macro Trends


```python
OUTPUT_DIR = "FinalHousingStudies"
os.makedirs(OUTPUT_DIR, exist_ok=True)

VAR_LABEL_MAP = {
    "Intercept": "Constant / Intercept",
    "SPP": "Safe Parking Program (SPP Baseline)",
    "Year_Trend": "Annual Linear Trend (Years Post-2017)",
    "Year_Trend:SPP": "Annual Trend × SPP Interaction",
    "SPP:Year_Trend": "Annual Trend × SPP Interaction",
    "Woman": "Woman",
    "LGBTQ": "LGBTQ+",
    "Family": "Family Household",
    "VeteranStatus": "Veteran",
    "BlackAfAmerican": "Black / African American",
    "Age": "Age (Years)",
    "Employed_Entry": "Employed at Entry",
    "ChronicHomelessness": "Chronically Unhoused",
    "DVSurvivor_Entry": "Domestic Violence Survivor",
    "DisablingCondition": "Disabling Condition",
    "UnableToWork_Entry": "Unable to Work",
    "Prior_Unsheltered": "Prior Location: Unsheltered",
    "received_service_housing_financial": (
        "Received Housing & Financial Services"
    ),
    "received_service_jobs_employment": "Received Jobs & Employment Services",
    "DaysEnrolled": "Days Enrolled in Program (100s)",
    "Earned_Entry": "Earned Monthly Income ($1,000s)",
    "Total_Lifetime_Enrollments": "Total Lifetime Enrollments",
}

CONTROL_LIST = [
    "Woman",
    "LGBTQ",
    "Family",
    "VeteranStatus",
    "BlackAfAmerican",
    "Age",
    "Employed_Entry",
    "ChronicHomelessness",
    "DVSurvivor_Entry",
    "DisablingCondition",
    "UnableToWork_Entry",
    "Prior_Unsheltered",
    "received_service_housing_financial",
    "received_service_jobs_employment",
    "DaysEnrolled",
    "Earned_Entry",
    "Total_Lifetime_Enrollments",
]


def clean_term_label(term):
  return VAR_LABEL_MAP.get(term, term)


def get_significance_asterisks(p):
  if pd.isna(p):
    return ""
  if p < 0.01:
    return "**"
  elif p < 0.05:
    return "*"
  return ""


def format_p_value(p):
  if pd.isna(p):
    return "N/A"
  if p < 0.001:
    return "< 0.001"
  return f"{p:.4f}"


def run_appendix_iv_trends(df_first_clean, df_repeat_clean):
  print(
      "\n>>> Running Technical Appendix IV: Continuous Average Macro"
      " Trends..."
  )
  results = []

  for df, label in [
      (df_first_clean, "First-Time Cohort"),
      (df_repeat_clean, "Repeat Cohort"),
  ]:
    df_tr = df.copy()
    df_tr["Year_Trend"] = df_tr["CalendarYear"].astype(int) - 2017

    # Dynamic check to exclude zero-variance variables
    valid_controls = [
        c for c in CONTROL_LIST if c in df_tr.columns and df_tr[c].nunique() > 1
    ]
    controls_str = " + ".join(valid_controls)

    trend_formula = f"Exit_Permanent ~ Year_Trend * SPP + {controls_str}"

    model = smf.logit(formula=trend_formula, data=df_tr).fit(
        cov_type="cluster",
        cov_kwds={"groups": df_tr["ProjectID"]},
        disp=False,
    )

    conf = model.conf_int()
    res_df = pd.DataFrame({
        "Raw_Term": model.params.index,
        "Coef": model.params.values,
        "Robust_SE": model.bse.values,
        "aOR": np.exp(model.params.values),
        "CI_Lower": np.exp(conf[0].values),
        "CI_Upper": np.exp(conf[1].values),
        "p_value": model.pvalues.values,
        "Cohort": label,
    })
    results.append(res_df)

  tr_all = pd.concat(results, ignore_index=True)
  tr_all["Variable / Predictor"] = tr_all["Raw_Term"].apply(clean_term_label)

  # Markdown export with separate Sig. column
  md_df = tr_all[[
      "Cohort",
      "Variable / Predictor",
      "Coef",
      "Robust_SE",
      "aOR",
      "CI_Lower",
      "CI_Upper",
      "p_value",
  ]].copy()

  md_df["p_value_clean"] = md_df["p_value"].apply(format_p_value)
  md_df["Sig."] = md_df["p_value"].apply(get_significance_asterisks)
  md_df["aOR [95% CI]"] = md_df.apply(
      lambda r: f"{r['aOR']:.3f} [{r['CI_Lower']:.3f}, {r['CI_Upper']:.3f}]",
      axis=1,
  )

  export_table = md_df[[
      "Cohort",
      "Variable / Predictor",
      "Coef",
      "Robust_SE",
      "aOR [95% CI]",
      "p_value_clean",
      "Sig.",
  ]].rename(columns={"p_value_clean": "p_value"})

  md_path = os.path.join(OUTPUT_DIR, "IV_Technical_Appendix_Macro_Trends.md")
  with open(md_path, "w", encoding="utf-8") as f:
    f.write("# Technical Appendix IV: Continuous Average Macro Trends\n\n")
    f.write(
        "**Specification:** Binomial Logit | **Trend Variable:** Year - 2017 |"
        " **Clustering:** `ProjectID`\n\n"
    )
    f.write(export_table.to_markdown(index=False))
    f.write(
        "\n\n*`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*\n"
    )

  print(f"[Appendix IV Markdown Exported] {md_path}")
  return tr_all


# Run estimation
app4_results = run_appendix_iv_trends(df_first_clean, df_repeat_clean)
```

## Technical Appendix V
### Average Marginal Effects (AME)


```python
# Directory Setup
OUTPUT_DIR = "FinalHousingStudies"
os.makedirs(OUTPUT_DIR, exist_ok=True)

# Master Variable Label Mapping Dictionary
VAR_LABEL_MAP = {
    "Intercept": "Constant / Intercept",
    "SPP": "Safe Parking Program (SPP Baseline)",
    "Post2022": "Enrolled Post-2022 Era",
    "SPP:Post2022": "SPP × Post-2022 Era (DiD Estimator)",
    "Post2022:SPP": "SPP × Post-2022 Era (DiD Estimator)",
    "Woman": "Woman",
    "LGBTQ": "LGBTQ+",
    "Family": "Family Household",
    "SingleParentWChild": "Single Parent Household with Child",
    "VeteranStatus": "Veteran",
    "White": "White",
    "BlackAfAmerican": "Black / African American",
    "Age": "Age (Years)",
    "Employed_Entry": "Employed at Entry",
    "ChronicHomelessness": "Chronically Unhoused",
    "DVSurvivor_Entry": "Domestic Violence Survivor",
    "DisablingCondition": "Disabling Condition",
    "UnableToWork_Entry": "Unable to Work",
    "Prior_Unsheltered": "Prior Location: Unsheltered",
    "received_service_housing_financial": (
        "Received Housing & Financial Services"
    ),
    "received_service_jobs_employment": "Received Jobs & Employment Services",
    "DaysEnrolled": "Days Enrolled in Program (100s)",
    "Earned_Entry": "Earned Monthly Income ($1,000s)",
    "Total_Lifetime_Enrollments": "Total Lifetime Enrollments",
}

CONTROL_LIST = [
    "Woman",
    "LGBTQ",
    "Family",
    "VeteranStatus",
    "BlackAfAmerican",
    "Age",
    "Employed_Entry",
    "ChronicHomelessness",
    "DVSurvivor_Entry",
    "DisablingCondition",
    "UnableToWork_Entry",
    "Prior_Unsheltered",
    "received_service_housing_financial",
    "received_service_jobs_employment",
    "DaysEnrolled",
    "Earned_Entry",
    "Total_Lifetime_Enrollments",
]


def clean_term_label(term):
  """Remaps raw model term strings into exact academic labels matching the LPM DiD."""
  return VAR_LABEL_MAP.get(term, term)


def get_significance_asterisks(p):
  """Returns significance asterisks: ** p < 0.01, * p < 0.05."""
  if pd.isna(p):
    return ""
  if p < 0.01:
    return "**"
  elif p < 0.05:
    return "*"
  return ""


def format_p_value(p):
  """Formats clean numeric p-value string without embedded asterisks."""
  if pd.isna(p):
    return "N/A"
  if p < 0.001:
    return "< 0.001"
  return f"{p:.4f}"


def run_appendix_v_ame(df_first_clean, df_repeat_clean):
  print(
      "\n>>> Running Technical Appendix V: Average Marginal Effects (AME)..."
  )
  results = []

  for df, label in [
      (df_first_clean, "First-Time Cohort"),
      (df_repeat_clean, "Repeat Cohort"),
  ]:
    df_ame = df.copy()

    # Dynamic variance check to drop invariant variables (e.g., Total_Lifetime_Enrollments in first-time)
    valid_controls = [
        c
        for c in CONTROL_LIST
        if c in df_ame.columns and df_ame[c].nunique() > 1
    ]
    controls_str = " + ".join(valid_controls)

    logit_formula = f"Exit_Permanent ~ SPP * Post2022 + {controls_str}"

    model = smf.logit(formula=logit_formula, data=df_ame).fit(
        cov_type="cluster",
        cov_kwds={"groups": df_ame["ProjectID"]},
        disp=False,
    )

    # Compute overall Average Marginal Effects
    margeff = model.get_margeff(at="overall")

    # Array-based extraction to prevent AttributeError (stderr) and KeyError (conf_int_low)
    conf_bounds = margeff.conf_int()  # Returns Nx2 numpy array

    res_df = pd.DataFrame({
        "Raw_Term": margeff.summary_frame().index,
        "AME_pp": margeff.margeff * 100,  # Expressed in Percentage Points (Δpp)
        "SE_pp": (
            margeff.margeff_se * 100
        ),  # Standard error array (margeff_se)
        "p_value": margeff.pvalues,  # p-values array
        "CI_Lower_pp": conf_bounds[:, 0] * 100,  # 95% CI Lower bound
        "CI_Upper_pp": conf_bounds[:, 1] * 100,  # 95% CI Upper bound
        "Cohort": label,
    })
    results.append(res_df)

  ame_all = pd.concat(results, ignore_index=True)
  ame_all["Variable / Predictor"] = ame_all["Raw_Term"].apply(clean_term_label)

  # Markdown export formatting with dedicated Sig. column
  md_df = ame_all[[
      "Cohort",
      "Variable / Predictor",
      "AME_pp",
      "SE_pp",
      "CI_Lower_pp",
      "CI_Upper_pp",
      "p_value",
  ]].copy()

  md_df["p_value_clean"] = md_df["p_value"].apply(format_p_value)
  md_df["Sig."] = md_df["p_value"].apply(get_significance_asterisks)
  md_df["AME [95% CI] (pp)"] = md_df.apply(
      lambda r: (
          f"{r['AME_pp']:+.2f}pp [{r['CI_Lower_pp']:+.2f},"
          f" {r['CI_Upper_pp']:+.2f}]"
      ),
      axis=1,
  )

  export_table = md_df[[
      "Cohort",
      "Variable / Predictor",
      "AME [95% CI] (pp)",
      "SE_pp",
      "p_value_clean",
      "Sig.",
  ]].rename(columns={"p_value_clean": "p_value"})

  md_path = os.path.join(OUTPUT_DIR, "V_Technical_Appendix_AME.md")
  with open(md_path, "w", encoding="utf-8") as f:
    f.write(
        "# Technical Appendix V: Binomial Logit Average Marginal Effects"
        " (AME)\n\n"
    )
    f.write(
        "**Methodology:** Overall Average Marginal Effects expressed in"
        " Percentage Points (Δpp) | **Clustering:** `ProjectID`\n\n"
    )
    f.write(export_table.to_markdown(index=False))
    f.write(
        "\n\n*`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*\n"
    )

  print(f"[Appendix V Markdown Exported] {md_path}")
  return ame_all


# Execute Appendix V
app5_results = run_appendix_v_ame(df_first_clean, df_repeat_clean)
```

## Technical Appendix VI
### Dynamic Event-Study Trajectories


```python
# ensure directory export
OUTPUT_DIR = "FinalHousingStudies"
os.makedirs(OUTPUT_DIR, exist_ok=True)

# publication specifications
plt.rcParams.update({
    "font.family": "serif",
    "font.serif": ["Times New Roman", "DejaVu Serif"],
    "font.size": 10,
    "axes.labelsize": 10.5,
    "axes.titlesize": 11.5,
})

# same color palette
COLOR_PRIMARY_GREEN = "#00843D"  # First-Time Cohort
COLOR_CPP_GOLD = "#FFB500"  # Repeat Cohort
COLOR_NULL_RED = "#D9381E"  # Vertical Policy Cutoff Line

CONTROL_LIST = [
    "Woman",
    "LGBTQ",
    "Family",
    "VeteranStatus",
    "BlackAfAmerican",
    "Age",
    "Employed_Entry",
    "ChronicHomelessness",
    "DVSurvivor_Entry",
    "DisablingCondition",
    "UnableToWork_Entry",
    "Prior_Unsheltered",
    "received_service_housing_financial",
    "received_service_jobs_employment",
    "DaysEnrolled",
    "Earned_Entry",
    "Total_Lifetime_Enrollments",
]


def clean_term_label(term):
  """Remaps dynamic event-study interaction terms to publication labels."""
  es_match = re.search(
      r"C\(CalendarYear,\s*Treatment\(reference=2021\)\)\[T\.(\d+)\]:SPP", term
  )
  if not es_match:
    es_match = re.search(
        r"SPP:C\(CalendarYear,\s*Treatment\(reference=2021\)\)\[T\.(\d+)\]",
        term,
    )
  if es_match:
    return f"Year {es_match.group(1)} × SPP Policy Lift (vs. 2021)"
  return term


def get_significance_asterisks(p):
  if pd.isna(p):
    return ""
  if p < 0.01:
    return "**"
  elif p < 0.05:
    return "*"
  return ""


def format_p_value(p):
  if pd.isna(p):
    return "N/A"
  if p < 0.001:
    return "< 0.001"
  return f"{p:.4f}"


def run_appendix_vi_event_study(df_first_clean, df_repeat_clean):
  print(
      "\n>>> Running Technical Appendix VI: Dynamic Event-Study (DiD)"
      " Trajectories..."
  )

  es_data = {}
  results = []

  for df, label in [
      (df_first_clean, "First-Time Cohort"),
      (df_repeat_clean, "Repeat Cohort"),
  ]:
    df_es = df.copy()
    df_es["CalendarYear"] = df_es["CalendarYear"].astype(int)

    # Dynamic variance check for controls
    valid_controls = [
        c for c in CONTROL_LIST if c in df_es.columns and df_es[c].nunique() > 1
    ]
    controls_str = " + ".join(valid_controls)

    es_formula = f"Exit_Permanent ~ C(CalendarYear, Treatment(reference=2021)) * SPP + {controls_str}"

    model = smf.ols(formula=es_formula, data=df_es).fit(
        cov_type="cluster", cov_kwds={"groups": df_es["ProjectID"]}
    )

    # Filter interaction terms (Year x SPP)
    terms = [
        t for t in model.params.index if "C(CalendarYear" in t and "SPP" in t
    ]
    conf = model.conf_int()

    res_df = pd.DataFrame({
        "Raw_Term": terms,
        "Coef_pp": model.params[terms].values * 100,  # Expressed in Δpp
        "SE_pp": model.bse[terms].values * 100,
        "CI_Lower_pp": conf.loc[terms, 0].values * 100,
        "CI_Upper_pp": conf.loc[terms, 1].values * 100,
        "p_value": model.pvalues[terms].values,
    })

    res_df["Year"] = (
        res_df["Raw_Term"].str.extract(r"\[T\.(\d+)\]")[0].astype(int)
    )

    # Insert normalized reference year (2021)
    ref_row = pd.DataFrame([{
        "Raw_Term": "Reference (2021)",
        "Coef_pp": 0.0,
        "SE_pp": 0.0,
        "CI_Lower_pp": 0.0,
        "CI_Upper_pp": 0.0,
        "p_value": 1.0,
        "Year": 2021,
    }])

    res_df = (
        pd.concat([res_df, ref_row], ignore_index=True)
        .sort_values("Year")
        .reset_index(drop=True)
    )
    res_df["Cohort"] = label
    es_data[label] = res_df
    results.append(res_df)

  es_all = pd.concat(results, ignore_index=True)
  es_all["Dynamic Interaction Term"] = es_all["Raw_Term"].apply(
      clean_term_label
  )

  # Markdown export formatting with separate Sig. column
  md_df = es_all[[
      "Cohort",
      "Year",
      "Dynamic Interaction Term",
      "Coef_pp",
      "SE_pp",
      "CI_Lower_pp",
      "CI_Upper_pp",
      "p_value",
  ]].copy()

  md_df["p_value_clean"] = md_df["p_value"].apply(format_p_value)
  md_df["Sig."] = md_df["p_value"].apply(get_significance_asterisks)
  md_df["Policy Lift [95% CI] (pp)"] = md_df.apply(
      lambda r: (
          f"{r['Coef_pp']:+.2f}pp [{r['CI_Lower_pp']:+.2f},"
          f" {r['CI_Upper_pp']:+.2f}]"
      ),
      axis=1,
  )

  export_table = md_df[[
      "Cohort",
      "Year",
      "Dynamic Interaction Term",
      "Policy Lift [95% CI] (pp)",
      "SE_pp",
      "p_value_clean",
      "Sig.",
  ]].rename(columns={"p_value_clean": "p_value"})

  md_path = os.path.join(
      OUTPUT_DIR, "VI_Technical_Appendix_Event_Study_Results.md"
  )
  with open(md_path, "w", encoding="utf-8") as f:
    f.write(
        "# Technical Appendix VI: Dynamic DiD Event-Study Trajectories"
        " (2017–2025)\n\n"
    )
    f.write(
        "**Pre-Shift Reference Year ($t_0$):** 2021 | **Clustering:**"
        " `ProjectID`\n\n"
    )
    f.write(export_table.to_markdown(index=False))
    f.write(
        "\n\n*Significance levels: ** 99% (p < 0.01) & * 95% (p < 0.05)*\n"
    )

  print(f"[Appendix VI Markdown Exported] {md_path}")

  # 600 DPI Two-Panel High-Resolution Publication Plot
  fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5.5), dpi=600, sharey=True)

  cohort_configs = [
      (
          ax1,
          es_data["First-Time Cohort"],
          "First-Time Enrollments (df_first)",
          COLOR_PRIMARY_GREEN,
      ),
      (
          ax2,
          es_data["Repeat Cohort"],
          "Repeat Enrollments (df_repeat)",
          COLOR_CPP_GOLD,
      ),
  ]

  for ax, df_sub, title, color_hex in cohort_configs:
    years = df_sub["Year"].values
    coefs = df_sub["Coef_pp"].values
    yerr = [
        coefs - df_sub["CI_Lower_pp"].values,
        df_sub["CI_Upper_pp"].values - coefs,
    ]

    ax.axhline(0, color="black", linestyle="--", linewidth=1.0)
    ax.axvline(
        2021.5,
        color=COLOR_NULL_RED,
        linestyle=":",
        linewidth=1.2,
        label="2022 Policy Expansion Cutoff",
    )

    ax.errorbar(
        years,
        coefs,
        yerr=yerr,
        fmt="o-",
        color=color_hex,
        ecolor=color_hex,
        capsize=3,
        linewidth=1.8,
        label="Policy Lift (Δpp vs. 2021 Baseline)",
    )

    # Significance annotations on chart points
    for _, r in df_sub.iterrows():
      ast = get_significance_asterisks(r["p_value"])
      if ast and r["Year"] != 2021:
        y_pos = (
            r["CI_Upper_pp"] + 1.2 if r["Coef_pp"] >= 0 else r["CI_Lower_pp"] - 2.8
        )
        ax.text(
            r["Year"],
            y_pos,
            ast,
            color=color_hex,
            ha="center",
            va="center",
            fontsize=9,
            fontweight="bold",
        )

    ax.set_title(
        f"Dynamic Policy Trajectory: {title}", fontweight="bold", pad=10
    )
    ax.set_xlabel("Calendar Year", fontweight="bold")
    ax.set_xticks(range(2017, 2026))
    ax.grid(True, linestyle=":", alpha=0.6)
    ax.legend(loc="lower left", frameon=True, facecolor="white")

  ax1.set_ylabel(
      "Dynamic DiD Treatment Lift (Percentage Points)", fontweight="bold"
  )
  #fig.suptitle(
      "Technical Appendix VI: Dynamic Event-Study Policy Trajectories"
      " (2017–2025)\nValidation of Parallel Pre-Trends (2017–2020) and Post-2022"
      " Policy Expansion Lift",
      fontsize=12,
      fontweight="bold",
      y=0.98,
  )
  plt.tight_layout(rect=[0, 0, 1, 0.94])

  fig_path = os.path.join(
      OUTPUT_DIR, "VI_Appendix_Dynamic_Event_Study_DiD.jpg"
  )
  plt.savefig(fig_path, dpi=600, format="jpg", bbox_inches="tight")
  plt.close()

  print(f"[Appendix VI Figure Exported] {fig_path}")
  return es_all


# Execute Appendix VI
app6_results = run_appendix_vi_event_study(df_first_clean, df_repeat_clean)
```

## Study complete! (Maybe!)
