---
name: cell-type-specific-differential-expression-analysis
description: Use when single-cell RNA-seq data (e.g. BALF or PBMC count matrices from
  GEO) has been clustered and cell-type-annotated, and the goal is to test candidate
  genes (e.g. MCEMP1, HLA-DRA, ETS1) for differential expression between severity
  groups per cell type.
when_to_use_negative:
- Input is bulk RNAseq data rather than single-cell matrices — use the bulk severity
  database workflow (t-test, BH-adjusted p-values, fold-change per dataset) instead
- Cell clusters have not yet been defined or annotated — run clustering and annotation
  first before stratified testing
- The question requires timing/kinetics of expression rather than severity-group contrasts
  — longitudinal transcriptomics analysis is needed instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: R toolkit for constructing Seurat objects, QC filtering, LogNormalize normalization,
    PCA/UMAP, graph-based clustering, and cell-type-stratified Wilcoxon rank-sum differential
    expression testing
  repo: https://github.com/kuanrongchan/COVID19-severity
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:2e1711d12bc62a497e439aa7f22539a8ffcb66f20db0bb03bb2da715d6c3cf00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cell-type-specific-differential-expression-analysis

## Summary

Stratified differential expression testing on annotated single-cell clusters, comparing disease severity groups (severe vs mild COVID-19) within each cell type using Wilcoxon rank-sum tests in Seurat, to reveal cell-type-resolved gene dysregulation such as HLA-DRA suppression in antigen-presenting cells.

## When to use

Use when single-cell RNA-seq data (e.g. BALF or PBMC count matrices from GEO) has been clustered and cell-type-annotated, and the goal is to test candidate genes (e.g. MCEMP1, HLA-DRA, ETS1) for differential expression between severity groups per cell type.

## When NOT to use

- Input is bulk RNAseq data rather than single-cell matrices — use the bulk severity database workflow (t-test, BH-adjusted p-values, fold-change per dataset) instead
- Cell clusters have not yet been defined or annotated — run clustering and annotation first before stratified testing
- The question requires timing/kinetics of expression rather than severity-group contrasts — longitudinal transcriptomics analysis is needed instead

## Inputs

- Single-cell count matrices from GEO (e.g. GSE145926 BALF, 78,666 cells), supplemented with authors' processed data from Zenodo/GitHub deposit
- Cell-type cluster annotations for the Seurat object (11 immune/epithelial clusters)
- Per-cell severity metadata labels (severe vs mild COVID-19)

## Outputs

- Per-cell-type differential expression table with log fold changes and adjusted p-values for each candidate gene (e.g. balf_celltype_de_results.csv)
- Annotated Seurat object with UMAP embeddings and 11 cell-type clusters

## How to apply

Build Seurat objects from the count matrices with per-cell QC filtering, LogNormalize normalization and scaling of variable genes, then run PCA and UMAP followed by graph-based clustering (11 clusters were recovered for GSE145926 BALF, 78,666 cells) and annotate clusters into immune and epithelial cell types. For each annotated cell type, subset severe and mild COVID-19 cells and run cell-type-stratified Wilcoxon rank-sum differential expression for the genes of interest. Summarize log fold changes and adjusted p-values per cell type and write the results table (e.g. balf_celltype_de_results.csv). The rationale is that bulk-level signals (MCEMP1/HLA-DRA from the 7 RNAseq datasets) map to specific immune subsets — the article found dysregulation occurs primarily in CD14+ cells, and in BALF reduced HLA-DRA appeared in macrophages, DC, mast cells, neutrophils, epithelial cells and NK cells while MCEMP1 and ETS1 showed no differential expression.

## Related tools

- **Seurat** (R toolkit for constructing Seurat objects, QC filtering, LogNormalize normalization, PCA/UMAP, graph-based clustering, and cell-type-stratified Wilcoxon rank-sum differential expression testing) — https://github.com/kuanrongchan/COVID19-severity

## Examples

```
Run cell-type-stratified DE per Seurat cluster: for each cell type c, `de <- subset(balf, idents = c); FindMarkers(de, ident.1 = "severe", ident.2 = "mild", test.use = "wilcox", logfc.threshold = 0)`
```

## Evaluation signals

- Reproduction is accepted when 11 annotated clusters are recovered from the GSE145926 BALF object
- Results show no differential expression of MCEMP1 and ETS1 between severe and mild COVID-19 across BALF cell types
- Reduced HLA-DRA expression is observed in macrophages, DC, mast cells, neutrophils, epithelial cells and NK cells
- Output table (balf_celltype_de_results.csv) contains log fold changes and adjusted p-values for every annotated cell type
- Differential testing is performed within each cell type rather than on the pooled object, so per-cluster group sizes must be checked before trusting p-values

## Limitations

- Heterogeneity in the study datasets was noted; future studies are needed to understand heterogeneity in patients influencing MCEMP1 and HLA-DRA expression
- Datasets did not directly evaluate the effect of different SARS-CoV-2 variants on transcriptomic responses to severe COVID-19
- In BALF cells, no differential expression of MCEMP1 and ETS1 was detected — the bulk blood signal does not replicate uniformly at single-cell level across tissues

## Evidence

- [methods] Load the count matrices in R and construct Seurat objects with per-cell QC filtering, LogNormalize normalization and scaling of variable genes.: [evidence span withheld — anchor status: not_found]
- [methods] Run cell-type-stratified differential expression testing (Wilcoxon rank-sum) comparing severe versus mild COVID-19 cells: [evidence span withheld — anchor status: not_found]
- [results] reduced HLA-DRA expression was observed in macrophages, DC, mast cells, neutrophils, epithelial cells and NK cells: [evidence span withheld — anchor status: not_found]
- [intro] we ascertained that the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets: "we ascertained that the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets"
- [discussion] we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF: "we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF"
