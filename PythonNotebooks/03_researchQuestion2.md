# Question 2: 
## Where do clients come from, how long do they stay, and what are the rates of exit for first-time and long-time residents?


```python
# client origins and length of stay
# styles
plt.style.use(
    'seaborn-v0_8-whitegrid'
    if 'seaborn-v0_8-whitegrid' in plt.style.available
    else 'default'
)

COLOR_GREEN = '#2E7D32'
COLOR_YELLOW = '#FBC02D'
COLOR_GREY = '#757575'
COLOR_RED = '#D32F2F'

#column diagnostics
print(list(df_all.columns))
```


```python
df_repeat.head()
```


```python
# data types for EntryDate and ExitDate are datetime
df_repeat.dtypes
```


```python
df_repeat.shape
```


```python
# calculating where do they come from and how long on average they are enrolled 
prior_vars = ['Prior_Unsheltered', 'Prior_TemporaryStable', 'Prior_TemporaryUnstable', 'Prior_Permanent', 'Prior_Institutional']
print("Prior Location Percentages by Cohort")
print(df_first.groupby('SPP')[prior_vars].mean() * 100)

print("\nMedian Days Enrolled by Program & Repeat Status")
print(df_first.groupby(['SPP', 'True_Repeat_Client'])['DaysEnrolled'].median())
```

## Multi-level linear probability model 


```python
# create the full LPM formula with all predictors
lpm_formula = (
    "Exit_Permanent ~ SPP * Post2022 + "
    "Woman + LGBTQ + Family + VeteranStatus + BlackAfAmerican + "
    "Age + Employed_Entry + ChronicHomelessness + DVSurvivor_Entry + DisablingCondition + "
    "UnableToWork_Entry + Prior_Unsheltered + received_service_housing_financial + "
    "received_service_jobs_employment + DaysEnrolled + Earned_Entry + Total_Lifetime_Enrollments"
)

# model variables 
variables = [
    'Exit_Permanent',
    'SPP',
    'Post2022',
    'Woman',
    'LGBTQ',
    'Family',
    'VeteranStatus',
    'BlackAfAmerican',
    'Age',
    'Employed_Entry',
    'ChronicHomelessness',
    'DVSurvivor_Entry',
    'DisablingCondition',
    'UnableToWork_Entry',
    'Prior_Unsheltered',
    'received_service_housing_financial',
    'received_service_jobs_employment',
    'DaysEnrolled',
    'Earned_Entry',
    'Total_Lifetime_Enrollments',
    'ProjectID',
]

#drop na
df_first_clean = df_first.dropna(subset=variables).copy()
df_repeat_clean = df_repeat.dropna(subset=variables).copy()
```


```python
df_first_clean.shape
```


```python
df_repeat_clean.shape
```

### running lpm for both first-time and repeat with null variables cleaned up


```python
# lpm for first-time df_first
lpm_model_first = smf.ols(lpm_formula, data=df_first_clean).fit(
    cov_type='cluster', cov_kwds={'groups': df_first_clean['ProjectID']}
)

# lpm for repeat df_repeat
lpm_model_repeat = smf.ols(lpm_formula, data=df_repeat_clean).fit(
    cov_type='cluster', cov_kwds={'groups': df_repeat_clean['ProjectID']}
)

print("LPM first-time cohort")
print(lpm_model_first.summary())

print("\nLPM repeat cohort")
print(lpm_model_repeat.summary())
```


```python
# create variable dictionary for table
variable_mapping = {
    "Intercept": "Constant / Intercept",
    "SPP": "Safe Parking Program (SPP Baseline)",
    "Post2022": "Enrolled Post-2022 Era",
    "SPP:Post2022": "SPP × Post-2022 Era (DiD Estimator)",
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
    "received_service_housing_financial": "Received Housing & Financial Services",
    "received_service_jobs_employment": "Received Jobs & Employment Services",
    "DaysEnrolled": "Days Enrolled in Program (100s)",
    "Earned_Entry": "Earned Monthly Income ($1,000s)",
    "Total_Lifetime_Enrollments": "Total Lifetime Enrollments",
}


# add statistical significance
def get_stars(p_val):
    if p_val < 0.01:
        return "**"
    elif p_val < 0.05:
        return "*"
    return ""


# create formatted table
def generate_publication_table(model_first, model_repeat, mapping):
    rows = []

    # Get all unique predictor names
    all_vars = list(
        dict.fromkeys(
            list(model_first.params.index) + list(model_repeat.params.index)
        )
    )

    for var in all_vars:
        # Map raw variable name to spelled-out string
        clean_name = mapping.get(var, var)

        # First-time cohort stats
        b_f = model_first.params[var]
        se_f = model_first.bse[var]
        p_f = model_first.pvalues[var]
        str_f = f"{b_f:+.4f}{get_stars(p_f)} ({se_f:.4f})"

        # Repeat cohort stats
        b_r = model_repeat.params[var]
        se_r = model_repeat.bse[var]
        p_r = model_repeat.pvalues[var]
        str_r = f"{b_r:+.4f}{get_stars(p_r)} ({se_r:.4f})"

        rows.append(
            {
                "Predictor Variable": clean_name,
                "First-Time Enrollments β (SE)": str_f,
                "Repeat Enrollments β (SE)": str_r,
            }
        )

    df_pub = pd.DataFrame(rows)

    # Reorder table so DiD Policy drivers are at top
    focal_top = [
        "SPP × Post-2022 Era (DiD Estimator)",
        "Safe Parking Program (SPP Baseline)",
        "Enrolled Post-2022 Era",
    ]
    df_focal = df_pub[df_pub["Predictor Variable"].isin(focal_top)].sort_values(
        by="Predictor Variable", key=lambda x: x.map(lambda v: focal_top.index(v))
    )
    df_controls = df_pub[~df_pub["Predictor Variable"].isin(focal_top)]

    return pd.concat([df_focal, df_controls], ignore_index=True)


# create table
df_clean_results = generate_publication_table(
    lpm_model_first, lpm_model_repeat, variable_mapping
)

# Display in notebook
print(df_clean_results.to_string(index=False))

# export to markdown 
output_dir = Path("FinalHousingStudies")
output_dir.mkdir(parents=True, exist_ok=True)
export_file = output_dir / "Table2_LPM_DiD_Regression_Results.md"

markdown_output = f"""# Linear Probability Model Results (DiD)
### Policy Drivers and Control Predictors of Permanent Housing Exits

{df_clean_results.to_markdown(index=False)}

---
*\Standard errors clustered by ProjectID to control for program differences. * p<0.05, ** p<0.01.*
* First-Time Cohort: N = {int(lpm_model_first.nobs):,}, R² = {lpm_model_first.rsquared:.3f}
* Repeat Cohort: N = {int(lpm_model_repeat.nobs):,}, R² = {lpm_model_repeat.rsquared:.3f}
"""

with open(export_file, "w", encoding="utf-8") as f:
    f.write(markdown_output)

print(f"\nClean publication table saved to: {export_file}")
```

# Linear Probability Model Results (DiD)
### Policy Drivers and Control Predictors of Permanent Housing Exits

| Predictor Variable                    | First-Time Enrollments β (SE)   | Repeat Enrollments β (SE)   |
|:--------------------------------------|:--------------------------------|:----------------------------|
| SPP × Post-2022 Era (DiD Estimator)   | +0.0779* (0.0303)               | +0.0657** (0.0249)          |
| Safe Parking Program (SPP Baseline)   | -0.1043** (0.0204)              | -0.0960** (0.0149)          |
| Enrolled Post-2022 Era                | -0.0750** (0.0123)              | -0.0189 (0.0106)            |
| Constant / Intercept                  | +0.2548** (0.0291)              | +0.1248** (0.0277)          |
| Woman                                 | +0.0338** (0.0077)              | +0.0239** (0.0062)          |
| LGBTQ+                                | -0.0076 (0.0207)                | +0.0116 (0.0131)            |
| Family Household                      | +0.1738** (0.0235)              | +0.1080** (0.0317)          |
| Veteran                               | +0.0095 (0.0064)                | +0.0081 (0.0049)            |
| Black / African American              | +0.0593** (0.0076)              | +0.0486** (0.0064)          |
| Age (Years)                           | +0.0010* (0.0004)               | +0.0011** (0.0004)          |
| Employed at Entry                     | +0.0960** (0.0099)              | +0.0432** (0.0103)          |
| Chronically Unhoused                  | +0.0022 (0.0077)                | +0.0045 (0.0067)            |
| Domestic Violence Survivor            | +0.0100 (0.0061)                | +0.0173** (0.0049)          |
| Disabling Condition                   | -0.0026 (0.0075)                | +0.0198** (0.0073)          |
| Unable to Work                        | +0.0135* (0.0062)               | +0.0067 (0.0052)            |
| Prior Location: Unsheltered           | -0.0493** (0.0095)              | -0.0653** (0.0086)          |
| Received Housing & Financial Services | +0.1246** (0.0263)              | +0.0912** (0.0298)          |
| Received Jobs & Employment Services   | -0.0343 (0.0255)                | +0.0071 (0.0345)            |
| Days Enrolled in Program (100s)       | +0.0494** (0.0030)              | +0.0497** (0.0027)          |
| Earned Monthly Income ($1,000s)       | +0.0045* (0.0019)               | +0.0338** (0.0057)          |
| Total Lifetime Enrollments            | -0.0765** (0.0043)              | -0.0174** (0.0013)          |

---
*\Standard errors clustered by ProjectID to control for program differences. *`*` = p < 0.05 (95% Confidence), `**` = p < 0.01 (99% Confidence).*
* First-Time Cohort: N = 48,690, R² = 0.146
* Repeat Cohort: N = 49,952, R² = 0.085


## LPM DiD Forest Plot


```python
# import additional matplotlib library to reorient legend
import matplotlib.patches as mpatches

# variable mapping for plot 
variable_labels = {
    "SPP:Post2022": "SPP × Post-2022 Era (DiD Estimator)",
    "SPP": "Safe Parking Program (SPP Baseline)",
    "Post2022": "Enrolled Post-2022 Era",
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

# extract parameters from LPM DiD
plot_rows = []
for var_key, clean_label in variable_labels.items():
  # First-time metrics (converted to percentage points, * 100)
  b_first = lpm_model_first.params.get(var_key, np.nan) * 100
  se_first = lpm_model_first.bse.get(var_key, np.nan) * 100
  p_first = lpm_model_first.pvalues.get(var_key, np.nan)

  # Repeat metrics (converted to percentage points, * 100)
  b_rep = lpm_model_repeat.params.get(var_key, np.nan) * 100
  se_rep = lpm_model_repeat.bse.get(var_key, np.nan) * 100
  p_rep = lpm_model_repeat.pvalues.get(var_key, np.nan)

  # significance asterisks
  star_first = (
      "**" if p_first < 0.01 else ("*" if p_first < 0.05 else "")
  ) if not np.isnan(p_first) else ""
  star_rep = (
      "**" if p_rep < 0.01 else ("*" if p_rep < 0.05 else "")
  ) if not np.isnan(p_rep) else ""

  plot_rows.append({
      "var": clean_label,
      "b_first": b_first,
      "se_first": se_first,
      "star_first": star_first,
      "b_rep": b_rep,
      "se_rep": se_rep,
      "star_rep": star_rep,
  })

df_forest = pd.DataFrame(plot_rows)

# show DiD interaction at top
df_forest = df_forest.iloc[::-1].reset_index(drop=True)

# Compute 95% Confidence Intervals (1.96 * SE)
df_forest["ci_first"] = 1.96 * df_forest["se_first"]
df_forest["ci_rep"] = 1.96 * df_forest["se_rep"]

# 4Style and pallette set up
plt.rcParams["font.family"] = "serif"
plt.rcParams["font.serif"] = ["Times New Roman", "Times", "DejaVu Serif"]

fig, ax = plt.subplots(figsize=(14, 12), dpi=300)

y_indices = np.arange(len(df_forest))
offset = 0.18

# Project Standard Colors
COLOR_GREEN = "#2E7D32"  # First-Time Cohort (Green)
COLOR_YELLOW = "#FBC02D"  # Repeat Cohort (Yellow)
COLOR_TEXT_YELLOW = (
    "#B8860B"  # Darker Gold/Yellow for clear text contrast on white background
)

# Plot First-Time Cohort Estimates
ax.errorbar(
    x=df_forest["b_first"],
    y=y_indices + offset,
    xerr=df_forest["ci_first"],
    fmt="o",
    color=COLOR_GREEN,
    ecolor=COLOR_GREEN,
    elinewidth=1.8,
    capsize=4,
    capthick=1.5,
    markersize=7,
    label=f"First-Time Clients (N = {int(lpm_model_first.nobs):,})",
)

# Plot Repeat Cohort Estimates
ax.errorbar(
    x=df_forest["b_rep"],
    y=y_indices - offset,
    xerr=df_forest["ci_rep"],
    fmt="s",
    color=COLOR_YELLOW,
    ecolor=COLOR_YELLOW,
    elinewidth=1.8,
    capsize=4,
    capthick=1.5,
    markersize=6,
    label=f"Repeat Clients (N = {int(lpm_model_repeat.nobs):,})",
)

# Annotate asterisks (* p<0.05, ** p<0.01) beside Error Caps
for idx in range(len(df_forest)):
  row = df_forest.iloc[idx]

  # First-Time Star Annotation
  if row["star_first"]:
    x_pos_first = row["b_first"] + row["ci_first"] + 0.4
    ax.text(
        x_pos_first,
        idx + offset,
        row["star_first"],
        va="center",
        ha="left",
        color=COLOR_GREEN,
        fontsize=10,
        fontweight="bold",
    )

  # Repeat Star Annotation
  if row["star_rep"]:
    x_pos_rep = row["b_rep"] + row["ci_rep"] + 0.4
    ax.text(
        x_pos_rep,
        idx - offset,
        row["star_rep"],
        va="center",
        ha="left",
        color=COLOR_TEXT_YELLOW,
        fontsize=10,
        fontweight="bold",
    )

# Reference Null Line (0 Δpp)
ax.axvline(x=0, color="#757575", linestyle="--", linewidth=1.2, alpha=0.85)

# Axis & Title Formatting
ax.set_yticks(y_indices)
ax.set_yticklabels(
    df_forest["var"], fontsize=11, fontweight="bold", color="#212121"
)
ax.set_xlabel(
    "Percentage Point Effect on Permanent Housing Exit (Δpp)",
    fontsize=10,
    fontweight="bold",
    labelpad=10,
)
#ax.set_title(
    #"Figure 4: Difference-in-Differences LPM Policy Drivers & Control"
    #" Predictors",
    #fontsize=14,
   # fontweight="bold",
    #pad=15,
#)

ax.set_xlim(-22, 28)
ax.grid(True, linestyle=":", alpha=0.5, axis="x")

# Legend with significance note customized
handles, labels = ax.get_legend_handles_labels()
note_handle = mpatches.Patch(color="none", label="")
handles.append(note_handle)
labels.append("* p < 0.05 (95% CI) | ** p < 0.01 (99% CI)")

ax.legend(
    handles = handles,
    labels = labels,
    loc="lower right",
    bbox_to_anchor=(
        0.98,
        0.02,
    ),
    frameon=True,
    facecolor="white",
    edgecolor='lightgray',
    framealpha=0.95,
    fontsize=10,
    title_fontsize=10,
)


plt.tight_layout()

# Save High-Res Image
export_jpg = output_dir / "Fig4_DiD_PolicyDrivers_ForestPlot.jpg"
plt.savefig(export_jpg, bbox_inches="tight", dpi=600, format="jpg")


print(f"Success! Complete Figure 4 Forest Plot exported to: {export_jpg}")

plt.show()
```

## Dynamic trajectories


```python
# set paths and font
output_dir = Path("FinalHousingStudies")
output_dir.mkdir(parents=True, exist_ok=True)

plt.rcParams.update(
    {
        "font.family": "serif",
        "font.serif": ["Times New Roman"],
        "font.size": 10,
        "axes.titlesize": 10,
        "axes.labelsize":10,
        "xtick.labelsize": 10,
        "ytick.labelsize": 10,
        "legend.fontsize": 10,
    }
)


# set axis and figure
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 6), sharey=True)
years = np.arange(2017, 2026)

# First-Time Trajectories
df_first = df_all[df_all["True_Repeat_Client"] == 0]
spp_first_exit = (
    df_first[df_first["SPP"] == 1]
    .groupby("CalendarYear")["Exit_Permanent"]
    .mean()
    * 100
)
nonspp_first_exit = (
    df_first[df_first["SPP"] == 0]
    .groupby("CalendarYear")["Exit_Permanent"]
    .mean()
    * 100
)

ax1.plot(
    spp_first_exit.index,
    spp_first_exit.values,
    color=COLOR_GREEN,
    marker="o",
    linewidth=2.5,
    label="SPP (First-Time)",
)
ax1.plot(
    nonspp_first_exit.index,
    nonspp_first_exit.values,
    color=COLOR_GREY,
    linestyle="--",
    marker="s",
    label="Non-SPP Shelter",
)
ax1.axvline(
    x=2022, color=COLOR_RED, linestyle="--", label="2022 Policy Shift"
)
ax1.set_title("First-Time Clients", fontweight="bold")
ax1.set_ylabel("Permanent Exit Rate (%)", fontweight="bold")
ax1.legend(
    loc="upper right",
    frameon=True,
    facecolor="white",
    edgecolor='lightgray',
    framealpha=0.95,
    fontsize=10,
    title_fontsize=10,
            
)

# Repeat Trajectories
df_rep = df_all[df_all["True_Repeat_Client"] == 1]
spp_rep_exit = (
    df_rep[df_rep["SPP"] == 1]
    .groupby("CalendarYear")["Exit_Permanent"]
    .mean()
    * 100
)
nonspp_rep_exit = (
    df_rep[df_rep["SPP"] == 0]
    .groupby("CalendarYear")["Exit_Permanent"]
    .mean()
    * 100
)

ax2.plot(
    spp_rep_exit.index,
    spp_rep_exit.values,
    color=COLOR_YELLOW,
    marker="o",
    linewidth=2.5,
    label="SPP (Repeat)",
)
ax2.plot(
    nonspp_rep_exit.index,
    nonspp_rep_exit.values,
    color=COLOR_GREY,
    linestyle="--",
    marker="s",
    label="Non-SPP Shelter",
)
ax2.axvline(
    x=2022, color=COLOR_RED, linestyle="--", label="2022 Policy Shift"
)
ax2.set_title("Repeat Clients", fontweight="bold")
ax2.legend(
    loc="upper right",
    frameon=True,
    facecolor="white",
    edgecolor='lightgray',
    framealpha=0.95,
    fontsize=10,
    title_fontsize=10,
)

plt.tight_layout()

# export chart
export_path = output_dir / "Fig3_DynamicTrajectoriesPlot.jpg"
plt.savefig(export_path, dpi=600, bbox_inches="tight", format="jpg")
print(f"Chart successfully saved to: {export_path}")

plt.show()
```
