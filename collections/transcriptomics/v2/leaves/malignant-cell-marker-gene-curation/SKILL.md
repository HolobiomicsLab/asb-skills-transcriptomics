---
name: malignant-cell-marker-gene-curation
description: Use when building malignant vs non-malignant cell classification features
  from annotated cancer scRNA-seq datasets, or when a pan-cancer marker gene set common
  across multiple tumor types is needed.
when_to_use_negative:
- The goal is to identify subpopulations of non-cancer cells (immune, stromal, epithelial)
  — PreCanCell and its DEG features are limited to annotating cancer vs non-cancer
  cells
- Only one training dataset is available, since the TMG/NMG sets require intersection
  across all five cancer-type datasets
- Input expression data is raw counts rather than library-depth-normalized values
  (e.g. TPM, CPM), which PreCanCell_data() requires
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Seurat
  role: Used to normalize expression (NormalizeData()) and perform differential expression
    analysis with FindMarkers() comparing malignant vs non-malignant cells
- name: PreCanCell
  role: 'Downstream consumer: the curated TMG/NMG sets serve as features for k-NN
    (k=5) malignant/non-malignant classification'
  repo: https://github.com/WangX-Lab/PreCanCell
- name: R
  role: Implementation language for the DEG intersection workflow and the PreCanCell
    R package
  repo: https://github.com/WangX-Lab/PreCanCell
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:7a5498da23f766d83dae26ed44ba65275b6d92c2112574d11b20e5db97f30657
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
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# malignant-cell-marker-gene-curation

## Summary

Curates tumor marker gene (TMG) and non-malignant marker gene (NMG) feature sets by intersecting differentially expressed genes (DEGs) between malignant and non-malignant cells across five cancer scRNA-seq training datasets (RCC, HNSCC, melanoma, LUAD, BC), yielding 73 TMGs and 186 NMGs for use as k-NN classification features in PreCanCell.

## When to use

Use when building malignant vs non-malignant cell classification features from annotated cancer scRNA-seq datasets, or when a pan-cancer marker gene set common across multiple tumor types is needed.

## When NOT to use

- The goal is to identify subpopulations of non-cancer cells (immune, stromal, epithelial) — PreCanCell and its DEG features are limited to annotating cancer vs non-cancer cells
- Only one training dataset is available, since the TMG/NMG sets require intersection across all five cancer-type datasets
- Input expression data is raw counts rather than library-depth-normalized values (e.g. TPM, CPM), which PreCanCell_data() requires

## Inputs

- Normalized expression matrices (rownames = genes, colnames = cells) for five training scRNA-seq datasets: RCC, HNSCC, melanoma, LUAD, BC
- Per-cell malignant/non-malignant annotations for each training dataset

## Outputs

- TMG gene list: 73 genes upregulated in malignant cells, common to all five training datasets
- NMG gene list: 186 genes upregulated in non-malignant cells, common to all five training datasets
- deg_gene_sets.tsv with gene identifiers and set membership labels (TMG/NMG)

## How to apply

Load normalized expression matrices and malignant/non-malignant cell annotations for each training dataset (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC). For each dataset, run Seurat's FindMarkers() comparing malignant versus non-malignant cells with the Wilcoxon rank-sum test, keeping only genes with Bonferroni-adjusted P-value < 0.05. Classify genes as upregulated in malignant cells if detected in at least 10% of malignant cells with mean log-scale fold change > 0.25 (and analogously for non-malignant cells). Intersect the malignant-upregulated genes across all five datasets to define the TMG set, and intersect the non-malignant-upregulated genes across all five datasets to define the NMG set. The rationale is that only genes recurring as DEGs in every cancer type are robust, generalizable markers, avoiding tumor-type-specific artifacts.

## Related tools

- **Seurat** (Used to normalize expression (NormalizeData()) and perform differential expression analysis with FindMarkers() comparing malignant vs non-malignant cells)
- **PreCanCell** (Downstream consumer: the curated TMG/NMG sets serve as features for k-NN (k=5) malignant/non-malignant classification) — https://github.com/WangX-Lab/PreCanCell
- **R** (Implementation language for the DEG intersection workflow and the PreCanCell R package) — https://github.com/WangX-Lab/PreCanCell

## Examples

```
markers <- FindMarkers(seurat_obj, ident.1 = "malignant", ident.2 = "non-malignant", test.use = "wilcox"); tmg <- Reduce(intersect, lapply(marker_lists, function(m) subset(m, avg_log2FC > 0.25 & pct.1 >= 0.10 & p_val_adj < 0.05)$gene))
```

## Evaluation signals

- The resulting DEG sets match the reported composition of 73 TMGs and 186 NMGs
- Every gene in each set satisfies the 10% detection and 0.25 log fold-change criteria in each of the five training datasets
- Every gene in each set has a Bonferroni-adjusted Wilcoxon rank-sum test P-value < 0.05 in all five training datasets
- The TMG set is exactly the intersection of malignant-upregulated genes across T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, and T5_BC (no dataset omitted)
- TMGs show significantly higher expression in TCGA pan-cancer than normal controls (P = 2.22e-39), as an external sanity check

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- The DEG feature set is limited to annotating cancer vs non-cancer cells; it does not resolve non-cancer subpopulations
- DEGs common to the five training cancer types may miss markers relevant to tumor types not represented among RCC, HNSCC, melanoma, LUAD, and BC

## Evidence

- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [methods] Bonferroni Wilcoxon threshold: "the gene expression difference between malignant and non-malignant cells was statistically significant if the adjusted Wilcoxon rank-sum test P-value < 0.05 by the Bonferroni correction"
- [methods] Intersection defining TMGs: "we obtained the gene set, which was the intersection of the upregulated genes in malignant cells identified in each of the five training datasets; this gene set was defined as tumor marker genes"
- [methods] FindMarkers usage: "We performed the differential gene expression analysis with the function “FindMarkers()” in the R package “Seurat”"
- [intro] DEGs as k-NN features: "With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5)"
