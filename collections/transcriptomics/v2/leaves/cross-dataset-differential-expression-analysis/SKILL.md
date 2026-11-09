---
name: cross-dataset-differential-expression-analysis
description: Use when the goal is to identify genes consistently dysregulated across
  several public transcriptomics cohorts (e.g. severe vs mild COVID-19 peripheral
  blood RNA-seq) instead of relying on a single-dataset DE result.
when_to_use_negative:
- The analysis is limited to a single dataset with no cross-cohort consistency requirement
  — a per-dataset DE workflow alone suffices.
- The data are single-cell RNA-seq PBMC/BALF matrices; the article handles those with
  a separate subsetting workflow, not this bulk-matrix severity pipeline.
- The comparison groups are severe vs healthy rather than severe vs mild, unless that
  contrast is explicitly the intended output.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: pip
  role: Installs the Python dependencies of the COVID19-severity repository needed
    to run the Streamlit app and reproduce the per-dataset DE tables
  repo: https://github.com/kuanrongchan/COVID19-severity
- name: Streamlit
  role: Serves the curated 7-dataset database so users can query per-gene severe-vs-mild
    statistics and plot box and strip plots
  repo: https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:c0767c6cfb29d3c3a1a1057f7912352fa3875395ecdb3c9b5db89f2c8916756e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Streamlit
  - pip
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-dataset-differential-expression-analysis

## Summary

A skill for re-analysing multiple independent bulk RNA-seq/transcriptomics datasets to rank genes by the consistency of their differential expression between severe and mild COVID-19, rather than in any single cohort. It grounds gene-level calls in per-dataset fold changes and t-test p-values aggregated across 7 public datasets.

## When to use

Use when the goal is to identify genes consistently dysregulated across several public transcriptomics cohorts (e.g. severe vs mild COVID-19 peripheral blood RNA-seq) instead of relying on a single-dataset DE result.

## When NOT to use

- The analysis is limited to a single dataset with no cross-cohort consistency requirement — a per-dataset DE workflow alone suffices.
- The data are single-cell RNA-seq PBMC/BALF matrices; the article handles those with a separate subsetting workflow, not this bulk-matrix severity pipeline.
- The comparison groups are severe vs healthy rather than severe vs mild, unless that contrast is explicitly the intended output.

## Inputs

- Processed per-gene expression matrices (severe and mild subject groups annotated) from 7 COVID-19 datasets: GSE155454, GSE161731, GSE157103, GSE152418, GSE172114, GSE164805, Bibert 2021 (10.17632/8wxhhykfnh.2)
- Raw count data and processed files from the counts and anova folders of github.com/kuanrongchan/COVID19-severity

## Outputs

- Concatenated per-dataset fold-change and p-value table saved as severity_fold_changes.csv
- Dataset-by-gene heatmap of significant dysregulation counts saved as severity_heatmap.png
- Ranked list of consistently dysregulated genes (MCEMP1, HLA-DRA, ETS1)

## How to apply

Retrieve the processed expression matrices from the COVID19-severity GitHub repository and Zenodo deposit (10.5281/zenodo.7114840), plus accessions GSE155454, GSE161731, GSE157103, GSE152418, GSE172114, GSE164805 and Bibert 2021 from Mendeley Data. Load each of the 7 matrices with pandas, retaining the annotated severe/mild groups (e.g. 14 severe vs 18 mild in GSE155454; 50 severe vs 50 mild in GSE157103; 5 vs 5 in GSE164805). For each dataset compute log2 fold changes (severe vs mild) and p-values per gene via scipy.stats t-tests, calling genes differentially expressed at fold change > 1.3 and p < 0.05. Rank genes by consistency of dysregulation direction across datasets, build a dataset-by-gene significant-dysregulation count matrix for candidates (S100A12, S100A9, CDKN2D, IL11RA, MARCHF9, PEA15, MCEMP1, HLA-DRA, ETS1), and visualise it as a seaborn heatmap. The rationale is that reproducibility across 140 severe and 181 mild patients in 7 independent cohorts is a stronger generalisability filter than any single-cohort statistic.

## Related tools

- **pip** (Installs the Python dependencies of the COVID19-severity repository needed to run the Streamlit app and reproduce the per-dataset DE tables) — https://github.com/kuanrongchan/COVID19-severity
- **Streamlit** (Serves the curated 7-dataset database so users can query per-gene severe-vs-mild statistics and plot box and strip plots) — https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py

## Examples

```
cd path/to/COVID19-severity && pip install -r requirements.txt && streamlit run app.py
```

## Evaluation signals

- MCEMP1 is upregulated in 6 of 7 datasets with mean fold change ≈ 3.62 (severe vs mild).
- HLA-DRA and ETS1 are the most consistently downregulated genes, mean fold changes ≈ 2.28 and ≈ 2.10 down respectively.
- Every per-dataset gene call passes the FC > 1.3 and p < 0.05 thresholds (t-test via scipy.stats).
- severity_fold_changes.csv contains per-dataset log2 fold changes and p-values for all 7 datasets, with group sizes matching the README (e.g. 140 severe, 181 mild in total).
- severity_heatmap.png reproduces the dataset-by-gene significance matrix for S100A12, S100A9, CDKN2D, IL11RA, MARCHF9, PEA15, MCEMP1, HLA-DRA and ETS1.

## Limitations

- The included datasets did not directly evaluate the effect of different SARS-CoV-2 variants on transcriptomic responses.
- Heterogeneity among the 7 study datasets was noted, so patient-level heterogeneity influencing MCEMP1 and HLA-DRA expression is not resolved.
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severity have not been prospectively validated.
- Some datasets also include healthy subjects, so contrast definitions (severe vs mild vs healthy) must be chosen explicitly per dataset.

## Evidence

- [abstract] The most consistent differentially regulated genes in peripheral blood of severe COVID-19 patients were MCEMP1, HLA-DRA and ETS1 across the 7 transcriptomics datasets.: "The most consistent differentially regulated genes in peripheral blood of severe COVID-19 patients were MCEMP1, HLA-DRA and ETS1 across the 7 transcriptomics datasets."
- [readme] README description of processed files: "The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data between severe COVID-19 and mild COVID-19"
- [readme] README scope statement: "This database is a curation of 7 transcriptomics datasets comparing the gene expression in severe COVID-19 and mild COVID-19 subjects."
- [methods] Methods dependency installation: "navigate to the directory the file is located at and install its dependencies as shown below"
