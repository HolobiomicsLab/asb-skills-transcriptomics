---
name: seurat-findmarkers-workflow
description: Use when you have normalized scRNA-seq expression matrices with malignant/non-malignant
  cell annotations from multiple cancer datasets (e.g., RCC, HNSCC, melanoma, LUAD,
  BC) and need a consensus marker gene feature set for cell-type classification.
when_to_use_negative:
- You already have a preprocessed, gene-matched, scaled feature matrix ready for classification
  — use PreCanCell_data() and PreCanCell_classifier() instead of re-deriving DEGs
- Your goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial)
  — this workflow only distinguishes cancer vs non-cancer cells
- You have only a single dataset — the TMG/NMG sets require intersection across all
  five training datasets and may not generalize from one cancer type
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: R package providing NormalizeData() and FindMarkers() used to normalize expression
    and perform the Wilcoxon rank-sum differential expression test between malignant
    and non-malignant cells
- name: PreCanCell
  role: Downstream consumer of the TMG/NMG DEG feature sets; PreCanCell_data() selects
    matched genes and PreCanCell_classifier() classifies cells by k-NN (k=5) with
    majority vote
  repo: https://github.com/WangX-Lab/PreCanCell
- name: R
  role: Implementation language; DEG analysis performed with FindMarkers() in the
    R package Seurat
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:7da277a04b7ea3125eb013a06313d5a6b2b3e5debbada0fcc0fddaab76fa992c
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# seurat-findmarkers-workflow

## Summary

A differential expression workflow using Seurat's FindMarkers() (Wilcoxon rank-sum test with Bonferroni correction) to identify genes consistently upregulated in malignant versus non-malignant cells across five cancer scRNA-seq training datasets, yielding tumor marker gene (TMG) and non-malignant marker gene (NMG) feature sets (73 TMGs, 186 NMGs) for the PreCanCell classifier.

## When to use

Use when you have normalized scRNA-seq expression matrices with malignant/non-malignant cell annotations from multiple cancer datasets (e.g., RCC, HNSCC, melanoma, LUAD, BC) and need a consensus marker gene feature set for cell-type classification.

## When NOT to use

- You already have a preprocessed, gene-matched, scaled feature matrix ready for classification — use PreCanCell_data() and PreCanCell_classifier() instead of re-deriving DEGs
- Your goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial) — this workflow only distinguishes cancer vs non-cancer cells
- You have only a single dataset — the TMG/NMG sets require intersection across all five training datasets and may not generalize from one cancer type

## Inputs

- Normalized gene expression matrices (e.g., library-depth normalized TPM/CPM) with rownames as genes and colnames as cells for each training dataset (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC)
- Cell-level malignant/non-malignant annotations for each dataset

## Outputs

- TMG gene set: 73 genes upregulated in malignant cells, common across all five training datasets
- NMG gene set: 186 genes upregulated in non-malignant cells, common across all five training datasets
- deg_gene_sets.tsv with gene identifiers and set membership labels

## How to apply

For each of the five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC), run Seurat's FindMarkers() comparing malignant versus non-malignant cells with the Wilcoxon rank-sum test, retaining only genes with Bonferroni-adjusted P-value < 0.05. From the significant genes, call a gene upregulated in malignant cells if it is expressed in at least 10% of malignant cells with mean log-scale fold change > 0.25 (and analogously for non-malignant cells). Then compute the intersection of malignant-upregulated genes across all five datasets to define the TMG set, and the intersection of non-malignant-upregulated genes to define the NMG set. The rationale is that genes recurrently differentially expressed across diverse cancer types form robust pan-cancer features for k-NN (k=5) classification of malignant versus non-malignant cells. Write the resulting gene lists to a file such as deg_gene_sets.tsv with gene identifiers and set membership labels.

## Related tools

- **Seurat** (R package providing NormalizeData() and FindMarkers() used to normalize expression and perform the Wilcoxon rank-sum differential expression test between malignant and non-malignant cells)
- **PreCanCell** (Downstream consumer of the TMG/NMG DEG feature sets; PreCanCell_data() selects matched genes and PreCanCell_classifier() classifies cells by k-NN (k=5) with majority vote) — https://github.com/WangX-Lab/PreCanCell
- **R** (Implementation language; DEG analysis performed with FindMarkers() in the R package Seurat)

## Examples

```
markers <- FindMarkers(seurat_obj, ident.1 = "malignant", ident.2 = "non_malignant", test.use = "wilcox"); sig <- subset(markers, p_val_adj < 0.05)
```

## Evaluation signals

- The resulting TMG set must contain exactly 73 genes and the NMG set exactly 186 genes
- Every gene in each set must satisfy the 10% cell-detection and 0.25 mean log fold-change criteria in each of the five training datasets
- Every retained gene must have a Bonferroni-adjusted Wilcoxon rank-sum test P-value < 0.05 in each dataset
- TMG genes should show significantly higher expression in TCGA pan-cancer than normal controls (P = 2.22 × 10−39), with FDR < 0.05 in 25 of 30 individual cancer types
- deg_gene_sets.tsv must contain gene identifiers with valid set membership labels (TMG or NMG) and no overlap-handling ambiguity

## Limitations

- Training-set predictive performance may be overestimated because the DEG feature sets were derived from the same data used for training
- The workflow is limited to annotating cancer vs non-cancer cells; it does not identify non-cancer cell subpopulations (immune, stromal, epithelial)
- No accession IDs or repository links are provided for the named train/test datasets (P1_BC, T5_BC, P4_SyS, P5_SyS, P7_GBM, P8_PDAC, P10_GBM)

## Evidence

- [methods] We performed the differential gene expression analysis with the function “FindMarkers()” in the R package “Seurat”: "We performed the differential gene expression analysis with the function “FindMarkers()” in the R package “Seurat”"
- [methods] Bonferroni-corrected Wilcoxon P-value threshold: "the gene expression difference between malignant and non-malignant cells was statistically significant if the adjusted Wilcoxon rank-sum test P-value < 0.05 by the Bonferroni correction"
- [methods] Intersection of upregulated genes defines TMGs: "we obtained the gene set, which was the intersection of the upregulated genes in malignant cells identified in each of the five training datasets; this gene set was defined as tumor marker genes"
- [results] DEG composition result: "The DEGs included 73 TMGs and 186 NMGs."
- [methods] Normalization step in Seurat: "normalized gene expression values using the “NormalizeData()” function in the R package “Seurat”"
