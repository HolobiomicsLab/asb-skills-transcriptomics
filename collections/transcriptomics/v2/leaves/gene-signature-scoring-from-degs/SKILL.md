---
name: gene-signature-scoring-from-degs
description: Use when a single-cell cluster's DEGs (e.g., scanpyobj.h5ad / cluster_DEGs.csv)
  must be tested against bulk RNA-seq cohorts (e.g., TCGA-LIHC) for prognostic significance
  via signature scoring and survival analysis.
when_to_use_negative:
- Input is already a bulk expression feature table with no cluster-derived DEG signature
  from scRNA-seq — the scoring relies on cluster-specific DEGs like Cluster 4's PTMA/STMN1
- No matched clinical survival data exist for the bulk cohort — the log-rank test
  and hazard ratio cannot be computed
- The goal is drug response prediction (IC50/AUC via GDSC or PRISM models) rather
  than survival stratification — use scDrug's Drug Response Prediction module instead
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDrug
  role: Provides single_cell_analysis.py with --survival, --tcga, --id, and --not_treated
    flags that automate signature-based patient grouping and Kaplan-Meier/log-rank
    analysis against TCGA data
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Performs the upstream scRNA-seq preprocessing and clustering that produces
    the Anndata object and cluster DEGs used as the gene signature
  repo: https://github.com/ailabstw/scDrug
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:c08c671d20c157fe5abb18802e2ac3016b4d09d553a623b9010a63b8e23b02cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - Python
  - Scanpy
  - scDrug
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gene-signature-scoring-from-degs

## Summary

Converts cluster-specific differentially expressed genes (e.g., the top 20 DEGs of a tumor Cluster 4 from GSE156625 scRNA-seq) into a binary activity score per bulk RNA-seq patient, then stratifies patients into high/low-expressing groups for Kaplan-Meier survival analysis linking scRNA-seq subpopulations to clinical outcome.

## When to use

Use when a single-cell cluster's DEGs (e.g., scanpyobj.h5ad / cluster_DEGs.csv) must be tested against bulk RNA-seq cohorts (e.g., TCGA-LIHC) for prognostic significance via signature scoring and survival analysis.

## When NOT to use

- Input is already a bulk expression feature table with no cluster-derived DEG signature from scRNA-seq — the scoring relies on cluster-specific DEGs like Cluster 4's PTMA/STMN1
- No matched clinical survival data exist for the bulk cohort — the log-rank test and hazard ratio cannot be computed
- The goal is drug response prediction (IC50/AUC via GDSC or PRISM models) rather than survival stratification — use scDrug's Drug Response Prediction module instead

## Inputs

- Clustering output from upstream single-cell analysis: scanpyobj.h5ad and cluster_DEGs.csv (cluster-specific DEGs, e.g., top 20 of Cluster 4 with PTMA, STMN1)
- TCGA bulk RNA expression profiles and matching clinical information (e.g., TCGA-LIHC, optionally restricted to untreated samples)

## Outputs

- Kaplan-Meier survival plot (e.g., cluster4_kaplan_meier.png)
- Group labels, hazard ratio, and log-rank p-value table (e.g., cluster4_survival_results.csv)

## How to apply

First select the cluster-specific gene signature from the upstream single-cell clustering output (cluster_DEGs.csv), e.g., the top 20 DEGs of Cluster 4 including PTMA and STMN1, typically DEGs with log2 fold change above 2 and p-value and adjusted p-value both below 0.01. Next, download TCGA bulk RNA expression and matching clinical data (optionally restricting to untreated samples with --not_treated). For each signature gene, assign a binary 1 if a patient's expression exceeds the gene's median expression across all patients, else 0, and sum to obtain the cluster activity score per patient. Divide patients into 'high-expressing' and 'low-expressing' groups using the top and bottom quartiles of the activity score. Fit Kaplan-Meier curves per group, run a log-rank test, and compute the hazard ratio; the scDrug single_cell_analysis.py --survival flag automates this with --tcga and --id TCGA-LIHC.

## Related tools

- **scDrug** (Provides single_cell_analysis.py with --survival, --tcga, --id, and --not_treated flags that automate signature-based patient grouping and Kaplan-Meier/log-rank analysis against TCGA data) — https://github.com/ailabstw/scDrug
- **Scanpy** (Performs the upstream scRNA-seq preprocessing and clustering that produces the Anndata object and cluster DEGs used as the gene signature) — https://github.com/ailabstw/scDrug

## Examples

```
python3 single_cell_analysis.py -f h5ad --input scanpyobj.h5ad --clusters 4 --survival --tcga /path/to/tcga --id TCGA-LIHC --not_treated
```

## Evaluation signals

- Reproduced analysis shows a hazard ratio of approximately 2.52 with -log2(p) ≈ 14 (log-rank p-value far below 0.05), matching the paper's Supplementary Fig. S1
- High-expressing and low-expressing groups each contain patients in the top and bottom quartiles of the summed binary activity score
- Binary gene indicator is exactly 1 when patient expression exceeds the gene's median across all patients, else 0 — spot-check individual gene columns
- cluster4_kaplan_meier.png and cluster4_survival_results.csv are produced with the group labels, hazard ratio, and log-rank p-value populated
- Signature genes match the cluster_DEGs.csv top DEGs (e.g., PTMA and STMN1 among the top 20 of Cluster 4)

## Limitations

- In the authors' prostate cancer dataset, none of the tumor clusters yielded survival significance at p < 0.05, attributed to poor alignment of scRNA-seq expression profiles and limited TCGA data
- Medians and quartile cut points are cohort-dependent, so the activity score grouping is sensitive to the TCGA sample set used
- Survival analysis in scDrug depends on the availability and quality of TCGA clinical annotations matching the project id supplied (e.g., TCGA-LIHC)

## Evidence

- [methods] patients were divided into "high-expressing" and "low-expressing" groups if their activity score for that cluster was in the top or bottom quartile: [evidence span withheld — anchor status: not_found]
- [intro] Single-Cell Data Analysis performed data preprocessing, clustering, cell type annotation, Gene Set Enrichment Analysis (GSEA), and survival analysis.: "**Single-Cell Data Analysis** performed data preprocessing, clustering, cell type annotation, Gene Set Enrichment Analysis (GSEA), and survival analysis."
- [readme] --not_treated only consider untreated samples from TCGA for survival analysis.: "--not_treated         only consider untreated samples from TCGA for survival analysis."
- [results] none of tumor clusters in this dataset yielded a statistical significance at p-value <0.05 for the survival analysis.: "none of tumor clusters in this dataset yielded a statistical significance at p-value <0.05 for the survival analysis."
- [methods] DEGs with log2 fold change above 2 and p-value and adjusted p-value both below 0.01: "DEGs with log2 fold change above 2 and p-value and adjusted p-value both below 0.01"
