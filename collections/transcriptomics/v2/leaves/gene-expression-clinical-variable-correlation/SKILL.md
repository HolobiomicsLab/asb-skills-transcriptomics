---
name: gene-expression-clinical-variable-correlation
description: Use when whole-blood gene expression measurements and matched per-sample
  clinical laboratory values (e.g. CRP, lymphocyte counts, IFN scores) are available
  and you need to test whether gene abundance is linearly associated with disease-severity
  markers.
when_to_use_negative:
- The data are group-level differential-expression summaries (p-value, adjusted p-value,
  fold-change between severe and mild COVID-19, as in the anova folder files) rather
  than sample-wise matched expression and clinical values, so per-sample correlation
  is impossible.
- The relationship between gene and clinical variable is expected to be non-linear
  or monotonic-only, where a Pearson r would misrepresent the association.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0091
tools:
- name: Streamlit
  role: Hosted webtool for querying the COVID19-severity processed gene-expression
    databases used as the correlation input
  repo: https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
- name: pip
  role: Installs repository dependencies (requirements.txt) needed to run the Streamlit
    app and reproduce the processed data environment
  repo: https://github.com/kuanrongchan/COVID19-severity
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:80f3a13be4abbf59f4f76671cdf7a0ad79e003d45a56717f2ef201ad47667f8a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Streamlit
  - pip
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-expression-clinical-variable-correlation

## Summary

Computes sample-wise Pearson correlations between focal gene transcript abundances (e.g. MCEMP1, HLA-DRA, ETS1 from the Ong et al. 2021 COVID-19 cohort) and clinical laboratory variables (CRP, lymphocyte counts, 28-gene type I IFN score) to validate that severity-associated transcripts track clinical predictors of severe disease.

## When to use

Use when whole-blood gene expression measurements and matched per-sample clinical laboratory values (e.g. CRP, lymphocyte counts, IFN scores) are available and you need to test whether gene abundance is linearly associated with disease-severity markers.

## When NOT to use

- The data are group-level differential-expression summaries (p-value, adjusted p-value, fold-change between severe and mild COVID-19, as in the anova folder files) rather than sample-wise matched expression and clinical values, so per-sample correlation is impossible.
- The relationship between gene and clinical variable is expected to be non-linear or monotonic-only, where a Pearson r would misrepresent the association.

## Inputs

- Processed Ong et al. 2021 cohort whole-blood transcript abundance table (pandas-readable) containing rows for MCEMP1, HLA-DRA and ETS1
- Matched per-sample clinical variables: CRP levels, lymphocyte counts, and 28-gene type I IFN score from the processed COVID19-severity repository data

## Outputs

- Labelled table of Pearson r and two-sided p-values for each gene-by-variable pair, saved as fig2_pearson_correlations.csv
- Directional verification of each coefficient's sign against Fig. 2 (MCEMP1+/HLA-DRA- with CRP and IFN score; MCEMP1-/ETS1+/HLA-DRA+ with lymphocytes; ETS1-CRP non-significant)

## How to apply

Load the processed Ong et al. cohort transcript table and the matched CRP, lymphocyte-count and 28-gene type I IFN-score columns from the COVID19-severity repository (github.com/kuanrongchan/COVID19-severity) with pandas, then extract the rows for the focal genes and align them sample-wise with the clinical columns. Compute all pairwise Pearson coefficients (r and two-sided p-values) between each gene and each clinical/IFN-score variable using scipy.stats.pearsonr. Verify the sign of each coefficient against the reported Fig. 2 directions: MCEMP1 positive with CRP and IFN score, HLA-DRA negative with CRP and IFN score, MCEMP1 inverse and ETS1/HLA-DRA positive with lymphocyte counts, with ETS1-versus-CRP expected to be non-significant. Assemble r and p values into a labelled table (fig2_pearson_correlations.csv); accept the output if every recomputed coefficient reproduces the sign and approximate magnitude reported in Fig. 2.

## Related tools

- **Streamlit** (Hosted webtool for querying the COVID19-severity processed gene-expression databases used as the correlation input) — https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
- **pip** (Installs repository dependencies (requirements.txt) needed to run the Streamlit app and reproduce the processed data environment) — https://github.com/kuanrongchan/COVID19-severity

## Examples

```
import pandas as pd; from scipy.stats import pearsonr; df = pd.read_csv('ong_cohort_processed.csv'); r, p = pearsonr(df['MCEMP1'], df['CRP'])
```

## Evaluation signals

- Every recomputed Pearson coefficient reproduces the sign of the Fig. 2 association: MCEMP1 positive and HLA-DRA negative with CRP and IFN score; MCEMP1 inverse and ETS1/HLA-DRA positive with lymphocyte counts.
- ETS1-versus-CRP yields a non-significant r (p above the significance threshold), matching the reported absence of correlation.
- MCEMP1 and HLA-DRA (but not ETS1) show significant correlation with the type I IFN score, as reported.
- Output table fig2_pearson_correlations.csv contains labelled rows for all 3 genes x 3 variables with both r and two-sided p-value columns, and magnitudes approximate the reported values.

## Limitations

- Correlations are from a single longitudinal cohort (Ong et al. 2021); heterogeneity in patients influencing MCEMP1 and HLA-DRA expression was noted, so coefficients may not generalise across datasets.
- Validation depends on the reported Fig. 2 directions; sensitivity and specificity of MCEMP1/HLA-DRA as prognostic markers have not been prospectively validated.
- Only one gene can be queried at a time in the Streamlit app, so multi-gene correlation must be done from the downloaded processed files rather than the webtool.

## Evidence

- [other] Compute all pairwise Pearson correlation coefficients (r and two-sided p-values) between each of the three genes and each of the three clinical/IFN-score variables using scipy.stats.pearsonr.: [evidence span withheld — anchor status: not_found]
- [results] MCEMP1 transcript abundance was significantly positively correlated with CRP levels while HLA-DRA was inversely correlated: [evidence span withheld — anchor status: not_found]
- [discussion] All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity: "All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity"
- [readme] The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data: "The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data between severe COVID-19 and mild COVID-19"
