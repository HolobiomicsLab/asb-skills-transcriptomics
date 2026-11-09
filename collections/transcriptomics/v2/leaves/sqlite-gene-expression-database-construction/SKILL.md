---
name: sqlite-gene-expression-database-construction
description: Use when a Seurat or Scanpy/AnnData object must be turned into CellexalVR-ready
  input files (expression sqlite3 DB, .mds DR coordinate files) placed in a CellexalVR
  Data folder.
when_to_use_negative:
- The goal is visualization on a conventional flat-screen monitor with existing 2D
  software rather than preparing data for the CellexalVR platform
- The data are raw reads or an unprocessed count matrix that has not yet gone through
  a scRNA-seq processing workflow (e.g. Seurat/Scanpy)
- The data are already exported CellexalVR input files sitting in a Data folder —
  no reconversion is needed
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3366
tools:
- name: cellexalvrR
  role: R package providing as_cellexalvrR to convert Seurat/AnnData objects and export2cellexalvr
    to write the CellexalVR input file set, including the expression sqlite3 database
  repo: https://github.com/sonejilab/cellexalvrR
- name: Seurat
  role: R framework whose processed object is a supported input to as_cellexalvrR
- name: SCANPY
  role: Python framework whose AnnData object is a supported input to as_cellexalvrR
- name: CellexalVR
  role: Unity-based VR application that consumes the exported file set from its Data
    folder at runtime
  repo: https://github.com/sonejilab/cellexalvr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:42f8c1cfa054ded15372bef45644fdb3f3dedc63218fe3fdbec4284813a90544
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# sqlite-gene-expression-database-construction

## Summary

Converts processed single-cell objects (Seurat or AnnData) into the CellexalVR input file set, writing non-zero gene-expression values into an sqlite3 database and dimension-reduction coordinates into 4-column .mds text files via the cellexalvrR package.

## When to use

Use when a Seurat or Scanpy/AnnData object must be turned into CellexalVR-ready input files (expression sqlite3 DB, .mds DR coordinate files) placed in a CellexalVR Data folder.

## When NOT to use

- The goal is visualization on a conventional flat-screen monitor with existing 2D software rather than preparing data for the CellexalVR platform
- The data are raw reads or an unprocessed count matrix that has not yet gone through a scRNA-seq processing workflow (e.g. Seurat/Scanpy)
- The data are already exported CellexalVR input files sitting in a Data folder — no reconversion is needed

## Inputs

- Processed single-cell expression object (Seurat object or AnnData/h5ad from Scanpy)
- Dimension-reduction cell coordinates (2D/3D UMAP, tSNE, DDRTree, PHATE, etc.)
- Optionally, RNA velocity destination coordinates for each cell

## Outputs

- sqlite3 database of all non-zero expression values for all cells and all genes
- 4-column .mds text files with cell identifier + x/y/z coordinates per dimension reduction
- A cellexalvrR R object (loadable in R)
- Dataset folder with the input files under CellexalVR's Data/<dataset_name> directory

## How to apply

Process the scRNA-seq data with the user's preferred tool (e.g. Seurat or SCANPY), then run as_cellexalvrR from the cellexalvrR package (installed via devtools::install_github("sonejilab/cellexalvrr") under R >= 4.0.0) to produce a cellexalvrR object. Call export2cellexalvr on that object to write the CellexalVR input files, including the gene-expression values as an sqlite3 database holding all non-zero expression values of all cells and all genes. Export each dimension-reduction result (UMAP, tSNE, DDRTree, etc.) as a 4-column text file (.mds format) with a cell identifier plus x/y/z coordinates, placing everything in a sensibly named subfolder of the Data folder next to CellexalVR.exe. To enable RNA velocity visualization, additionally supply coordinates denoting each cell's destination alongside the DR coordinates. Set the Rscript.exe path in Config/config.xml (RscriptFilePath tag) so the Unity application can call the R back-end during sessions.

## Related tools

- **cellexalvrR** (R package providing as_cellexalvrR to convert Seurat/AnnData objects and export2cellexalvr to write the CellexalVR input file set, including the expression sqlite3 database) — https://github.com/sonejilab/cellexalvrR
- **Seurat** (R framework whose processed object is a supported input to as_cellexalvrR)
- **SCANPY** (Python framework whose AnnData object is a supported input to as_cellexalvrR)
- **CellexalVR** (Unity-based VR application that consumes the exported file set from its Data folder at runtime) — https://github.com/sonejilab/cellexalvr

## Examples

```
devtools::install_github("sonejilab/cellexalvrr"); library(cellexalvrR); cex <- as_cellexalvrR(seurat_obj); export2cellexalvr(cex, "Data/mouse_hspc")
```

## Evaluation signals

- The exported cellexalvrR object reloads in R without error
- The sqlite3 database can be queried and contains the expected number of non-zero expression entries covering all cells and genes
- Every DR coordinate file has the 4-column structure (cell identifier + x/y/z) with one row per cell
- Files are located under Data/<sensible_dataset_name>/ next to CellexalVR.exe, matching the README folder layout (Config, Data, CellexalVR.exe)
- CellexalVR launches and can load the dataset after the RscriptFilePath in Config/config.xml points to a valid Rscript.exe

## Limitations

- Requires R >= 4.0.0 (productively used under R 4.0.2 and 4.1.0) plus the devtools package; pandoc is additionally needed for the HTML session reports
- CellexalVR runs on Windows 10 with a VR-ready workstation (e.g. NVIDIA GTX1060 or higher) and was developed with HTC Vive and Valve Index; other VR systems are not yet supported
- In multiuser mode each participant must hold a local copy of the data files — data are not transmitted over RPCs
- Backend calculations triggered during sessions scale with the number of cells/groups selected and take longer on large datasets

## Evidence

- [results] conversion of Seurat/AnnData objects via as_cellexalvrR and export via export2cellexalvr: [evidence span withheld — anchor status: not_found]
- [results] export2cellexalvr writes the input files: "the resulting object is converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR"
- [results] RNA velocity needs destination coordinates: "To enable RNA velocity visualization users should supply coordinates denoting the destination of each cell alongside the coordinates of the DR method used"
- [readme] installation command and R version requirement: "Install the [CellexalVR R package](https://www.github.com/sonejilab/cellexalvrr) by running `devtools::install_github("sonejilab/cellexalvrr")` in an R terminal."
- [readme] Data folder placement requirement: "Create a folder called `Data` where CellexalVR is installed.
* Put your data inside a new folder with a sensible name in the `Data` folder."
- [readme] R version and install prerequisites: "Make sure at least [R 4.0.0](https://cran.r-project.org/src/base/R-4/) is installed."
