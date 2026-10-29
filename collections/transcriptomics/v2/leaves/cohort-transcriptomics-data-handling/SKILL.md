---
name: cohort-transcriptomics-data-handling
description: Use when processed cohort transcript tables must be aligned sample-wise
  with matched clinical/lab measurements (e.g. CRP, lymphocyte counts, IFN score)
  to test whether candidate genes such as MCEMP1, HLA-DRA or ETS1 track clinical predictors
  of severity.
when_to_use_negative:
- Goal is severe-vs-mild differential expression (fold-change, BH-adjusted p-values)
  or box/strip plot visualisation of raw counts — use the repository's Streamlit app
  workflow instead of correlation analysis
- Input is single-cell RNA-seq data or single-cell subset deconvolution rather than
  bulk whole-blood cohort tables with matched clinical variables
- Clinical variables are not matched sample-wise to the transcript rows (no CRP/lymphocyte/IFN-score
  columns aligned to the same samples)
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Streamlit
  role: Web app instance of the COVID19-severity database used to query gene expression
    differences; can also be launched locally to explore the processed data this skill
    consumes
  repo: https://github.com/kuanrongchan/COVID19-severity
- name: pip
  role: Installs the repository's Python dependencies (requirements.txt) needed to
    load and process the cohort tables
  repo: https://github.com/kuanrongchan/COVID19-severity
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:cc6811ef0cede83b9efac32c167488aa0b64374a8133f69f7e04755057e1530d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Streamlit
  - pip
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cohort-transcriptomics-data-handling

## Summary

Loads and aligns processed whole-blood transcript abundance tables from the COVID19-severity repository (Ong et al. 2021 longitudinal cohort) with matched clinical variables — CRP levels, lymphocyte counts and the 28-gene type I IFN score — to compute sample-wise gene–variable Pearson correlations for genes such as MCEMP1, HLA-DRA and ETS1.

## When to use

Use when processed cohort transcript tables must be aligned sample-wise with matched clinical/lab measurements (e.g. CRP, lymphocyte counts, IFN score) to test whether candidate genes such as MCEMP1, HLA-DRA or ETS1 track clinical predictors of severity.

## When NOT to use

- Goal is severe-vs-mild differential expression (fold-change, BH-adjusted p-values) or box/strip plot visualisation of raw counts — use the repository's Streamlit app workflow instead of correlation analysis
- Input is single-cell RNA-seq data or single-cell subset deconvolution rather than bulk whole-blood cohort tables with matched clinical variables
- Clinical variables are not matched sample-wise to the transcript rows (no CRP/lymphocyte/IFN-score columns aligned to the same samples)

## Inputs

- Processed transcript measurement tables from the COVID19-severity repository (Ong et al. 2021 longitudinal cohort)
- Matched per-sample clinical variables: CRP levels, lymphocyte counts
- Per-sample 28-gene type I IFN score
- Focal gene list (MCEMP1, HLA-DRA, ETS1)

## Outputs

- Labelled table of Pearson r and two-sided p-values for all gene–variable pairs, saved as fig2_pearson_correlations.csv

## How to apply

Download the processed data distributed with the COVID19-severity repository (github.com/kuanrongchan/COVID19-severity) and read the tables with pandas; note that processed files carry p-value (t-test), adjusted p-value (BH step-up procedure), ratio and fold-change between severe and mild COVID-19, while the raw counts are separate in the counts folder. Extract the rows for the three focal genes (MCEMP1, HLA-DRA, ETS1) and align them sample-wise with the CRP, lymphocyte-count and 28-gene type I IFN-score columns. Compute all pairwise Pearson coefficients (r and two-sided p-values) between each gene and each clinical/IFN-score variable using scipy.stats.pearsonr. Validate each coefficient's sign against the reported Fig. 2 directions — MCEMP1 positive and HLA-DRA negative with CRP and IFN score, MCEMP1 inverse and ETS1/HLA-DRA positive with lymphocyte counts, ETS1 not correlated with CRP — then assemble r and p values into a labelled CSV (fig2_pearson_correlations.csv).

## Related tools

- **Streamlit** (Web app instance of the COVID19-severity database used to query gene expression differences; can also be launched locally to explore the processed data this skill consumes) — https://github.com/kuanrongchan/COVID19-severity
- **pip** (Installs the repository's Python dependencies (requirements.txt) needed to load and process the cohort tables) — https://github.com/kuanrongchan/COVID19-severity

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(gene_df['MCEMP1'], clinical_df['CRP'])
```

## Evaluation signals

- Recomputed Pearson coefficients reproduce the sign (and approximate magnitude) of every association reported in Fig. 2: MCEMP1 positive and HLA-DRA negative with CRP and the 28-gene type I IFN score
- MCEMP1 is inversely correlated and ETS1/HLA-DRA positively correlated with lymphocyte counts
- ETS1-versus-CRP yields a non-significant r, matching the reported finding
- The output CSV contains r and p for all 9 gene–variable pairs (3 genes × 3 clinical/IFN variables) with correct labels
- Gene rows and clinical columns are aligned sample-wise (equal-length vectors with no unaligned samples) before pearsonr is called

## Limitations

- Correlation signs come from a single longitudinal cohort (Ong et al. 2021); dataset heterogeneity across the 7 transcriptomics cohorts was explicitly noted by the authors
- Only one gene can be selected at a time for graph plotting in the repository's app, so multi-gene correlation requires working directly with the processed tables
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severe COVID-19 have not been prospectively validated; these correlations do not constitute clinical validation

## Evidence

- [discussion] All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity: "All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity"
- [readme] Processed-file schema: "The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data between severe COVID-19 and mild COVID-19"
- [readme] Dependency install for the environment used to handle the cohort tables: "pip install -r requirements.txt"
- [abstract] Longitudinal cohort basis for aligning transcripts with clinical trajectories: "blood transcriptomics of COVID-19 patients were prospectively and longitudinally monitored previously, to track the time in which these gene expression changes occur"
