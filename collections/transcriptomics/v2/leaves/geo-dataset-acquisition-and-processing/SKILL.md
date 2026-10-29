---
name: geo-dataset-acquisition-and-processing
description: Use when starting from a public GEO scRNA-seq accession (e.g. GSE81682,
  GSE129788, GSE100866, GSE87038) and the goal is a processed Seurat/AnnData object
  ready for export to CellexalVR input files.
when_to_use_negative:
- The data are already exported CellexalVR input files (sqlite3 expression database
  plus .mds coordinate files) sitting in a Data folder — no reprocessing is needed
- The analysis target is VR-session back-end computation (differential expression
  via Wilcox test, heatmap clustering of top 250 marker genes, slingshot pseudotime)
  rather than data acquisition and conversion
- The input is not single-cell gene expression data, e.g. bulk RNA-seq or non-omics
  tabular data
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: cellexalvrR
  role: R package that converts Seurat/AnnData objects via as_cellexalvrR and writes
    CellexalVR input files via export2cellexalvr
  repo: https://github.com/sonejilab/cellexalvrR
- name: Seurat
  role: User-preferred R tool for processing the single-cell data before conversion
- name: SCANPY
  role: User-preferred Python tool for processing the single-cell data into AnnData
    objects before conversion
- name: CellexalVR
  role: VR platform that consumes the exported input files from a sensibly named dataset
    folder inside the Data folder
  repo: https://github.com/sonejilab/cellexalvr
- name: R
  role: Runtime (>= 4.0.0) hosting cellexalvrR; Rscript path must be configured in
    Config/config.xml for the VR client
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:ef4f579d8b0433e39ebc01f2a75df1dafa8517fa364c99c5681322630ae16519
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvrR
  tools:
  - CellexalVR
  - R
  - SCANPY
  - Seurat
  - cellexalvrR
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# geo-dataset-acquisition-and-processing

## Summary

Acquires a single-cell RNA-seq dataset from GEO (e.g. GSE81682 mouse HSPC from Nestorowa et al., 2016), processes it with Seurat or SCANPY including dimension reduction, and converts the resulting Seurat/AnnData object into CellexalVR input files using the cellexalvrR R package.

## When to use

Use when starting from a public GEO scRNA-seq accession (e.g. GSE81682, GSE129788, GSE100866, GSE87038) and the goal is a processed Seurat/AnnData object ready for export to CellexalVR input files.

## When NOT to use

- The data are already exported CellexalVR input files (sqlite3 expression database plus .mds coordinate files) sitting in a Data folder — no reprocessing is needed
- The analysis target is VR-session back-end computation (differential expression via Wilcox test, heatmap clustering of top 250 marker genes, slingshot pseudotime) rather than data acquisition and conversion
- The input is not single-cell gene expression data, e.g. bulk RNA-seq or non-omics tabular data

## Inputs

- GEO accession data (e.g. GSE81682 mouse HSPC, GSE129788 ageing brain, GSE100866 CITEseq, GSE87038 mouse gastrulation)
- Raw single-cell count matrix (scRNA-seq)
- Processed Seurat object or Scanpy/AnnData (h5ad) object
- Dimension-reduction cell coordinates (tSNE/UMAP/PHATE/DDRTree), optionally RNA velocity destination coordinates

## Outputs

- cellexalvrR R object (from as_cellexalvrR)
- SQLite3 database of all non-zero gene-expression values for all cells and genes
- 4-column .mds text files of DR cell coordinates (cell identifier + x/y/z)
- Data-folder dataset directory with a sensible name placed under the CellexalVR Data folder

## How to apply

Download the GEO accession (e.g. GSE81682 mouse hematopoietic stem and progenitor cells, Nestorowa et al. 2016) and process the raw counts with a user-preferred tool such as Seurat or SCANPY, performing dimension reduction (tSNE, UMAP, DDRTree, or PHATE) so cells are positioned in two- or three-dimensional space. Convert the processed object with the cellexalvrR package: install via devtools::install_github("sonejilab/cellexalvrr") under R >= 4.0.0, then run as_cellexalvrR on the Seurat or AnnData object to produce a cellexalvrR object. Export that object with export2cellexalvr to write the CellexalVR input files: gene-expression values as an sqlite3 database of all non-zero expression values for all cells and genes, and DR coordinates as 4-column text (.mds) files. Place the exported files inside a sensibly named subfolder of the Data folder next to CellexalVR.exe, and set RscriptFilePath in Config/config.xml if running the VR client. For RNA velocity visualization, additionally supply coordinates denoting each cell's destination alongside the DR coordinates.

## Related tools

- **cellexalvrR** (R package that converts Seurat/AnnData objects via as_cellexalvrR and writes CellexalVR input files via export2cellexalvr) — https://github.com/sonejilab/cellexalvrR
- **Seurat** (User-preferred R tool for processing the single-cell data before conversion)
- **SCANPY** (User-preferred Python tool for processing the single-cell data into AnnData objects before conversion)
- **CellexalVR** (VR platform that consumes the exported input files from a sensibly named dataset folder inside the Data folder) — https://github.com/sonejilab/cellexalvr
- **R** (Runtime (>= 4.0.0) hosting cellexalvrR; Rscript path must be configured in Config/config.xml for the VR client)

## Examples

```
install.packages("devtools"); devtools::install_github("sonejilab/cellexalvrr"); library(cellexalvrR); obj <- as_cellexalvrR(seurat_obj); export2cellexalvr(obj, "Data/mouse_hspc")
```

## Evaluation signals

- The exported R object (cellexalvrR object) loads without error in R >= 4.0.0
- The sqlite3 expression database can be queried and contains the expected number of non-zero expression entries across all cells and all genes
- Each DR coordinate (.mds) file has the 4-column structure: cell identifier followed by x/y/z values
- Dataset files sit in a sensibly named subfolder inside the CellexalVR Data folder, matching the documented folder layout (Config, Data/data_set_N, CellexalVR.exe)
- config.xml RscriptFilePath points to a valid Rscript.exe under an installed R 4.0.0+ so the VR client can invoke the R back end

## Limitations

- Requires an R >= 4.0.0 installation and devtools (and pandoc for HTML reports); cellexalvrR is productively used under R 4.0.2 and 4.1.0
- Backend calculations triggered after export (e.g. identifying differentially expressed genes) take longer depending on how many cells/groups are selected, so very large datasets slow in-session analysis
- RNA velocity visualization requires supplying destination coordinates for each cell in addition to the DR coordinates; without them this feature is unavailable

## Evidence

- [results] Those using Seurat and AnnData objects can use our "as_cellexalvrR" function which will produce a cellexalvrR object: [evidence span withheld — anchor status: not_found]
- [results] the resulting object is converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR: "the resulting object is converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR"
- [intro] A common step in the process is dimension reduction (DR) where cells are positioned in two/three dimensional space: "A common step in the process is dimension reduction (DR) where cells are positioned in two/three dimensional space to visualize heterogeneity within the assayed populations."
- [readme] Install the CellexalVR R package by running `devtools::install_github("sonejilab/cellexalvrr")` in an R terminal.: "Install the [CellexalVR R package](https://www.github.com/sonejilab/cellexalvrr) by running `devtools::install_github("sonejilab/cellexalvrr")` in an R terminal."
- [discussion] Mouse HSPCNestorowa et al. (2016)GEO: GSE81682: "Mouse HSPCNestorowa et al. (2016)GEO: GSE81682"
