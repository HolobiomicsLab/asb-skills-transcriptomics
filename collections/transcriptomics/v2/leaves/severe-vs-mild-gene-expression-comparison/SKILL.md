---
name: severe-vs-mild-gene-expression-comparison
description: Use when severe-vs-mill or severe-vs-mild gene expression data from a
  transcriptomic cohort (e.g. GEO RNAseq matrices) is available and the goal is to
  test whether a fixed two-gene signature discriminates disease severity, including
  cross-cohort or cross-pathogen validation.
when_to_use_negative:
- Input data lacks severity labels or a severe/mild grouping, since no positive class
  can be defined for roc_auc_score
- The candidate genes (MCEMP1, HLA-DRA) are absent from the expression matrix, as
  the combined Z-score cannot be formed
- Only one cohort with no cross-cohort or cross-pathogen replication goal exists and
  a full differential-expression re-analysis is wanted instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Streamlit
  role: Web app through which the curated severe-vs-mild COVID-19 transcriptomics
    datasets and per-gene severe/mild comparisons can be queried and downloaded
  repo: https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
- name: pip
  role: Installs dependencies (requirements.txt) needed to run the Streamlit app locally
    for dataset access
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:c9a84e6755d90c4851b1ec2ddbd255a04e6d4feaa257a9262d8b08b8071407f9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Streamlit
  - pip
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# severe-vs-mild-gene-expression-comparison

## Summary

Compares gene expression between severe and mild viral infection patients by z-scoring candidate genes (MCEMP1, HLA-DRA) per cohort, averaging the two Z-scores into a combined severity metric, and computing ROC/AUC with scikit-learn to test whether the signature discriminates severity within and across pathogens.

## When to use

Use when severe-vs-mill or severe-vs-mild gene expression data from a transcriptomic cohort (e.g. GEO RNAseq matrices) is available and the goal is to test whether a fixed two-gene signature discriminates disease severity, including cross-cohort or cross-pathogen validation.

## When NOT to use

- Input data lacks severity labels or a severe/mild grouping, since no positive class can be defined for roc_auc_score
- The candidate genes (MCEMP1, HLA-DRA) are absent from the expression matrix, as the combined Z-score cannot be formed
- Only one cohort with no cross-cohort or cross-pathogen replication goal exists and a full differential-expression re-analysis is wanted instead

## Inputs

- Severe-vs-mill severity-labelled expression matrices (raw counts or normalised) from GEO/Mendeley Data, e.g. GSE155454, GSE161731, GSE157103, GSE152418, GSE172114, GSE164805, Bibert 2021 (10.17632/8wxhhykfnh.2)
- Accession list mapping cohorts to diseases (Supplementary Table S5)
- Candidate gene list: MCEMP1 and HLA-DRA expression values per sample

## Outputs

- Per-subject combined MCEMP1/HLA-DRA Z-score severity metric per cohort
- ROC curves and AUC values per cohort (9 cohorts across influenza, RSV, dengue)
- roc_results.csv summarising AUC, confidence direction and discrimination outcome per cohort/disease

## How to apply

Retrieve accession numbers (e.g. from Supplementary Table S5) and download severe-vs-mild expression matrices from GEO/Mendeley Data. Per cohort, z-score normalise MCEMP1 and HLA-DRA expression across samples, then average the two gene Z-scores per subject to form the combined severity metric. Treat severe status as the positive class and compute ROC curves and AUC values for the combined Z-score in each cohort with scikit-learn's roc_curve/roc_auc_score. Tabulate AUC, confidence direction and discrimination outcome (significant vs no discrimination, AUC~0.5) per cohort and disease, saving the summary as roc_results.csv. The rationale is that MCEMP1 was heightened and HLA-DRA reduced as early as four days before nadir of respiratory function, so a directional combined score should generalise if the signature is pathogen-agnostic.

## Related tools

- **Streamlit** (Web app through which the curated severe-vs-mild COVID-19 transcriptomics datasets and per-gene severe/mild comparisons can be queried and downloaded) — https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
- **pip** (Installs dependencies (requirements.txt) needed to run the Streamlit app locally for dataset access)

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, _ = roc_curve(severe_labels, combined_zscore); auc = roc_auc_score(severe_labels, combined_zscore)
```

## Evaluation signals

- Reproduced pattern matches the paper: severe influenza discriminated in 2 of 3 cohorts, severe RSV in 2 of 2 studies, no discrimination for dengue (AUC~0.5)
- roc_results.csv exists and contains AUC, confidence direction and discrimination outcome columns for each of the 9 cohorts
- Per-cohort Z-scores of MCEMP1 and HLA-DRA have mean ~0 and standard deviation ~1 across samples before averaging
- ROC curves computed with severe status coded as the positive class (AUC > 0.5 indicates higher combined score in severe cases)

## Limitations

- The combined score may not generalise to all pathogens — no discrimination was observed for dengue
- Heterogeneity in the study datasets was noted; patient heterogeneity can influence MCEMP1 and HLA-DRA expression
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severity have not been prospectively validated

## Evidence

- [other] Per cohort, z-score normalise MCEMP1 and HLA-DRA expression across samples and average the two gene Z-scores per subject to form the combined Z-score severity metric for severe vs mild groups.: [evidence span withheld — anchor status: not_found]
- [other] Compute ROC curves and AUC values for the combined Z-score using scikit-learn, treating severe status as the positive class.: [evidence span withheld — anchor status: not_found]
- [abstract] The most consistent differentially regulated genes in peripheral blood of severe COVID-19 patients were MCEMP1, HLA-DRA and ETS1 across 7 transcriptomics datasets.: "The most consistent differentially regulated genes in peripheral blood of severe COVID-19 patients were MCEMP1, HLA-DRA and ETS1 across the 7 transcriptomics datasets."
- [discussion] The datasets and accession numbers are documented in Table 1 and Supplementary Table S5 of the paper.: "the accession numbers are shown in Table 1 and Supplementary Table S5"
- [readme] This database is a curation of 7 transcriptomics datasets comparing the gene expression in severe COVID-19 and mild COVID-19 subjects.: "This database is a curation of 7 transcriptomics datasets comparing the gene expression in severe COVID-19 and mild COVID-19 subjects."
- [readme] The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data between severe COVID-19 and mild COVID-19.: "The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data between severe COVID-19 and mild COVID-19"
