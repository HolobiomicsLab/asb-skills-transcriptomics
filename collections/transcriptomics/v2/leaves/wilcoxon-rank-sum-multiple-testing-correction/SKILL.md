---
name: wilcoxon-rank-sum-multiple-testing-correction
description: Use when you need to derive a robust, pan-dataset malignant-vs-non-malignant
  marker gene feature set by intersecting statistically significant DEGs across multiple
  annotated cancer scRNA-seq training datasets.
when_to_use_negative:
- The goal is to annotate non-cancer cell subpopulations (immune, stromal, epithelial)
  — PreCanCell's DEG features are limited to binary cancer vs non-cancer annotation
- Only one dataset is available for differential expression, since the TMG/NMG sets
  require genes to pass significance and fold-change criteria in all five training
  datasets
- Expression data are raw counts without library-depth normalization (e.g. TPM/CPM),
  since FindMarkers() is applied to normalized values produced via Seurat's NormalizeData()
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: R package whose NormalizeData() and FindMarkers() functions perform the normalization
    and Wilcoxon rank-sum differential expression testing with Bonferroni correction
    (version >= 4.3.0)
  repo: https://github.com/WangX-Lab/PreCanCell
- name: PreCanCell
  role: R package within which the DEG-derived TMG/NMG feature sets are computed and
    subsequently used as k-NN (k = 5) classification features
  repo: https://github.com/WangX-Lab/PreCanCell
- name: R
  role: Language in which the PreCanCell package and Seurat-based DEG identification
    workflow are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:b84e1ae03d3ef67fa09d4e0bc9ee56ce4a37be84d9a416e4a2462a29f69712c5
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# wilcoxon-rank-sum-multiple-testing-correction

## Summary

Identifies genes differentially expressed between malignant and non-malignant cells in each of five cancer scRNA-seq training datasets (RCC, HNSCC, melanoma, LUAD, BC) using Seurat's FindMarkers() with a Wilcoxon rank-sum test, Bonferroni-adjusted P < 0.05, and a 10% detection / logFC > 0.25 upregulation filter, then intersects results across datasets to form the 73-TMG and 186-NMG feature sets for PreCanCell.

## When to use

Use when you need to derive a robust, pan-dataset malignant-vs-non-malignant marker gene feature set by intersecting statistically significant DEGs across multiple annotated cancer scRNA-seq training datasets.

## When NOT to use

- The goal is to annotate non-cancer cell subpopulations (immune, stromal, epithelial) — PreCanCell's DEG features are limited to binary cancer vs non-cancer annotation
- Only one dataset is available for differential expression, since the TMG/NMG sets require genes to pass significance and fold-change criteria in all five training datasets
- Expression data are raw counts without library-depth normalization (e.g. TPM/CPM), since FindMarkers() is applied to normalized values produced via Seurat's NormalizeData()

## Inputs

- Normalized expression matrices (rownames = genes, colnames = cells) for five training datasets: T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC
- Per-cell malignant/non-malignant annotations for each training dataset

## Outputs

- deg_gene_sets.tsv containing gene identifiers and TMG/NMG set-membership labels
- 73 tumor marker genes (TMGs) upregulated in malignant cells
- 186 non-malignant marker genes (NMGs) upregulated in non-malignant cells

## How to apply

For each of the five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC), load the normalized expression matrix and malignant/non-malignant cell annotations and run Seurat's FindMarkers() comparing the two cell classes with the Wilcoxon rank-sum test, retaining only genes whose Bonferroni-adjusted P-value < 0.05. Among significant genes, call a gene upregulated in malignant cells if it is expressed in at least 10% of malignant cells with mean log-scale fold change > 0.25 (and analogously for non-malignant cells). Compute the intersection of malignant-upregulated genes across all five datasets to define the tumor marker gene (TMG) set, and the intersection of non-malignant-upregulated genes across all five datasets for the non-malignant marker gene (NMG) set. The rationale is that genes reproducibly differentially expressed across all five cancer types form generalizable k-NN features for PreCanCell's malignant/non-malignant classification. Write the resulting gene lists with set-membership labels to deg_gene_sets.tsv.

## Related tools

- **Seurat** (R package whose NormalizeData() and FindMarkers() functions perform the normalization and Wilcoxon rank-sum differential expression testing with Bonferroni correction (version >= 4.3.0)) — https://github.com/WangX-Lab/PreCanCell
- **PreCanCell** (R package within which the DEG-derived TMG/NMG feature sets are computed and subsequently used as k-NN (k = 5) classification features) — https://github.com/WangX-Lab/PreCanCell
- **R** (Language in which the PreCanCell package and Seurat-based DEG identification workflow are implemented)

## Examples

```
markers <- FindMarkers(obj, ident.1 = "malignant", ident.2 = "non_malignant", test.use = "wilcox"); sig <- markers[markers$p_val_adj < 0.05, ]
```

## Evaluation signals

- Resulting DEG sets match the reported composition of exactly 73 TMGs and 186 NMGs
- Every gene in each set passes the 10% detection and 0.25 log fold-change criteria in each of the five training datasets
- Every retained gene has Bonferroni-adjusted Wilcoxon rank-sum test P-value < 0.05 in each training dataset
- Each TMG/NMG gene is present in the intersection across all five datasets (i.e., appears in the per-dataset upregulated lists for T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, and T5_BC)
- deg_gene_sets.tsv parses with two columns (gene identifier and set label) and row counts summing to 73 + 186

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- The feature set is limited to annotating cancer vs non-cancer cells and is not designed for identifying subpopulations of non-cancer cells
- Common DEGs across five datasets may exclude biologically relevant markers specific to individual cancer types due to the strict full-intersection requirement

## Evidence

- [methods] the gene expression difference between malignant and non-malignant cells was statistically significant if the adjusted Wilcoxon rank-sum test P-value < 0.05 by the Bonferroni correction: "the gene expression difference between malignant and non-malignant cells was statistically significant if the adjusted Wilcoxon rank-sum test P-value < 0.05 by the Bonferroni correction"
- [methods] we obtained the gene set, which was the intersection of the upregulated genes in malignant cells identified in each of the five training datasets; this gene set was defined as tumor marker genes: "we obtained the gene set, which was the intersection of the upregulated genes in malignant cells identified in each of the five training datasets; this gene set was defined as tumor marker genes"
- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [methods] We performed the differential gene expression analysis with the function “FindMarkers()” in the R package “Seurat”: "We performed the differential gene expression analysis with the function “FindMarkers()” in the R package “Seurat”"
- [readme] The `Seurat` package (*version* >= 4.3.0) is used for data preprocessing.: "The `Seurat` package (*version* >= 4.3.0) is used for data preprocessing."
