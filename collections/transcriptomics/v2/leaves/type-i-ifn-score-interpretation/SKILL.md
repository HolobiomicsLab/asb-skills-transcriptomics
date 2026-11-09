---
name: type-i-ifn-score-interpretation
description: Use when whole-blood transcript levels (e.g. MCEMP1, HLA-DRA, ETS1) must
  be correlated sample-wise with clinical severity predictors — CRP, lymphocyte counts
  or the 28-gene type I IFN score — in the Ong et al. COVID-19 cohort.
when_to_use_negative:
- The goal is differential expression between severe and mild patients across the
  7 cross-sectional datasets (Fong, Bibert, McClain, Overmyer, Arunachalam, Carapito,
  Zhang) — use the repository's counts/anova box-and-strip workflow instead of this
  correlation analysis
- No matched CRP, lymphocyte-count or type I IFN-score measurements are available
  for the transcript samples, so sample-wise Pearson correlation cannot be computed
- The analysis requires non-linear or partial (confounder-adjusted) associations rather
  than two-sided Pearson correlation of raw values
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Streamlit
  role: Webtool underlying the COVID19-severity repository from which the processed
    Ong et al. cohort data and IFN-score inputs are queried and distributed
  repo: https://github.com/kuanrongchan/COVID19-severity
- name: pip
  role: Installs the repository dependencies (requirements.txt, including pandas and
    scipy) needed to run the correlation computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:0ae9c0df57a8315d19aa04c486d09a3086418e12122de249b6b00fc97b9a2235
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Streamlit
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# type-i-ifn-score-interpretation

## Summary

Interprets whole-blood expression of the COVID-19 severity genes MCEMP1, HLA-DRA and ETS1 by quantifying their Pearson correlations with the 28-gene type I IFN score, CRP levels and lymphocyte counts in the longitudinal Ong et al. 2021 cohort, reproducing the association directions reported in Fig. 2 of Chan et al. 2023.

## When to use

Use when whole-blood transcript levels (e.g. MCEMP1, HLA-DRA, ETS1) must be correlated sample-wise with clinical severity predictors — CRP, lymphocyte counts or the 28-gene type I IFN score — in the Ong et al. COVID-19 cohort.

## When NOT to use

- The goal is differential expression between severe and mild patients across the 7 cross-sectional datasets (Fong, Bibert, McClain, Overmyer, Arunachalam, Carapito, Zhang) — use the repository's counts/anova box-and-strip workflow instead of this correlation analysis
- No matched CRP, lymphocyte-count or type I IFN-score measurements are available for the transcript samples, so sample-wise Pearson correlation cannot be computed
- The analysis requires non-linear or partial (confounder-adjusted) associations rather than two-sided Pearson correlation of raw values

## Inputs

- Processed Ong et al. cohort transcript measurement table (rows = genes including MCEMP1, HLA-DRA, ETS1; columns = samples) from the COVID19-severity repository
- Matched per-sample clinical variables: CRP levels, lymphocyte counts
- Per-sample 28-gene type I IFN score values
- requirements.txt listing pandas/scipy dependencies for the COVID19-severity repository

## Outputs

- fig2_pearson_correlations.csv — labelled table of Pearson r and two-sided p-values for each gene-by-variable pair
- Signed correlation matrix confirming Fig. 2 directions (MCEMP1+/HLA-DRA- with CRP and IFN score; MCEMP1- and ETS1/HLA-DRA+ with lymphocyte counts)

## How to apply

Load the processed Ong et al. cohort transcript measurements together with matched CRP, lymphocyte-count and 28-gene type I IFN-score values from the processed data distributed with the COVID19-severity repository, reading the tables with pandas. Extract the rows for MCEMP1, HLA-DRA and ETS1 and align them sample-wise with the CRP, lymphocyte and IFN-score columns. Compute all pairwise Pearson correlation coefficients (r and two-sided p-values) between each of the three genes and each of the three clinical/IFN-score variables using scipy.stats.pearsonr. Verify each coefficient's sign against the reported Fig. 2 directions: MCEMP1 positive and HLA-DRA negative with CRP and IFN score, MCEMP1 inverse and ETS1/HLA-DRA positive with lymphocyte counts, and ETS1 not correlated with CRP. Assemble the r and p values into a labelled table saved as fig2_pearson_correlations.csv.

## Related tools

- **Streamlit** (Webtool underlying the COVID19-severity repository from which the processed Ong et al. cohort data and IFN-score inputs are queried and distributed) — https://github.com/kuanrongchan/COVID19-severity
- **pip** (Installs the repository dependencies (requirements.txt, including pandas and scipy) needed to run the correlation computation)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(mcmp1_expr_aligned, ifn_score_aligned)
```

## Evaluation signals

- Recomputed Pearson r for every gene-by-variable pair has the same sign as Fig. 2: MCEMP1 positive with CRP, HLA-DRA inverse with CRP
- MCEMP1 and HLA-DRA correlations with the type I IFN score are significant, while ETS1 shows no significant correlation with the IFN score
- ETS1-versus-CRP yields a non-significant r, matching the reported finding
- With lymphocyte counts, MCEMP1 is inversely correlated and both ETS1 and HLA-DRA positively correlated
- Output file fig2_pearson_correlations.csv exists and contains labelled r and two-sided p-value columns for all nine gene-by-variable pairs

## Limitations

- Correlations with CRP, lymphocyte counts and the IFN score are associative and do not establish that MCEMP1/HLA-DRA expression causally tracks disease mechanisms
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severe COVID-19 have not been validated; prospective studies are still needed
- The repository README documents only 7 cross-sectional severe-vs-mild datasets; the longitudinal Ong et al. cohort used for this correlation skill must be obtained separately as processed data
- Only one gene can be selected at a time in the repository's app for graph plotting, so multi-gene comparison requires running the correlation code outside the app

## Evidence

- [results] MCEMP1 transcript abundance was significantly positively correlated with CRP levels while HLA-DRA was inversely correlated: [evidence span withheld — anchor status: not_found]
- [methods] Compute all pairwise Pearson correlation coefficients: [evidence span withheld — anchor status: not_found]
- [abstract] Elevated MCEMP1 and reduced HLA-DRA gene expression in CD14+ cells during the early phase of disease are prognostic of severe COVID-19.: "Elevated MCEMP1 and reduced HLA-DRA gene expression in CD14+ cells during the early phase of disease are prognostic of severe COVID-19."
- [readme] README: processed data distribution: "This database is a curation of 7 transcriptomics datasets comparing the gene expression in severe COVID-19 and mild COVID-19 subjects."
- [readme] README: launching the analysis environment: "In the command-line interface (command prompt on Windows, Terminal on MacOS/Linux), navigate to the directory the file is located at and install its dependencies"
