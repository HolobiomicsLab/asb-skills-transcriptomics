---
name: respiratory-nadir-time-alignment
description: Use when a longitudinal COVID-19 transcriptomics cohort has per-sample
  clinical timing (respiratory function measurements) and the goal is to anchor expression
  trajectories to the day of respiratory nadir to determine how early severe-disease
  signatures emerge.
when_to_use_negative:
- Cross-sectional datasets comparing severe vs mild patients (e.g. the 7 RNAseq datasets
  like GSE155454 or GSE157103) — these lack repeated sampling and cannot be aligned
  to a nadir timeline
- Analyses that need gene expression profiles more than 4 days before respiratory
  nadir — no such data exist in available studies, so precise timing of dysregulation
  onset is undetermined
- Single-cell PBMC/BALF deconvolution aiming to identify cell subsets — use cell-subset
  annotation workflows instead of nadir time alignment
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3308
tools:
- name: pandas
  role: Load the E-MTAB-9721 expression matrix and clinical metadata and align samples
    to days relative to respiratory nadir
- name: matplotlib
  role: Plot MCEMP1, HLA-DRA and ETS1 expression trajectories grouped by day-relative-to-nadir
    bins
- name: Streamlit
  role: Downstream query tool for gene-expression differences between severe and mild
    COVID-19 patients in the 7 cross-sectional datasets
  repo: https://github.com/kuanrongchan/COVID19-severity
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:8a187eff86db249730741acbb4b2068bbb83c5c7b2b88c3c99a2deb41a8d9aa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Streamlit
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# respiratory-nadir-time-alignment

## Summary

Aligns longitudinal whole-blood transcriptomics samples (e.g. E-MTAB-9721, Ong et al.) to each patient's nadir of respiratory function so gene-expression kinetics can be interpreted in disease-time rather than sampling-time, enabling detection of early prognostic shifts in MCEMP1, HLA-DRA and ETS1.

## When to use

Use when a longitudinal COVID-19 transcriptomics cohort has per-sample clinical timing (respiratory function measurements) and the goal is to anchor expression trajectories to the day of respiratory nadir to determine how early severe-disease signatures emerge.

## When NOT to use

- Cross-sectional datasets comparing severe vs mild patients (e.g. the 7 RNAseq datasets like GSE155454 or GSE157103) — these lack repeated sampling and cannot be aligned to a nadir timeline
- Analyses that need gene expression profiles more than 4 days before respiratory nadir — no such data exist in available studies, so precise timing of dysregulation onset is undetermined
- Single-cell PBMC/BALF deconvolution aiming to identify cell subsets — use cell-subset annotation workflows instead of nadir time alignment

## Inputs

- Longitudinal whole-blood microarray expression matrix from E-MTAB-9721 (ArrayExpress)
- Clinical metadata table with per-sample respiratory function measurements and sampling days
- Gene list of interest (e.g. MCEMP1, HLA-DRA, ETS1)

## Outputs

- nadir_aligned_expression.csv — per-sample expression table aligned to days relative to respiratory nadir
- nadir_trajectories.png — gene-expression trajectory plots around the nadir (matplotlib/plotly)

## How to apply

Download the longitudinal whole-blood microarray dataset E-MTAB-9721 from ArrayExpress and load the expression matrix and clinical metadata with Python (pandas). Align each sample to days relative to that patient's nadir of respiratory function, then extract expression values for MCEMP1, HLA-DRA and ETS1 and bin samples by day-relative-to-nadir, with the earliest available bin at 4 days before nadir. Plot the per-bin expression trajectories with matplotlib or plotly, checking the expected pattern: MCEMP1 heightened and HLA-DRA reduced as early as 4 days pre-nadir, ETS1 reduced only at nadir, and all three transcripts returning to baseline 7–8 days post-nadir. Rationale: anchoring to respiratory nadir converts irregular sampling into a common disease-time axis, revealing that early differential expression of MCEMP1 and HLA-DRA occurs predominantly in CD14+ cells and is prognostic of severe COVID-19 before overt respiratory deterioration.

## Related tools

- **pandas** (Load the E-MTAB-9721 expression matrix and clinical metadata and align samples to days relative to respiratory nadir)
- **matplotlib** (Plot MCEMP1, HLA-DRA and ETS1 expression trajectories grouped by day-relative-to-nadir bins)
- **Streamlit** (Downstream query tool for gene-expression differences between severe and mild COVID-19 patients in the 7 cross-sectional datasets) — https://github.com/kuanrongchan/COVID19-severity

## Examples

```
import pandas as pd; df = pd.read_csv('E-MTAB-9721_expression.csv'); df['day_vs_nadir'] = df['sample_day'] - df['nadir_day']; nadir_expr = df[df['gene'].isin(['MCEMP1','HLA-DRA','ETS1'])]; nadir_expr.to_csv('nadir_aligned_expression.csv', index=False)
```

## Evaluation signals

- MCEMP1 expression is significantly heightened and HLA-DRA significantly reduced in the bin at 4 days before respiratory nadir in patients who developed severe COVID-19
- ETS1 downregulation appears only at the nadir bin, not at 4 days pre-nadir
- All three transcripts (MCEMP1, HLA-DRA, ETS1) return to baseline in bins 7–8 days after nadir
- Every sample in nadir_aligned_expression.csv maps to a valid day-relative-to-nadir bin derived from that patient's respiratory function metadata
- The trajectory figure shows monotonically interpretable disease-time ordering rather than raw sampling-date ordering

## Limitations

- No gene expression profiles exist more than 4 days before respiratory nadir, so the precise timing of MCEMP1/HLA-DRA dysregulation onset is undetermined
- Future longitudinal studies are needed to characterise the kinetics of MCEMP1 and HLA-DRA gene and protein expression over the entire course of infection
- Sensitivity and specificity of using MCEMP1/HLA-DRA expression to predict severe COVID-19 have not yet been validated prospectively

## Evidence

- [other] In the longitudinal cohort (Ong et al., E-MTAB-9721), significantly heightened MCEMP1 expression and downregulated HLA-DRA expression were detectable 4 days before the nadir: [evidence span withheld — anchor status: not_found]
- [abstract] we found significantly heightened MCEMP1 and reduced HLA-DRA expression as early as four days before the nadir of respiratory function: "we found significantly heightened MCEMP1 and reduced HLA-DRA expression as early as four days before the nadir of respiratory function, and the differential expression of MCEMP1 and HLA-DRA occurred"
- [discussion] as there are currently no studies documenting gene expression profiles more than 4 days before nadir of respiratory function, we were unable to determine precisely when the dysregulation of MCEMP1: "as there are currently no studies documenting gene expression profiles more than 4 days before nadir of respiratory function, we were unable to determine precisely when the dysregulation of MCEMP1"
- [abstract] blood transcriptomics of COVID-19 patients were prospectively and longitudinally monitored previously, to track the time in which these gene expression changes occur before nadir of respiratory: "blood transcriptomics of COVID-19 patients were prospectively and longitudinally monitored previously, to track the time in which these gene expression changes occur before nadir of respiratory"
- [readme] All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity: "All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity"
