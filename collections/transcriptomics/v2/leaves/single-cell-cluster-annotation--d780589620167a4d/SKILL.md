---
name: single-cell-cluster-annotation
description: Use when a Seurat (or equivalent) single-cell object has been clustered
  (e.g. 11 BALF clusters from GSE145926) and cluster labels must be assigned to cell
  types before stratified DE testing.
when_to_use_negative:
- Input is already a cell-type-annotated object or a bulk/deconvolved expression matrix
  — no cluster annotation step is needed
- The goal is cohort-level bulk RNAseq differential expression (e.g. the 7 GSE/M data­sets
  queried via the Streamlit app), which does not involve clustering single cells
- Sample size or depth is insufficient to resolve discrete transcriptional clusters
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: R toolkit used to construct the Seurat object, run QC/LogNormalize, PCA, UMAP,
    graph-based clustering and cell-type annotation of the BALF single-cell data
  repo: https://github.com/kuanrongchan/COVID19-severity
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:1b364d06c8fd28051914db8cc8f72f2a71c8fb3daf07caa13a875299e0e94d88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Seurat
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-cluster-annotation

## Summary

Annotate data-driven single-cell clusters into biologically meaningful cell types (immune and epithelial subsets) after PCA/UMAP and graph-based clustering, so that downstream cell-type-stratified differential expression (e.g. severe vs mild COVID-19) is interpretable.

## When to use

Use when a Seurat (or equivalent) single-cell object has been clustered (e.g. 11 BALF clusters from GSE145926) and cluster labels must be assigned to cell types before stratified DE testing.

## When NOT to use

- Input is already a cell-type-annotated object or a bulk/deconvolved expression matrix — no cluster annotation step is needed
- The goal is cohort-level bulk RNAseq differential expression (e.g. the 7 GSE/M data­sets queried via the Streamlit app), which does not involve clustering single cells
- Sample size or depth is insufficient to resolve discrete transcriptional clusters

## Inputs

- BALF or PBMC single-cell count matrices (e.g. GSE145926, 78,666 cells; GEO raw plus authors' processed Zenodo/GitHub deposit)
- Seurat object after PCA, UMAP and graph-based clustering
- Canonical cell-type marker gene lists

## Outputs

- Cluster-to-cell-type annotation map over 11 clusters (immune and epithelial cell types)
- Annotated Seurat object supporting cell-type-stratified DE testing
- Per-cell-type DE table (log fold changes and adjusted p-values, e.g. balf_celltype_de_results.csv)

## How to apply

Build a Seurat object from the BALF count matrices (78,666 cells from GSE145926), apply per-cell QC filtering, LogNormalize normalization, and scaling of variable genes. Run PCA, then UMAP, then graph-based clustering to recover discrete clusters. Annotate each cluster to a cell type by matching canonical marker expression (e.g. HLA-DRA and other APC markers for macrophages/DC, epithelial and NK markers for the remaining clusters) against the annotated cell populations reported for the dataset. Only after annotation, run cell-type-stratified Wilcoxon rank-sum differential expression (severe vs mild COVID-19) for genes of interest such as MCEMP1, ETS1 and HLA-DRA, writing log fold changes and adjusted p-values per cell type (e.g. to balf_celltype_de_results.csv).

## Related tools

- **Seurat** (R toolkit used to construct the Seurat object, run QC/LogNormalize, PCA, UMAP, graph-based clustering and cell-type annotation of the BALF single-cell data) — https://github.com/kuanrongchan/COVID19-severity

## Examples

```
Rscript -e 'balf <- CreateSeuratObject(Read10X("GSE145926/BALF")); balf <- SCTransform/LogNormalize; balf <- RunUMAP(RunPCA(balf)); balf <- FindClusters(balf); Idents(balf) <- annotated_celltypes; balf_de <- DE_sev_vs_mild(balf, genes=c("MCEMP1","ETS1","HLA-DRA"))'
```

## Evaluation signals

- Exactly 11 annotated clusters are recovered, matching the source card's validation criterion
- Annotation assigns clusters to both immune and epithelial cell types, including macrophages, DC, mast cells, neutrophils, epithelial cells and NK cells
- Cell-type-stratified DE reproduces no differential expression of MCEMP1 and ETS1 between severe and mild COVID-19
- Reduced HLA-DRA (negative log fold change, significant adjusted p-value) is observed in macrophages, DC, mast cells, neutrophils, epithelial cells and NK cells
- DE results are exported with per-cell-type log fold changes and adjusted p-values to balf_celltype_de_results.csv

## Limitations

- Differential expression of MCEMP1 and ETS1 may not be detected in BALF cell types even though these genes are consistent bulk blood biomarkers — absence of signal is an expected outcome, not a pipeline failure
- Study heterogeneity across datasets was noted by the authors, so cluster annotations and marker definitions may not transfer uniformly between cohorts
- Annotation quality depends on the dataset's published cell-type labels (authors' processed Zenodo/GitHub deposit) and canonical marker expression; mismatched references can mislabel clusters

## Evidence

- [other] Run cell-type-stratified differential expression testing (Wilcoxon rank-sum) comparing severe versus mild COVID-19 cells for MCEMP1, ETS1 and HLA-DRA.: [evidence span withheld — anchor status: not_found]
- [other] Validation: the reproduced output is accepted when 11 annotated clusters are recovered and the results show no differential expression of MCEMP1 and ETS1 while reduced HLA-DRA is observed.: [evidence span withheld — anchor status: not_found]
- [discussion] we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF: "we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF"
- [readme] All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity: "All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity"
