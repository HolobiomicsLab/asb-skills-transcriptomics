---
name: dimensionality-reduction-coordinate-export
description: Use when a processed scRNA-seq object (Seurat or AnnData) with 2D/3D
  DR embeddings (UMAP, tSNE, PHATE, DDRTree) must be converted into CellexalVR-compatible
  input files for VR visualization.
when_to_use_negative:
- Input is raw sequencing data (FASTQ) that has not yet been processed with Seurat/Scanpy
  — DR coordinates and a processed object are prerequisites
- The target platform is not CellexalVR (e.g. exporting for a conventional 2D viewer)
  — this pipeline produces CellexalVR-specific input files
- Only a 2D embedding exists and no 3D projection can be computed — CellexalVR's value
  derives from 3D projections, and .mds files require x/y/z coordinates
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: cellexalvrR
  role: R package providing as_cellexalvrR (converts Seurat/AnnData objects to a cellexalvrR
    object) and export2cellexalvr (writes CellexalVR input files including the sqlite3
    expression database and .mds coordinate files)
  repo: https://github.com/sonejilab/cellexalvrR
- name: Seurat
  role: User-side processing of single-cell data to produce the Seurat object that
    as_cellexalvrR accepts as input
- name: SCANPY
  role: User-side processing of single-cell data to produce the AnnData object that
    as_cellexalvrR accepts as input
- name: CellexalVR
  role: VR platform whose InputReader loads the exported .mds coordinate files and
    expression database from the Data folder for visualization
  repo: https://github.com/sonejilab/cellexalvr
- name: scvelo
  role: Computes per-cell RNA velocity destination coordinates that extend .mds exports
    from 4 columns to 7 columns for animated velocity visualization
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:3b36b782e7c5f757ce2b1175ee3b30f6bae29a56196079e278b8152702c0611f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvrR
  tools:
  - CellexalVR
  - Python
  - R
  - SCANPY
  - Seurat
  - cellexalvrR
  - scvelo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dimensionality-reduction-coordinate-export

## Summary

Exports dimension-reduction (DR) cell coordinates and expression data from processed Seurat/AnnData single-cell objects into the CellexalVR input files using the cellexalvrR R package (as_cellexalvrR then export2cellexalvr), producing 4-column .mds coordinate files (or 7-column files when RNA velocity destinations are supplied).

## When to use

Use when a processed scRNA-seq object (Seurat or AnnData) with 2D/3D DR embeddings (UMAP, tSNE, PHATE, DDRTree) must be converted into CellexalVR-compatible input files for VR visualization.

## When NOT to use

- Input is raw sequencing data (FASTQ) that has not yet been processed with Seurat/Scanpy — DR coordinates and a processed object are prerequisites
- The target platform is not CellexalVR (e.g. exporting for a conventional 2D viewer) — this pipeline produces CellexalVR-specific input files
- Only a 2D embedding exists and no 3D projection can be computed — CellexalVR's value derives from 3D projections, and .mds files require x/y/z coordinates

## Inputs

- Processed Seurat object or Scanpy/AnnData object with gene expression matrix
- Dimension-reduction cell coordinates (2D/3D UMAP, tSNE, PHATE, or DDRTree embeddings)
- Optional: per-cell RNA velocity destination coordinates (vx, vy, vz) from scvelo spliced/unspliced velocity estimation

## Outputs

- sqlite3 database of all non-zero expression values for all cells and all genes
- 4-column .mds text files (cell identifier + x/y/z coordinates), one per DR method
- 7-column .mds trajectory files (cell ID + 3 position + 3 velocity values) when RNA velocity is enabled
- cellexalvrR R object (validates via loading in R)

## How to apply

Process the scRNA-seq data with the user's preferred tool (Seurat or Scanpy), then run the as_cellexalvrR function from the cellexalvrR package (installed via devtools::install_github("sonejilab/cellexalvrr"), productive under R 4.0.2/4.1.0) to convert the Seurat or AnnData object into a cellexalvrR object. Export that object with export2cellexalvr, which writes the gene-expression values as an sqlite3 database containing all non-zero expression values of all cells and genes, plus the DR cell coordinates as 4-column text files (.mds format) with one row per cell. For RNA velocity visualization, additionally supply a second set of per-cell destination coordinates (from scvelo velocity estimation), producing 7-column .mds files (cell ID + x/y/z + velocity vx/vy/vz). Place the exported files in a named subfolder of the CellexalVR 'Data' folder so the InputReader loads them for graph rendering.

## Related tools

- **cellexalvrR** (R package providing as_cellexalvrR (converts Seurat/AnnData objects to a cellexalvrR object) and export2cellexalvr (writes CellexalVR input files including the sqlite3 expression database and .mds coordinate files)) — https://github.com/sonejilab/cellexalvrR
- **Seurat** (User-side processing of single-cell data to produce the Seurat object that as_cellexalvrR accepts as input)
- **SCANPY** (User-side processing of single-cell data to produce the AnnData object that as_cellexalvrR accepts as input)
- **CellexalVR** (VR platform whose InputReader loads the exported .mds coordinate files and expression database from the Data folder for visualization) — https://github.com/sonejilab/cellexalvr
- **scvelo** (Computes per-cell RNA velocity destination coordinates that extend .mds exports from 4 columns to 7 columns for animated velocity visualization)

## Examples

```
library(cellexalvrR); obj <- as_cellexalvrR(seurat_obj, drRanks=list(umap3d)); export2cellexalvr(obj, "Data/data_set_1")
```

## Evaluation signals

- The exported cellexalvrR R object loads without error in R (R >= 4.0.0)
- The SQLite expression database can be queried and contains the expected non-zero expression entries for all cells and genes
- Each DR coordinate .mds file has the exact 4-column structure (cell identifier + x/y/z) with one row per cell
- Velocity-enabled .mds files contain exactly 7 columns (cell ID + 3 position + 3 velocity values), one row per cell, and are accepted by the CellexalVR InputReader
- Files reside in a named subfolder of the CellexalVR 'Data' folder (e.g. Data/data_set_1/) and the dataset appears when CellexalVR is launched

## Limitations

- Backend calculations (e.g. differentially expressed gene identification) scale with the number of cells/groups selected and take longer for large datasets
- CellexalVR requires a VR-ready workstation with a suitable GPU (NVIDIA GTX1060 or higher recommended) running Windows 10; only HTC Vive and Valve Index were used during development
- The exported .mds coordinate files must be 3D (x/y/z); the paper's motivation is that 3D projections separate cell populations better than 2D, so 2D-only exports do not leverage the platform
- cellexalvrR is described as productively used under R 4.0.2 and 4.1.0; other R versions may not be supported

## Evidence

- [results] as_cellexalvrR on Seurat/AnnData objects: [evidence span withheld — anchor status: not_found]
- [results] export via export2cellexalvr: "the resulting object is converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR"
- [results] RNA velocity requires destination coordinates: "To enable RNA velocity visualization users should supply coordinates denoting the destination of each cell alongside the coordinates of the DR method used"
- [intro] DR step rationale: "A common step in the process is dimension reduction (DR) where cells are positioned in two/three dimensional space to visualize heterogeneity within the assayed populations."
- [readme] Data folder structure requirement: "Put your data inside a new folder with a sensible name in the `Data` folder."
- [readme] cellexalvrR installation and R version: [evidence span withheld — anchor status: external_doc]
