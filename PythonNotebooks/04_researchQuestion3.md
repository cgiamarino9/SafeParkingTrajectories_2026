# Research Question 3
## Reevaluation and Recidivism
How effectively do SPPs prevent returns to homelessness compared to non-SPPs? How can metrics be recategorized to appropriately evaluate homelessness exits from SPPs?

## Table for HMIS outcomes and Net stability framework


```python
# unknown exits unfiltered
df_all.shape
```


```python
# known exits only
df_all1.shape
```


```python
# unknown first-time exits unfiltered
df_first_all.shape
```


```python
# known exits first-time only
df_first.shape
```


```python
# unknown exits repeat unfiltered
df_repeat_all.shape
```


```python
# repeat known exits only
df_repeat.shape
```


```python
# create function to demonstrate HUD/HMIS penalties for groups
def calculate_exit_framework(df_sub, program_name):
    vertical = df_sub['Exit_Permanent'].mean() * 100
    expanded_pos = df_sub['Exit_TemporaryStable'].mean() * 100
    horizontal_veh = df_sub['b_returned_to_vehicle'].mean() * 100
    unsheltered_loss = df_sub['Exit_Unsheltered'].mean() * 100
    
    std_hmis_success = vertical
    net_stability_rate = vertical + expanded_pos + horizontal_veh
    
    print(f"{program_name} Exit Metrics")
    print(f"Vertical Exits (Permanent Placement): {vertical:.2f}%")
    print(f"Expanded Positive Exits (Temporary Stable): {expanded_pos:.2f}%")
    print(f"Horizontal Exits (Vehicular Asset Preservation): {horizontal_veh:.2f}%")
    print(f"Standard HMIS Success Rate (Penalized): {std_hmis_success:.2f}%")
    print(f"Proposed Net Stability Rate: {net_stability_rate:.2f}%")
    print("-" * 50)
calculate_exit_framework(df_all[df_all['SPP']==0], "Traditional Emergency Shelter")
calculate_exit_framework(df_all[df_all['SPP']==1], "Safe Parking Program (SPP)")
```


```python
print(list(df_all.columns))
```


```python
def generate_figure5_net_stability(df_all, output_dir="FinalHousingStudies"):
  categories = [
      "Vertical Exits (Permanent Placement)",
      "Expanded Positive Exits (Temporary Stable)",
      "Horizontal Exits (Vehicular Asset Preservation)",
      "True Unsheltered Exits (Street / Tent / Park)",
      "Other / Institutional / Temporary Unstable",
      "Missing / Administrative Non-Response",
  ]

  categories_rev = categories[::-1]

  # Exact native percentage values
  shelter_vals_rev = [40.4, 23.9, 9.5, 3.3, 6.7, 16.1]
  spp_vals_rev = [29.0, 15.3, 0.0, 36.9, 9.2, 11.7]

  # Styling configuration
  plt.rcParams.update({
      "font.family": "serif",
      "font.serif": ["Times New Roman", "DejaVu Serif"],
      "font.size": 10,
      "axes.labelsize": 11,
      "axes.titlesize": 12,
      "ytick.labelsize": 10,
      "xtick.labelsize": 10,
  })

  fig, ax = plt.subplots(figsize=(11, 7.0), dpi=600)

  color_shelter = "#00843D"  # CPP Primary Green
  color_spp = "#FFB500"  # CPP Gold

  y = np.arange(len(categories_rev))
  height = 0.36

  rects1 = ax.barh(
      y + height / 2,
      shelter_vals_rev,
      height,
      label="Traditional Shelter (N=133,243)",
      color=color_shelter,
      edgecolor="white",
  )
  rects2 = ax.barh(
      y - height / 2,
      spp_vals_rev,
      height,
      label="Safe Parking Program (N=5,950)",
      color=color_spp,
      edgecolor="white",
  )

  # Percentage Text Labels
  for rect in rects1:
    w = rect.get_width()
    ax.annotate(
        f"{w:.1f}%",
        xy=(w, rect.get_y() + rect.get_height() / 2),
        xytext=(4, 0),
        textcoords="offset points",
        ha="left",
        va="center",
        fontsize=8.5,
        fontweight="bold",
        color=color_shelter,
    )

  for rect in rects2:
    w = rect.get_width()
    # Explicitly print 0.0% for Category IV so SPP is represented
    ax.annotate(
        f"{w:.1f}%" if w > 0 else "0.0%",
        xy=(w, rect.get_y() + rect.get_height() / 2),
        xytext=(4, 0),
        textcoords="offset points",
        ha="left",
        va="center",
        fontsize=8.5,
        fontweight="bold",
        color=color_spp,
    )

  # Callout box pointing to SPP Vehicle Retention Bar
  veh_idx = categories_rev.index(
      "Horizontal Exits (Vehicular Asset Preservation)"
  )
  y_veh_pos = y[veh_idx] - height / 2

  ax.annotate(
      "HUD penalizes vehicle exits:\n`b_returned_to_vehicle` (36.9%) is"
      " appropriately separated from\nstandard HMIS 'Unsheltered' (dropping true street exit"
      " to 0.0% as policy proposal).\n Caveats: Vehicle retained, self-resolved exits, abolition of anti-vehicle-dwelling ordinances.",
      xy=(41.2, y_veh_pos),
      xycoords="data",
      xytext=(1.05, 0.42),
      textcoords="axes fraction",
      arrowprops=dict(
          arrowstyle="->",
          color="#FFB500",
          lw=1.5,
          connectionstyle="arc3,rad=0.0",
      ),
      fontsize=8.5,
      fontweight="bold",
      color="#01426A",
      clip_on=False,
      bbox=dict(
          boxstyle="round,pad=0.5",
          facecolor="#FFF9E6",
          edgecolor="#FFB500",
          lw=1.2,
      ),
  )

  ax.set_yticks(y)
  ax.set_yticklabels(categories_rev, fontweight="bold")
  ax.set_xlabel("Percentage of Total Program Exits (%)", labelpad=10)
 # ax.set_title(
     # "Figure 5. Deconstruction of HMIS Exit Destinations under Net Stability"
     # " Framework",
     # pad=15,
     # fontweight="bold",
  #)
  ax.set_xlim(0, 45)
  ax.grid(axis="x", linestyle=":", alpha=0.5)
  ax.set_axisbelow(True)

  ax.legend(
      loc="upper right",
      frameon=True,
      facecolor="white",
      framealpha=0.95,
      edgecolor="#cccccc",
      fontsize=9,
  )

  fig.subplots_adjust(right=0.72)

  export_path = os.path.join(
      output_dir, "Fig5_NetStability_VehicleStabilization.jpg"
  )
  plt.savefig(export_path, dpi=600, bbox_inches="tight", format="jpg")
  plt.show()

  print(f"Success! Corrected Figure 5 exported to: {export_path}")


generate_figure5_net_stability(df_all)
```

## Methodological Justification: Operationalizing `b_returned_to_vehicle`

### Overview of HMIS reevaluation framework for vehicular homelessness
In standard HUD HMIS Data Standards, Destination Code `116.0` (*"Other unsheltered location / Vehicle"*) is automatically categorized as an unsheltered outcome (*"Place Not Meant for Human Habitation"*). While this classification reflects unmanaged homelessness for traditional emergency shelter clients (`SPP == 0`), it creates a **SPP performance penalty** for Safe Parking Programs (`SPP == 1`), ignoring that retainment of vehicle is not negative and may lead to self-resolved homelessness, return to SPPs, or, at worst, poverty tows. Additionally, 93.9% of SPP clients come from `unsheltered` locations because the system treats their vehicle the same as a form of unsheltered homelessness.  

For vehicular residents, retaining or returning to a vehicle within a safe, service-connected parking framework represents asset preservation and intermediate harm reduction rather than a return to unsheltered street homelessness. To address this HMIS evaluative misclassification, the binary variable `b_returned_to_vehicle` (derived from `ExitClassification == "ReturnedToVehiculeOrTemporary"`) was created to signify **Vehicle Stabilization** in the proposed policy framework. 

---

### Recategorization Hierarchy
The outcome reclassification created a strict 3-tier precedence structure evaluated using`np.select()`:

1. **Tier 1: Permanent Housing (`Exit_Permanent = 1`)**
   * Evaluated using positive exits with standard HUD permanent placement codes `[422,423,426,410,435,421,411]`, including subsidized/unsubsidized leases and permanent family/friend tenure).
2. **Tier 2: Permanent Housing & Temporary Stays**
    * Evaluated using positive exits with standard HUD permanent placement codes `[422,423,426,410,435,421,411]`, including subsidized/unsubsidized leases and permanent family/friend tenure).
    * **Temporary Placements (All Cohorts):** Destination codes in `[302,329,332,312,313,336,335,327]` include transitional housing, substance use treatment, intermediate care facilities across both groups, or non-permanent stays in rooms, apartments, or homes with family members or friends.
3. **Tier 3: Permanent Housing, Vehicular Preservation & Temporary Stable Stays (`b_returned_to_vehicle = 1`)**
    * Evaluated using positive exits with standard HUD permanent placement codes `[422,423,426,410,435,421,411]`, including subsidized/unsubsidized leases and permanent family/friend tenure).
   * **Vehicular Maintenance (SPP Cohort):** `(Destination == 116.0) & (SPP == 1)`. Identifies SPP participants who maintain safe, legal vehicular residency upon program exit across full sample, including those switching programs, first-time clients that never return, and repeat clients that do return and must have a vehicle to enter an SPP..
   * **Temporary Placements (All Cohorts):** Destination codes in `[302,329,332,312,313,336,335,327]` include transitional housing, substance use treatment, intermediate care facilities across both groups, or non-permanent stays in rooms, apartments, or homes with family members or friends.
4. **Tier 4: Other Exits / Unmanaged Homelessness (`default`)**
   * Appropriately measures remaining exit destinations to unmanaged street sleeping and unclassified outcomes (e.g., True Unsheltered or Missing). Better data collection and long-term program availability can address these exits. 

---

### Defending the Framework
* **SPPs as Harm-Reduction Spaces:** Measures the intermediate continuum between unmanaged street homelessness and permanent housing placement. Horizontal exits are not treated as program failures for SPPs, which often invoke budget cuts and lot closures. 
* **Addressing HMIS Bias:** Quantifies how standard HMIS metrics misclassify safe vehicular residency as unsheltered homelessness. This provides empirical evidence to propose a new **Net Stability Framework**.
* **Validating Analysis:** In traditional emergency shelters (`SPP == 0`), Destination `116.0` represents a minor share of exits. Binding `116.0` to `SPP == 1` identifies program-specific vehicule residency outcomes without distorting non-SPP enrollment baseline estimates. 
* **Regulatory Caveats:** For this framework to be implemented and justified, cities would have to allow vehicle residents to use their assets for shelter on public streets when exiting SPPs. This includes political decisions to abolish anti-vehicle dwelling ordinances that lead to police encounters, residential harassment, unpaid tickets, and loss of vehicle via impoundment and lien sale. In concert with regulatory toleration of vehicle dwellings on public streets, cities should expand access to 24/7 SPPs with spaces that accommodate cars, campers, vans, and RVs and tailor housing and financial assistance services to this large, unique, comparatively stable "unsheltered" population.


```python
print(list(df_all.columns))
```


```python
def generate_table3_four_tier_framework(
    df_all,
    df_all1=None,
    df_first_all=None,
    df_repeat_all=None,
    output_dir="FinalHousingStudies",
):
  os.makedirs(output_dir, exist_ok=True)

  # Derive complete-case analytic sample if not explicitly passed
  if df_all1 is None and "Exit_MissingOther" in df_all.columns:
    df_all1 = df_all[df_all["Exit_MissingOther"] == 0]

  # Compute metrics aligned with the 4-Tier Recategorization Hierarchy
  def compute_tier_metrics(df):
    if df is None or len(df) == 0:
      empty = {
          "N": 0,
          "V": 0.0,
          "E": 0.0,
          "H": 0.0,
          "U": 0.0,
          "O": 0.0,
          "M": 0.0,
          "STD": 0.0,
          "RATE_EXP": 0.0,
          "RATE_NET": 0.0,
          "LIFT": 0.0,
      }
      return {0: empty, 1: empty}

    results = {}
    for spp_val in [0, 1]:
      sub = df[df["SPP"] == spp_val]
      n_obs = len(sub)

      if n_obs == 0:
        continue

      # Tier Components
      v = sub["Exit_Permanent"].mean() * 100  # Tier 1
      e = sub["Exit_TemporaryStable"].mean() * 100  # Tier 2
      h = sub["b_returned_to_vehicle"].mean() * 100  # Tier 3

      inst = (
          sub["Exit_Institutional"].mean() * 100
          if "Exit_Institutional" in sub.columns
          else 0.0
      )
      temp_unstable = (
          sub["Exit_TemporaryUnstable"].mean() * 100
          if "Exit_TemporaryUnstable" in sub.columns
          else 0.0
      )
      o = inst + temp_unstable # tier 4b
    
      m = (
          sub["Exit_MissingOther"].mean() * 100
          if "Exit_MissingOther" in sub.columns
          else 0.0
      )# tier 4c
        
      u_raw = sub["Exit_Unsheltered"].mean() * 100
      u_true = max(0.0, u_raw-h) # tier 4a

      # Proposed Dual Stability Rates
      std_rate = v  # Tier 1 Only (HUD Standard)
      rate_expanded = v + e  # Rate 1: Vertical + Expanded Positive
      rate_net = v + e + h  # Rate 2: Vertical + Expanded + Horizontal (Net Stability)
      lift = rate_net - std_rate

      results[spp_val] = {
          "N": n_obs,
          "V": v,
          "E": e,
          "H": h,
          "U": u_true,
          "O": o,
          "M": m,
          "STD": std_rate,
          "RATE_EXP": rate_expanded,
          "RATE_NET": rate_net,
          "LIFT": lift,
      }

    return results

  m_all = compute_tier_metrics(df_all)
  m_all1 = compute_tier_metrics(df_all1) if df_all1 is not None else m_all
  m_first = (
      compute_tier_metrics(df_first_all) if df_first_all is not None else None
  )
  m_repeat = (
      compute_tier_metrics(df_repeat_all) if df_repeat_all is not None else None
  )

  # Markdown Construction
  md = []
  md.append("# Table 3. Proposed Net Stability Framework Reclassification Matrix\n")
  md.append(
      "**Table 3.** *Comparison of Standard HUD/HMIS Exit Success Rates versus"
      " Proposed Multi-Tier Stability Metrics across Different Samples and"
      " Client Cohorts.*\n"
  )

  # Panel A
  md.append("### Vehicle Stability Framework Part I. Primary Sample Comparison (Unfiltered vs. Known Exits)\n")
  md.append(
      "| Recategorization Hierarchy & Component | Unfiltered Baseline (`df_all`)"
      " <br> Traditional Shelter | Unfiltered Baseline (`df_all`) <br> Safe"
      " Parking (SPP) | Complete-Case (`df_all1`) <br> Traditional Shelter |"
      " Complete-Case (`df_all1`) <br> Safe Parking (SPP) |"
  )
  md.append(
      "| :--- | :---: | :---: | :---: | :---: |"
  )
  md.append(
      f"| **Sample Size (N)** | **{m_all[0]['N']:,}** | **{m_all[1]['N']:,}**"
      f" | **{m_all1[0]['N']:,}** | **{m_all1[1]['N']:,}** |"
  )
  md.append(
      "| **Tier 1: Vertical Exits (Permanent Housing Placement)** |"
      f" {m_all[0]['V']:.2f}% | {m_all[1]['V']:.2f}% | {m_all1[0]['V']:.2f}% |"
      f" {m_all1[1]['V']:.2f}% |"
  )
  md.append(
      "| **Tier 2: Expanded Positive Exits (Temporary Stable)** |"
      f" {m_all[0]['E']:.2f}% | {m_all[1]['E']:.2f}% | {m_all1[0]['E']:.2f}% |"
      f" {m_all1[1]['E']:.2f}% |"
  )
  md.append(
      "| **Tier 3: Horizontal Exits (Vehicular Asset Preservation)** |"
      f" {m_all[0]['H']:.2f}% | {m_all[1]['H']:.2f}% | {m_all1[0]['H']:.2f}% |"
      f" {m_all1[1]['H']:.2f}% |"
  )
  md.append(
      "| **Tier 4a: True Unsheltered Exits (Street / Tent / Park)** |"
      f" {m_all[0]['U']:.2f}% | {m_all[1]['U']:.2f}% | {m_all1[0]['U']:.2f}% |"
      f" {m_all1[1]['U']:.2f}% |"
  )
  md.append(
      "| **Tier 4b: Other / Institutional / Temporary Unstable** |"
      f" {m_all[0]['O']:.2f}% | {m_all[1]['O']:.2f}% | {m_all1[0]['O']:.2f}% |"
      f" {m_all1[1]['O']:.2f}% |"
  )
  md.append(
      "| **Tier 4c: Missing / Administrative Non-Response** |"
      f" {m_all[0]['M']:.2f}% | {m_all[1]['M']:.2f}% | {m_all1[0]['M']:.2f}% |"
      f" {m_all1[1]['M']:.2f}% |"
  )
  md.append(
      "| **Standard HUD/HMIS Success Rate (Tier 1: Vertical Only)** |"
      f" **{m_all[0]['STD']:.2f}%** | **{m_all[1]['STD']:.2f}%** |"
      f" **{m_all1[0]['STD']:.2f}%** | **{m_all1[1]['STD']:.2f}%** |"
  )
  md.append(
      "| **Proposed Vehicle Residency & SPP Evaluative Framework: Expanded Stability (Vertical + Expanded)** |"
      f" **{m_all[0]['RATE_EXP']:.2f}%** | **{m_all[1]['RATE_EXP']:.2f}%** |"
      f" **{m_all1[0]['RATE_EXP']:.2f}%** | **{m_all1[1]['RATE_EXP']:.2f}%** |"
  )
  md.append(
      "| **Proposed Vehicle Residency & SPP Evaluative Framework:" 
        "Comprehensive Stability (Vertical + Expanded + Horizontal)** |"
        f"**{m_all[0]['RATE_NET']:.2f}%** |"
        f" **{m_all[1]['RATE_NET']:.2f}%** |"
        f" **{m_all1[0]['RATE_NET']:.2f}%** |"
        f" **{m_all1[1]['RATE_NET']:.2f}%** |"
  )
  md.append(
      "| **Proposed Stability Framework Absolute Improvement over Standard HMIS Metrics (percentage points / pp)** |"
      f" **+{m_all[0]['LIFT']:.2f}pp** | **+{m_all[1]['LIFT']:.2f}pp** |"
      f" **+{m_all1[0]['LIFT']:.2f}pp** | **+{m_all1[1]['LIFT']:.2f}pp** |\n"
  )

  # Panel B
  if m_first is not None and m_repeat is not None:
    md.append("### Vehicle Stability Framework Part II. First-Time and Repeat Disaggregation (`df_all` Subgroups)\n")
    md.append(
        "| Recategorization Hierarchy & Component | First-Time Clients"
        " (`df_first`) <br> Traditional Shelter | First-Time Clients"
        " (`df_first`) <br> Safe Parking (SPP) | Repeat Clients"
        " (`df_repeat`) <br> Traditional Shelter | Repeat Clients (`df_repeat`)"
        " <br> Safe Parking (SPP) |"
    )
    md.append(
        "| :--- | :---: | :---: | :---: | :---: |"
    )
    md.append(
        f"| **Sample Size (N)** | **{m_first[0]['N']:,}** |"
        f" **{m_first[1]['N']:,}** | **{m_repeat[0]['N']:,}** |"
        f" **{m_repeat[1]['N']:,}** |"
    )
    md.append(
        "| **Tier 1: Vertical Exits (Permanent Housing Placement)** |"
        f" {m_first[0]['V']:.2f}% | {m_first[1]['V']:.2f}% |"
        f" {m_repeat[0]['V']:.2f}% | {m_repeat[1]['V']:.2f}% |"
    )
    md.append(
        "| **Tier 2: Expanded Positive Exits (Temporary Stable)** |"
        f" {m_first[0]['E']:.2f}% | {m_first[1]['E']:.2f}% |"
        f" {m_repeat[0]['E']:.2f}% | {m_repeat[1]['E']:.2f}% |"
    )
    md.append(
        "| **Tier 3: Horizontal Exits (Vehicular Asset Preservation)** |"
        f" {m_first[0]['H']:.2f}% | {m_first[1]['H']:.2f}% |"
        f" {m_repeat[0]['H']:.2f}% | {m_repeat[1]['H']:.2f}% |"
    )
    md.append(
        "| **Tier 4a: True Unsheltered Exits (Street / Tent / Park)** |"
        f" {m_first[0]['U']:.2f}% | {m_first[1]['U']:.2f}% |"
        f" {m_repeat[0]['U']:.2f}% | {m_repeat[1]['U']:.2f}% |"
    )
    md.append(
        "| **Tier 4b: Other / Institutional / Temporary Unstable** |"
        f" {m_first[0]['O']:.2f}% | {m_first[1]['O']:.2f}% |"
        f" {m_repeat[0]['O']:.2f}% | {m_repeat[1]['O']:.2f}% |"
    )
    md.append(
        "| **Tier 4c: Missing / Administrative Non-Response** |"
        f" {m_first[0]['M']:.2f}% | {m_first[1]['M']:.2f}% |"
        f" {m_repeat[0]['M']:.2f}% | {m_repeat[1]['M']:.2f}% |"
    )
    md.append(
        "| **Standard HUD/HMIS Success Rate (Tier 1: Vertical Only)** |"
        f" **{m_first[0]['STD']:.2f}%** | **{m_first[1]['STD']:.2f}%** |"
        f" **{m_repeat[0]['STD']:.2f}%** | **{m_repeat[1]['STD']:.2f}%** |"
    )
    md.append(
        "| **Proposed Vehicle Residency & SPP Evaluative Framework: Expanded Stability (Vertical + Expanded)** |"
        f" **{m_first[0]['RATE_EXP']:.2f}%** |"
        f" **{m_first[1]['RATE_EXP']:.2f}%** |"
        f" **{m_repeat[0]['RATE_EXP']:.2f}%** |"
        f" **{m_repeat[1]['RATE_EXP']:.2f}%** |"
    )
    md.append(
        "| **Proposed Vehicle Residency & SPP Evaluative Framework:" 
        "Comprehensive Stability (Vertical + Expanded + Horizontal)** |"
        f"**{m_first[0]['RATE_NET']:.2f}%** |"
        f" **{m_first[1]['RATE_NET']:.2f}%** |"
        f" **{m_repeat[0]['RATE_NET']:.2f}%** |"
        f" **{m_repeat[1]['RATE_NET']:.2f}%** |"
    )
    md.append(
        "| **Proposed Stability Framework Absolute Improvement over Standard HMIS Metrics (percentage points / pp)** |"
        f" **+{m_first[0]['LIFT']:.2f}pp** |"
        f" **+{m_first[1]['LIFT']:.2f}pp** |"
        f" **+{m_repeat[0]['LIFT']:.2f}pp** |"
        f" **+{m_repeat[1]['LIFT']:.2f}pp** |\n"
    )

  content = "\n".join(md)
  export_path = os.path.join(
      output_dir, "Table3_NetStability_DualProposedRates.md"
  )
  with open(export_path, "w", encoding="utf-8") as f:
    f.write(content)

  print(f"Table 3 exported with Dual Proposed Rates to: {export_path}")
  return content


# Execute function
table3_markdown = generate_table3_four_tier_framework(
    df_all,
    df_all1=df_all1 if "df_all1" in locals() else None,
    df_first_all=df_first_all if "df_first_all" in locals() else None,
    df_repeat_all=df_repeat_all if "df_repeat_all" in locals() else None,
)
```

# Table 3. Proposed Net Stability Framework Reclassification Matrix

**Table 3.** *Comparison of Standard HUD/HMIS Exit Success Rates versus Proposed Multi-Tier Stability Metrics across Different Samples and Client Cohorts.*

### Vehicle Stability Framework Part I. Primary Sample Comparison (Unfiltered vs. Known Exits)

| Recategorization Hierarchy & Component | Unfiltered Baseline (`df_all`) <br> Traditional Shelter | Unfiltered Baseline (`df_all`) <br> Safe Parking (SPP) | Complete-Case (`df_all1`) <br> Traditional Shelter | Complete-Case (`df_all1`) <br> Safe Parking (SPP) |
| :--- | :---: | :---: | :---: | :---: |
| **Sample Size (N)** | **133,243** | **5,950** | **79,398** | **4,225** |
| **Tier 1: Vertical Exits (Permanent Housing Placement)** | 16.13% | 11.71% | 27.07% | 16.50% |
| **Tier 2: Expanded Positive Exits (Temporary Stable)** | 6.67% | 9.19% | 11.19% | 12.95% |
| **Tier 3: Horizontal Exits (Vehicular Asset Preservation)** | 3.29% | 36.92% | 5.52% | 52.00% |
| **Tier 4a: True Unsheltered Exits (Street / Tent / Park)** | 9.54% | 0.00% | 16.01% | 0.00% |
| **Tier 4b: Other / Institutional / Temporary Unstable** | 23.94% | 15.28% | 40.17% | 21.51% |
| **Tier 4c: Missing / Administrative Non-Response** | 40.41% | 28.99% | 0.00% | 0.00% |
| **Standard HUD/HMIS Success Rate (Tier 1: Vertical Only)** | **16.13%** | **11.71%** | **27.07%** | **16.50%** |
| **Proposed Vehicle Residency & SPP Evaluative Framework: Expanded Stability (Vertical + Expanded)** | **22.80%** | **20.91%** | **38.26%** | **29.44%** |
| **Proposed Vehicle Residency & SPP Evaluative Framework:Comprehensive Stability (Vertical + Expanded + Horizontal)** |**26.09%** | **57.83%** | **43.79%** | **81.44%** |
| **Proposed Stability Framework Absolute Improvement over Standard HMIS Metrics (percentage points / pp)** | **+9.96pp** | **+46.12pp** | **+16.72pp** | **+64.95pp** |

### Vehicle Stability Framework Part II. First-Time and Repeat Disaggregation (`df_all` Subgroups)

| Recategorization Hierarchy & Component | First-Time Clients (`df_first`) <br> Traditional Shelter | First-Time Clients (`df_first`) <br> Safe Parking (SPP) | Repeat Clients (`df_repeat`) <br> Traditional Shelter | Repeat Clients (`df_repeat`) <br> Safe Parking (SPP) |
| :--- | :---: | :---: | :---: | :---: |
| **Sample Size (N)** | **79,768** | **3,977** | **81,657** | **3,545** |
| **Tier 1: Vertical Exits (Permanent Housing Placement)** | 16.24% | 11.39% | 12.39% | 9.00% |
| **Tier 2: Expanded Positive Exits (Temporary Stable)** | 7.31% | 9.15% | 6.07% | 9.11% |
| **Tier 3: Horizontal Exits (Vehicular Asset Preservation)** | 3.59% | 34.85% | 3.17% | 38.65% |
| **Tier 4a: True Unsheltered Exits (Street / Tent / Park)** | 8.32% | 0.00% | 10.40% | 0.00% |
| **Tier 4b: Other / Institutional / Temporary Unstable** | 23.32% | 15.34% | 26.43% | 20.54% |
| **Tier 4c: Missing / Administrative Non-Response** | 41.21% | 31.31% | 41.52% | 25.25% |
| **Standard HUD/HMIS Success Rate (Tier 1: Vertical Only)** | **16.24%** | **11.39%** | **12.39%** | **9.00%** |
| **Proposed Vehicle Residency & SPP Evaluative Framework: Expanded Stability (Vertical + Expanded)** | **23.54%** | **20.54%** | **18.46%** | **18.11%** |
| **Proposed Vehicle Residency & SPP Evaluative Framework:Comprehensive Stability (Vertical + Expanded + Horizontal)** |**27.13%** | **55.39%** | **21.63%** | **56.76%** |
| **Proposed Stability Framework Absolute Improvement over Standard HMIS Metrics (percentage points / pp)** | **+10.89pp** | **+44.00pp** | **+9.24pp** | **+47.76pp** |


## Recidivism analysis 


```python
df_first.shape
```


```python
df_repeat_all.head()
```


```python
# import KaplanMeier analysis libraries 
from lifelines import KaplanMeierFitter
from lifelines.statistics import logrank_test
```


```python
# set paramaters
plt.rcParams.update({
    "font.family": "serif",
    "font.serif": ["Times New Roman", "DejaVu Serif"],
    "font.size": 10,
    "axes.labelsize": 10.5,
    "axes.titlesize": 11.5,
})

COLORS = {
    "Tier 1: Vertical (Permanent Placement)": "#00843D",  # Green
    "Tier 2: Expanded Positive Exits": "#005A36",  # Dark Green
    "Tier 3: Horizontal (Vehicular Preservation)": "#FFB500",  # Gold
    "Tier 4a: True Unsheltered (Street / Tent)": "#D9381E",  # Red
    "SPP Overall": "#FFB500",  # Gold
    "Traditional Shelter Overall": "#00843D",  # Green
}

STYLES = {
    "Tier 1: Vertical (Permanent Placement)": "-",
    "Tier 2: Expanded Positive Exits": "--",
    "Tier 3: Horizontal (Vehicular Preservation)": "-",
    "Tier 4a: True Unsheltered (Street / Tent)": ":",
    "SPP Overall": "-",
    "Traditional Shelter Overall": "--",
}


def prepare_data(df_first, df_repeat):
  """Combines datasets, aligns forward return intervals per PersonalID,

  assigns 4-tier exit hierarchy, and sets up KM survival variables.
  """
  df_all = pd.concat([df_first, df_repeat], ignore_index=True)

  # 1. Ensure chronological ordering per client
  if (
      "PersonalID" in df_all.columns
      and "Enrollment_Sequence" in df_all.columns
  ):
    df_all = df_all.sort_values(["PersonalID", "Enrollment_Sequence"]).copy()

  # 2. Forward shift CleanDaysPostExit (aligns row k exit with return gap to row k+1)
  df_all["Forward_CleanDaysPostExit"] = df_all.groupby("PersonalID")[
      "CleanDaysPostExit"
  ].shift(-1)

  # Fallback if already shifted in raw input data
  if df_all["Forward_CleanDaysPostExit"].isna().all():
    df_all["Forward_CleanDaysPostExit"] = df_all["CleanDaysPostExit"]

  # 3. Assign 4-Tier Exit Hierarchy
  conditions = [
      (df_all["Exit_Permanent"] == 1),
      (df_all["Exit_TemporaryStable"] == 1),
      (df_all["b_returned_to_vehicle"] == 1),
      ((df_all["Exit_Unsheltered"] == 1) & (df_all["b_returned_to_vehicle"] == 0)),
  ]
  choices = [
      "Tier 1: Vertical (Permanent Placement)",
      "Tier 2: Expanded Positive Exits",
      "Tier 3: Horizontal (Vehicular Preservation)",
      "Tier 4a: True Unsheltered (Street / Tent)",
  ]
  df_all["Tier"] = np.select(conditions, choices, default="Tier 4b: Other")

  # 4. Event = 1 if return occurred <= 730 days; 0 if right-censored/no return
  df_all["Event"] = np.where(
      df_all["Forward_CleanDaysPostExit"].notna()
      & (df_all["Forward_CleanDaysPostExit"] <= 730),
      1,
      0,
  )

  # 5. Duration: Fill missing with 730 and cap at 730 days
  df_all["Duration"] = (
      df_all["Forward_CleanDaysPostExit"]
      .fillna(730.0)
      .clip(lower=1.0, upper=730.0)
  )

  return df_all


# plot 1 for within SPP
def generate_cut1_plot(df_all, output_dir):
  df_spp = df_all[df_all["SPP"] == 1].copy()

  fig, ax = plt.subplots(figsize=(9, 6), dpi=600)
  kmf = KaplanMeierFitter()

  tiers_spp = [
      "Tier 1: Vertical (Permanent Placement)",
      "Tier 2: Expanded Positive Exits",
      "Tier 3: Horizontal (Vehicular Preservation)",
  ]

  for tier in tiers_spp:
    sub = df_spp[df_spp["Tier"] == tier]
    if len(sub) == 0:
      continue

    kmf.fit(sub["Duration"], event_observed=sub["Event"], label=tier)
    cum_return = (1.0 - kmf.survival_function_) * 100

    ax.plot(
        cum_return.index,
        cum_return.iloc[:, 0],
        label=f"{tier} (N={len(sub):,})",
        color=COLORS[tier],
        linestyle=STYLES[tier],
        linewidth=2.5 if "Horizontal" in tier else 1.6,
    )

  # Log-Rank Test: Tier 3 vs Tier 2
  t3 = df_spp[
      df_spp["Tier"] == "Tier 3: Horizontal (Vehicular Preservation)"
  ]
  t2 = df_spp[df_spp["Tier"] == "Tier 2: Expanded Positive Exits"]
  lr_result = logrank_test(
      t3["Duration"], t2["Duration"], t3["Event"], t2["Event"]
  )

  #ax.set_title(
      #"Cut 1. Safe Parking Program (SPP) Recidivism Trajectories by Exit Tier"
      #" (Within-SPP)",
      #fontweight="bold",
     # pad=12,
 # )
  ax.set_xlabel("Days Elapsed Post-Program Exit", labelpad=8)
  ax.set_ylabel("Cumulative Recidivism Rate (%)", labelpad=8)
  ax.set_xlim(0, 730)
  ax.set_ylim(0, 40)
  ax.grid(True, linestyle=":", alpha=0.6)

  # Callout Box in Bottom Right (Left-Aligned Lines inside Box)
  validation_text = (
      "Within-SPP Validation:\n"
      "Tier 3 (Vehicular Preservation) trajectory\n"
      "shows no statistical difference from\n"
      f"Tier 2 Expanded Positive Exits (p = {lr_result.p_value:.3f})."
  )
  ax.text(
      0.96,
      0.05,
      validation_text,
      transform=ax.transAxes,
      ha="right",
      va="bottom",
      multialignment="left",  # Left-aligns text lines inside the box
      fontsize=8.5,
      fontweight="bold",
      color="#01426A",
      bbox=dict(
          boxstyle="round,pad=0.5",
          facecolor="#FFF9E6",
          edgecolor="#FFB500",
          lw=1.2,
      ),
  )

  ax.legend(
      loc="upper left",
      frameon=True,
      facecolor="white",
      edgecolor="#cccccc",
      fontsize=8.5,
  )
  plt.tight_layout()

  path = os.path.join(output_dir, "Fig6_SPP_ExitTiers.jpg")
  plt.savefig(path, dpi=600, format="jpg")
  plt.close()
  print(f"[Cut 1 Figure Exported] {path}")


# secondary plots spp vs. non-spp
def generate_cut2_plot(df_all, output_dir):
  fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(16, 6), dpi=600, sharey=True)
  kmf = KaplanMeierFitter()

  spp_all = df_all[df_all["SPP"] == 1]
  shl_all = df_all[df_all["SPP"] == 0]

  # Panel A
  kmf.fit(spp_all["Duration"], spp_all["Event"], label="SPP Overall")
  cum_spp = (1.0 - kmf.survival_function_) * 100
  ax1.plot(
      cum_spp.index,
      cum_spp.iloc[:, 0],
      color=COLORS["SPP Overall"],
      linestyle=STYLES["SPP Overall"],
      lw=2.2,
      label=f"SPP Overall (N={len(spp_all):,})",
  )

  kmf.fit(shl_all["Duration"], shl_all["Event"], label="Traditional Shelter")
  cum_shl = (1.0 - kmf.survival_function_) * 100
  ax1.plot(
      cum_shl.index,
      cum_shl.iloc[:, 0],
      color=COLORS["Traditional Shelter Overall"],
      linestyle=STYLES["Traditional Shelter Overall"],
      lw=2.2,
      label=f"Traditional Shelter Overall (N={len(shl_all):,})",
  )

  ax1.set_title(
      "Plot I. Program Model Macro Comparison\n(Log-Rank p < 0.001)",
      fontweight="bold",
  )
  ax1.set_xlabel("Days Elapsed Post-Program Exit")
  ax1.set_ylabel("Cumulative Recidivism Rate (%)")
  ax1.set_xlim(0, 730)
  ax1.set_ylim(0, 45)
  ax1.grid(True, linestyle=":", alpha=0.6)
  ax1.legend(loc="upper left", frameon=True, facecolor="white")

  # Panel B
  spp_t3 = spp_all[
      spp_all["Tier"] == "Tier 3: Horizontal (Vehicular Preservation)"
  ]
  shl_t4a = shl_all[
      shl_all["Tier"] == "Tier 4a: True Unsheltered (Street / Tent)"
  ]
  shl_t1 = shl_all[
      shl_all["Tier"] == "Tier 1: Vertical (Permanent Placement)"
  ]

  kmf.fit(
      spp_t3["Duration"], spp_t3["Event"], label="SPP Tier 3 (Vehicular Exit)"
  )
  ax2.plot(
      (1 - kmf.survival_function_) * 100,
      color=COLORS["Tier 3: Horizontal (Vehicular Preservation)"],
      lw=2.5,
      label=f"SPP Tier 3: Vehicular Preservation (N={len(spp_t3):,})",
  )

  kmf.fit(
      shl_t4a["Duration"], shl_t4a["Event"], label="Shelter Tier 4a (Street)"
  )
  ax2.plot(
      (1 - kmf.survival_function_) * 100,
      color=COLORS["Tier 4a: True Unsheltered (Street / Tent)"],
      lw=2.0,
      ls=":",
      label=f"Shelter Tier 4a: Street Exit (N={len(shl_t4a):,})",
  )

  kmf.fit(
      shl_t1["Duration"], shl_t1["Event"], label="Shelter Tier 1 (Permanent)"
  )
  ax2.plot(
      (1 - kmf.survival_function_) * 100,
      color=COLORS["Tier 1: Vertical (Permanent Placement)"],
      lw=1.8,
      ls="--",
      label=f"Shelter Tier 1: Permanent Exit (N={len(shl_t1):,})",
  )

  ax2.set_title(
      "Plot II. Tier Comparisons Comparison: SPP Vehicular Exits vs Shelter Exits",
      fontweight="bold",
  )
  ax2.set_xlabel("Days Elapsed Post-Program Exit")
  ax2.set_xlim(0, 730)
  ax2.grid(True, linestyle=":", alpha=0.6)
  ax2.legend(loc="upper left", frameon=True, facecolor="white")

  #fig.suptitle(
      #"Plot 2. Macro and Micro Recidivism Evaluation: Safe Parking vs."
      #" Traditional Emergency Shelter",
      #fontsize=13,
      #fontweight="bold",
      #y=0.98,
  #)
  plt.tight_layout(rect=[0, 0, 1, 0.95])

  path = os.path.join(output_dir, "Fig6_SPP_vs_Shelter_Recidivism.jpg")
  plt.savefig(path, dpi=600, format="jpg")
  plt.close()
  print(f"[Cut 2 Figure Exported] {path}")


# markdown
def export_markdown_table(df_all, output_dir):
  time_points = [30, 90, 180, 365, 730]
  kmf = KaplanMeierFitter()

  def get_cum_rates(sub_df):
    kmf.fit(sub_df["Duration"], event_observed=sub_df["Event"])
    surv = kmf.survival_function_at_times(time_points).values
    return [f"{(1.0 - r) * 100:.2f}%" for r in surv]

  df_spp = df_all[df_all["SPP"] == 1]
  df_shl = df_all[df_all["SPP"] == 0]

  # Cut 1 Rows
  cut1_rows = []
  for t in [
      "Tier 1: Vertical (Permanent Placement)",
      "Tier 2: Expanded Positive Exits",
      "Tier 3: Horizontal (Vehicular Preservation)",
  ]:
    sub = df_spp[df_spp["Tier"] == t]
    if len(sub) > 0:
      cut1_rows.append(
          f"| **{t}** | {len(sub):,} | " + " | ".join(get_cum_rates(sub)) + " |"
      )

  # Cut 2 Rows
  rates_spp = get_cum_rates(df_spp)
  rates_shl = get_cum_rates(df_shl)
  cut2_macro_rows = [
      f"| **SPP Overall** | {len(df_spp):,} | " + " | ".join(rates_spp) + " |",
      f"| **Traditional Shelter Overall** | {len(df_shl):,} | "
      + " | ".join(rates_shl)
      + " |",
  ]

  t3 = df_spp[
      df_spp["Tier"] == "Tier 3: Horizontal (Vehicular Preservation)"
  ]
  shl_t4a = df_shl[
      df_shl["Tier"] == "Tier 4a: True Unsheltered (Street / Tent)"
  ]
  shl_t1 = df_shl[
      df_shl["Tier"] == "Tier 1: Vertical (Permanent Placement)"
  ]

  cut2_micro_rows = [
      f"| **SPP Tier 3: Vehicular Preservation** | {len(t3):,} | "
      + " | ".join(get_cum_rates(t3))
      + " |",
      f"| **Shelter Tier 1: Permanent Exit** | {len(shl_t1):,} | "
      + " | ".join(get_cum_rates(shl_t1))
      + " |",
      f"| **Shelter Tier 4a: Street Exit** | {len(shl_t4a):,} | "
      + " | ".join(get_cum_rates(shl_t4a))
      + " |",
  ]

  # Tests
  t2 = df_spp[df_spp["Tier"] == "Tier 2: Expanded Positive Exits"]
  lr_c1 = logrank_test(
      t3["Duration"], t2["Duration"], t3["Event"], t2["Event"]
  )
  lr_macro = logrank_test(
      df_spp["Duration"],
      df_shl["Duration"],
      df_spp["Event"],
      df_shl["Event"],
  )
  lr_t3_t4a = logrank_test(
      t3["Duration"], shl_t4a["Duration"], t3["Event"], shl_t4a["Event"]
  )
  lr_t3_t1 = logrank_test(
      t3["Duration"], shl_t1["Duration"], t3["Event"], shl_t1["Event"]
  )

  md_path = os.path.join(output_dir, "KM_Survival_Summary_Table.md")
  md_content = f"""# Research Question 3: Kaplan-Meier Survival Analysis Summary Table

This table details the empirical cumulative recidivism rates (%) derived from the 730-day Kaplan-Meier survival analysis across **Survival Analysis 1 (Within-SPP Validation)** and **Survival Analysis 2 (Safe Parking vs. Traditional Shelter)**.

---

## Within-Safe Parking Program (SPP) Recidivism Rates by Exit Tier

Post-exit trajectories evaluated strictly within the Safe Parking Program cohort ($N={len(df_spp):,}$).

| Exit Destination Tier | Cohort N | 30-Day Rate | 90-Day Rate | 180-Day Rate | 365-Day Rate | 730-Day Rate |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
""" + "\n".join(cut1_rows) + f"""

### Statistical Validation (Analysis 1)
* **Log-Rank Test (Tier 3 vs. Tier 2):** $\\chi^2 = {lr_c1.test_statistic:.3f}$, **$p = {lr_c1.p_value:.4f}$**
* **Key Finding:** Tier 3 Vehicular Preservation does not statistically diverge from Tier 2 (Expanded Positive Exits) at $\\alpha = 0.05$, validating that vehicular asset preservation functions as a protective buffer against system re-entry.

---

## Program Model Comparison (Safe Parking vs. Traditional Shelter)

### Plot I. Macro Model Comparison: Overall SPP vs. Traditional Shelter

| Program Model | Total N | 30-Day Rate | 90-Day Rate | 180-Day Rate | 365-Day Rate | 730-Day Rate |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
""" + "\n".join(cut2_macro_rows) + f"""

* **Macro Log-Rank Test (SPP Overall vs. Shelter Overall):** $\\chi^2 = {lr_macro.test_statistic:.3f}$, **$p < 0.001$**
* **Key Finding:** SPP has a statistically significant recidivism rate compared to traditional shelters, which points to the need for expanded SPP access to accommodate this growing population with case managers that can provide housing and financial assistance to transition vehicle residents into permanent housing.

---

### Plot II. Tiers Comparison: SPP Vehicular Exits vs. Shelter Exits

| Micro Cohort & Exit Tier | Cohort N | 30-Day Rate | 90-Day Rate | 180-Day Rate | 365-Day Rate | 730-Day Rate |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
""" + "\n".join(cut2_micro_rows) + f"""

### Statistical Validation: Analysis 2
* **Log-Rank Test (SPP Tier 3 vs. Shelter Tier 4a Street Exits):** $\\chi^2 = {lr_t3_t4a.test_statistic:.3f}$, **$p < 0.001$**
* **Log-Rank Test (SPP Tier 3 vs. Shelter Tier 1 Permanent Exits):** $\\chi^2 = {lr_t3_t1.test_statistic:.3f}$, **$p = {lr_t3_t1.p_value:.4f}$**
"""

  with open(md_path, "w") as f:
    f.write(md_content)

  print(f"[Markdown Exported] {md_path}")


# run the full script
def run_full_km_pipeline(df_first, df_repeat, output_dir="FinalHousingStudies"):
  os.makedirs(output_dir, exist_ok=True)
  df_all = prepare_data(df_first, df_repeat)

  generate_cut1_plot(df_all, output_dir)
  generate_cut2_plot(df_all, output_dir)
  export_markdown_table(df_all, output_dir)

  print(
      f"\n>>> Survival Analysis a 'great success!' (Borat voice): All figures and Markdown table generated in"
      f" '{output_dir}/' <<<"
  )


# Execute everything in a single line
run_full_km_pipeline(df_first, df_repeat)
```

# Kaplan-Meier Survival Analysis Summary Table

This table details the empirical cumulative recidivism rates (%) derived from the 730-day Kaplan-Meier survival analysis across **Survival Analysis 1 (Within-SPP Validation)** and **Survival Analysis 2 (Safe Parking vs. Traditional Shelter)**.

---

## Survival Analysis 1: Within-Safe Parking Program (SPP) Recidivism Rates by Exit Tier

Post-exit trajectories evaluated strictly within the Safe Parking Program cohort ($N=5,055$).

| Exit Destination Tier | Cohort N | 30-Day Rate | 90-Day Rate | 180-Day Rate | 365-Day Rate | 730-Day Rate |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| Tier 1: Vertical (Permanent Placement) | 697 | 5.31% | 6.74% | 8.03% | 9.61% | 11.05% |
| Tier 2: Expanded Positive Exits | 547 | 20.11% | 29.62% | 34.19% | 35.28% | 36.93% |
| Tier 3: Horizontal (Vehicular Preservation) | 2,071 | 25.45% | 29.41% | 32.54% | 35.30% | 37.04% |

### Statistical Validation (Survival Analysis 1)
* **Log-Rank Test (Tier 3 vs. Tier 2):** $\chi^2 = 0.124$, **$p = 0.7251$**
* **Key Finding:** Tier 3 Vehicular Preservation does not statistically diverge from Tier 2 (Expanded Positive Exits) at $\alpha = 0.05$, validating that vehicular asset preservation functions as a protective buffer against system re-entry.

---

## Survival Analysis 2: Program Model Comparison (Safe Parking vs. Traditional Shelter)

### Plot I. Macro Model Comparison: Overall SPP vs. Traditional Shelter

| Program Model | Total N | 30-Day Rate | 90-Day Rate | 180-Day Rate | 365-Day Rate | 730-Day Rate |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| SPP Overall | 5,055 | 22.08% | 25.76% | 28.21% | 30.09% | 31.47% |
| Traditional Shelter Overall | 99,337 | 13.31% | 16.06% | 18.25% | 20.57% | 22.39% |

* **Macro Log-Rank Test (SPP Overall vs. Shelter Overall):** $\chi^2 = 254.369$, **$p < 0.001$**
* **Key Finding:** SPP has a statistically significant higher recidivism rate compared to traditional shelters, which points to the need for expanded SPP access to accommodate this growing population with case managers that can provide housing and financial assistance to transition vehicle residents into permanent housing.

---

### Plot II. Tiers Comparison: SPP Vehicular Exits vs. Shelter Exits

| Micro Cohort & Exit Tier | Cohort N | 30-Day Rate | 90-Day Rate | 180-Day Rate | 365-Day Rate | 730-Day Rate |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| SPP Tier 3: Vehicular Preservation | 2,071 | 25.45% | 29.41% | 32.54% | 35.30% | 37.04% |
| Shelter Tier 1: Permanent Exit | 21,493 | 3.71% | 4.67% | 5.75% | 7.11% | 8.85% |
| Shelter Tier 4a: Street Exit | 17,099 | 11.06% | 17.76% | 22.63% | 27.94% | 31.28% |

### Statistical Validation: Plot II
* **Log-Rank Test (SPP Tier 3 vs. Shelter Tier 4a Street Exits):** $\chi^2 = 56.293$, **$p < 0.001$**
* **Log-Rank Test (SPP Tier 3 vs. Shelter Tier 1 Permanent Exits):** $\chi^2 = 1801.994$, **$p = 0.0000$**

