---
name: bootstrap-confidence-interval-estimation
description: Use when aggregating per-dataset performance metrics (e.g. ROC AUCs of
  the MCEMP1/HLA-DRA Z-score across GSE155454, Bibert 2021, GSE161731, GSE157103,
  GSE152418, GSE172114, GSE164805) and you need a 95% CI that reflects cohort-level
  variability rather than subject-level sampling.
when_to_use_negative:
- You need patient-level confidence intervals within a single cohort — bootstrap over
  dataset-level AUCs measures cross-cohort variability, not within-cohort sampling
  error
- The input is a single AUC from one dataset (e.g. only GSE172114) with no multi-cohort
  replication, so there is nothing to resample at the dataset level
- The gene set or severity definition differs from severe-vs-mild COVID-19 classification
  (e.g. severe-vs-healthy comparisons in the processed anova files), since the reported
  0.865 AUC benchmark applies to severe-vs-mild only
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scikit-learn (sklearn.metrics)
  role: roc_curve and roc_auc_score compute the severe-vs-mild ROC curves and dataset-level
    AUCs that are bootstrapped
- name: sklearn.preprocessing.StandardScaler
  role: Z-score normalises MCEMP1 and HLA-DRA raw counts within each dataset before
    computing the combined score
- name: matplotlib
  role: Plots the per-dataset ROC curves summarised by the bootstrapped mean AUC and
    CI
- name: pandas
  role: Loads the processed raw counts for MCEMP1 and HLA-DRA from the kuanrongchan/COVID19-severity
    repository and writes roc_auc_results.csv
  repo: https://github.com/kuanrongchan/COVID19-severity
- name: pip
  role: Installs the repository dependencies (pip install -r requirements.txt) needed
    to run the analysis environment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:7fa482ea2383ccd15be40a19bc4359ca1f9ff1002e9b085d425a5cba216edc80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Streamlit
  - matplotlib
  - pandas
  - pip
  - scikit-learn (sklearn.metrics)
  - sklearn.preprocessing.StandardScaler
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# bootstrap-confidence-interval-estimation

## Summary

Estimate a mean AUC with a 95% confidence interval by bootstrapping over the 7 dataset-level AUCs of the combined Z(MCEMP1) - Z(HLA-DRA) severity score, quantifying how consistently the biomarker pair discriminates severe from mild COVID-19 across independent RNAseq cohorts.

## When to use

Use when aggregating per-dataset performance metrics (e.g. ROC AUCs of the MCEMP1/HLA-DRA Z-score across GSE155454, Bibert 2021, GSE161731, GSE157103, GSE152418, GSE172114, GSE164805) and you need a 95% CI that reflects cohort-level variability rather than subject-level sampling.

## When NOT to use

- You need patient-level confidence intervals within a single cohort — bootstrap over dataset-level AUCs measures cross-cohort variability, not within-cohort sampling error
- The input is a single AUC from one dataset (e.g. only GSE172114) with no multi-cohort replication, so there is nothing to resample at the dataset level
- The gene set or severity definition differs from severe-vs-mild COVID-19 classification (e.g. severe-vs-healthy comparisons in the processed anova files), since the reported 0.865 AUC benchmark applies to severe-vs-mild only

## Inputs

- 7 dataset-level AUC values of the combined Z(MCEMP1) - Z(HLA-DRA) score (one per dataset: GSE155454, Bibert 2021, GSE161731, GSE157103, GSE152418, GSE172114, GSE164805)
- Subject-level MCEMP1 and HLA-DRA raw counts with severe/mild labels from the kuanrongchan/COVID19-severity GitHub repository
- Pairwise subject-level scores and labels in GSE172114 for the correlated DeLong comparison

## Outputs

- Mean AUC with bootstrapped 95% confidence interval for the combined score (expected ~0.865, CI 0.75–1.00)
- Per-dataset AUC values and ROC curves plotted with matplotlib
- roc_auc_results.csv containing per-dataset AUCs, mean AUC, 95% CI, and DeLong p-value
- DeLong test p-value for combined score vs HLA-DRA alone in GSE172114 (significant if < 0.05)

## How to apply

First compute a dataset-level AUC per cohort using sklearn.metrics.roc_curve and roc_auc_score on the subject-level combined score Z(MCEMP1) - Z(HLA-DRA), where each gene's raw counts from the kuanrongchan/COVID19-severity repository are z-score normalised within dataset with sklearn.preprocessing.StandardScaler. Then resample the 7 dataset-level AUCs with replacement (bootstrap) to obtain the distribution of the mean AUC, and take the 2.5th and 97.5th percentiles as the 95% CI. The rationale is that with only 7 independent cohorts (140 severe, 181 mild patients total), the dominant uncertainty is between-cohort heterogeneity, so bootstrapping at the dataset level gives an honest CI. In the GSE172114 sub-analysis (46 severe, 23 mild), pair the AUC comparison against HLA-DRA alone with a DeLong test for correlated ROC curves at alpha = 0.05. Write per-dataset AUCs, mean AUC, 95% CI, and the DeLong p-value to roc_auc_results.csv alongside the per-dataset ROC plot.

## Related tools

- **scikit-learn (sklearn.metrics)** (roc_curve and roc_auc_score compute the severe-vs-mild ROC curves and dataset-level AUCs that are bootstrapped)
- **sklearn.preprocessing.StandardScaler** (Z-score normalises MCEMP1 and HLA-DRA raw counts within each dataset before computing the combined score)
- **matplotlib** (Plots the per-dataset ROC curves summarised by the bootstrapped mean AUC and CI)
- **pandas** (Loads the processed raw counts for MCEMP1 and HLA-DRA from the kuanrongchan/COVID19-severity repository and writes roc_auc_results.csv) — https://github.com/kuanrongchan/COVID19-severity
- **pip** (Installs the repository dependencies (pip install -r requirements.txt) needed to run the analysis environment)

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, _ = roc_curve(y_sev, z_mcmp1 - z_hladra); auc = roc_auc_score(y_sev, z_mcmp1 - z_hladra)
```

## Evaluation signals

- The bootstrapped mean AUC of the combined score reproduces the reported 0.865 with 95% CI 0.75–1.00
- ROC curves consistently separate severe from mild patients in all 7 datasets, so every per-dataset AUC should be well above 0.5
- The DeLong test in GSE172114 (46 severe, 23 mild) yields a significant p-value < 0.05 favouring the combined score over HLA-DRA alone
- roc_auc_results.csv contains exactly 7 per-dataset AUC rows plus mean AUC, 95% CI, and DeLong p-value columns
- Per-gene z-scores within each dataset have mean ~0 and standard deviation ~1 before the combined score is computed

## Limitations

- Only 7 dataset-level AUCs are resampled, so the bootstrap CI is coarse and sensitive to cohort heterogeneity, which the authors explicitly noted (Supplementary Fig. S9)
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severe COVID-19 have not been prospectively validated
- The included datasets did not directly evaluate the effect of different SARS-CoV-2 variants on transcriptomic responses, limiting generalisation of the AUC estimate

## Evidence

- [methods] bootstrap mean AUC and 95% CI computation over the 7 datasets: [evidence span withheld — anchor status: not_found]
- [results] reported benchmark performance of the combined score: [evidence span withheld — anchor status: not_found]
- [readme] GSE172114 cohort composition used for the DeLong sub-analysis: "Dataset compares the gene expression differences between 46 severe COVID-19 and 23 mild COVID-19 subjects."
- [abstract] biological basis of the combined score across cohorts: "The most consistent differentially regulated genes in peripheral blood of severe COVID-19 patients were MCEMP1, HLA-DRA and ETS1 across the 7 transcriptomics datasets."
