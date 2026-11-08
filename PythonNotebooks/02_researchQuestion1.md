# Research Question 1: 
## How do Safe Parking Program (SPP) and non-SPP client characteristics differ? How did these characteristics shift after the CoC funding scandals and SPP expansions since 2022?

Policy context: Mayor Bass is elected in 2022. More investments in Safe Parking and Inside Safe. Scandals and defunding follow.


```python
# set chart styles and color consistency for all plots
plt.style.use('seaborn-v0_8-whitegrid' if 'seaborn-v0_8-whitegrid' in plt.style.available else 'default')
COLOR_GREEN = '#2E7D32'
COLOR_YELLOW = '#FBC02D'
COLOR_GREY = '#757575'
COLOR_RED = '#D32F2F'

```


```python
df_first_all.shape
```

## Descriptive statistics
### Uses `df_first_all` to capture all enrollments regardless of exits (e.g., program characteristics)


```python
# set up environment
output_dir = Path("FinalHousingStudies")
output_dir.mkdir(parents=True, exist_ok=True)

if "EntryDate" in df_first_all.columns:
    df_first_all["EntryDate"] = pd.to_datetime(df_first_all["EntryDate"])
    df_first_all["Post2022"] = (df_first_all["EntryDate"].dt.year >= 2022).astype(int)

variable_mapping = {
    "Woman": "Woman",
    "LGBTQ": "LGBTQ",
    "Family": "Family",
   # "SingleParentWChild": "Single Parent Household with Child",
    "VeteranStatus": "Veteran",
    #"White": "White",
    "BlackAfAmerican": "Black/African American",
    "Employed_Entry": "Employed",
    "ChronicHomelessness": "Chronically Unhoused",
    "DVSurvivor_Entry": "Domestic Violence Survivor",
    "DisablingCondition": "Disabling Condition",
    "UnableToWork_Entry": "Unable to Work",
    "Prior_Unsheltered": "Prior Location: Unsheltered",
    "received_service_housing_financial": "Received Housing and Financial Services",
    "received_service_jobs_employment": "Received Jobs and Employment Services",
    "Post2022": "Enrolled Post-2022 Era",
    "Age": "Age",
     "DaysEnrolled": "Days Enrolled (100s)",
    "Earned_Entry": "Earned Income ($1,000s)",
    "Total_Lifetime_Enrollments": "Total Lifetime Enrollments",
}

continuous_vars = [
    "Age",
    "Earned_Entry",
    "Total_Lifetime_Enrollments",
    "DaysEnrolled",
]

binary_vars = [v for v in variable_mapping.keys() if v not in continuous_vars]

descriptive_rows = []

# chi square test 
for var in binary_vars:
    clean_df = df_first_all[[var, "SPP"]].dropna()

    total_spp = (clean_df["SPP"] == 1).sum()
    total_non_spp = (clean_df["SPP"] == 0).sum()

    count_spp = clean_df[(clean_df["SPP"] == 1) & (clean_df[var] == 1)].shape[0]
    count_non_spp = clean_df[(clean_df["SPP"] == 0) & (clean_df[var] == 1)].shape[0]

    pct_spp = (count_spp / total_spp) * 100 if total_spp > 0 else 0
    pct_non_spp = (count_non_spp / total_non_spp) * 100 if total_non_spp > 0 else 0

    contingency = pd.crosstab(clean_df[var], clean_df["SPP"])

    if contingency.shape == (2, 2):
        _, p_val, _, _ = stats.chi2_contingency(contingency)
    else:
        p_val = 1.0

    if p_val < 0.01:
        sig_marker = "**"
    elif p_val < 0.05:
        sig_marker = "*"
    else:
        sig_marker = " "

    descriptive_rows.append(
        {
            "Variable": variable_mapping[var],
            "Non-SPP Shelter": f"{count_non_spp:,} ({pct_non_spp:.1f}%)",
            "Safe Parking (SPP)": f"{count_spp:,} ({pct_spp:.1f}%)",
            "Significance": sig_marker,
        }
    )

#continuous variable mapping t-test
for var in continuous_vars:
    clean_df = df_first_all[[var, "SPP"]].dropna()

    g1 = clean_df[clean_df["SPP"] == 1][var]
    g0 = clean_df[clean_df["SPP"] == 0][var]

    _, p_val = stats.mannwhitneyu(g1, g0, alternative="two-sided")

    if p_val < 0.01:
        sig_marker = "**"
    elif p_val < 0.05:
        sig_marker = "*"
    else:
        sig_marker = "ns"

    if var in ["Earned_Entry", "Total_Lifetime_Enrollments"]:
        mean_spp, std_spp = g1.mean(), g1.std()
        mean_non, std_non = g0.mean(), g0.std()

        if var == "Earned_Entry":
            label_non = f"${mean_non:,.2f} [±${std_non:,.2f}]"
            label_spp = f"${mean_spp:,.2f} [±${std_spp:,.2f}]"
        else:
            label_non = f"{mean_non:.2f} [±{std_non:.2f}]"
            label_spp = f"{mean_spp:.2f} [±{std_spp:.2f}]"

        descriptive_rows.append(
            {
                "Variable": f"{variable_mapping[var]} (Mean [± SD])",
                "Non-SPP Shelter": label_non,
                "Safe Parking (SPP)": label_spp,
                "Significance": sig_marker,
            }
        )
    else:
        med_spp = g1.median()
        q25_spp, q75_spp = np.percentile(g1, 25), np.percentile(g1, 75)

        med_non = g0.median()
        q25_non, q75_non = np.percentile(g0, 25), np.percentile(g0, 75)

        descriptive_rows.append(
            {
                "Variable": f"{variable_mapping[var]} (Median [IQR])",
                "Non-SPP Shelter": f"{med_non:.1f} [{q25_non:.1f} - {q75_non:.1f}]",
                "Safe Parking (SPP)": f"{med_spp:.1f} [{q25_spp:.1f} - {q75_spp:.1f}]",
                "Significance": sig_marker,
            }
        )

#create academic markdown 
df_table1 = pd.DataFrame(descriptive_rows)

n_spp = (df_first_all["SPP"] == 1).sum()
n_non_spp = (df_first_all["SPP"] == 0).sum()

col_non_spp = f"Non-SPP Shelter (N = {n_non_spp:,})"
col_spp = f"Safe Parking (SPP) (N = {n_spp:,})"

df_table1_export = df_table1.rename(
    columns={
        "Non-SPP Shelter": col_non_spp,
        "Safe Parking (SPP)": col_spp,
    }
)

output_path = output_dir / "Table1_DescriptiveStatistics_SPP.md"

with open(output_path, "w", encoding="utf-8") as f:
    f.write("# Table 1: Demographics and Acuity Types for Non-SPPs and SPPs\n\n")
    f.write(
        f"**Total Baseline Sample Size (N):** {len(df_first_all):,} (Non-SPP: N = {n_non_spp:,} | SPP: N = {n_spp:,})\n\n"
    )
    f.write(
        df_table1_export[
            ["Variable", col_non_spp, col_spp, "Significance"]
        ].to_markdown(index=False)
    )
    f.write(
        "\n\n*Significance: `*` = significant at the 95% confidence level (p < 0.05), `**` = significant at the 99% confidence level (p < 0.01). Earned Income and Total Lifetime Enrollments are reported as Mean [± SD] to show population distributions.*"
    )

print(f"Table 1 successfully compiled and exported to: {output_path}")
```

# Table 1: Demographics and Acuity Types for Non-SPPs and SPPs

**Total Baseline Sample Size (N):** 83,745 (Non-SPP: N = 79,768 | SPP: N = 3,977)

| Variable                                 | Non-SPP Shelter (N = 79,768)   | Safe Parking (SPP) (N = 3,977)   | Significance   |
|:-----------------------------------------|:-------------------------------|:---------------------------------|:---------------|
| Woman                                    | 32,823 (41.1%)                 | 1,798 (45.2%)                    | **             |
| LGBTQ                                    | 1,214 (1.5%)                   | 42 (1.1%)                        | *              |
| Family                                   | 6,312 (7.9%)                   | 269 (6.8%)                       | **             |
| Veteran                                  | 3,015 (3.9%)                   | 347 (8.9%)                       |                |
| Black/African American                   | 32,436 (40.7%)                 | 1,723 (43.3%)                    | **             |
| Employed                                 | 8,238 (10.3%)                  | 1,150 (28.9%)                    | **             |
| Chronically Unhoused                     | 15,515 (19.5%)                 | 698 (17.6%)                      | **             |
| Domestic Violence Survivor               | 17,129 (21.5%)                 | 868 (21.8%)                      |                |
| Disabling Condition                      | 36,408 (45.6%)                 | 1,683 (42.3%)                    | **             |
| Unable to Work                           | 30,346 (38.0%)                 | 1,206 (30.3%)                    | **             |
| Prior Location: Unsheltered              | 47,455 (59.5%)                 | 3,735 (93.9%)                    | **             |
| Received Housing and Financial Services  | 1,649 (2.1%)                   | 103 (2.6%)                       | *              |
| Received Jobs and Employment Services    | 437 (0.5%)                     | 29 (0.7%)                        |                |
| Enrolled Post-2022 Era                   | 34,743 (43.6%)                 | 1,804 (45.4%)                    | *              |
| Age (Median [IQR])                       | 46.0 [36.0 - 60.0]             | 52.0 [40.0 - 63.0]               | **             |
| Earned Income ($1,000s) (Mean [± SD])    | \\$0.10 [±\\$0.59]                 | \\$0.45 [±\\$4.76]                   | **             |
| Total Lifetime Enrollments (Mean [± SD]) | 1.66 [±1.22]                   | 1.73 [±1.26]                     | **             |
| Days Enrolled (100s) (Median [IQR])      | 0.6 [0.1 - 1.6]                | 0.5 [0.1 - 1.1]                  | **             |

*Significance: `*` = significant at the 95% confidence level (p < 0.05), `**` = significant at the 99% confidence level (p < 0.01). Earned Income and Total Lifetime Enrollments are reported as Mean [± SD] to show population distributions.*

## Multivariable regression and bivariate predictors 



```python
# export to final folder
output_dir = Path("FinalHousingStudies")
output_dir.mkdir(parents=True, exist_ok=True)

# map variables
variable_mapping = {
    "Woman": "Woman",
    "LGBTQ": "LGBTQ",
    "Family": "Family",
   # "SingleParentWChild": "Single Parent Household with Child",
    "VeteranStatus": "Veteran",
    #"White": "White",
    "BlackAfAmerican": "Black/African American",
    "Employed_Entry": "Employed",
    "ChronicHomelessness": "Chronically Unhoused",
    "DVSurvivor_Entry": "Domestic Violence Survivor",
    "DisablingCondition": "Disabling Condition",
    "UnableToWork_Entry": "Unable to Work",
    "Prior_Unsheltered": "Prior Location: Unsheltered",
    "received_service_housing_financial": "Received Housing and Financial Services",
    "received_service_jobs_employment": "Received Jobs and Employment Services",
    "Post2022": "Enrolled Post-2022 Era",
    "Age": "Age",
    "DaysEnrolled": "Days Enrolled (100s)",
    "Earned_Entry": "Earned Income ($1,000s)",
    "Total_Lifetime_Enrollments": "Total Lifetime Enrollments"
}

predictors = list(variable_mapping.keys())

# assign significance asterisks to table
def format_p_value(p):
    if p < 0.01:
        return f"{p:.3f} **"
    elif p < 0.05:
        return f"{p:.3f} *"
    else:
        return f"{p:.3f} "

# run unadjusted bivariate predictor analysis 
bivariate_results = []

for var in predictors:
    try:
        # Calculate valid sample sizes for SPP = 0 and SPP = 1 for this specific variable
        n_spp0 = df_first_all[(df_first_all["SPP"] == 0) & (df_first_all[var].notna())].shape[0]
        n_spp1 = df_first_all[(df_first_all["SPP"] == 1) & (df_first_all[var].notna())].shape[0]
        
        # Run single-predictor logistic regression
        formula_bivariate = f"SPP ~ {var}"
        model_biv = smf.logit(formula_bivariate, data=df_first_all).fit(disp=0)
        
        # Extract metrics
        or_val = np.exp(model_biv.params[var])
        ci_lower = np.exp(model_biv.conf_int().loc[var, 0])
        ci_upper = np.exp(model_biv.conf_int().loc[var, 1])
        p_val = model_biv.pvalues[var]
        
        bivariate_results.append({
            "Variable_Key": var,
            "Predictors": variable_mapping[var],
            "N (SPP=0)": f"{n_spp0:,}",
            "N (SPP=1)": f"{n_spp1:,}",
            "Unadjusted OR": f"{or_val:.2f}",
            "Bivariate 95% CI": f"[{ci_lower:.2f} - {ci_upper:.2f}]",
            "Bivariate p-value": format_p_value(p_val)
        })
    except Exception as e:
        print(f"Could not run bivariate model for {var}: {e}")

df_bivariate = pd.DataFrame(bivariate_results)


# run multivariable predictor adjusted analysis 
formula_multivariable = "SPP ~ " + " + ".join(predictors)
model_multi = smf.logit(formula_multivariable, data=df_first_all).fit()

multivariable_results = []
for var in predictors:
    try:
        aor_val = np.exp(model_multi.params[var])
        a_ci_lower = np.exp(model_multi.conf_int().loc[var, 0])
        a_ci_upper = np.exp(model_multi.conf_int().loc[var, 1])
        a_p_val = model_multi.pvalues[var]
        
        multivariable_results.append({
            "Variable_Key": var,
            "Adjusted aOR": f"{aor_val:.2f}",
            "Multivariable 95% CI": f"[{a_ci_lower:.2f} - {a_ci_upper:.2f}]",
            "Multivariable p-value": format_p_value(a_p_val)
        })
    except Exception as e:
        print(f"Could not extract multivariable data for {var}: {e}")

df_multivariable = pd.DataFrame(multivariable_results)


# merge and save table 
final_summary_df = pd.merge(df_bivariate, df_multivariable, on="Variable_Key")
final_summary_df = final_summary_df.drop(columns=["Variable_Key"])

# reorder with sample sizes first 
final_summary_df = final_summary_df[[
    "Predictors", "N (SPP=0)", "N (SPP=1)",
    "Unadjusted OR", "Bivariate 95% CI", "Bivariate p-value",
    "Adjusted aOR", "Multivariable 95% CI", "Multivariable p-value"
]]

# Generate Markdown text
markdown_output = "# Regression Analysis Results: Predictors of Safe Parking Program (SPP) Placement\n\n"
markdown_output += f"*`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*\n\n"
markdown_output += final_summary_df.to_markdown(index=False)

# Save markdown document
output_file_path = output_dir / "Fig1_BivariateSummary.md"
with open(output_file_path, "w", encoding="utf-8") as f:
    f.write(markdown_output)

print(f"Success! Updated markdown summary table saved to: {output_file_path}")

```

# Regression Analysis Results: Predictors of Safe Parking Program (SPP) Placement


| Predictors                              | N (SPP=0)   | N (SPP=1)   |   Unadjusted OR | Bivariate 95% CI   | Bivariate p-value   |   Adjusted aOR | Multivariable 95% CI   | Multivariable p-value   |
|:----------------------------------------|:------------|:------------|----------------:|:-------------------|:--------------------|---------------:|:-----------------------|:------------------------|
| Woman                                   | 79,768      | 3,977       |            1.18 | [1.11 - 1.26]      | 0.000 **            |           1.3  | [1.21 - 1.40]          | 0.000 **                |
| LGBTQ                                   | 79,768      | 3,977       |            0.69 | [0.51 - 0.94]      | 0.019 *             |           0.94 | [0.68 - 1.29]          | 0.689                   |
| Family                                  | 79,768      | 3,977       |            0.84 | [0.74 - 0.96]      | 0.009 **            |           0.88 | [0.76 - 1.01]          | 0.065                   |
| Veteran                                 | 77,186      | 3,892       |            1.11 | [1.07 - 1.16]      | 0.000 **            |           1.12 | [1.07 - 1.17]          | 0.000 **                |
| Black/African American                  | 79,768      | 3,977       |            1.12 | [1.05 - 1.19]      | 0.001 **            |           1.06 | [0.99 - 1.13]          | 0.106                   |
| Employed                                | 79,768      | 3,977       |            3.53 | [3.29 - 3.80]      | 0.000 **            |           3.18 | [2.89 - 3.50]          | 0.000 **                |
| Chronically Unhoused                    | 79,768      | 3,977       |            0.88 | [0.81 - 0.96]      | 0.003 **            |           0.4  | [0.36 - 0.45]          | 0.000 **                |
| Domestic Violence Survivor              | 79,768      | 3,977       |            1.02 | [0.95 - 1.10]      | 0.598               |           0.98 | [0.90 - 1.07]          | 0.629                   |
| Disabling Condition                     | 79,768      | 3,977       |            0.87 | [0.82 - 0.93]      | 0.000 **            |           1.39 | [1.27 - 1.51]          | 0.000 **                |
| Unable to Work                          | 79,768      | 3,977       |            0.71 | [0.66 - 0.76]      | 0.000 **            |           0.74 | [0.68 - 0.81]          | 0.000 **                |
| Prior Location: Unsheltered             | 79,768      | 3,977       |           10.51 | [9.22 - 11.98]     | 0.000 **            |          12.89 | [11.25 - 14.76]        | 0.000 **                |
| Received Housing and Financial Services | 79,768      | 3,977       |            1.26 | [1.03 - 1.54]      | 0.025 *             |           1.31 | [1.06 - 1.63]          | 0.014 *                 |
| Received Jobs and Employment Services   | 79,768      | 3,977       |            1.33 | [0.91 - 1.94]      | 0.135               |           1.3  | [0.87 - 1.94]          | 0.197                   |
| Enrolled Post-2022 Era                  | 79,768      | 3,977       |            1.08 | [1.01 - 1.15]      | 0.025 *             |           1.13 | [1.05 - 1.21]          | 0.001 **                |
| Age                                     | 79,768      | 3,977       |            1.02 | [1.01 - 1.02]      | 0.000 **            |           1.03 | [1.03 - 1.03]          | 0.000 **                |
| Days Enrolled (100s)                    | 79,768      | 3,977       |            0.85 | [0.83 - 0.88]      | 0.000 **            |           0.82 | [0.80 - 0.84]          | 0.000 **                |
| Earned Income ($1,000s)                 | 79,768      | 3,977       |            1.61 | [1.54 - 1.67]      | 0.000 **            |           1.13 | [1.08 - 1.18]          | 0.000 **                |
| Total Lifetime Enrollments              | 79,768      | 3,977       |            1.04 | [1.02 - 1.07]      | 0.001 **            |           1.04 | [1.01 - 1.06]          | 0.009 **                |

*`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*

### Bivariate forest plot


```python
# filter warnings
warnings.filterwarnings("ignore", message="findfont: Font family.*not found.")

# set variable mapping
plt.rcParams["font.family"] = "serif"
plt.rcParams["font.serif"] = ["Times New Roman", "Times", "DejaVu Serif"]

variable_mapping = {
    "Woman": "Woman",
    "LGBTQ": "LGBTQ",
    "Family": "Family",
    #"SingleParentWChild": "Single Parent Household with Child",
    #"IsRepeatEnrollment": "Is Repeat Enrollment",
    "VeteranStatus": "Veteran",
    #"White": "White",
    "BlackAfAmerican": "Black/African American",
    "Employed_Entry": "Employed",
    "ChronicHomelessness": "Chronically Unhoused",
    "DVSurvivor_Entry": "Domestic Violence Survivor",
    "DisablingCondition": "Disabling Condition",
    "UnableToWork_Entry": "Unable to Work",
    "Prior_Unsheltered": "Prior Location: Unsheltered",
    "received_service_housing_financial": "Received Housing and Financial Services",
    "received_service_jobs_employment": "Received Jobs and Employment Services",
    "Post2022": "Enrolled Post-2022 Era",
    "Age": "Age",
    "DaysEnrolled": "Days Enrolled (100s)",
    "Earned_Entry": "Earned Income ($1,000s)",
    "Total_Lifetime_Enrollments": "Total Lifetime Enrollments",
}

continuous_vars = ["Age", "Earned_Entry", "Total_Lifetime_Enrollments", "DaysEnrolled"]

binary_vars = [
    v for v in variable_mapping.keys() 
    if v not in continuous_vars
]

plot_data = []

z_95 = 1.960
z_99 = 2.576

# Colors
COLOR_GREEN_95 = "#2e7d32"  # Forest Green
COLOR_GREEN_99 = "#81c784"  # Light Green
COLOR_YELLOW_95 = "#d4af37" # Golden Yellow
COLOR_YELLOW_99 = "#f0e68c" # Light Yellow

BG_GREEN = "#d4edda"  # Soft green for SPP significance
BG_RED = "#f8d7da"    # Soft red for Non-SPP significance

# process binary variables first
for var in binary_vars:
    clean_df = df_first_all[[var, "SPP"]].dropna()
    g1 = clean_df[clean_df["SPP"] == 1][var]
    g0 = clean_df[clean_df["SPP"] == 0][var]

    a = (g1 == 1).sum()
    b = (g1 == 0).sum()
    c = (g0 == 1).sum()
    d = (g0 == 0).sum()
    n_valid = a + b + c + d

    if a == 0 or b == 0 or c == 0 or d == 0:
        a, b, c, d = a + 0.5, b + 0.5, c + 0.5, d + 0.5

    or_val = (a * d) / (b * c)
    se_log_or = np.sqrt((1 / a) + (1 / b) + (1 / c) + (1 / d))

    ci_lower_95 = np.exp(np.log(or_val) - z_95 * se_log_or)
    ci_upper_95 = np.exp(np.log(or_val) + z_95 * se_log_or)
    ci_lower_99 = np.exp(np.log(or_val) - z_99 * se_log_or)
    ci_upper_99 = np.exp(np.log(or_val) + z_99 * se_log_or)

    sig_95 = "Yes" if (ci_upper_95 < 1.0 or ci_lower_95 > 1.0) else "No"
    sig_99 = "Yes" if (ci_upper_99 < 1.0 or ci_lower_99 > 1.0) else "No"

    plot_data.append(
        {
            "Display_Name": variable_mapping[var],
            "Type": "Binary",
            "Effect": or_val,
            "Lower_95": ci_lower_95,
            "Upper_95": ci_upper_95,
            "Lower_99": ci_lower_99,
            "Upper_99": ci_upper_99,
            "Sig_95": sig_95,
            "Sig_99": sig_99,
            "Sample_Size": f"{n_valid:,}",
        }
    )

# process continuous for Cohen's D
for var in continuous_vars:
    clean_df = df_first_all[[var, "SPP"]].dropna()
    g1 = clean_df[clean_df["SPP"] == 1][var]
    g0 = clean_df[clean_df["SPP"] == 0][var]

    m1, s1, n1 = g1.mean(), g1.std(), len(g1)
    m0, s0, n0 = g0.mean(), g0.std(), len(g0)
    n_valid = n1 + n0

    s_pooled = np.sqrt(((n1 - 1) * s1**2 + (n0 - 1) * s0**2) / (n1 + n0 - 2))
    cohen_d = (m1 - m0) / s_pooled
    se_d = np.sqrt((n1 + n0) / (n1 * n0) + (cohen_d**2) / (2 * (n1 + n0)))

    ci_lower_95 = cohen_d - z_95 * se_d
    ci_upper_95 = cohen_d + z_95 * se_d
    ci_lower_99 = cohen_d - z_99 * se_d
    ci_upper_99 = cohen_d + z_99 * se_d

    sig_95 = "Yes" if (ci_upper_95 < 0.0 or ci_lower_95 > 0.0) else "No"
    sig_99 = "Yes" if (ci_upper_99 < 0.0 or ci_lower_99 > 0.0) else "No"

    plot_data.append(
        {
            "Display_Name": variable_mapping[var],
            "Type": "Continuous",
            "Effect": cohen_d,
            "Lower_95": ci_lower_95,
            "Upper_95": ci_upper_95,
            "Lower_99": ci_lower_99,
            "Upper_99": ci_upper_99,
            "Sig_95": sig_95,
            "Sig_99": sig_99,
            "Sample_Size": f"{n_valid:,}",
        }
    )

# create polished forest plot
plot_df = pd.DataFrame(plot_data)

fig, ax = plt.subplots(figsize=(10, len(plot_df) * 0.45 + 2), dpi=300)

y_positions = np.arange(len(plot_df))

for idx, row in plot_df.iterrows():
    y = y_positions[idx]

    if row["Type"] == "Binary":
        val = np.log(row["Effect"])
        l95 = np.log(row["Lower_95"])
        u95 = np.log(row["Upper_95"])
        l99 = np.log(row["Lower_99"])
        u99 = np.log(row["Upper_99"])
        marker = "o"
        marker_color = COLOR_GREEN_95
        c_95 = COLOR_GREEN_95
        c_99 = "gray"
    else:
        val = row["Effect"]
        l95 = row["Lower_95"]
        u95 = row["Upper_95"]
        l99 = row["Lower_99"]
        u99 = row["Upper_99"]
        marker = "^"
        marker_color = COLOR_YELLOW_95
        c_95 = COLOR_YELLOW_95
        c_99 = "gray"

    # Add background shading based on significance direction
    if l95 > 0:  # Significantly higher in SPP
        ax.axhspan(y - 0.4, y + 0.4, color=BG_GREEN, alpha=0.5, zorder=0)
    elif u95 < 0:  # Significantly higher in Non-SPP
        ax.axhspan(y - 0.4, y + 0.4, color=BG_RED, alpha=0.5, zorder=0)

    # 99% CI Line (Thinner)
    ax.plot([l99, u99], [y, y], color=c_99, linewidth=1.5, alpha=0.9, zorder=3)
    # 95% CI Line (Thicker)
    ax.plot([l95, u95], [y, y], color=c_95, linewidth=4.5, alpha=0.9, zorder=4)
    # Point Estimate Marker
    ax.plot(
        val,
        y,
        marker=marker,
        color="none",
        markerfacecolor=marker_color,
        markeredgecolor="black",
        markersize=7,
        zorder=5,
    )

# Reference line at null effect (Log-Odds = 0 / Cohen's d = 0)
ax.axvline(x=0, color="black", linestyle="--", linewidth=1.0, alpha=0.8, zorder=2)

ax.set_yticks(y_positions)
ax.set_yticklabels(plot_df["Display_Name"], fontsize=10, family="Times New Roman")
ax.invert_yaxis()  # Top-to-bottom display order

ax.set_xlabel(
    "Standardized Bivariate Effect Size (Log-Odds Ratio for Binary / Cohen's d for Continuous)",
    fontsize=10,
    fontweight="bold",
    family="Times New Roman",
)
#ax.set_title(
   # "Bivariate Predictors of SPP vs. Non-SPP Enrollment (All First-Time Enrollees, 2017-2025)",
   # fontsize=10,
    #fontweight="bold",
   # pad=15,
    #family="Times New Roman",
#)

ax.grid(axis="x", linestyle=":", alpha=0.6, zorder=1)
ax.grid(axis="y", visible=False)
ax.spines["top"].set_visible(False)
ax.spines["right"].set_visible(False)

# Legend elements
legend_elements = [
    Line2D([0], [0], color=COLOR_GREEN_95, linewidth=4.5, label="Binary 95% CI (Green)"),
    Line2D([0], [0], color=COLOR_YELLOW_95, linewidth=4.5, label="Continuous 95% CI (Yellow)"),
    Line2D([0], [0], color="gray", linewidth=1.5, label="99% Confidence Interval"),
    Line2D([0], [0], marker="o", color="none", markerfacecolor=COLOR_GREEN_95, markeredgecolor="black", markersize=7, label="Binary Effect Estimate"),
    Line2D([0], [0], marker="^", color="none", markerfacecolor=COLOR_YELLOW_95, markeredgecolor="black", markersize=7, label="Continuous Effect Estimate"),
]

ax.legend(
    handles=legend_elements,
    loc="upper left",
    bbox_to_anchor=(1.02, 0.3),
    frameon=False,
    prop={"family": "Times New Roman", "size": 10},
)

ax.tick_params(axis='x', labelsize=10)

plt.tight_layout()

# Save final product directly to directory
output_folder = Path("FinalHousingStudies")
output_folder.mkdir(parents=True, exist_ok=True)
output_path = output_folder / "Fig1_BivariatePlot.jpg"

plt.savefig(output_path, dpi=600, bbox_inches="tight", format="jpg")
print(f"Polished forest plot successfully saved to: {output_path}")
plt.show()
```

## Compositional shifts


```python
# export to folder
output_dir = Path("FinalHousingStudies")
output_dir.mkdir(parents=True, exist_ok=True)

# variable mapping
variable_mapping = {
    "Woman": "Woman",
    "LGBTQ": "LGBTQ",
    "Family": "Family",
    #"SingleParentWChild": "Single Parent Household with Child",
    #"IsRepeatEnrollment": "Is Repeat Enrollment",
    "VeteranStatus": "Veteran",
   # "White": "White",
    "BlackAfAmerican": "Black/African American",
    "Employed_Entry": "Employed",
    "ChronicHomelessness": "Chronically Unhoused",
    "DVSurvivor_Entry": "Domestic Violence Survivor",
    "DisablingCondition": "Disabling Condition",
    "UnableToWork_Entry": "Unable to Work",
    "Prior_Unsheltered": "Prior Location: Unsheltered",
    "received_service_housing_financial": "Received Housing and Financial Services",
    "received_service_jobs_employment": "Received Jobs and Employment Services",
    "Post2022": "Enrolled Post-2022 Era",
    "Age": "Age",
    "DaysEnrolled": "Days Enrolled (100s)",
    "Earned_Entry": "Earned Income ($1,000s)",
    "Total_Lifetime_Enrollments": "Total Lifetime Enrollments"
}

# order variables for layout
ordered_keys = [
    "Woman",  "LGBTQ", "Family", "VeteranStatus", "BlackAfAmerican", "Employed_Entry", "ChronicHomelessness", "DVSurvivor_Entry", "DisablingCondition",
    "UnableToWork_Entry", "Prior_Unsheltered", "received_service_housing_financial",  "received_service_jobs_employment", "Age", "DaysEnrolled", "Earned_Entry", 
    "Total_Lifetime_Enrollments"
]

# run interaction regression
formula_interaction = (
    "SPP ~ Post2022 * (" + " + ".join(ordered_keys) + ")"
)

print("Fitting complete interaction logit model...")
model2 = smf.logit(formula_interaction, data=df_first_all).fit()

# get effects and create dataframe
table_rows = []
plot_data_list = []

def get_sig_asterisks(p_val):
    if p_val < 0.01: return "**"
    elif p_val < 0.05: return "*"
    return " "

for var in ordered_keys:
    interaction_term = f"Post2022:{var}"
    if interaction_term not in model2.params:
        interaction_term = f"{var}:Post2022"
        
    coef = model2.params[interaction_term]
    p_val = model2.pvalues[interaction_term]
    ci_bounds = model2.conf_int().loc[interaction_term]
    
    aor = np.exp(coef)
    ci_lower = np.exp(ci_bounds[0])
    ci_upper = np.exp(ci_bounds[1])
    sig = get_sig_asterisks(p_val)
    
    clean_name = variable_mapping[var]
    
    table_rows.append({
        "Variable": clean_name,
        "Interaction aOR": f"{aor:.2f}",
        "95% CI": f"[{ci_lower:.2f} - {ci_upper:.2f}]",
        "p-value": f"{p_val:.4f}",
        "Significance": sig
    })
    
    plot_data_list.append({
        "clean_name": clean_name,
        "aor": aor,
        "lower": ci_lower,
        "upper": ci_upper,
        "sig": sig
    })

df_table = pd.DataFrame(table_rows)

# save table
markdown_content = "Compositional Shifts Post-2022 for SPP and Non-SPP Enrollments (2017-2025)\n\n"
markdown_content += df_table.to_markdown(index=False)
table_path = output_dir / "Fig2_CompositionalTable.md"
with open(table_path, "w", encoding="utf-8") as f:
    f.write(markdown_content)
print(f"Table successfully saved to {table_path}")


# create plot
plot_elements = plot_data_list[::-1]

fig, ax = plt.subplots(figsize=(12, 11), dpi=300)

color_significant = "#D32F2F"    # Crimson Red
color_insignificant = "#778899"  # Slate Grey
grid_line_color = "#E0E0E0"

y_positions = np.arange(len(plot_elements))

ax.axvline(x=1.0, color="#333333", linestyle="--", linewidth=1.2, zorder=1)
for y in y_positions:
    ax.axhline(y=y, color=grid_line_color, linestyle="-", linewidth=0.7, zorder=0)

for idx, item in enumerate(plot_elements):
    aor = item["aor"]
    low = item["lower"]
    high = item["upper"]
    is_sig = item["sig"] in ["*", "**"]
    
    use_color = color_significant if is_sig else color_insignificant
    
    ax.plot([low, high], [idx, idx], color=use_color, linewidth=2.0, zorder=2)
    ax.plot(aor, idx, marker="o", markersize=8, color=use_color, markerfacecolor=use_color, zorder=3)
    
    label_text = f"{aor:.2f}{item['sig'] if item['sig'] != 'ns' else ''}"
    ax.text(high + 0.04, idx, label_text, va="center", ha="left", 
            color=use_color, fontsize=10, fontweight="bold" if is_sig else "normal")

ax.set_yticks(y_positions)
ax.set_yticklabels([item["clean_name"] for item in plot_elements], fontsize=10.5)
ax.set_xlim(-0.1, 2.1)
ax.set_xticks(np.arange(0.0, 2.25, 0.25))

ax.tick_params(axis='x', labelsize=10)

#ax.set_title("Post-2022 Scandals and Expansions: Client Enrollment Compositional Shifts\nin Safe Parking Programs (SPP), 2017-2025", 
            # fontsize=14, fontweight="bold", pad=25, loc="center")
ax.set_xlabel("Interaction Adjusted Odds Ratio (aOR) for SPP Enrollment Post-2022", 
              fontsize=11, fontweight="bold", labelpad=15)

ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.spines['left'].set_color('#999999')
ax.spines['bottom'].set_color('#999999')

ax.text(0.95, len(plot_elements), "Left of line (< 1.0) = Lower SPP intake post-2022", 
        fontsize=8.5, bbox=dict(facecolor='none', edgecolor='#CCCCCC', boxstyle='round,pad=0.3'), ha='right', va='bottom')
ax.text(1.05, len(plot_elements), "Right of line (> 1.0) = Higher SPP intake post-2022", 
        fontsize=8.5, bbox=dict(facecolor='none', edgecolor='#CCCCCC', boxstyle='round,pad=0.3'), ha='left', va='bottom')

from matplotlib.lines import Line2D
legend_elements = [
    Line2D([0], [0], color='#333333', linestyle='--', linewidth=1.2, label="No Shift Baseline (aOR = 1.0)"),
    Line2D([0], [0], marker='o', color='none', markerfacecolor=color_significant, markeredgecolor='none',markersize=8, label="Significant Shift (* p < 0.05, ** p < 0.01)"),
    Line2D([0], [0], marker='o', color='none', markerfacecolor=color_insignificant, markeredgecolor='none',markersize=8, label="Non-Significant Shift")
]
ax.legend(handles=legend_elements, loc='lower center', bbox_to_anchor=(0.5, -0.13),
          ncol=3, frameon=False, fontsize=9.5, handletextpad=0.5, columnspacing=1.5)

footer_text = (
    "Cohort Sample Sizes (Total Cohort N = 83,745):\n"
    "• Pre-2022 Era (N = 47,198): Emergency Shelter N = 45,025 | Safe Parking N = 2,173\n"
    "• Post-2022 Era (N = 36,547): Emergency Shelter N = 34,743 | Safe Parking N = 1,804\n"
    "(Model sample sizes across specific characteristics range from N = 74,116 to N = 83,745)"
)
plt.text(0.5, -0.21, footer_text, transform=ax.transAxes, ha="center", va="top", fontsize=8.5,
         color="#444444", bbox=dict(facecolor='#FAFAFA', edgecolor='#D3D3D3', boxstyle='round,pad=0.6'))

plt.tight_layout()

figure_path = output_dir / "Fig2_CompositionalFigure.jpg"
plt.savefig(figure_path, dpi=600, bbox_inches="tight", format="jpg")
plt.close()
print(f"Figure successfully saved to {figure_path}")

```

# Compositional Shifts Post-2022 for SPP and Non-SPP Enrollments (2017-2025)

| Variable                                |   Interaction aOR | 95% CI        |   p-value | Significance   |
|:----------------------------------------|------------------:|:--------------|----------:|:---------------|
| Woman                                   |              0.72 | [0.62 - 0.83] |    0      | **             |
| LGBTQ                                   |              0.57 | [0.29 - 1.12] |    0.1024 |                |
| Family                                  |              1.23 | [0.92 - 1.63] |    0.157  |                |
| Veteran                                 |              0.92 | [0.84 - 1.00] |    0.0589 |                |
| Black/African American                  |              1.46 | [1.27 - 1.67] |    0      | **             |
| Employed                                |              1.32 | [1.10 - 1.59] |    0.0034 | **             |
| Chronically Unhoused                    |              0.88 | [0.71 - 1.08] |    0.2192 |                |
| Domestic Violence Survivor              |              1.17 | [0.98 - 1.39] |    0.081  |                |
| Disabling Condition                     |              1.11 | [0.94 - 1.32] |    0.2195 |                |
| Unable to Work                          |              1.43 | [1.21 - 1.71] |    0      | **             |
| Prior Location: Unsheltered             |              0.82 | [0.62 - 1.08] |    0.1509 |                |
| Received Housing and Financial Services |              0.93 | [0.58 - 1.49] |    0.7639 |                |
| Received Jobs and Employment Services   |              0.08 | [0.02 - 0.36] |    0.0009 | **             |
| Age                                     |              1.01 | [1.00 - 1.01] |    0.0051 | **             |
| Days Enrolled (100s)                    |              0.84 | [0.80 - 0.89] |    0      | **             |
| Earned Income ($1,000s)                 |              1.1  | [1.03 - 1.18] |    0.0035 | **             |
| Total Lifetime Enrollments              |              1.09 | [1.02 - 1.16] |    0.0143 | *              |

*`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*


```python
# validate sample sizes for pre 2022
print(df_first_all[df_first_all['Post2022']==0]['SPP'].value_counts())
```


```python
# validate sample sizes for post 2022
print(df_first_all[df_first_all['Post2022']==1]['SPP'].value_counts())
```


```python
df_first_all.shape
```
