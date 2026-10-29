---
name: covid-19-severity-biomarker-association-analysis
description: Use when whole-blood transcript measurements for candidate severity biomarkers
  must be aligned sample-wise with clinical/lab variables (CRP, lymphocyte counts,
  type I IFN score) and tested for Pearson correlation sign and significance.
when_to_use_negative:
- Input is raw counts from one of the 7 severe-vs-mild comparison datasets with no
  matched clinical/lab metadata — those support differential expression analysis,
  not biomarker-clinical correlation.
- The goal is differential expression between severe and mild patients (t-test, BH
  adjusted p-value, fold-change), not association of transcripts with continuous clinical
  predictors.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Streamlit
  role: The COVID19-severity app distributes and lets users query the processed transcriptomics
    databases from which the Ong et al. cohort data and clinical variables are drawn
  repo: https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
- name: pip
  role: Installs the repository dependencies (requirements.txt) needed to run the
    app and the analysis environment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:a93362d729727e3da9472cdbfcb83d9fe2a275b478a32780b3c4d83f83609d16
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Streamlit
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# covid-19-severity-biomarker-association-analysis

## Summary

Reproduces and validates Pearson correlations between prognostic COVID-19 biomarker transcripts (MCEMP1, HLA-DRA, ETS1) and clinical severity predictors (CRP, 28-gene type I IFN score, lymphocyte counts) using the processed Ong et al. 2021 longitudinal cohort data distributed with the COVID19-severity repository.

## When to use

Use when whole-blood transcript measurements for candidate severity biomarkers must be aligned sample-wise with clinical/lab variables (CRP, lymphocyte counts, type I IFN score) and tested for Pearson correlation sign and significance.

## When NOT to use

- Input is raw counts from one of the 7 severe-vs-mild comparison datasets with no matched clinical/lab metadata — those support differential expression analysis, not biomarker-clinical correlation.
- The goal is differential expression between severe and mild patients (t-test, BH adjusted p-value, fold-change), not association of transcripts with continuous clinical predictors.

## Inputs

- Processed Ong et al. 2021 longitudinal COVID-19 cohort transcript table (whole-blood gene expression, from the COVID19-severity repository counts/anova data)
- Matched clinical variables per sample: CRP levels, lymphocyte counts
- 28-gene type I IFN score per sample

## Outputs

- Labelled table of Pearson r and two-sided p-values for each gene-by-variable pair, saved as fig2_pearson_correlations.csv
- Sign-validation verdict for each association against the reported Fig. 2 directions

## How to apply

Load the processed Ong et al. cohort transcript table together with matched CRP, lymphocyte-count and 28-gene type I IFN-score values from the COVID19-severity GitHub repository using pandas. Extract the rows for MCEMP1, HLA-DRA and ETS1 and align them sample-wise with the clinical variable columns. Compute all pairwise Pearson correlation coefficients (r and two-sided p-values) between each of the three genes and each of the three clinical/IFN-score variables with scipy.stats.pearsonr. Verify each coefficient's sign against the Fig. 2 directions: MCEMP1 positive and HLA-DRA negative with CRP and the IFN score, MCEMP1 inverse and ETS1/HLA-DRA positive with lymphocyte counts, and ETS1 not significantly correlated with CRP. Assemble the r and p values into a labelled table and save as fig2_pearson_correlations.csv.

## Related tools

- **Streamlit** (The COVID19-severity app distributes and lets users query the processed transcriptomics databases from which the Ong et al. cohort data and clinical variables are drawn) — https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
- **pip** (Installs the repository dependencies (requirements.txt) needed to run the app and the analysis environment)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(df['MCEMP1'], df['CRP'])
```

## Evaluation signals

- MCEMP1-versus-CRP Pearson r is positive and statistically significant.
- HLA-DRA-versus-CRP Pearson r is negative and statistically significant.
- ETS1-versus-CRP yields a non-significant r.
- MCEMP1 and HLA-DRA are significantly correlated with the 28-gene type I IFN score with the expected signs (MCEMP1 positive, HLA-DRA negative).
- With lymphocyte counts, MCEMP1 is inversely correlated while ETS1 and HLA-DRA are positively correlated, matching the reported Fig. 2 signs and approximate magnitudes.

## Limitations

- Validation accepts output only if recomputed coefficients reproduce the sign (and approximate magnitude) of every Fig. 2 association; sign flips or magnitude drifts indicate misalignment of samples or wrong variable columns.
- Heterogeneity across the study datasets was noted, so associations derived from the Ong et al. cohort may not generalise to other cohorts or variants.
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severe COVID-19 have not been validated; prospective validation is still needed.

## Evidence

- [methods] Compute all pairwise Pearson correlation coefficients (r and two-sided p-values) between each of the three genes and each of the three clinical/IFN-score variables using scipy.stats.pearsonr.: [evidence span withheld — anchor status: not_found]
- [results] Validation criterion: [evidence span withheld — anchor status: not_found]
- [readme] Data source: "This database is a curation of 7 transcriptomics datasets comparing the gene expression in severe COVID-19 and mild COVID-19 subjects."
- [discussion] Reproducibility note: "All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity"
