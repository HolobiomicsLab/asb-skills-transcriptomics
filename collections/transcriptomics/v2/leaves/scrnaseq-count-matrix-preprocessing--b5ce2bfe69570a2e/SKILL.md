---
name: scrnaseq-count-matrix-preprocessing
description: Use when raw scRNA-seq count data (e.g. GEO accession like GSE81682 mouse
  HSPC) must be normalized and converted with cellexalvrR into CellexalVR-consumable
  input files.
when_to_use_negative:
- The data are already exported CellexalVR input files sitting in the Data folder
  — re-preprocessing is unnecessary
- The analysis goal is 3D overlap/entropy comparison of existing dimension reductions
  (entropy_2dvs3d / 2d_vs_3d tooling), not preparing expression data
- The input is not single-cell gene expression data (e.g. CITE-seq protein abundance
  alone), which the as_cellexalvrR conversion path does not target
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Normalizes and processes the raw count matrix into a Seurat object before
    conversion
- name: cellexalvrR
  role: Provides as_cellexalvrR to convert Seurat/AnnData objects and export2cellexalvr
    to write CellexalVR input files
  repo: https://github.com/sonejilab/cellexalvrR
- name: R
  role: Runtime for the preprocessing pipeline; must be at least 4.0.0 with Rscript.exe
    configured in Config/config.xml
- name: CellexalVR
  role: Consumes the exported input files from the Data folder for visualization and
    in-session analysis
  repo: https://github.com/sonejilab/cellexalvr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:a37afcf087d4e57f64954b651cfbea8caacba65ef6aedefdfc1c549e2310c20a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvrR
  tools:
  - CellexalVR
  - R
  - Seurat
  - cellexalvrR
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scrnaseq-count-matrix-preprocessing

## Summary

Prepares raw single-cell RNA-seq expression data (e.g. GEO-deposited mouse HSPC counts such as GSE81682 from Nestorowa et al., 2016) into the normalized expression matrices and CellexalVR input files required by downstream in-session analyses like slingshot pseudotime.

## When to use

Use when raw scRNA-seq count data (e.g. GEO accession like GSE81682 mouse HSPC) must be normalized and converted with cellexalvrR into CellexalVR-consumable input files.

## When NOT to use

- The data are already exported CellexalVR input files sitting in the Data folder — re-preprocessing is unnecessary
- The analysis goal is 3D overlap/entropy comparison of existing dimension reductions (entropy_2dvs3d / 2d_vs_3d tooling), not preparing expression data
- The input is not single-cell gene expression data (e.g. CITE-seq protein abundance alone), which the as_cellexalvrR conversion path does not target

## Inputs

- Raw single-cell expression counts from GEO (e.g. GSE81682 mouse HSPC, Nestorowa et al. 2016)
- Seurat object or AnnData object with a normalized expression matrix
- R environment >= 4.0.0 with the cellexalvrR package installed via devtools

## Outputs

- A cellexalvrR object produced by as_cellexalvrR
- A set of CellexalVR input files written by export2cellexalvr into a Data/<data_set> folder
- A normalized expression matrix suitable for downstream slingshot pseudotime and heatmap steps

## How to apply

Download the raw expression data (e.g. GSE81682 mouse HSPC from GEO) and process it with a user-chosen tool such as Seurat to obtain a normalized expression matrix. Convert the processed object using the cellexalvrR 'as_cellexalvrR' function, which accepts Seurat and AnnData objects and produces a cellexalvrR object. Export the result with 'export2cellexalvr' so the backend R scripts that perform in-session calculations can consume it. Place the exported files in a named subfolder of the 'Data' folder next to CellexalVR.exe, and ensure the RscriptFilePath tag in Config/config.xml points to Rscript.exe from R 4.0.0 or newer, since cellexalvrR is productively used under R 4.0.2 and 4.1.0. This preprocessing stage is a prerequisite for any subsequent in-session step (clustering, Wilcox-test heatmaps with the default top 250 marker genes, or four-cluster slingshot pseudotime with ten colour groups).

## Related tools

- **Seurat** (Normalizes and processes the raw count matrix into a Seurat object before conversion)
- **cellexalvrR** (Provides as_cellexalvrR to convert Seurat/AnnData objects and export2cellexalvr to write CellexalVR input files) — https://github.com/sonejilab/cellexalvrR
- **R** (Runtime for the preprocessing pipeline; must be at least 4.0.0 with Rscript.exe configured in Config/config.xml)
- **CellexalVR** (Consumes the exported input files from the Data folder for visualization and in-session analysis) — https://github.com/sonejilab/cellexalvr

## Examples

```
devtools::install_github("sonejilab/cellexalvrR"); cello <- as_cellexalvrR(seurat_obj); export2cellexalvr(cello, "Data/data_set_1")
```

## Evaluation signals

- export2cellexalvr completes without error and writes data files into Data/<data_set_name>/ with the folder structure CellexalVR expects (Config/, Data/, CellexalVR.exe)
- The resulting normalized expression matrix matches the cell/gene dimensions of the source object (e.g. the Nestorowa et al. mouse HSPC cells from GSE81682)
- as_cellexalvrR accepts the Seurat object without complaining about missing expression/metadata slots, yielding a valid cellexalvrR object
- CellexalVR loads the data set from the Data folder on session start, confirming the input files are correctly formatted
- RscriptFilePath in Config/config.xml resolves to a valid Rscript.exe of R >= 4.0.0 so backend scripts can run

## Limitations

- cellexalvrR is only reported as productively used under R 4.0.2 and 4.1.0; other R versions are untested in the provided context
- Backend calculations on the converted data scale with the number of cells/groups selected and take longer for large datasets
- Running the converted data in CellexalVR itself requires a VR-ready workstation (NVIDIA GTX1060 or higher recommended) running Windows 10, with HTC Vive or Valve Index during development
- The R package must be installed with devtools::install_github, requiring the devtools dependency; pandoc is additionally needed for HTML session reports

## Evidence

- [results] Single-cell data is processed using any method preferred by the user (Seurat/Scanpy for example): "Single-cell data is processed using any method preferred by the user (Seurat/Scanpy for example)"
- [results] Those using Seurat and AnnData objects can use our "as_cellexalvrR" function which will produce a cellexalvrR object: [evidence span withheld — anchor status: not_found]
- [readme] cellexalvrR is productively used under R 4.0.2 and 4.1.0.: [evidence span withheld — anchor status: external_doc]
- [readme] cellexalvrR is an R package to prep single-cell expression data for exploration/use in our virtual realitiy data analysis platform: [evidence span withheld — anchor status: external_doc]
- [results] the resulting object is converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR: "the resulting object is converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR"
