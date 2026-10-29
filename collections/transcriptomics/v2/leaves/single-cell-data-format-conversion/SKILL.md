---
name: single-cell-data-format-conversion
description: Use when a processed scRNA-seq object (Seurat or AnnData) must be made
  loadable in CellexalVR, which requires exported CellexalVR input files rather than
  native R/Python object formats.
when_to_use_negative:
- The input is raw sequencing data (FASTQ etc.) rather than a processed Seurat/AnnData
  object — process it first with Seurat or SCANPY
- The target platform is not CellexalVR; this skill produces CellexalVR-specific input
  files only
- The environment lacks R >= 4.0.0 (cellexalvrR is productively used under R 4.0.2
  and 4.1.0)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cellexalvrR
  role: R package providing as_cellexalvrR and export2cellexalvr to convert Seurat/AnnData
    objects into CellexalVR input files
  repo: https://github.com/sonejilab/cellexalvrR
- name: Seurat
  role: Source processed single-cell object format accepted by as_cellexalvrR
- name: SCANPY
  role: Source processed AnnData object format accepted by as_cellexalvrR
- name: CellexalVR
  role: Target platform that consumes the exported input files from a dataset folder
    under its Data directory
  repo: https://github.com/sonejilab/cellexalvr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:6eebf2fd504df4cdd806393283fab7dac4c76047f213ba91b6c906bdc2954ca3
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# single-cell-data-format-conversion

## Summary

Converts processed single-cell objects (Seurat or Scanpy/AnnData) into the input files required by CellexalVR — an SQLite expression database plus 4-column .mds dimension-reduction coordinate files — using the cellexalvrR R package.

## When to use

Use when a processed scRNA-seq object (Seurat or AnnData) must be made loadable in CellexalVR, which requires exported CellexalVR input files rather than native R/Python object formats.

## When NOT to use

- The input is raw sequencing data (FASTQ etc.) rather than a processed Seurat/AnnData object — process it first with Seurat or SCANPY
- The target platform is not CellexalVR; this skill produces CellexalVR-specific input files only
- The environment lacks R >= 4.0.0 (cellexalvrR is productively used under R 4.0.2 and 4.1.0)

## Inputs

- Processed Seurat object (e.g. from GSE81682 mouse HSPC scRNA-seq data)
- Processed Scanpy/AnnData object
- Dimension-reduction cell coordinates (e.g. UMAP, tSNE, DDRTree) embedded in the object

## Outputs

- cellexalvrR object (intermediate, loadable in R)
- SQLite3 database of all non-zero expression values for all cells and all genes
- 4-column .mds text files with cell identifier and x/y/z coordinates, in a dataset folder under the Data directory

## How to apply

First install the R package with devtools::install_github('sonejilab/cellexalvrr') under R >= 4.0.0. Run as_cellexalvrR on the Seurat or AnnData object to produce a cellexalvrR object, then call export2cellexalvr on that object to write the CellexalVR input files: gene-expression values are written as an sqlite3 database containing all non-zero expression values of all cells and all genes, and dimension-reduction cell coordinates are written as 4-column text (.mds) files with cell identifier plus x/y/z. Place the exported files in a sensibly named subfolder of the Data folder where CellexalVR is installed. Validate by checking the R object reloads, the SQLite database queries with the expected non-zero entries, and each DR file has the 4-column structure.

## Related tools

- **cellexalvrR** (R package providing as_cellexalvrR and export2cellexalvr to convert Seurat/AnnData objects into CellexalVR input files) — https://github.com/sonejilab/cellexalvrR
- **Seurat** (Source processed single-cell object format accepted by as_cellexalvrR)
- **SCANPY** (Source processed AnnData object format accepted by as_cellexalvrR)
- **CellexalVR** (Target platform that consumes the exported input files from a dataset folder under its Data directory) — https://github.com/sonejilab/cellexalvr

## Examples

```
library(cellexalvrR); obj <- as_cellexalvrR(seurat_obj); export2cellexalvr(obj, "Data/GSE81682_HSPC")
```

## Evaluation signals

- The exported cellexalvrR object loads successfully in R
- The SQLite database can be queried and contains the expected non-zero expression entries for all cells and genes
- Each dimension-reduction coordinate file has the 4-column structure: cell identifier plus x/y/z
- Exported files reside in a sensibly named dataset folder inside the Data folder alongside CellexalVR.exe (Config/config.xml RscriptFilePath set correctly)

## Limitations

- cellexalvrR requires R >= 4.0.0 (productively used under R 4.0.2 and 4.1.0) and installation via devtools::install_github
- HTML report generation additionally requires pandoc to be installed
- Expression is exported as non-zero values only; the SQLite database deliberately excludes zero expression entries
- If RNA velocity visualization is desired, destination coordinates for each cell must be supplied alongside the DR coordinates during export

## Evidence

- [results] as_cellexalvrR and export2cellexalvr workflow: [evidence span withheld — anchor status: not_found]
- [results] conversion to CellexalVR input files: "the resulting object is converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR"
- [readme] installation command: "Install the [CellexalVR R package](https://www.github.com/sonejilab/cellexalvrr) by running `devtools::install_github("sonejilab/cellexalvrr")` in an R terminal."
- [readme] Data folder structure: "Put your data inside a new folder with a sensible name in the `Data` folder."
- [results] RNA velocity coordinates requirement: "To enable RNA velocity visualization users should supply coordinates denoting the destination of each cell alongside the coordinates of the DR method used"
