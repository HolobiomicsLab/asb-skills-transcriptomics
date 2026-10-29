---
name: covid19-severity-biomarker-reproduction
description: Use when reproducing or validating the timing of severe-COVID-19 prognostic
  gene-expression changes (MCEMP1, HLA-DRA, ETS1) in a longitudinal whole-blood microarray
  cohort aligned to days relative to respiratory nadir.
when_to_use_negative:
- No gene expression profiles exist more than 4 days before respiratory nadir, so
  this skill cannot determine the precise onset time of MCEMP1/HLA-DRA dysregulation
  earlier than day −4
- Input is single-cell RNA-seq data for immune-cell-subset attribution rather than
  a longitudinal bulk whole-blood cohort with nadir-aligned sampling
- Goal is variant-specific transcriptomic analysis — the included datasets did not
  directly evaluate the effect of different SARS-CoV-2 variants
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3517
tools:
- name: pandas
  role: Load the expression matrix and clinical metadata and align samples to days
    relative to respiratory nadir
- name: matplotlib
  role: Plot MCEMP1, HLA-DRA and ETS1 expression trajectories across day-relative-to-nadir
    bins
- name: Streamlit
  role: Webtool for querying gene expression differences between severe and mild COVID-19
    patients in the 7 cross-sectional datasets
  repo: https://github.com/kuanrongchan/COVID19-severity
- name: pip
  role: Install repository dependencies to run the Streamlit app locally
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:1722e1e63624b19a1a0893ac1c0de62ea92971069d1b05f2ba75262bac7be4fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Streamlit
  - matplotlib
  - pandas
  - pip
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# covid19-severity-biomarker-reproduction

## Summary

Reproduces the longitudinal trajectory analysis of MCEMP1, HLA-DRA and ETS1 expression around the respiratory-function nadir in the E-MTAB-9721 cohort (Ong et al.), verifying that MCEMP1 is heightened and HLA-DRA reduced 4 days pre-nadir, ETS1 falls only at nadir, and all transcripts return to baseline 7–8 days post-nadir.

## When to use

Use when reproducing or validating the timing of severe-COVID-19 prognostic gene-expression changes (MCEMP1, HLA-DRA, ETS1) in a longitudinal whole-blood microarray cohort aligned to days relative to respiratory nadir.

## When NOT to use

- No gene expression profiles exist more than 4 days before respiratory nadir, so this skill cannot determine the precise onset time of MCEMP1/HLA-DRA dysregulation earlier than day −4
- Input is single-cell RNA-seq data for immune-cell-subset attribution rather than a longitudinal bulk whole-blood cohort with nadir-aligned sampling
- Goal is variant-specific transcriptomic analysis — the included datasets did not directly evaluate the effect of different SARS-CoV-2 variants

## Inputs

- Longitudinal whole-blood microarray dataset E-MTAB-9721 (Ong et al., 2021) from ArrayExpress
- Clinical metadata mapping each sample to days relative to the patient's respiratory-function nadir
- Processed severe-vs-mild count/anova files from the COVID19-severity GitHub repository (optional cross-check)

## Outputs

- nadir_aligned_expression.csv (per-day expression table for MCEMP1, HLA-DRA, ETS1 aligned to respiratory nadir)
- nadir_trajectories.png (gene-expression trajectory figure around the nadir)

## How to apply

Download the longitudinal whole-blood microarray dataset E-MTAB-9721 (Ong et al., 2021) from ArrayExpress, load the expression matrix and clinical metadata with Python/pandas, and align each sample to days relative to that patient's nadir of respiratory function. Extract expression values for MCEMP1, HLA-DRA and ETS1 and bin samples by day-relative-to-nadir, with the earliest available bin at 4 days before nadir. Plot per-day trajectories with matplotlib/plotly and check the expected pattern: MCEMP1 heightened and HLA-DRA reduced as early as 4 days pre-nadir, ETS1 downregulated only at nadir, and all three transcripts returning to baseline 7–8 days post-nadir. Save the aligned per-day expression table as nadir_aligned_expression.csv and the trajectory figure as nadir_trajectories.png. Cross-check the cross-sectional direction of dysregulation against the 7-dataset severity database (GSE155454, Bibert 2021, GSE161731, GSE157103, GSE152418, GSE172114, GSE164805) hosted in the COVID19-severity repository.

## Related tools

- **pandas** (Load the expression matrix and clinical metadata and align samples to days relative to respiratory nadir)
- **matplotlib** (Plot MCEMP1, HLA-DRA and ETS1 expression trajectories across day-relative-to-nadir bins)
- **Streamlit** (Webtool for querying gene expression differences between severe and mild COVID-19 patients in the 7 cross-sectional datasets) — https://github.com/kuanrongchan/COVID19-severity
- **pip** (Install repository dependencies to run the Streamlit app locally)

## Examples

```
streamlit run app.py
```

## Evaluation signals

- MCEMP1 expression is significantly heightened and HLA-DRA significantly reduced in the day −4 pre-nadir bin in patients who developed severe COVID-19
- ETS1 downregulation is observed only at the nadir bin, not at day −4
- All three transcripts return to baseline 7–8 days after nadir in the trajectory plot
- nadir_aligned_expression.csv contains one row per sample with a valid day-relative-to-nadir bin and columns for MCEMP1, HLA-DRA and ETS1
- Cross-sectional direction (MCEMP1 up, HLA-DRA and ETS1 down in severe) is consistent with the processed severe-vs-mild files in the COVID19-severity repo

## Limitations

- No studies document gene expression profiles more than 4 days before nadir of respiratory function, so the precise onset of MCEMP1/HLA-DRA dysregulation cannot be determined
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severe COVID-19 have not been validated; prospective studies are needed
- Study-dataset heterogeneity was noted, and future longitudinal studies are needed to characterise kinetics of MCEMP1 and HLA-DRA gene and protein expression over the entire course of infection

## Evidence

- [abstract] MCEMP1 heightened pre-nadir, HLA-DRA reduced: "we found significantly heightened MCEMP1 and reduced HLA-DRA expression as early as four days before the nadir of respiratory function"
- [abstract] Longitudinal tracking workflow: "blood transcriptomics of COVID-19 patients were prospectively and longitudinally monitored previously, to track the time in which these gene expression changes occur before nadir of respiratory"
- [intro] Cell-subset attribution: "we ascertained that the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets"
- [abstract] CD14+ prognostic finding: "Elevated MCEMP1 and reduced HLA-DRA gene expression in CD14+ cells during the early phase of disease are prognostic of severe COVID-19."
- [discussion] No data before day −4: "as there are currently no studies documenting gene expression profiles more than 4 days before nadir of respiratory function, we were unable to determine precisely when the dysregulation of MCEMP1"
- [readme] README app usage: "1. Select a database to query
2. Search for your gene of interest using the filters indicated in the dataframe headers."
