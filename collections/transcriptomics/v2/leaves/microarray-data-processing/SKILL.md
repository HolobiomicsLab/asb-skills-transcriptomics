---
name: microarray-data-processing
description: Use when a longitudinal peripheral-blood transcriptomics dataset needs
  per-patient time alignment to clinical nadir, and trajectories of candidate prognostic
  transcripts must be extracted and plotted across disease phases.
when_to_use_negative:
- When no per-patient longitudinal sampling exists — cross-sectional cohorts cannot
  be aligned to a respiratory-function nadir
- When the goal is single-cell immune-cell-subset deconvolution; this skill operates
  on bulk whole-blood microarray/RNAseq matrices
- When gene expression profiles more than 4 days before nadir are required — no such
  data exist in E-MTAB-9721
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: pandas
  role: Load the E-MTAB-9721 expression matrix and clinical metadata and align samples
    to days relative to respiratory nadir
- name: matplotlib/plotly
  role: Plot MCEMP1, HLA-DRA and ETS1 expression trajectories across day-relative-to-nadir
    bins
- name: Streamlit
  role: Query gene expression differences between severe and mild COVID-19 patients
    in the 7-dataset COVID19-severity database
  repo: https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
- name: pip
  role: Install dependencies to run the COVID19-severity query app locally
  repo: https://github.com/kuanrongchan/COVID19-severity
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:81e12d1dadb0672f7a5df4ea37dcae8236788424f7ebb5dafa7708032df5a4ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Streamlit
  - matplotlib/plotly
  - pandas
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# microarray-data-processing

## Summary

Processing of longitudinal whole-blood microarray data (e.g. E-MTAB-9721, Ong et al. 2021) to align samples to days relative to each patient's respiratory-function nadir and extract trajectories of severity-associated transcripts (MCEMP1, HLA-DRA, ETS1) in severe vs non-severe COVID-19.

## When to use

Use when a longitudinal peripheral-blood transcriptomics dataset needs per-patient time alignment to clinical nadir, and trajectories of candidate prognostic transcripts must be extracted and plotted across disease phases.

## When NOT to use

- When no per-patient longitudinal sampling exists — cross-sectional cohorts cannot be aligned to a respiratory-function nadir
- When the goal is single-cell immune-cell-subset deconvolution; this skill operates on bulk whole-blood microarray/RNAseq matrices
- When gene expression profiles more than 4 days before nadir are required — no such data exist in E-MTAB-9721

## Inputs

- Whole-blood microarray expression matrix from E-MTAB-9721 (ArrayExpress, Ong et al. 2021)
- Clinical metadata with respiratory-function measurements per patient and sampling timepoints
- Gene list of interest: MCEMP1, HLA-DRA, ETS1
- Processed severe-vs-mild RNAseq tables from GSE155454, Bibert 2021 (10.17632/8wxhhykfnh.2), GSE161731, GSE157103, GSE152418, GSE172114, GSE164805

## Outputs

- nadir_aligned_expression.csv (per-sample table aligned to days relative to respiratory nadir)
- nadir_trajectories.png (gene-expression trajectories around the nadir, per day-relative bin)
- Per-dataset box/strip plots of selected genes with t-test p-values, BH-adjusted p-values, ratio and fold-change

## How to apply

Download the ArrayExpress dataset (e.g. E-MTAB-9721) and load the expression matrix and clinical metadata with Python (pandas). Align each sample to days relative to that patient's nadir of respiratory function, then extract expression values for MCEMP1, HLA-DRA and ETS1 and group samples into day-relative-to-nadir bins (earliest available bin at 4 days pre-nadir). Plot trajectories with matplotlib/plotly and verify expected kinetics: MCEMP1 heightened and HLA-DRA reduced as early as 4 days before nadir, ETS1 downregulated only at nadir, and all three transcripts returning to baseline 7-8 days post-nadir. For cross-sectional validation of the same transcripts, query the 7 RNAseq severe-vs-mild datasets (e.g. GSE155454, GSE161731, GSE157103) via the COVID19-severity database, whose processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio and fold-change data.

## Related tools

- **pandas** (Load the E-MTAB-9721 expression matrix and clinical metadata and align samples to days relative to respiratory nadir)
- **matplotlib/plotly** (Plot MCEMP1, HLA-DRA and ETS1 expression trajectories across day-relative-to-nadir bins)
- **Streamlit** (Query gene expression differences between severe and mild COVID-19 patients in the 7-dataset COVID19-severity database) — https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
- **pip** (Install dependencies to run the COVID19-severity query app locally) — https://github.com/kuanrongchan/COVID19-severity

## Examples

```
import pandas as pd; expr = pd.read_csv('E-MTAB-9721_expression.csv', index_col=0); meta = pd.read_csv('E-MTAB-9721_meta.csv'); expr.loc[['MCEMP1','HLA-DRA','ETS1']].groupby(meta['day_to_nadir']).mean()
```

## Evaluation signals

- MCEMP1 expression is significantly heightened and HLA-DRA reduced at the 4-days-before-nadir bin in patients who developed severe COVID-19
- ETS1 is downregulated only at the nadir bin, not 4 days before nadir
- All three transcripts return to baseline 7-8 days after nadir
- nadir_aligned_expression.csv contains every sample mapped to a day-relative-to-nadir bin with no unmapped samples
- Severe-vs-mild processed tables include t-test p-values, BH-adjusted p-values, ratio and fold-change for each gene

## Limitations

- No gene expression profiles exist more than 4 days before respiratory nadir, so the precise onset of MCEMP1/HLA-DRA dysregulation is undetermined
- Heterogeneity across study datasets (Supplementary Fig. S9) affects generalisability of the trajectories
- Only one gene at a time can be plotted in the Streamlit box/strip plot interface

## Evidence

- [abstract] MCEMP1 heightened pre-nadir: "we found significantly heightened MCEMP1 and reduced HLA-DRA expression as early as four days before the nadir of respiratory function"
- [methods] workflow steps: [evidence span withheld — anchor status: not_found]
- [readme] processed statistics: "The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data between severe COVID-19 and mild COVID-19"
- [discussion] onset timing limit: "as there are currently no studies documenting gene expression profiles more than 4 days before nadir of respiratory function, we were unable to determine precisely when the dysregulation of MCEMP1"
