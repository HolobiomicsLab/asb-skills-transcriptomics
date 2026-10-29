---
name: cascc-attractor-based-clustering-pipeline-execution
description: 'Use when clustering scRNA-seq count matrices with a co-expression-assisted
  method: gene-by-cell matrices needing attractor-based feature selection, cluster-number
  prediction, and UMAP visualization with cluster correspondence.'
when_to_use_negative:
- Input is not a gene-by-cell scRNA-seq expression matrix (e.g. bulk RNA-seq, proteomics,
  or already-aggregated feature tables)
- Input expression data are not well-normalized and preprocessing/filtering (filterGenes
  with removeMT.RP.ERCC, removeNonProtein) has not been performed
- A no-marker-seed approach is required when no known marker gene is available to
  seed findAttractor.adaptive — use the unsupervised adaptive variant or run.CASCC
  defaults instead
edam_operation: http://edamontology.org/operation_3620
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CASCC
  role: Core R package executing the attractor-based clustering pipeline (run.CASCC,
    CASCC.featureSelection, predictClusterK, kmeansCenterModify)
  repo: https://github.com/LingyiC/CASCC
- name: cafr
  role: Dependency package providing the adaptive attractor algorithm for co-expression
    feature identification
  repo: https://github.com/weiyi-bitw/cafr
- name: Seurat 4.3.0
  role: Dependency used to run CASCC within a Seurat workflow and for downstream analysis
    (FindAllMarkers, DimPlot visualization of the UMAP)
- name: devtools/remotes
  role: Installation of the pinned Seurat 4.3.0 dependency and GitHub packages
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:875521638a3771b0321be558fa8207011ff3f37c043eb15fc9daac76eaba548c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC
  tools:
  - CASCC
  - NbClust
  - R
  - Seurat 4.3.0
  - cafr
  - devtools/remotes
  - ggplot2
  edam_operation: http://edamontology.org/operation_3620
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# cascc-attractor-based-clustering-pipeline-execution

## Summary

Executes the CASCC clustering pipeline for single-cell RNA-seq data, which improves biological accuracy by using gene co-expression features identified with an unsupervised adaptive attractor algorithm, then predicts cluster number via silhouette index and performs k-means clustering with modified attractor centers.

## When to use

Use when clustering scRNA-seq count matrices with a co-expression-assisted method: gene-by-cell matrices needing attractor-based feature selection, cluster-number prediction, and UMAP visualization with cluster correspondence.

## When NOT to use

- Input is not a gene-by-cell scRNA-seq expression matrix (e.g. bulk RNA-seq, proteomics, or already-aggregated feature tables)
- Input expression data are not well-normalized and preprocessing/filtering (filterGenes with removeMT.RP.ERCC, removeNonProtein) has not been performed
- A no-marker-seed approach is required when no known marker gene is available to seed findAttractor.adaptive — use the unsupervised adaptive variant or run.CASCC defaults instead

## Inputs

- Gene-by-cell single-cell RNA-seq expression matrix (e.g. Data_PDAC_peng_2k) or raw data from a study accession such as PRJCA001063
- Well-normalized input expression data (inputDataType = "well-normalized")
- Optional marker gene seed for attractor identification (e.g. "AIF1")

## Outputs

- CASCC result object res with res$res.predictK$adata (feature-selected UMAP, an adata object)
- Attractor co-expression output and attractor feature list
- Predicted cluster number with silhouette index (method = "ward.D2", index = "silhouette")
- Main-type clustering output via kmeansCenterModify (res$mainType.output)
- DEGs from Seurat::FindAllMarkers(adata, only.pos = T)

## How to apply

Run the pipeline in R 4.0.5 with Seurat 4.3.0 and the cafr dependency installed via devtools/remotes. Start with CASCC::filterGenes(data, min.cells, removeNonProtein, removeMT.RP.ERCC) for gene filtering, then CASCC.featureSelection(Data, inputDataType = "well-normalized") to identify co-expression attractor features, and predictClusterK(data, fs.res, method = "ward.D2", index = "silhouette") to determine the cluster count. The main run via CASCC::run.CASCC(data) returns a result object in which res$res.predictK$adata holds the feature-selected UMAP for downstream inspection. Optional refinement includes kmeansCenterModify(res) for k-means clustering with modified attractor centers, findAttractor.adaptive() seeded with a known marker (e.g. "AIF1"), and visualization with attractorDimplot(). In the published PDAC example (Data_PDAC_peng_2k), the analysis identified eight distinct clusters, each corresponding to a specific attractor, and DEGs can be found with Seurat::FindAllMarkers(adata, only.pos = T).

## Related tools

- **CASCC** (Core R package executing the attractor-based clustering pipeline (run.CASCC, CASCC.featureSelection, predictClusterK, kmeansCenterModify)) — https://github.com/LingyiC/CASCC
- **cafr** (Dependency package providing the adaptive attractor algorithm for co-expression feature identification) — https://github.com/weiyi-bitw/cafr
- **Seurat 4.3.0** (Dependency used to run CASCC within a Seurat workflow and for downstream analysis (FindAllMarkers, DimPlot visualization of the UMAP))
- **devtools/remotes** (Installation of the pinned Seurat 4.3.0 dependency and GitHub packages)

## Examples

```
res <- CASCC::run.CASCC(Data_PDAC_peng_2k); adata <- res$res.predictK$adata; DEGs <- Seurat::FindAllMarkers(adata, only.pos = T)
```

## Evaluation signals

- dim(Data_PDAC_peng_2k) confirms expected matrix dimensions before running the pipeline
- res$res.predictK$adata is non-null and contains a feature-selected UMAP usable with Seurat DimPlot
- str(Res_PDAC_peng_2k, max.level = 2) on the deposited reference result object matches the structure returned by run.CASCC
- Cluster number prediction with predictClusterK(method = "ward.D2", index = "silhouette") yields a defined cluster count; the published PDAC example recovered eight distinct clusters, each corresponding to a specific attractor
- CASCC::checkDependencies() confirms Seurat 4.3.0 and cafr dependencies are installed

## Limitations

- Requires R version 4.0.5 (2021-03-31) with Seurat pinned at 4.3.0 and the cafr dependency — other versions may break the pipeline
- Feature selection has parameter constraints (exponent.max = 10, exponent.min = 2) that bound attractor exponent range
- findAttractor.adaptive requires a seed marker gene (e.g. "AIF1"); without a biologically meaningful seed, attractor identification may not reflect co-expression structure
- Attribution/citation details are incomplete in the repository README (no explicit citation list or changelog)

## Evidence

- [intro] CASCC is a clustering method: "CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm."
- [readme] installation pinned deps: "remotes::install_version("Seurat", "4.3.0") # dependency
install_github("weiyi-bitw/cafr") # dependency"
- [methods] run.CASCC call: "res <- CASCC::run.CASCC(Data_PDAC_peng_2k)"
- [methods] predictClusterK call: "res.predictK <- predictClusterK(data, fs.res, method = "ward.D2", index = "silhouette", min.nc_fix = FALSE)"
- [methods] feature selection params: "fs.res <- CASCC.featureSelection(Data_PDAC_peng_2k, inputDataType = "well-normalized", attr.raw = NULL, exponent.max = 10, exponent.min = 2"
- [methods] eight clusters finding: "The analysis identifies the presence of eight distinct clusters, each corresponding to a specific attractor"
- [methods] UMAP output: "adata <- res$res.predictK$adata # feature-selected UMAP"
