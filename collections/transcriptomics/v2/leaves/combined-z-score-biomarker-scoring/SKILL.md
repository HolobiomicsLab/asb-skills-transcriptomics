---
name: combined-z-score-biomarker-scoring
description: Use when you need to convert differential expression of a small gene
  panel (e.g. MCEMP1 up, HLA-DRA down) into a single per-subject severity score and
  quantify class separation between severe and mild disease groups via AUC.
when_to_use_negative:
- The analysis goal is differential expression testing (t-test with BH-adjusted p-values)
  rather than building a per-subject severity classifier score
- The marker panel includes genes regulated in the same direction whose Z-scores cannot
  simply be averaged without re-deriving coefficient directions
- Cohorts lack severe-vs-mild group labels, so no positive class exists for ROC analysis
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: Streamlit
  role: Webtool developed by the authors to query gene expression differences between
    severe and mild COVID-19 patients across the curated datasets
  repo: https://github.com/kuanrongchan/COVID19-severity
- name: pip
  role: Installs the dependencies (requirements.txt) needed to run the Streamlit app
    locally for exploring the datasets used to derive the combined score
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:f9783593c7c1a82ce470ca59e7873df6c5c10740884449dbeef6ac43a7044953
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Streamlit
  - pip
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# combined-z-score-biomarker-scoring

## Summary

Forms a two-gene severity metric by z-score normalising MCEMP1 and HLA-DRA expression per cohort and averaging the two Z-scores per subject, then evaluating severe-vs-mild discrimination with ROC/AUC. Used to test whether the COVID-19-derived biomarker signature generalises to influenza, RSV and dengue cohorts.

## When to use

Use when you need to convert differential expression of a small gene panel (e.g. MCEMP1 up, HLA-DRA down) into a single per-subject severity score and quantify class separation between severe and mild disease groups via AUC.

## When NOT to use

- The analysis goal is differential expression testing (t-test with BH-adjusted p-values) rather than building a per-subject severity classifier score
- The marker panel includes genes regulated in the same direction whose Z-scores cannot simply be averaged without re-deriving coefficient directions
- Cohorts lack severe-vs-mild group labels, so no positive class exists for ROC analysis

## Inputs

- Severe-vs-mild expression matrices for each cohort (raw/processed count data from GEO or Mendeley Data, e.g. GSE155454, GSE161731, GSE157103, Bibert 2021 10.17632/8wxhhykfnh.2)
- Per-sample severe/mild phenotype labels
- Gene expression values for the marker panel genes (MCEMP1, HLA-DRA)

## Outputs

- Per-subject combined Z-score severity metric
- ROC curves and AUC values per cohort (scikit-learn roc_curve/roc_auc_score)
- Summary table roc_results.csv with AUC, confidence direction and discrimination outcome per cohort and disease

## How to apply

Per cohort, z-score normalise the expression values of each marker gene (MCEMP1 and HLA-DRA) across all samples, then average the two gene Z-scores per subject to form the combined Z-score severity metric. Treat severe status as the positive class and compute ROC curves and AUC with scikit-learn's roc_curve/roc_auc_score for severe vs mild groups in each cohort. Tabulate AUC, confidence direction and discrimination outcome (significant vs no discrimination, AUC~0.5) per cohort and disease, saving the summary as roc_results.csv. Because MCEMP1 is heightened and HLA-DRA reduced in severe disease, the two Z-scores are averaged directly so the combined score rises with severity. Validation requires reproducing the reported cross-pathogen pattern: severe influenza discriminated in 2 of 3 cohorts, severe RSV in 2 independent studies, and no discrimination for dengue.

## Related tools

- **Streamlit** (Webtool developed by the authors to query gene expression differences between severe and mild COVID-19 patients across the curated datasets) — https://github.com/kuanrongchan/COVID19-severity
- **pip** (Installs the dependencies (requirements.txt) needed to run the Streamlit app locally for exploring the datasets used to derive the combined score)

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; z = (expr['MCEMP1'].mean() - expr['MCEMP1']) / expr['MCEMP1'].std(); auc = roc_auc_score(severe_labels, combined_z_score)
```

## Evaluation signals

- Reproduced ROC pattern must match the paper: discrimination of severe influenza in 2 of 3 cohorts, severe RSV in 2 of 2 studies, and no discrimination for dengue (AUC~0.5)
- Each cohort's AUC is computed treating severe status as the positive class, so AUC > 0.5 indicates the combined Z-score is higher in severe subjects
- roc_results.csv contains one row per cohort with AUC, confidence direction and a significant-vs-no-discrimination outcome label
- Per-cohort Z-scores for each gene have mean ~0 and SD ~1 across samples before averaging

## Limitations

- Discrimination of severe influenza was achieved in only 2 of 3 cohorts and no discrimination was observed for dengue, indicating incomplete cross-pathogen generalisability
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severe disease have not been prospectively validated
- Heterogeneity across study datasets was noted, which may influence MCEMP1 and HLA-DRA expression

## Evidence

- [results] Applying the combined MCEMP1/HLA-DRA score, severe from mild influenza was discriminated in only 2 of 3 cohorts, severe RSV was discriminated in 2 independent studies, and no discrimination was observed for dengue.: [evidence span withheld — anchor status: not_found]
- [supplementary] Z-score normalisation and averaging workflow: [evidence span withheld — anchor status: not_found]
- [supplementary] ROC computation with scikit-learn: [evidence span withheld — anchor status: not_found]
- [abstract] Direction of differential expression: "Elevated MCEMP1 and reduced HLA-DRA gene expression in CD14+ cells during the early phase of disease are prognostic of severe COVID-19"
- [supplementary] Validation criterion: [evidence span withheld — anchor status: not_found]
- [supplementary] Output file specification: [evidence span withheld — anchor status: not_found]
- [discussion] Prospective validation gap: "Future prospective studies will also be needed to validate the sensitivity and specificity of using MCEMP1 and HLA-DRA expression levels in predicting severe COVID-19"
