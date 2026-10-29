---
name: cellexalvr-input-file-conversion
description: Use when a processed scRNA-seq object (Seurat or AnnData) with dimension-reduction
  coordinates must be exported into CellexalVR input files, including 7-column .mds
  trajectory files for RNA velocity visualization.
when_to_use_negative:
- Input is raw sequencing data that has not yet been processed with Seurat/Scanpy
  into an object carrying DR coordinates
- The target platform is not CellexalVR — the exported file formats (.mds, CellexalVR
  input files) are specific to its InputReader
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cellexalvrR
  role: R package providing as_cellexalvrR and export2cellexalvr functions to convert
    Seurat/AnnData objects into CellexalVR input files
  repo: https://github.com/sonejilab/cellexalvrR
- name: CellexalVR
  role: Target VR platform whose InputReader loads the exported files placed in the
    Data folder to create graphs and RNA velocity visualizations
  repo: https://github.com/sonejilab/cellexalvr
- name: scvelo
  role: Python tool to compute per-cell RNA velocity (spliced/unspliced velocity estimation
    and velocity graph) yielding destination coordinates for the .mds export
- name: R
  role: Language in which the conversion pipeline (cellexalvrR) runs; Rscript.exe
    path must be configured in Config/config.xml
  repo: https://cran.r-project.org/src/base/R-4/
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:a2f53b7db1337de1492cf54d2d0a93ea84e483e53283123c4186ac26f5390d8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvrR
  tools:
  - CellexalVR
  - Python
  - R
  - cellexalvrR
  - scvelo
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cellexalvr-input-file-conversion

## Summary

Converts processed single-cell objects (Seurat/AnnData) into CellexalVR input files via the cellexalvrR R package, using as_cellexalvrR and export2cellexalvr, including 7-column .mds files that pair DR coordinates with RNA velocity destination coordinates. This is the mandatory data-preparation bridge that makes scRNA-seq data loadable in CellexalVR.

## When to use

Use when a processed scRNA-seq object (Seurat or AnnData) with dimension-reduction coordinates must be exported into CellexalVR input files, including 7-column .mds trajectory files for RNA velocity visualization.

## When NOT to use

- Input is raw sequencing data that has not yet been processed with Seurat/Scanpy into an object carrying DR coordinates
- The target platform is not CellexalVR — the exported file formats (.mds, CellexalVR input files) are specific to its InputReader

## Inputs

- Seurat or AnnData object with dimension-reduction coordinates (processed scRNA-seq data)
- Per-cell velocity destination coordinates (vx, vy, vz) computed with scvelo for RNA velocity visualization
- R 4.0.0 or newer with devtools and the cellexalvrR package installed

## Outputs

- Set of CellexalVR input files exported via export2cellexalvr
- 7-column .mds trajectory files (cell ID + 3 position + 3 velocity values, one row per cell)
- Dataset subfolder placed inside the CellexalVR 'Data' folder

## How to apply

Process the single-cell data in the user's preferred tool (Seurat or Scanpy), then apply the as_cellexalvrR function to the Seurat or AnnData object to produce a cellexalvrR object, and export it with export2cellexalvr. For RNA velocity visualization, assemble a per-cell table with cell ID, the 3D DR embedding coordinates (x, y, z), and velocity destination coordinates (vx, vy, vz) computed with scvelo, then export as 7-column .mds trajectory files (cell ID + 3 position + 3 velocity columns). Place the exported files in a named subfolder of the CellexalVR 'Data' folder (one folder per dataset) so the InputReader script in the Unity project loads them and creates graphs, heatmaps, and transcription factor networks. Validate that each .mds file has exactly 7 columns with one row per cell.

## Related tools

- **cellexalvrR** (R package providing as_cellexalvrR and export2cellexalvr functions to convert Seurat/AnnData objects into CellexalVR input files) — https://github.com/sonejilab/cellexalvrR
- **CellexalVR** (Target VR platform whose InputReader loads the exported files placed in the Data folder to create graphs and RNA velocity visualizations) — https://github.com/sonejilab/cellexalvr
- **scvelo** (Python tool to compute per-cell RNA velocity (spliced/unspliced velocity estimation and velocity graph) yielding destination coordinates for the .mds export)
- **R** (Language in which the conversion pipeline (cellexalvrR) runs; Rscript.exe path must be configured in Config/config.xml) — https://cran.r-project.org/src/base/R-4/

## Examples

```
devtools::install_github("sonejilab/cellexalvrR"); cellexal_obj <- as_cellexalvrR(seurat_obj); export2cellexalvr(cellexal_obj, "Data/data_set_1/")
```

## Evaluation signals

- Each exported .mds file contains exactly 7 columns (cell ID + 3 position + 3 velocity values) with one row per cell
- CellexalVR's InputReader accepts the files and creates the expected graphs, heatmaps, and transcription factor networks from the dataset subfolder
- The as_cellexalvrR call produces a valid cellexalvrR object from Seurat or AnnData input without error
- Animated RNA velocity visualization activates on a UMAP/PHATE reduction (e.g., Liver Cell Atlas) confirming the destination coordinates were parsed
- Files are organized as one named subfolder per dataset inside the CellexalVR 'Data' folder, matching the documented folder structure

## Limitations

- Each user of a multi-user session must have a local copy of the data files; data files are not transmitted over RPCs
- cellexalvrR is productively used under R 4.0.2 and 4.1.0; Rscript path must be configured in Config/config.xml on first run
- Backend calculations scale with the number of cells/groups selected and will take longer for large datasets

## Evidence

- [readme] cellexalvrR is an R package to prep single-cell expression data for exploration/use in our virtual realitiy data analysis platform: [evidence span withheld — anchor status: external_doc]
- [results] as_cellexalvrR function produces a cellexalvrR object: [evidence span withheld — anchor status: not_found]
- [results] export using export2cellexalvr: [evidence span withheld — anchor status: not_found]
- [results] RNA velocity requires destination coordinates: "To enable RNA velocity visualization users should supply coordinates denoting the destination of each cell alongside the coordinates of the DR method used"
- [readme] Data folder structure requirement: "Create a folder called `Data` where CellexalVR is installed.
* Put your data inside a new folder with a sensible name in the `Data` folder."
