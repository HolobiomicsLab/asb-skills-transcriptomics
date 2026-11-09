---
name: cross-dataset-gene-set-intersection
description: Use when building robust cancer-cell classification features by intersecting
  malignant-vs-non-malignant DEGs across multiple training scRNA-seq datasets (e.g.,
  RCC, HNSCC, melanoma, LUAD, BC) rather than relying on a single dataset.
when_to_use_negative:
- You need to classify individual cells as malignant/non-malignant rather than derive
  the feature genes — use PreCanCell_classifier() on a PreCanCell_data()-preprocessed
  test matrix instead
- You only have a single scRNA-seq dataset; the skill's defining step is the intersection
  across five training datasets, and a one-dataset DEG call does not reproduce the
  TMG/NMG sets
- You need annotation of non-cancer cell subpopulations (immune, stromal, epithelial)
  — the resulting marker framework only distinguishes cancer vs. non-cancer cells
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: Seurat
  role: Provides NormalizeData() for expression normalization and FindMarkers() for
    the malignant-vs-non-malignant differential expression (Wilcoxon rank-sum test)
    per training dataset
- name: PreCanCell
  role: R package whose downstream classifier consumes the intersected TMG/NMG DEG
    features; PreCanCell_data() selects matched genes and scales expression prior
    to k-NN
  repo: https://github.com/WangX-Lab/PreCanCell
- name: R
  role: Implementation language for the DEG analysis, gene-set intersection, and PreCanCell
    package
  repo: https://github.com/WangX-Lab/PreCanCell
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:88c62428389245ed41afc399896389daba9c182c35325bed0dca477318770fd6
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
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-dataset-gene-set-intersection

## Summary

Identifies differentially expressed genes reproducibly shared across multiple cancer scRNA-seq datasets by intersecting malignant-vs-non-malignant DEG sets, yielding the 73-gene tumor marker gene (TMG) and 186-gene non-malignant marker gene (NMG) feature sets used by PreCanCell.

## When to use

Use when building robust cancer-cell classification features by intersecting malignant-vs-non-malignant DEGs across multiple training scRNA-seq datasets (e.g., RCC, HNSCC, melanoma, LUAD, BC) rather than relying on a single dataset.

## When NOT to use

- You need to classify individual cells as malignant/non-malignant rather than derive the feature genes — use PreCanCell_classifier() on a PreCanCell_data()-preprocessed test matrix instead
- You only have a single scRNA-seq dataset; the skill's defining step is the intersection across five training datasets, and a one-dataset DEG call does not reproduce the TMG/NMG sets
- You need annotation of non-cancer cell subpopulations (immune, stromal, epithelial) — the resulting marker framework only distinguishes cancer vs. non-cancer cells

## Inputs

- Normalized scRNA-seq expression matrices (e.g., TPM/CPM, rownames = genes, colnames = cells) for five training datasets: T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC
- Per-cell malignant vs. non-malignant cell annotations for each training dataset

## Outputs

- deg_gene_sets.tsv with gene identifiers and set membership labels (TMG / NMG)
- 73-gene tumor marker gene (TMG) set upregulated in malignant cells
- 186-gene non-malignant marker gene (NMG) set upregulated in non-malignant cells

## How to apply

For each training dataset (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC), run Seurat's FindMarkers() comparing malignant versus non-malignant cells with the Wilcoxon rank-sum test, retaining genes with Bonferroni-adjusted P-value < 0.05. Call genes upregulated in malignant cells if expressed in at least 10% of malignant cells with mean log-scale fold change > 0.25, and analogously for non-malignant cells. Intersect the malignant-upregulated genes across all five datasets to obtain the TMG set (73 genes), and the non-malignant-upregulated genes across all five to obtain the NMG set (186 genes). Write both gene lists to deg_gene_sets.tsv with set membership labels; these DEGs then serve as k-NN (k=5) features for PreCanCell. Rationale: requiring commonality across five cancer types filters dataset-specific artifacts, producing pan-cancer marker features.

## Related tools

- **Seurat** (Provides NormalizeData() for expression normalization and FindMarkers() for the malignant-vs-non-malignant differential expression (Wilcoxon rank-sum test) per training dataset)
- **PreCanCell** (R package whose downstream classifier consumes the intersected TMG/NMG DEG features; PreCanCell_data() selects matched genes and scales expression prior to k-NN) — https://github.com/WangX-Lab/PreCanCell
- **R** (Implementation language for the DEG analysis, gene-set intersection, and PreCanCell package) — https://github.com/WangX-Lab/PreCanCell

## Examples

```
markers_rcc <- FindMarkers(seurat_obj, ident.1 = "malignant", ident.2 = "non_malignant", test.use = "wilcox"); tmg <- Reduce(intersect, list(malignant_up_rcc, malignant_up_hnscc, malignant_up_melanoma, malignant_up_luad, malignant_up_bc))
```

## Evaluation signals

- The intersected sets match the reported composition: exactly 73 TMGs and 186 NMGs
- Every gene in each set satisfies the detection (≥10% cells) and log fold-change (> 0.25) criteria in each of the five training datasets
- Every retained gene has Bonferroni-adjusted Wilcoxon rank-sum P-value < 0.05 in the malignant-vs-non-malignant comparison per dataset
- The output deg_gene_sets.tsv contains gene identifiers plus set membership labels, and each gene is assigned to exactly one set (TMG or NMG)
- Genes in the final sets are present in the matched-gene space accepted by PreCanCell_data() preprocessing

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- The resulting feature sets only annotate cancer vs. non-cancer cells and are not designed for identifying subpopulations of non-cancer cells
- No accession IDs are provided for the training datasets (e.g., T5_BC), which can complicate reproducing the exact intersection

## Evidence

- [methods] Bonferroni-corrected Wilcoxon rank-sum test with <0.05 threshold and 10%/0.25 logFC criteria: "the gene expression difference between malignant and non-malignant cells was statistically significant if the adjusted Wilcoxon rank-sum test P-value < 0.05 by the Bonferroni correction"
- [methods] TMG defined as intersection of upregulated genes across five training datasets: "we obtained the gene set, which was the intersection of the upregulated genes in malignant cells identified in each of the five training datasets; this gene set was defined as tumor marker genes"
- [results] DEG counts: 73 TMGs and 186 NMGs: "The DEGs included 73 TMGs and 186 NMGs."
- [abstract] DEGs identified commonly across five cancer scRNA-seq datasets: "PreCanCell first identifies the differentially expressed genes (DEGs) between malignant and non-malignant cells commonly in five common cancer types-associated single-cell transcriptome datasets"
- [methods] FindMarkers() in Seurat used for differential expression analysis: "We performed the differential gene expression analysis with the function “FindMarkers()” in the R package “Seurat”"
