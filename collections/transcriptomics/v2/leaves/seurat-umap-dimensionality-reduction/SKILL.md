---
name: seurat-umap-dimensionality-reduction
description: Use when raw scRNA-seq count matrices (e.g., GEO-deposited 10x-style
  BALF counts) must be reduced to a low-dimensional embedding for clustering and cell-type
  annotation before severe-vs-mild differential expression testing.
when_to_use_negative:
- Input is bulk RNA-seq transcriptomics data (e.g., the 7 bulk datasets GSE155454,
  GSE161731, GSE157103, GSE152418, GSE172114, GSE164805, Bibert 2021) — use the Streamlit
  database/app workflow instead
- The Seurat object already has PCA/UMAP and cluster annotations computed and the
  goal is only to rerun or read differential expression results
- Only one gene's summary statistics across pre-computed datasets is needed (query
  the Streamlit app rather than reconstructing the single-cell pipeline)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: R package used to construct Seurat objects, run PCA and UMAP dimensionality
    reduction, and perform graph-based clustering of the integrated BALF object prior
    to cell-type annotation and Wilcoxon differential testing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:5b92c18bef3436f145dc8a8b108b5ad28920d3d261cea8bcacd6df5053412d2e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# seurat-umap-dimensionality-reduction

## Summary

Applies Seurat's PCA-then-UMAP dimensionality reduction to single-cell RNA-seq count matrices (here, 78,666 BALF cells from GSE145926) after per-cell QC and LogNormalize normalization, so that immune and epithelial cell populations can be graph-clustered and cell-type-stratified differential expression (e.g., MCEMP1, HLA-DRA, ETS1) can be interpreted per cell type.

## When to use

Use when raw scRNA-seq count matrices (e.g., GEO-deposited 10x-style BALF counts) must be reduced to a low-dimensional embedding for clustering and cell-type annotation before severe-vs-mild differential expression testing.

## When NOT to use

- Input is bulk RNA-seq transcriptomics data (e.g., the 7 bulk datasets GSE155454, GSE161731, GSE157103, GSE152418, GSE172114, GSE164805, Bibert 2021) — use the Streamlit database/app workflow instead
- The Seurat object already has PCA/UMAP and cluster annotations computed and the goal is only to rerun or read differential expression results
- Only one gene's summary statistics across pre-computed datasets is needed (query the Streamlit app rather than reconstructing the single-cell pipeline)

## Inputs

- BALF single-cell count matrices from GSE145926 (78,666 cells) downloaded from GEO, supplemented with authors' processed data from the Zenodo/GitHub deposit
- Seurat object after per-cell QC filtering and LogNormalize normalization

## Outputs

- PCA and UMAP embeddings for the integrated BALF Seurat object
- Graph-based clusters (11 clusters) with immune/epithelial cell-type annotations
- Annotated cell-type labels used for cell-type-stratified differential expression (balf_celltype_de_results.csv)

## How to apply

Load the GSE145926 BALF count matrices (78,666 cells) in R and build Seurat objects, applying per-cell QC filtering, LogNormalize normalization, and scaling of variable genes. Run RunPCA on the scaled variable genes, then RunUMAP on the top PCs of the integrated BALF object to obtain the low-dimensional embedding. Use the PCA/UMAP space for graph-based clustering, annotating the resulting 11 cell clusters into immune and epithelial cell types. The rationale is that cell-type stratification after dimensionality reduction is what enables the Wilcoxon rank-sum severe-vs-mild tests for MCEMP1, ETS1 and HLA-DRA to be resolved per cell type; without an embedding and clustering, the broad HLA-DRA suppression across antigen-presenting and other BALF cell populations cannot be localized.

## Related tools

- **Seurat** (R package used to construct Seurat objects, run PCA and UMAP dimensionality reduction, and perform graph-based clustering of the integrated BALF object prior to cell-type annotation and Wilcoxon differential testing)

## Examples

```
balf <- RunPCA(balf, verbose = FALSE); balf <- RunUMAP(balf, dims = 1:30)
```

## Evaluation signals

- Graph-based clustering in the PCA/UMAP space recovers 11 annotated clusters spanning immune and epithelial cell types
- Cell-type-stratified Wilcoxon tests show no differential expression of MCEMP1 and ETS1 between severe and mild COVID-19 BALF cells
- Reduced HLA-DRA expression is observed in macrophages, DC, mast cells, neutrophils, epithelial cells and NK cells (log fold changes and adjusted p-values recorded per cell type)
- Per-cell-type log fold changes and adjusted p-values are written to balf_celltype_de_results.csv without missing cell types

## Limitations

- Validation is dataset-specific: acceptance requires recovering the 11 annotated clusters and the expected MCEMP1/ETS1-null, HLA-DRA-suppressed pattern; other BALF datasets may cluster differently
- The skill was demonstrated on BALF single cells from GSE145926 only; generalization to other tissues or cohorts is untested in this context
- Cluster resolution and annotation of the 11 clusters influence downstream cell-type-stratified DE, so results are sensitive to clustering choices

## Evidence

- [abstract] Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved.: "Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved."
- [discussion] we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF: "we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF"
