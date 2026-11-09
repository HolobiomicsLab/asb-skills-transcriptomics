---
name: single-cell-rna-seq-processing
description: Use when raw single-cell count matrices from GEO deposits (e.g. GSE145926
  BALF or PBMC scRNA-seq) need QC filtering, LogNormalize normalization, PCA/UMAP
  embedding, graph-based cluster annotation, and severe-vs-mild differential expression
  testing.
when_to_use_negative:
- Input is bulk RNA-seq data (e.g. the 7 transcriptomics datasets like GSE155454 or
  GSE157103 analysed via t-test with BH correction) — use the bulk transcriptomics
  comparison skill instead
- Input is already a processed gene-level statistics table (p-value, adjusted p-value,
  ratio, fold-change) — query it directly via the Streamlit app rather than re-clustering
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: R package used to construct per-cell QC-filtered Seurat objects, LogNormalize
    normalization, scaling of variable genes, PCA/UMAP, graph-based clustering, cell-type
    annotation, and Wilcoxon rank-sum severe-vs-mild DE testing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:d874681d158f50c6242df23b0faf67800a12d9cae21ee8ba5c10a2b84bd474ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-rna-seq-processing

## Summary

A Seurat-based workflow for building, dimensionality-reducing, clustering and annotating single-cell RNA-seq objects from public COVID-19 count matrices (e.g. the 78,666-cell GSE145926 BALF dataset), enabling cell-type-stratified differential expression of severity biomarkers such as MCEMP1, HLA-DRA and ETS1.

## When to use

Use when raw single-cell count matrices from GEO deposits (e.g. GSE145926 BALF or PBMC scRNA-seq) need QC filtering, LogNormalize normalization, PCA/UMAP embedding, graph-based cluster annotation, and severe-vs-mild differential expression testing.

## When NOT to use

- Input is bulk RNA-seq data (e.g. the 7 transcriptomics datasets like GSE155454 or GSE157103 analysed via t-test with BH correction) — use the bulk transcriptomics comparison skill instead
- Input is already a processed gene-level statistics table (p-value, adjusted p-value, ratio, fold-change) — query it directly via the Streamlit app rather than re-clustering

## Inputs

- Raw single-cell count matrices from GEO (e.g. GSE145926 BALF matrices, 78,666 cells)
- Authors' processed single-cell data from Zenodo/GitHub deposit (10.5281/zenodo.7114840)
- PBMC and BALF publicly available single-cell sequencing datasets

## Outputs

- Annotated Seurat object with per-cell QC-filtered, LogNormalize-normalized, PCA/UMAP-embedded cells
- Cluster annotation mapping cell clusters to immune and epithelial cell types
- Cell-type-stratified DE results table (log fold changes and adjusted p-values per cell type, e.g. balf_celltype_de_results.csv)

## How to apply

Download the BALF count matrices (78,666 cells) from GEO, supplemented by the authors' processed data from the Zenodo/GitHub deposit (10.5281/zenodo.7114840). Load the matrices in R, construct Seurat objects with per-cell QC filtering, apply LogNormalize normalization and scale variable genes, then run PCA followed by UMAP on the integrated object. Perform graph-based clustering and annotate the clusters into immune and epithelial cell types (e.g. the 11 clusters recovered for GSE145926). Run cell-type-stratified differential expression testing (Wilcoxon rank-sum) comparing severe versus mild COVID-19 cells for genes of interest such as MCEMP1, ETS1 and HLA-DRA, and export log fold changes and adjusted p-values per cell type (e.g. to balf_celltype_de_results.csv).

## Related tools

- **Seurat** (R package used to construct per-cell QC-filtered Seurat objects, LogNormalize normalization, scaling of variable genes, PCA/UMAP, graph-based clustering, cell-type annotation, and Wilcoxon rank-sum severe-vs-mild DE testing)

## Examples

```
seurat_obj <- CreateSeuratObject(counts = counts); seurat_obj <- NormalizeData(seurat_obj, normalization.method = "LogNormalize"); seurat_obj <- RunPCA(seurat_obj); seurat_obj <- RunUMAP(seurat_obj, dims = 1:10)
```

## Evaluation signals

- The integrated BALF object recovers 11 annotated clusters that map to known immune and epithelial cell types
- DE testing shows no differential expression of MCEMP1 and ETS1 between severe and mild BALF cells
- Reduced HLA-DRA expression is observed in macrophages, DC, mast cells, neutrophils, epithelial cells and NK cells
- Output CSV contains log fold changes and adjusted p-values per cell type (balf_celltype_de_results.csv schema)

## Limitations

- Differential expression conclusions can diverge across compartments: MCEMP1 and HLA-DRA are consistently regulated in peripheral blood (primarily CD14+ cells) but MCEMP1/ETS1 show no differential expression in BALF single cells
- No gene expression profiles exist more than 4 days before respiratory nadir, so timing of MCEMP1/HLA-DRA dysregulation cannot be precisely determined
- Heterogeneity across study datasets noted by the authors complicates interpretation of cell-type-level expression differences

## Evidence

- [abstract] Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved.: "Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved."
- [discussion] we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF: "we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF"
- [other] construct Seurat objects with per-cell QC filtering, LogNormalize normalization and scaling of variable genes: [evidence span withheld — anchor status: not_found]
- [other] reduced HLA-DRA is observed in macrophages, DC, mast cells, neutrophils, epithelial cells and NK cells: [evidence span withheld — anchor status: not_found]
- [readme] All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity: "This database is a curation of 7 transcriptomics datasets comparing the gene expression in severe COVID-19 and mild COVID-19 subjects."
