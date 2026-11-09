---
name: log-rank-test-statistics
description: Use when a cell-cluster DEG signature (e.g., Cluster 4 from GSE156625
  with top DEGs PTMA and STMN1) must be validated against TCGA bulk cohorts (e.g.,
  TCGA-LIHC) for prognostic survival separation.
when_to_use_negative:
- Input is drug response data (IC50/AUC prediction) rather than a cluster gene signature
  to be prognostically validated — use scDrug's Drug Response Prediction module instead
- No matched clinical survival data exist for the bulk cohort, or the tumor type/scRNA-seq
  profiles align poorly with available TCGA data (as with the prostate dataset where
  no cluster reached significance)
- Goal is cell-type annotation, GSEA, or batch correction rather than cluster-signature
  survival testing
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: scDrug
  role: Implements the survival analysis step via the --survival, --tcga, --id, and
    --not_treated flags of single_cell_analysis.py
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Produces the upstream clustering and DEG outputs (scanpyobj.h5ad, cluster_DEGs.csv)
    that supply the gene signature
  repo: https://github.com/ailabstw/scDrug
- name: Python
  role: Environment in which the scDrug single-cell analysis and survival pipeline
    is run
  repo: https://github.com/ailabstw/scDrug
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:1cffa963b25e0d676cf1765ece0c2b2f8893976b21b0aa00ac0a59703758710c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - Python
  - Scanpy
  - scDrug
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# log-rank-test-statistics

## Summary

Survival-analysis procedure from scDrug's Single-Cell Data Analysis module that scores a tumor-cluster gene signature in TCGA bulk RNA-seq patients, splits them into high/low-expressing quartile groups, and tests survival separation with Kaplan-Meier curves, log-rank p-value, and hazard ratio.

## When to use

Use when a cell-cluster DEG signature (e.g., Cluster 4 from GSE156625 with top DEGs PTMA and STMN1) must be validated against TCGA bulk cohorts (e.g., TCGA-LIHC) for prognostic survival separation.

## When NOT to use

- Input is drug response data (IC50/AUC prediction) rather than a cluster gene signature to be prognostically validated — use scDrug's Drug Response Prediction module instead
- No matched clinical survival data exist for the bulk cohort, or the tumor type/scRNA-seq profiles align poorly with available TCGA data (as with the prostate dataset where no cluster reached significance)
- Goal is cell-type annotation, GSEA, or batch correction rather than cluster-signature survival testing

## Inputs

- Cluster clustering output: scanpyobj.h5ad or cluster_DEGs.csv from upstream single-cell analysis
- Top N cluster-specific DEG signature (e.g., top 20 Cluster 4 DEGs including PTMA and STMN1)
- TCGA bulk RNA expression matrix (e.g., TCGA-LIHC)
- Matching TCGA clinical survival information

## Outputs

- Kaplan-Meier survival plot (e.g., cluster4_kaplan_meier.png)
- CSV of group labels, hazard ratio, and log-rank p-value (e.g., cluster4_survival_results.csv)

## How to apply

Take the top 20 differentially expressed genes of the target cluster (log2 fold change above 2, p-value and adjusted p-value below 0.01) from cluster_DEGs.csv or scanpyobj.h5ad as the signature. Download TCGA bulk RNA expression and matching clinical data (optionally restricting to untreated samples via --not_treated). For each signature gene, assign 1 if a patient's expression exceeds the gene's median across all patients, else 0, and sum to get a cluster activity score per patient. Divide patients into 'high-expressing' and 'low-expressing' groups using the top and bottom quartiles of the activity score. Fit Kaplan-Meier curves per group with a log-rank test and compute the hazard ratio; outputs are a Kaplan-Meier plot and a CSV of group labels, hazard ratio, and log-rank p-value. Significance is judged at log-rank p-value < 0.05 (e.g., the reproduced LIHC Cluster 4 result: hazard ratio ≈ 2.52, −log2(p) ≈ 14).

## Related tools

- **scDrug** (Implements the survival analysis step via the --survival, --tcga, --id, and --not_treated flags of single_cell_analysis.py) — https://github.com/ailabstw/scDrug
- **Scanpy** (Produces the upstream clustering and DEG outputs (scanpyobj.h5ad, cluster_DEGs.csv) that supply the gene signature) — https://github.com/ailabstw/scDrug
- **Python** (Environment in which the scDrug single-cell analysis and survival pipeline is run) — https://github.com/ailabstw/scDrug

## Examples

```
python3 single_cell_analysis.py -f h5ad --input scanpyobj.h5ad --clusters 4 --survival --tcga TCGA_DATA --id TCGA-LIHC --not_treated
```

## Evaluation signals

- Reproduced TCGA-LIHC Cluster 4 analysis yields hazard ratio ≈ 2.52 with −log2(p) ≈ 14 (log-rank p far below 0.05), matching Supplementary Fig. S1
- Patient grouping uses strictly the top and bottom quartiles of the summed binary (above-median-expression) activity score
- Outputs match expected artifacts: cluster4_kaplan_meier.png plot and cluster4_survival_results.csv containing group labels, hazard ratio, and log-rank p-value
- The signature is limited to the top 20 Cluster 4 DEGs meeting log2FC > 2 and p-value/adjusted p-value < 0.01
- With --not_treated, only untreated TCGA samples contribute to the survival comparison

## Limitations

- In the prostate cancer dataset, none of the tumor clusters yielded log-rank p < 0.05; the authors attribute this to poor alignment of expression and limited TCGA data
- Survival analysis in scDrug was applied only to specific tumor-cluster signatures; sign of association and quartile cutoff choice can affect reproducibility across cohorts

## Evidence

- [methods] Survival analysis groups patients by cluster activity score quartiles: [evidence span withheld — anchor status: not_found]
- [methods] DEG signature selection thresholds: "DEGs with log2 fold change above 2 and p-value and adjusted p-value both below 0.01"
- [readme] README documents survival analysis in the single-cell pipeline: "**Single-Cell Data Analysis** performed data preprocessing, clustering, cell type annotation, Gene Set Enrichment Analysis (GSEA), and survival analysis."
- [readme] README flags for restricting to untreated TCGA samples: "--not_treated         only consider untreated samples from TCGA for survival analysis."
- [results] Negative result showing p < 0.05 is not guaranteed: "none of tumor clusters in this dataset yielded a statistical significance at p-value <0.05 for the survival analysis."
