---
name: tcga-clinical-data-integration
description: Use when a tumor subpopulation gene signature (e.g., top DEGs from a
  Scanpy cluster like Cluster 4 with PTMA and STMN1) must be validated against TCGA
  patient survival data to assess prognostic significance.
when_to_use_negative:
- The scRNA-seq cluster profiles align poorly with the available TCGA bulk data (e.g.,
  limited or treated patient cohorts), as the paper reports no tumor cluster reached
  p < 0.05 in such a case
- The question is drug response prediction (IC50/AUC from GDSC or PRISM) rather than
  prognostic survival validation of a cluster signature
tools:
- name: scDrug
  role: Provides single_cell_analysis.py with --survival, --tcga, --id, and --not_treated
    flags to run the TCGA survival analysis against cluster DEG signatures
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Produces the clustered Anndata object (scanpyobj.h5ad) and cluster DEGs (cluster_DEGs.csv)
    that supply the gene signature input
- name: Python
  role: Environment in which the scDrug survival analysis pipeline (median binarization,
    activity scoring, Kaplan-Meier fitting) is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:b5383c79d0367f587d002ed5e205f01c3820dab509d15e2de19bc24243a2d758
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - Python
  - Scanpy
  - scDrug
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tcga-clinical-data-integration

## Summary

Integrates TCGA bulk RNA expression profiles and matching clinical survival data with a cluster-derived gene signature from single-cell analysis to test whether the signature stratifies patients into high- and low-survival groups via Kaplan-Meier analysis and hazard ratio estimation.

## When to use

Use when a tumor subpopulation gene signature (e.g., top DEGs from a Scanpy cluster like Cluster 4 with PTMA and STMN1) must be validated against TCGA patient survival data to assess prognostic significance.

## When NOT to use

- The scRNA-seq cluster profiles align poorly with the available TCGA bulk data (e.g., limited or treated patient cohorts), as the paper reports no tumor cluster reached p < 0.05 in such a case
- The question is drug response prediction (IC50/AUC from GDSC or PRISM) rather than prognostic survival validation of a cluster signature

## Inputs

- Clustering outputs from upstream single-cell analysis: scanpyobj.h5ad and cluster_DEGs.csv (e.g., Cluster 4 top DEGs including PTMA and STMN1)
- TCGA-LIHC bulk RNA expression profiles downloaded from the TCGA database
- Matching TCGA clinical/survival information (optionally filtered to untreated samples)

## Outputs

- Kaplan-Meier plot saved as cluster4_kaplan_meier.png
- Group labels, hazard ratio, and log-rank p-value saved as cluster4_survival_results.csv

## How to apply

Load the upstream clustering outputs (scanpyobj.h5ad and cluster_DEGs.csv) and select the top differentially expressed genes of the target cluster (e.g., top 20 DEGs of Cluster 4, including PTMA and STMN1) as the signature. Download TCGA-LIHC bulk RNA expression and matching clinical information (optionally restricted to untreated samples using the --not_treated flag). Binarize each signature gene per patient to 1 if expression exceeds that gene's median across all patients, else 0, and sum to obtain a per-patient activity score; assign 'high-expressing' and 'low-expressing' labels using the top and bottom quartiles of the activity score. Fit Kaplan-Meier curves for the two groups, run the log-rank test, and compute the hazard ratio. scDrug's single_cell_analysis.py exposes this via --survival, --tcga, --id (e.g., 'TCGA-LIHC'), and --not_treated.

## Related tools

- **scDrug** (Provides single_cell_analysis.py with --survival, --tcga, --id, and --not_treated flags to run the TCGA survival analysis against cluster DEG signatures) — https://github.com/ailabstw/scDrug
- **Scanpy** (Produces the clustered Anndata object (scanpyobj.h5ad) and cluster DEGs (cluster_DEGs.csv) that supply the gene signature input)
- **Python** (Environment in which the scDrug survival analysis pipeline (median binarization, activity scoring, Kaplan-Meier fitting) is executed)

## Examples

```
python3 single_cell_analysis.py -f h5ad --input scanpyobj.h5ad --survival --tcga TCGA_LIHC_expression.csv --id TCGA-LIHC --not_treated
```

## Evaluation signals

- Reproduced hazard ratio approximates 2.52 for the Cluster 4 signature in TCGA-LIHC
- -log2(p-value) ≈ 14 (log-rank p-value far below 0.05), matching the paper's Supplementary Fig. S1 survival separation
- cluster4_survival_results.csv contains group labels, hazard ratio, and log-rank p-value with the expected schema
- Activity score per patient equals the count of signature genes whose expression exceeds that gene's median across all patients
- Grouping uses only the top and bottom quartiles of the activity score, leaving intermediate patients ungrouped

## Limitations

- In the paper's prostate cancer dataset, no tumor cluster reached survival significance at p < 0.05, attributed to poor alignment of expression profiles between scRNA-seq and limited TCGA data
- Restricting to untreated TCGA samples (--not_treated) reduces the cohort size and can weaken statistical power

## Evidence

- [methods] quartile grouping for survival stratification: [evidence span withheld — anchor status: not_found]
- [readme] survival analysis within Single-Cell Data Analysis module: "**Single-Cell Data Analysis** performed data preprocessing, clustering, cell type annotation, Gene Set Enrichment Analysis (GSEA), and survival analysis."
- [readme] TCGA project id specification for survival analysis: "--id ID               Specify TCGA project id in the format "TCGA-xxxx", e.g., "TCGA-LIHC""
- [readme] untreated sample filtering for TCGA survival analysis: "--not_treated         only consider untreated samples from TCGA for survival analysis."
- [methods] TCGA as bulk transcriptomic data source: "downloaded from the TCGA database [29]"
