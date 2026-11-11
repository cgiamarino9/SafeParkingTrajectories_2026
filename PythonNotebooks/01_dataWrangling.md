# Part 1: Data wrangling notebook 
## This notebook imported the final csv and output dataframes for the three research questions.

# This is the final notebook to produce graphics and statistics for my SPP paper
## The first few code cells will be to bring in the cleaned and merged CSV.
I created six dataframes for the full 2017-2025 sample, first-time enrollments, and repeat enrollments, with known and unknown exits. 


```python
# system stuff
import os # paths and operating systems
import warnings # alerts
import textwrap # text wrapping
from pathlib import Path # work directory

# chart stuff 
import matplotlib.pyplot as plt # plotting charts
from matplotlib.lines import Line2D

# numbers and tests stuff 
from lifelines import KaplanMeierFitter # surival model for third research question 
import numpy as np # array processing
import pandas as pd #data frame wrangling

# statistics and plots stuff
import scipy.stats as stats
from scipy.stats import chi2_contingency, ttest_ind # import chi2 and ttests
import seaborn as sns # advanced charts
import statsmodels.api as sm # Logit and GLM
import statsmodels.formula.api as smf # OLS/GLM
from patsy import dmatrices # describe for technical appendices
from statsmodels.stats.outliers_influence import variance_inflation_factor # check multicollinearity
from drdid.drdid import drdid_panel, drdid_rc # doubly robust DID
from drdid.ipwd_did improt std_ipw_did_panel, std_ipw_did_rc # inverse probability weighted DID
from drdid.reg_did improt reg_did_panel, reg_did_rc # outcome regression DID

```


```python
# read in csv
# import final csv
final_sppAll = pd.read_csv("finalEnrollExit_stats092526.csv", parse_dates=["EntryDate","ExitDate","OperatingStartDate","OperatingEndDate"])

final_sppAll.head()

# subset columns 
cols_stats = ['EnrollmentID', 'PersonalID', 'EntryDate', 'ProjectID', 'LivingSituation', 'RentalSubsidyType', 'DisablingCondition', 'ExitDate', 'Destination', 'DaysEnrolled', 
              'MonthsEnrolled', 'YearsEnrolled', 'OperatingStartDate', 'OperatingEndDate', 'SPP', 'Address2', 'City', 'ZIP', 'YearsOpen', 'MonthsOpen', 'DaysOpen', 'StillOpen', 'UnitInventory', 'BedInventory', 
              'Year_Round', 'LocalFunding', 'Prior_Unsheltered', 'Prior_TemporaryStable', 'Prior_TemporaryUnstable', 'Prior_Permanent', 'Prior_Institutional', 'LOS_Days', 'ChronicHomelessness', 'Family', 'Couple', 
              'SingleParentWChild', 'BaseRankYear', 'ExitSameYear', 'PositiveExitHousing', 'PositiveExit', 'PositiveExitAll_PlusReturn', 'ExitClassification', 'b_permanent_housing', 'b_returned_to_vehicle', 'next_enrollment_permanent', 
              'Subsidy_Grouped', 'Destination_Grouped', 'BlackAfAmerican', 'White', 'Woman', 'VeteranStatus', 'LGBTQ', 'Age', 'EnrollmentNumber', 'IsRepeatEnrollment', 'EnrollmentType', 'ExitOutcome', 'PrioritizedAssessment', 
              'HousingNeedsAssessed', 'InPersonAssessment', 'Total_Acuity_Score', 'ExitYearRank', 'HighPriorityHousingNeeds', 'Employed_Entry', 'College_Entry', 'UnableToWork_Entry', 'DVSurvivor_Entry', 'GoodHealth_Entry', 'GoodHealth_Exit', 
              'count_health_benefits', 'count_housing_financial', 'count_jobs_employment', 'received_service_health_benefits', 'received_service_housing_financial', 'received_service_jobs_employment', 'BenefitsFromAnySource_Entry', 
              'Earned_Entry', 'InsuranceFromAnySource_Entry', 'OtherAssistance_Entry', 'RetireDisability_Entry', 'Total_Entry']

final_sppAll = final_sppAll[cols_stats].copy()

final_sppAll.head()
```


```python
# different exit types
# destination map
# adding five binary values as to where people were before entering SPP or non-SPP
# place not meant for human habitation 
final_sppAll["Exit_Unsheltered"] = final_sppAll["Destination"].isin([116]).astype(int)

# transitional housing, residential project or halfway house with no criteria, host home, satying with family or friends temporary tenure, staying in friend or family room, moved from HOPWA to HOPWA TH
final_sppAll["Exit_TemporaryStable"] = final_sppAll["Destination"].isin([302,329,332,312,313,336,335,327]).astype(int)


# Emergency shelter, safe haven, hotel/motel without voucher 
final_sppAll["Exit_TemporaryUnstable"]=final_sppAll["Destination"].isin([101,118,314]).astype(int)


# living with friends or family permanently, moved from HOPWA funded to HOPWA PH, rental by client without subsidy, rnetal by client without subsidy, owned by client with subsidy, owned by client without subsidy 
final_sppAll["Exit_Permanent"] = final_sppAll["Destination"].isin([422,423,426,410,435,421,411]).astype(int)


# foster home, hospital, jail, long-term nursing, psychiatric hospital, substance abuse 
final_sppAll["Exit_Institutional"] = final_sppAll["Destination"].isin([214,206,207,225,204,205]).astype(int)

# client doesnt know, client doesnt want to answer, missing, no exit interview completed, other, deceased, worker unable to determine 
final_sppAll["Exit_MissingOther"] = final_sppAll["Destination"].isin([8, 9, 99, 30,17,24,37]).astype(int)



final_sppAll.head()
    
```


```python
# convert ZIP from text to int
final_sppAll["ZIP"] = final_sppAll["ZIP"].astype(str).str.replace(r"\.0$", "", regex=True)
final_sppAll["ZIP"] = final_sppAll["ZIP"].replace({"None": "0", "nan": "0"}).fillna("0")
final_sppAll["ZIP"] = pd.to_numeric(final_sppAll["ZIP"], errors="coerce").fillna(0).astype("int64")
print(final_sppAll["ZIP"].dtype)
```


```python
print(list(final_sppAll.columns))
```


```python
# re-enrollment classification
final_sppAll = final_sppAll.sort_values(by=["PersonalID", "EntryDate"])

final_sppAll["Total_Lifetime_Enrollments"] = final_sppAll.groupby(
    "PersonalID"
)["EnrollmentID"].transform("count")

final_sppAll["True_Repeat_Client"] = np.where(
    final_sppAll["Total_Lifetime_Enrollments"] > 1, 1, 0
)

final_sppAll["Episode_Type"] = "One-Time Enrollee"
final_sppAll.loc[
    (final_sppAll["True_Repeat_Client"] == 1)
    & (final_sppAll["EnrollmentNumber"] == 1),
    "Episode_Type",
] = "Repeat Client: First Entry"
final_sppAll.loc[
    (final_sppAll["True_Repeat_Client"] == 1)
    & (final_sppAll["EnrollmentNumber"] > 1),
    "Episode_Type",
] = "Repeat Client: Subsequent Return"

custom_order = [
    "One-Time Enrollee",
    "Repeat Client: First Entry",
    "Repeat Client: Subsequent Return",
]

final_sppAll["Episode_Type"] = pd.Categorical(
    final_sppAll["Episode_Type"], categories=custom_order, ordered=True
)

# extract original versus non_original
df_filtered = final_sppAll[final_sppAll["Exit_MissingOther"] == 0].copy()

counts_orig = (
    final_sppAll.groupby("Episode_Type", observed=False)
    .size()
    .reset_index(name="Original Count")
)
counts_filt = (
    df_filtered.groupby("Episode_Type", observed=False)
    .size()
    .reset_index(name="Filtered Count")
)

counts_df = pd.merge(counts_orig, counts_filt, on="Episode_Type")
counts_df.rename(
    columns={"Episode_Type": "Episode Classification"}, inplace=True
)

total_orig = counts_df["Original Count"].sum()
total_filt = counts_df["Filtered Count"].sum()

counts_df["% of Original"] = (
    (counts_df["Original Count"] / total_orig) * 100
).round(1)
counts_df["% of Filtered"] = (
    (counts_df["Filtered Count"] / total_filt) * 100
).round(1)

# Descriptions for table
talking_points = {
    "One-Time Enrollee": (
        "People who used the program once to resolve their housing crisis"
        " and/or never returned to the system."
    ),
    "Repeat Client: First Entry": (
        "The initial program entry point for individuals who eventually"
        " returned to the system from 2017 to 2025."
    ),
    "Repeat Client: Subsequent Return": (
        "People who cycled back from first repeat entry for a second, third, or"
        " subsequent SPP or non-SPP episode."
    ),
}
counts_df["Description of Recidivism"] = counts_df[
    "Episode Classification"
].map(talking_points)

# restructure total row
total_row = pd.DataFrame([
    {
        "Episode Classification": "Total Rows",
        "Original Count": total_orig,
        "% of Original": 100.0,
        "Filtered Count": total_filt,
        "% of Filtered": 100.0,
        "Description of Recidivism": (
            f"Original N = {total_orig:,} | Filtered N = {total_filt:,}"
        ),
    }
])

summary_df = pd.concat([counts_df, total_row], ignore_index=True)

column_order = [
    "Episode Classification",
    "Original Count",
    "% of Original",
    "Filtered Count",
    "% of Filtered",
    "Description of Recidivism",
]
summary_df = summary_df[column_order]

summary_df["Original Count"] = summary_df["Original Count"].apply(
    lambda x: f"{x:,}"
)
summary_df["% of Original"] = summary_df["% of Original"].apply(
    lambda x: f"{x}%"
)
summary_df["Filtered Count"] = summary_df["Filtered Count"].apply(
    lambda x: f"{x:,}"
)
summary_df["% of Filtered"] = summary_df["% of Filtered"].apply(
    lambda x: f"{x}%"
)

# stylize the table 
plt.rcParams["font.family"] = "serif"
plt.rcParams["font.serif"] = ["Times New Roman", "Times", "DejaVu Serif"]

fig, ax = plt.subplots(figsize=(16, 8), dpi=300)
ax.axis("off")

# Fix text clipping
summary_df["Episode Classification"] = summary_df[
    "Episode Classification"
].apply(lambda x: "\n".join(textwrap.wrap(str(x), width=18)))
summary_df["Description of Recidivism"] = summary_df[
    "Description of Recidivism"
].apply(lambda x: "\n".join(textwrap.wrap(str(x), width=32)))

header_labels = [
    "Episode Classification",
    "Original\nEnrollments",
    "% of Original\nClients",
    "Filtered\nEnrollments",
    "% of Filtered\nClients",
    "Description of Recidivism",
]

table = ax.table(
    cellText=summary_df.values,
    colLabels=header_labels,
    cellLoc="left",
    loc="center",
)

table.auto_set_font_size(False)

# Style cells
for key, cell in table.get_celld().items():
  row_idx, col_idx = key
  cell.set_fontsize(10)

  if col_idx == 0:
    cell.set_width(0.20)  # Episode Classification
  elif col_idx == 1:
    cell.set_width(0.11)  # Original Count
  elif col_idx == 2:
    cell.set_width(0.12)  # % of Original
  elif col_idx == 3:
    cell.set_width(0.11)  # Filtered Count
  elif col_idx == 4:
    cell.set_width(0.12)  # % of Filtered
  elif col_idx == 5:
    cell.set_width(0.34)  # Description of Recidivism

  if row_idx == 0:
    cell.set_height(0.14)
  else:
    cell.set_height(0.26)

  cell.set_text_props(va="center", ha="left", multialignment="left")
  cell.pad = 0.04

  if row_idx == 0:
    cell.set_text_props(weight="bold", color="white", va="center", ha="left")
    cell.set_facecolor("#1E4D2B")
  else:
    cell.set_facecolor("white")
    if row_idx == len(summary_df):
      cell.set_text_props(weight="bold", color="black", va="center", ha="left")

# export jpg
os.makedirs("FinalHousingStudies", exist_ok=True)
plt.savefig(
    "FinalHousingStudies/00_SampleSizeDescriptionTable.jpg",
    bbox_inches="tight",
    dpi=300,
    format="jpg",
)
plt.close()

print(
    "Success! Your updated comparative table has been saved to"
    " FinalHousingStudies/00_SampleSizeDescriptionTable.jpg"
)
```


```python
print(list(final_sppAll.columns))
```


```python
final_sppAll.shape
```


```python
# check unique values and frequencies for Destination
counts = final_sppAll["Destination"].value_counts(dropna=False)
percentages = final_sppAll["Destination"].value_counts(dropna=False, normalize=True) * 100

# combine into data frame
destination = pd.DataFrame({
    "Raw Count": counts, 
    "Percentage": percentages.round(2)
})

print(destination)
```


```python
# check no exit interview
dest30 = final_sppAll[final_sppAll["Destination"]==30]

counts30 = dest30["SPP"].value_counts()

totals = final_sppAll["SPP"].value_counts()

rates = pd.DataFrame({
    "Total Incomplete Exits": counts30,
    "Total Program Enrollments": totals,
    "Incomplete Exit Rate": ((counts30 / totals) * 100).round(2)
})

rates.index = rates.index.map({0: "Non-SPP Shelter", 1: "SPP"})
print(rates)
```


```python
# check no exit interview
dest9 = final_sppAll[final_sppAll["Destination"]==9]

counts9 = dest9["SPP"].value_counts()

totals = final_sppAll["SPP"].value_counts()

rates = pd.DataFrame({
    "Total Refused Answers": counts9,
    "Total Program Enrollments": totals,
    "Refused Answer Rate": ((counts9 / totals) * 100).round(2)
})

rates.index = rates.index.map({0: "Non-SPP Shelter", 1: "SPP"})
print(rates)
```

## Creating six dataframes below


```python
# creating all three final dataframes to answer research questions

# filtering out for known outcomes
# full dataframe for 2017 to 2025
df_all = final_sppAll.copy()

df_all.shape
```


```python
df_all.head()
```


```python
print(list(df_all.columns))
```


```python
df_all["CalendarYear"] = pd.to_datetime(df_all["EntryDate"], format="%m%d%Y", errors="coerce").dt.year
df_all.head()
```


```python
df_all['Post2022'] = (df_all['EntryDate'].dt.year >= 2022).astype(int)
```


```python
if "Earned_Entry" in df_all.columns: 
    df_all["Earned_Entry"] = df_all["Earned_Entry"]/ 1000.0

if "DaysEnrolled" in df_all.columns: 
    df_all["DaysEnrolled"] = df_all["DaysEnrolled"] / 100.0

    
```


```python
# identify continuous variables and check them 
raw_continuous_vars = [
    "Age",
    "Earned_Entry",
    "Total_Lifetime_Enrollments",
    "DaysEnrolled",
]

# Identify available continuous columns in df_all
cols_to_check = [c for c in raw_continuous_vars if c in df_all.columns]

# Construct boolean mask where all continuous variables are >= 0 (ignoring NaNs if present)
non_negative_mask = (df_all[cols_to_check] >= 0).all(axis=1)

# Apply filter
df_all = df_all[non_negative_mask].copy()


```


```python
df_all.shape
```


```python
# overwrite disabling condition
df_all['DisablingCondition']  = np.where(
    df_all['DisablingCondition'] == 1, 1, 0
)
df_all.head()
```


```python
# sort again 
df_all = df_all.sort_values(by=['PersonalID', 'EntryDate']).reset_index(
    drop=True
)

# get previous exit date
df_all['Prev_ExitDate'] = df_all.groupby('PersonalID')['ExitDate'].shift(
    1
)

# create recidivism column
df_all['DaysPostExit'] = (
    df_all['EntryDate'] - df_all['Prev_ExitDate']
).dt.days

# check calculation
df_all.head(20)
```


```python
# flag negative overlapping enrollment
df_all['Is_Overalapping_Enrollment'] = (df_all['DaysPostExit'] < 0).astype(int)

# set true post-exit days to zero if already enrolled
df_all['CleanDaysPostExit'] = df_all['DaysPostExit'].apply(
    lambda x: 0 if x < 0 else x
)
```


```python
df_all.head(20)
```


```python
print(list(df_all.columns))
```


```python
# add sequential number for enrollment
df_all['Enrollment_Sequence'] = df_all.groupby('PersonalID').cumcount() + 1
df_all.head()
```


```python
spp_missing_summary = df_all.groupby('SPP')['Exit_MissingOther'].agg(
    Total_Enrollees='count',
    Missing_Exits='sum',
    Missing_Rate_Pct=lambda x: x.mean() * 100
).reset_index()

spp_missing_summary['Program'] = spp_missing_summary['SPP'].map({0: 'Non-SPP Shelter', 1: 'Safe Parking (SPP)'})
print("=== Missing Exit Rates by Program Type ===")
print(spp_missing_summary[['Program', 'Total_Enrollees', 'Missing_Exits', 'Missing_Rate_Pct']].to_string(index=False))

# 2. Filter out missing exits first
df_clean = df_all[df_all['Exit_MissingOther'] == 0].copy()

# 3. Create df_first and df_repeat from clean dataset
df_first = df_clean[df_clean['True_Repeat_Client'] == 0].copy()
df_repeat = df_clean[df_clean['True_Repeat_Client'] == 1].copy()

print(f"\n=== Clean Cohort Sample Sizes ===")
print(f"First-Time Cohort (df_first): {len(df_first):,} rows")
print(f"Repeat Cohort (df_repeat):     {len(df_repeat):,} rows")
```


```python
df_all1 = df_all[df_all["Exit_MissingOther"] == 0].copy()
df_all1.shape
```


```python
# one record per unique client 
df_first = df_all1[
    df_all1["Episode_Type"].isin(["Repeat Client: First Entry", "One-Time Enrollee"])
].copy()

# drop duplicates

df_first = df_first.drop_duplicates(subset=["PersonalID"], keep="first")

df_first.shape
```


```python
df_first.head(20)
```


```python
# repeat enrollments
df_repeat = df_all1[df_all1["Total_Lifetime_Enrollments"]>1].copy()
df_repeat.shape
```


```python
df_repeat.head(20)
```


```python
print(list(df_all1.columns))
```


```python
# df_all shape
df_all1.shape
```


```python
# first time shape
df_first.shape
```


```python
# repeat shape
df_repeat.shape
```


```python
df_all.shape
```


```python
# one record per unique client 
df_first_all = df_all[
    df_all["Episode_Type"].isin(["Repeat Client: First Entry", "One-Time Enrollee"])
].copy()

# drop duplicates

df_first_all = df_first_all.drop_duplicates(subset=["PersonalID"], keep="first")

df_first_all.shape
```


```python
# repeat enrollments
df_repeat_all = df_all[df_all["Total_Lifetime_Enrollments"]>1].copy()
df_repeat_all.shape
```

## Final dataframes
1. df_all (missing exits not filtered out)
1. df_all1 (missing exits filtered out)
1. df_first (missing exits filtered out)
1. df_first_all (missing exits not filtered out)
1. df_repeat (missing exits filtered out)
1. df_repeat_all (missing exits not filtered out)


```python
# Descriptive statistics for projects 
project_counts = pd.DataFrame({
    'df_all (Unfiltered)': df_all.groupby('SPP')['ProjectID'].nunique(),
    'df_all1 (Filtered)': df_all1.groupby('SPP')['ProjectID'].nunique(),
})

project_counts.index = ['Non-SPP (SPP=0)', 'SPP (SPP=1)']
print(project_counts)
```


```python
# check unique enrollments and personalIDs for projects
project_summary = (
    df_all.groupby(['SPP', 'ProjectID'])
    .agg(
        Total_Enrollments=('EnrollmentID', 'nunique'),
        Total_Clients=('PersonalID', 'nunique'),
    ).reset_index()
)

# calculate descriptive stats
descriptive_stats = (
    project_summary.groupby('SPP')
    .agg(
        Num_Projects=('ProjectID', 'count'),
        Total_Clients=('Total_Clients', 'sum'),
        Mean_Clients_Per_Project=('Total_Clients', 'mean'),
        Std_Clients=('Total_Clients', 'std'),
        Median_Clients=('Total_Clients', 'median'),
        Min_Clients=('Total_Clients', 'min'),
        Max_Clients=('Total_Clients', 'max'),
        Total_Enrollments=('Total_Enrollments', 'sum'),
        Mean_Enrollments_Per_Project=('Total_Enrollments', 'mean'),
        Std_Enrollments=('Total_Enrollments', 'std'),
        Median_Enrollments=('Total_Enrollments', 'median'),
        Min_Enrollments=('Total_Enrollments', 'min'),
        Max_Enrollments=('Total_Enrollments', 'max'),
    ).round(2)
)

descriptive_stats.index = ['Non-SPP Shelter', 'Safe Parking Programs']

# print
print(descriptive_stats.T)
```


```python
# check missing data
missing_exits = df_all['ExitDate'].isna().sum()
print(f"MissingExitDate values in df_all: {missing_exits}")

# check one-to-one enrollment
print(
    f"Total Rows: {len(df_all)} | Unique EnrollmentIDs:"
    f" {df_all['EnrollmentID'].nunique()}"
)

print(df_all['Exit_Permanent'].value_counts(dropna=False))
```


```python
# output directory
output_dir = "FinalHousingStudies"
os.makedirs(output_dir, exist_ok=True)

# stats per year
yearly_project_summary = (
    df_all.groupby(['CalendarYear', 'SPP', 'ProjectID'])
    .agg(
        Annual_Clients=('PersonalID', 'nunique'),
        Annual_Enrollments=('EnrollmentID', 'nunique'),
        #Annual_Exits=('ExitDate', 'count'),
        Annual_Permanent_Exits=('Exit_Permanent', 'sum')
    )
    .reset_index()
)

# desc stats
yearly_stats = (
    yearly_project_summary.groupby(['CalendarYear', 'SPP'])
    .agg(
        Active_Projects=('ProjectID', 'nunique'),
        Total_Clients=('Annual_Clients', 'sum'),
        Mean_Clients_Per_Project=('Annual_Clients', 'mean'),
        Median_Clients_Per_Project=('Annual_Clients', 'median'),
        Total_Enrollments=('Annual_Enrollments', 'sum'),
        Mean_Enrollments_Per_Project=('Annual_Enrollments', 'mean'),
        #Total_Exits=('Annual_Exits', 'sum'),
        #Mean_Exits_Per_Project=('Annual_Exits', 'mean'),
        Total_Permanent_Exits=('Annual_Permanent_Exits', 'sum'), 
        Mean_Permanent_Exits_Per_Project=('Annual_Permanent_Exits', 'mean'),
    )
    .round(2)
    .reset_index()
)

# calaculate exit rate percentage
yearly_stats['Permanent_Exit_Rate_Pct'] = (
    (yearly_stats['Total_Permanent_Exits'] / yearly_stats['Total_Enrollments'])
    * 100
).round(2)

# map spp labels
yearly_stats['Program_Type'] = yearly_stats['SPP'].map(
    {0: 'Non-SPP Shelter', 1: 'Safe Parking'}
)

cols = [
    'CalendarYear',
    'Program_Type',
    'Active_Projects',
    'Total_Clients',
    'Mean_Clients_Per_Project',
    'Median_Clients_Per_Project',
    'Total_Enrollments',
    'Mean_Enrollments_Per_Project',
    #'Total_Exits', 
    #'Mean_Exits_Per_Project', 
    'Total_Permanent_Exits',
    'Mean_Permanent_Exits_Per_Project',
    'Permanent_Exit_Rate_Pct',
]

# rename dictionary
rename_dict = {
    'CalendarYear': 'Calendar Year',
    'Program_Type': 'Program Type',
    'Active_Projects': 'Active Projects',
    'Total_Clients': 'Total Clients',
    'Mean_Clients_Per_Project': 'Average Clients per Project',
    'Median_Clients_Per_Project': 'Median Clients per Project',
    'Total_Enrollments': 'Total Enrollments',
    'Mean_Enrollments_Per_Project': 'Mean Enrollments Per Project',
    #'Total_Exits': 'Total Exits', 
    #'Mean_Exits_Per_Project': 'Mean Exits Per Project',
    'Total_Permanent_Exits': 'Total Permanent Exits', 
    'Mean_Permanent_Exits_Per_Project': 'Mean Permanent Exits Per Project',
    'Permanent_Exit_Rate_Pct': 'Permanent Exit Rate (%)',
}

output_path = os.path.join(output_dir, '00_DescStats_Unfiltered.md')
yearly_stats[cols].rename(columns=rename_dict).to_markdown(
    output_path, index=False
)

# print
print("Annual Breakdown by Year and Program Type")
print(yearly_stats[cols].to_string(index=False))
```


```python
# output directory
output_dir = "FinalHousingStudies"
os.makedirs(output_dir, exist_ok=True)

# stats per year
yearly_project_summary = (
    df_all1.groupby(['CalendarYear', 'SPP', 'ProjectID'])
    .agg(
        Annual_Clients=('PersonalID', 'nunique'),
        Annual_Enrollments=('EnrollmentID', 'nunique'),
        #Annual_Exits=('ExitDate', 'count'),
        Annual_Permanent_Exits=('Exit_Permanent', 'sum')
    )
    .reset_index()
)

# desc stats
yearly_stats = (
    yearly_project_summary.groupby(['CalendarYear', 'SPP'])
    .agg(
        Active_Projects=('ProjectID', 'nunique'),
        Total_Clients=('Annual_Clients', 'sum'),
        Mean_Clients_Per_Project=('Annual_Clients', 'mean'),
        Median_Clients_Per_Project=('Annual_Clients', 'median'),
        Total_Enrollments=('Annual_Enrollments', 'sum'),
        Mean_Enrollments_Per_Project=('Annual_Enrollments', 'mean'),
        #Total_Exits=('Annual_Exits', 'sum'),
        #Mean_Exits_Per_Project=('Annual_Exits', 'mean'),
        Total_Permanent_Exits=('Annual_Permanent_Exits', 'sum'), 
        Mean_Permanent_Exits_Per_Project=('Annual_Permanent_Exits', 'mean'),
    )
    .round(2)
    .reset_index()
)

# calaculate exit rate percentage
yearly_stats['Permanent_Exit_Rate_Pct'] = (
    (yearly_stats['Total_Permanent_Exits'] / yearly_stats['Total_Enrollments'])
    * 100
).round(2)

# map spp labels
yearly_stats['Program_Type'] = yearly_stats['SPP'].map(
    {0: 'Non-SPP Shelter', 1: 'Safe Parking'}
)

cols = [
    'CalendarYear',
    'Program_Type',
    'Active_Projects',
    'Total_Clients',
    'Mean_Clients_Per_Project',
    'Median_Clients_Per_Project',
    'Total_Enrollments',
    'Mean_Enrollments_Per_Project',
    #'Total_Exits', 
    #'Mean_Exits_Per_Project', 
    'Total_Permanent_Exits',
    'Mean_Permanent_Exits_Per_Project',
    'Permanent_Exit_Rate_Pct',
]

# rename dictionary
rename_dict = {
    'CalendarYear': 'Calendar Year',
    'Program_Type': 'Program Type',
    'Active_Projects': 'Active Projects',
    'Total_Clients': 'Total Clients',
    'Mean_Clients_Per_Project': 'Average Clients per Project',
    'Median_Clients_Per_Project': 'Median Clients per Project',
    'Total_Enrollments': 'Total Enrollments',
    'Mean_Enrollments_Per_Project': 'Mean Enrollments Per Project',
    #'Total_Exits': 'Total Exits', 
    #'Mean_Exits_Per_Project': 'Mean Exits Per Project',
    'Total_Permanent_Exits': 'Total Permanent Exits', 
    'Mean_Permanent_Exits_Per_Project': 'Mean Permanent Exits Per Project',
    'Permanent_Exit_Rate_Pct': 'Permanent Exit Rate (%)',
}

output_path = os.path.join(output_dir, '00_DescStats_Filtered.md')
yearly_stats[cols].rename(columns=rename_dict).to_markdown(
    output_path, index=False
)

# print
print("Annual Breakdown by Year and Program Type")
print(yearly_stats[cols].to_string(index=False))
```

|   Calendar Year | Program Type    |   Active Projects |   Total Clients |   Average Clients per Project |   Median Clients per Project |   Total Enrollments |   Mean Enrollments Per Project |   Total Permanent Exits |   Mean Permanent Exits Per Project |   Permanent Exit Rate (%) |
|----------------:|:----------------|------------------:|----------------:|------------------------------:|-----------------------------:|--------------------:|-------------------------------:|------------------------:|-----------------------------------:|--------------------------:|
|            2017 | Non-SPP Shelter |                84 |            2422 |                         28.83 |                         13   |                2443 |                          29.08 |                     567 |                               6.75 |                     23.21 |
|            2017 | Safe Parking    |                 1 |              17 |                         17    |                         17   |                  17 |                          17    |                       3 |                               3    |                     17.65 |
|            2018 | Non-SPP Shelter |               125 |            8937 |                         71.5  |                         32   |                9327 |                          74.62 |                    1673 |                              13.38 |                     17.94 |
|            2018 | Safe Parking    |                 7 |             126 |                         18    |                         17   |                 133 |                          19    |                      16 |                               2.29 |                     12.03 |
|            2019 | Non-SPP Shelter |               173 |           12557 |                         72.58 |                         31   |               13098 |                          75.71 |                    1932 |                              11.17 |                     14.75 |
|            2019 | Safe Parking    |                16 |             564 |                         35.25 |                         30   |                 752 |                          47    |                      45 |                               2.81 |                      5.98 |
|            2020 | Non-SPP Shelter |               295 |           21852 |                         74.07 |                         39   |               22851 |                          77.46 |                    3030 |                              10.27 |                     13.26 |
|            2020 | Safe Parking    |                22 |             947 |                         43.05 |                         33   |                1017 |                          46.23 |                      89 |                               4.05 |                      8.75 |
|            2021 | Non-SPP Shelter |               336 |           23548 |                         70.08 |                         40.5 |               25097 |                          74.69 |                    4719 |                              14.04 |                     18.8  |
|            2021 | Safe Parking    |                27 |            1143 |                         42.33 |                         37   |                1217 |                          45.07 |                     123 |                               4.56 |                     10.11 |
|            2022 | Non-SPP Shelter |               310 |           20716 |                         66.83 |                         35   |               21770 |                          70.23 |                    3503 |                              11.3  |                     16.09 |
|            2022 | Safe Parking    |                27 |             995 |                         36.85 |                         39   |                1093 |                          40.48 |                     151 |                               5.59 |                     13.82 |
|            2023 | Non-SPP Shelter |               260 |           19977 |                         76.83 |                         43   |               20933 |                          80.51 |                    3820 |                              14.69 |                     18.25 |
|            2023 | Safe Parking    |                22 |             774 |                         35.18 |                         32.5 |                 832 |                          37.82 |                     144 |                               6.55 |                     17.31 |
|            2024 | Non-SPP Shelter |               244 |           15592 |                         63.9  |                         44   |               16084 |                          65.92 |                    2182 |                               8.94 |                     13.57 |
|            2024 | Safe Parking    |                19 |             639 |                         33.63 |                         36   |                 679 |                          35.74 |                     108 |                               5.68 |                     15.91 |
|            2025 | Non-SPP Shelter |               149 |            1541 |                         10.34 |                          3   |                1640 |                          11.01 |                      67 |                               0.45 |                      4.09 |
|            2025 | Safe Parking    |                17 |             205 |                         12.06 |                         10   |                 210 |                          12.35 |                      18 |                               1.06 |                      8.57 |
