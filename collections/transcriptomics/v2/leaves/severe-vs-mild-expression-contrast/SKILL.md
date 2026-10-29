---
name: severe-vs-mild-expression-contrast
description: Use when re-analysing processed expression matrices from COVID-19 severity
  cohorts (e.g. GSE155454, GSE157103) to find genes consistently differentially expressed
  between severe and mild patients.
when_to_use_negative:
- Input is raw sequencing data or single-cell count matrices — this skill works on
  already-processed bulk expression matrices with severe/mild annotations
- Contrast is against healthy controls rather than severe-vs-mild; the README notes
  processed files also contain severe-vs-healthy comparisons, which answer a different
  question
- Goal is prognostic timing/kinetics (e.g. expression before nadir of respiratory
  function) rather than cross-cohort consistency of dysregulation
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: pip
  role: Installs the dependencies required to run the COVID19-severity analysis app
    and scripts (pip install -r requirements.txt)
  repo: https://github.com/kuanrongchan/COVID19-severity
- name: Streamlit
  role: Webtool for querying the per-dataset severe-vs-mild differential expression
    results (box and strip plots of a selected gene)
  repo: https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:6c61915a0250808a68b44b7520e73d04d0a31f9f377873fc9432d808fd3aa304
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Streamlit
  - pip
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# severe-vs-mild-expression-contrast

## Summary

Per-dataset differential expression of severe versus mild COVID-19 blood transcriptomics samples, followed by ranking of genes by consistency of dysregulation direction across datasets. This is the core analysis that identified MCEMP1, HLA-DRA and ETS1 as the most consistently dysregulated genes across 7 RNAseq datasets.

## When to use

Use when re-analysing processed expression matrices from COVID-19 severity cohorts (e.g. GSE155454, GSE157103) to find genes consistently differentially expressed between severe and mild patients.

## When NOT to use

- Input is raw sequencing data or single-cell count matrices — this skill works on already-processed bulk expression matrices with severe/mild annotations
- Contrast is against healthy controls rather than severe-vs-mild; the README notes processed files also contain severe-vs-healthy comparisons, which answer a different question
- Goal is prognostic timing/kinetics (e.g. expression before nadir of respiratory function) rather than cross-cohort consistency of dysregulation

## Inputs

- Processed expression matrices from 7 public datasets (GitHub kuanrongchan/COVID19-severity, Zenodo 10.5281/zenodo.7114840, GEO accessions GSE155454/GSE161731/GSE157103/GSE152418/GSE172114/GSE164805, Mendeley Bibert 2021 10.17632/8wxhhykfnh.2)
- Severe/mild group annotations per dataset (e.g. 50 severe vs 50 mild in GSE157103)

## Outputs

- severity_fold_changes.csv (concatenated per-dataset fold change and p-value table)
- severity_heatmap.png (dataset-by-gene heatmap of significant dysregulation counts)
- Ranked list of consistently dysregulated genes (MCEMP1 up ~3.62-fold; HLA-DRA down ~2.28-fold; ETS1 down ~2.10-fold)

## How to apply

Load each of the 7 processed expression matrices with pandas, retaining the severe and mild subject groups as annotated per study (e.g. 14 severe vs 18 mild in GSE155454; 15 severe vs 63 mild in Bibert 2021; 5 severe vs 5 mild in GSE164805). For every dataset compute log2 fold changes (severe vs mild) and per-gene p-values using scipy.stats t-tests with a pandas groupby comparison, filtering to differentially expressed genes at fold change > 1.3 and p < 0.05. Rank genes by consistency of dysregulation direction across the 7 datasets and build a dataset-by-gene matrix counting significant dysregulation for genes such as S100A12, S100A9, CDKN2D, IL11RA, MARCHF9, PEA15, MCEMP1, HLA-DRA and ETS1, plotting it as a seaborn heatmap. Concatenate per-dataset results and save them for downstream querying.

## Related tools

- **pip** (Installs the dependencies required to run the COVID19-severity analysis app and scripts (pip install -r requirements.txt)) — https://github.com/kuanrongchan/COVID19-severity
- **Streamlit** (Webtool for querying the per-dataset severe-vs-mild differential expression results (box and strip plots of a selected gene)) — https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py

## Examples

```
import pandas as pd; from scipy.stats import ttest_ind; df = pd.read_csv('GSE155454_counts.csv', index_col=0); fc = df.groupby('severity').mean().T; stat, p = ttest_ind(severe, mild)
```

## Evaluation signals

- MCEMP1 emerges as the most consistently upregulated gene (6 of 7 datasets) with mean fold change ≈ 3.62
- HLA-DRA (≈ 2.28-fold) and ETS1 (≈ 2.10-fold) emerge as the most consistently downregulated genes in severe patients
- Every per-dataset DE call passes the FC > 1.3 and p < 0.05 thresholds (t-test)
- severity_fold_changes.csv contains fold-change and p-value rows for all 7 datasets; severity_heatmap.png shows a dataset-by-gene count matrix including S100A12, S100A9, MCEMP1, HLA-DRA, ETS1
- Sample group sizes match published annotations (e.g. 14 severe vs 18 mild in GSE155454; 46 severe vs 23 mild in GSE172114; 5 severe vs 5 mild in GSE164805)

## Limitations

- Datasets did not directly evaluate the effect of different SARS-CoV-2 variants on transcriptomic responses to severe COVID-19
- Heterogeneity in the study datasets was noted; future studies are needed to understand heterogeneity in patients influencing MCEMP1 and HLA-DRA expression
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severe COVID-19 have not been validated; prospective studies are needed

## Evidence

- [methods] Thresholds: [evidence span withheld — anchor status: not_found]
- [other] Consistency ranking: [evidence span withheld — anchor status: not_found]
- [abstract] Systematic review scope: "We performed a systematic review and re-analysis of 7 publicly available datasets, analysing a total of 140 severe and 181 mild COVID-19 patients, to determine the most consistent differentially"
- [abstract] Top genes: "The most consistent differentially regulated genes in peripheral blood of severe COVID-19 patients were MCEMP1, HLA-DRA and ETS1 across the 7 transcriptomics datasets."
- [readme] Data provenance: "This database is a curation of 7 transcriptomics datasets comparing the gene expression in severe COVID-19 and mild COVID-19 subjects. Each database contains raw count data and a processed file"
- [readme] App usage: "Box and strip plots of the selected gene of interest in the dataset will be plotted. Mouse over the various plots for more in-depth statistics of each group."
