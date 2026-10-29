---
name: differential-expression-trajectory-visualization
description: Use when a longitudinal cohort has per-patient timepoints relative to
  a clinical event (respiratory nadir) and you need to determine how early differential
  expression of candidate genes emerges and when levels return to baseline.
when_to_use_negative:
- Only cross-sectional severe-vs-mild expression data are available — no timepoints
  exist to align to a nadir, so trajectories cannot be computed
- The question is which genes are consistently differentially regulated across cohorts
  (use the 7-dataset systematic re-analysis instead), rather than when dysregulation
  occurs in time
- Gene expression profiles more than 4 days before nadir are required — no studies
  document expression that far in advance, so precise timing earlier than day -4 cannot
  be determined
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3517
tools:
- name: pandas
  role: Load the E-MTAB-9721 expression matrix and clinical metadata and align samples
    to days relative to respiratory nadir
- name: matplotlib
  role: Plot MCEMP1/HLA-DRA/ETS1 expression trajectories across day-relative-to-nadir
    bins
- name: plotly
  role: Alternative plotting library for the nadir-aligned trajectory figure
- name: Streamlit
  role: Companion webtool for querying gene-expression differences between severe
    and mild COVID-19 patients in the 7 curated datasets
  repo: https://github.com/kuanrongchan/COVID19-severity
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:a73919dc9b80770a616ea851ac6567c58e2d7dfd504addb6bc1c49982863d272
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Streamlit
  - matplotlib
  - pandas
  - plotly
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# differential-expression-trajectory-visualization

## Summary

Aligns longitudinal whole-blood transcriptomics samples to each patient's respiratory-function nadir and plots per-gene expression trajectories (e.g. MCEMP1, HLA-DRA, ETS1) to establish the timing of severity-associated dysregulation before nadir.

## When to use

Use when a longitudinal cohort has per-patient timepoints relative to a clinical event (respiratory nadir) and you need to determine how early differential expression of candidate genes emerges and when levels return to baseline.

## When NOT to use

- Only cross-sectional severe-vs-mild expression data are available — no timepoints exist to align to a nadir, so trajectories cannot be computed
- The question is which genes are consistently differentially regulated across cohorts (use the 7-dataset systematic re-analysis instead), rather than when dysregulation occurs in time
- Gene expression profiles more than 4 days before nadir are required — no studies document expression that far in advance, so precise timing earlier than day -4 cannot be determined

## Inputs

- Longitudinal whole-blood microarray expression matrix (ArrayExpress accession E-MTAB-9721)
- Clinical metadata mapping samples to days relative to each patient's respiratory-function nadir
- List of candidate genes (e.g. MCEMP1, HLA-DRA, ETS1) identified from cross-cohort differential-expression analysis

## Outputs

- nadir_aligned_expression.csv (per-day expression table aligned to respiratory nadir)
- nadir_trajectories.png (gene-expression trajectory figure around nadir)

## How to apply

Download the longitudinal whole-blood microarray dataset E-MTAB-9721 (Ong et al., 2021) from ArrayExpress, load the expression matrix and clinical metadata with pandas, and align each sample to days relative to that patient's nadir of respiratory function. Extract expression values for the target transcripts (MCEMP1, HLA-DRA, ETS1), restricting to patients who developed severe COVID-19, and bin samples by day-relative-to-nadir, with the earliest available bin at 4 days before nadir. Plot the trajectories with matplotlib or plotly and verify the expected pattern: MCEMP1 heightened and HLA-DRA reduced as early as 4 days pre-nadir, ETS1 downregulated only at nadir, and all three transcripts returning to baseline 7-8 days post-nadir. Save the aligned per-day table as nadir_aligned_expression.csv and the figure as nadir_trajectories.png.

## Related tools

- **pandas** (Load the E-MTAB-9721 expression matrix and clinical metadata and align samples to days relative to respiratory nadir)
- **matplotlib** (Plot MCEMP1/HLA-DRA/ETS1 expression trajectories across day-relative-to-nadir bins)
- **plotly** (Alternative plotting library for the nadir-aligned trajectory figure)
- **Streamlit** (Companion webtool for querying gene-expression differences between severe and mild COVID-19 patients in the 7 curated datasets) — https://github.com/kuanrongchan/COVID19-severity

## Examples

```
import pandas as pd; df = pd.read_csv('E-MTAB-9721_expression.csv'); df['day_to_nadir'] = df['sample'].map(meta.set_index('sample')['day_to_nadir']); df.groupby(['day_to_nadir'])[['MCEMP1','HLA-DRA','ETS1']].mean().to_csv('nadir_aligned_expression.csv')
```

## Evaluation signals

- MCEMP1 expression is significantly heightened in the day -4 (pre-nadir) bin for patients who developed severe COVID-19
- HLA-DRA expression is significantly reduced in the day -4 pre-nadir bin
- ETS1 downregulation appears only at nadir (day 0), not at day -4
- All three transcripts (MCEMP1, HLA-DRA, ETS1) return to baseline in the 7-8 days post-nadir bins
- Outputs exist with expected names: nadir_aligned_expression.csv and nadir_trajectories.png, and the CSV rows map to sample day-relative-to-nadir bins

## Limitations

- There are currently no studies documenting gene expression profiles more than 4 days before nadir of respiratory function, so the precise onset of MCEMP1/HLA-DRA dysregulation is undetermined
- Future longitudinal studies are needed to characterise the kinetics of MCEMP1 and HLA-DRA gene and protein expression over the entire course of infection
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severe COVID-19 have not been validated in prospective studies
- Heterogeneity across study datasets may influence MCEMP1 and HLA-DRA expression levels

## Evidence

- [other] In the longitudinal cohort (Ong et al., E-MTAB-9721), significantly heightened MCEMP1 expression and downregulated HLA-DRA expression were detectable 4 days before the nadir of respiratory function: [evidence span withheld — anchor status: not_found]
- [abstract] MCEMP1 and HLA-DRA differential expression occurs in CD14+ cells: "we found significantly heightened MCEMP1 and reduced HLA-DRA expression as early as four days before the nadir of respiratory function"
- [discussion] Timing of dysregulation earlier than 4 days pre-nadir cannot be determined: "as there are currently no studies documenting gene expression profiles more than 4 days before nadir of respiratory function, we were unable to determine precisely when the dysregulation of MCEMP1"
- [readme] Database contents and app usage from the repository README: "This database is a curation of 7 transcriptomics datasets comparing the gene expression in severe COVID-19 and mild COVID-19 subjects."
