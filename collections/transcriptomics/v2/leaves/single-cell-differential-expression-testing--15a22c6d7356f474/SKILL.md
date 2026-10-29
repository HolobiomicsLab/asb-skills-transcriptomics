---
name: single-cell-differential-expression-testing
description: Use when annotated single-cell RNA-seq data with malignant/non-malignant
  labels is available and a robust, dataset-independent gene feature set (e.g., TMGs/NMGs)
  is needed for downstream cell classification.
when_to_use_negative:
- The input is already a reduced feature table or gene set — DE testing would be redundant
- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial),
  which PreCanCell's DE-derived features do not support
- Only one dataset is available and cross-dataset intersection (the source of TMG/NMG
  robustness) is impossible
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Provides NormalizeData() for expression normalization and FindMarkers() for
    Wilcoxon rank-sum differential expression testing with Bonferroni correction
- name: PreCanCell
  role: R package whose preprocessing (PreCanCell_data()) expects the DEG-derived
    TMG/NMG features for downstream malignant/non-malignant classification
  repo: https://github.com/WangX-Lab/PreCanCell
- name: R
  role: Language in which the differential expression analysis and gene-set intersection
    are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:95f2f21483491d446a77e1fb35b998321ff167318c82b65e05a3322904a7fa97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - PreCanCell
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-differential-expression-testing

## Summary

Identifies differentially expressed genes between malignant and non-malignant cells in single-cell transcriptome datasets using Seurat's FindMarkers() with a Wilcoxon rank-sum test, then intersects upregulated genes across multiple cancer datasets to build tumor marker gene (TMG) and non-malignant marker gene (NMG) feature sets.

## When to use

Use when annotated single-cell RNA-seq data with malignant/non-malignant labels is available and a robust, dataset-independent gene feature set (e.g., TMGs/NMGs) is needed for downstream cell classification.

## When NOT to use

- The input is already a reduced feature table or gene set — DE testing would be redundant
- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial), which PreCanCell's DE-derived features do not support
- Only one dataset is available and cross-dataset intersection (the source of TMG/NMG robustness) is impossible

## Inputs

- Normalized single-cell expression matrices (genes x cells, library-depth normalized e.g. TPM/CPM) for five training cancer scRNA-seq datasets (RCC, HNSCC, melanoma, LUAD, BC)
- Cell-level annotations labeling malignant vs non-malignant cells for each dataset
- Seurat R package (version >= 4.3.0) environment

## Outputs

- 73 tumor marker genes (TMGs) upregulated in malignant cells
- 186 non-malignant marker genes (NMGs) upregulated in non-malignant cells
- deg_gene_sets.tsv with gene identifiers and set membership labels

## How to apply

Load normalized expression matrices and malignant/non-malignant cell annotations for each dataset (e.g., T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC). For each dataset, run Seurat's FindMarkers() comparing malignant versus non-malignant cells with the Wilcoxon rank-sum test, retaining only genes with Bonferroni-adjusted P-value < 0.05. A gene is upregulated in malignant cells if expressed in at least 10% of malignant cells with mean log-scale fold change > 0.25 (analogously for non-malignant cells). Compute the intersection of malignant-upregulated genes across all five training datasets to obtain the 73-gene TMG set, and the intersection of non-malignant-upregulated genes to obtain the 186-gene NMG set; write both lists to deg_gene_sets.tsv. Rationale: requiring concordant DE across five cancer types yields features generalizable across cancers rather than tumor-type-specific.

## Related tools

- **Seurat** (Provides NormalizeData() for expression normalization and FindMarkers() for Wilcoxon rank-sum differential expression testing with Bonferroni correction)
- **PreCanCell** (R package whose preprocessing (PreCanCell_data()) expects the DEG-derived TMG/NMG features for downstream malignant/non-malignant classification) — https://github.com/WangX-Lab/PreCanCell
- **R** (Language in which the differential expression analysis and gene-set intersection are implemented)

## Examples

```
markers <- Seurat::FindMarkers(obj, ident.1 = "malignant", ident.2 = "non_malignant", test.use = "wilcox"); sig <- subset(markers, p_val_adj < 0.05)
```

## Evaluation signals

- The resulting sets must match the reported composition of 73 TMGs and 186 NMGs
- Every gene in each set must satisfy the 10% detection and 0.25 log fold-change criteria in each of the five training datasets
- All retained genes must have Bonferroni-adjusted Wilcoxon rank-sum test P-value < 0.05 in every dataset
- Each set must equal the strict intersection of upregulated genes across all five training datasets (no gene present in only a subset)
- deg_gene_sets.tsv must contain gene identifiers plus set membership labels parseable by PreCanCell_data()

## Limitations

- Because the predicted sets were involved in feature selection, downstream training-set predictive performance may be overestimated
- Requiring intersection across five specific cancer datasets limits applicability to cancer types represented in the training data
- The DEG feature set distinguishes only cancer vs non-cancer cells; it does not resolve non-cancer subpopulations

## Evidence

- [methods] the gene expression difference between malignant and non-malignant cells was statistically significant if the adjusted Wilcoxon rank-sum test P-value < 0.05 by the Bonferroni correction: "the gene expression difference between malignant and non-malignant cells was statistically significant if the adjusted Wilcoxon rank-sum test P-value < 0.05 by the Bonferroni correction"
- [methods] we obtained the gene set, which was the intersection of the upregulated genes in malignant cells identified in each of the five training datasets: "we obtained the gene set, which was the intersection of the upregulated genes in malignant cells identified in each of the five training datasets"
- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [methods] We performed the differential gene expression analysis with the function “FindMarkers()” in the R package “Seurat”: "We performed the differential gene expression analysis with the function “FindMarkers()” in the R package “Seurat”"
- [methods] normalized gene expression values using the “NormalizeData()” function in the R package “Seurat”: "normalized gene expression values using the “NormalizeData()” function in the R package “Seurat”"
