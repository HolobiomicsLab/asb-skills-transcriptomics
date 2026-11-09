---
name: seurat-object-wrangling
description: Use when multiple spatially resolved transcriptomics samples are held
  as Seurat objects (e.g. DLPFC Visium sections in seuList) and must be converted
  into an iSCMEBObj with an adjacency matrix list for iSC.MEB clustering.
when_to_use_negative:
- Input is raw count matrices or plain R objects that are not Seurat objects — CreateiSCMEBObject
  expects seuList of Seurat objects, so build Seurat objects first
- Analysis targets a non-Visium platform geometry without specifying the matching
  platform argument in CreateNeighbors (e.g. seqFISH sections need appropriate handling,
  not platform='Visium')
- The goal is single-sample clustering with the original SC.MEB rather than integrated
  multi-sample analysis — iSC.MEB is the extension for multiple SRT datasets
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: iSC.MEB
  role: Provides CreateiSCMEBObject and CreateNeighbors to convert a seuList of Seurat
    objects into an iSCMEBObj with spot adjacency matrices
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: R
  role: Open-source environment in which iSC.MEB is implemented and the object wrangling
    is executed
- name: devtools
  role: Used to install iSC.MEB from GitHub before object construction (install_github)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:8ed9baa7db50411d094428cb24962c90dce959e55528e64007d771daef2c2901
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - R
  - Seurat
  - devtools
  - iSC.MEB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# seurat-object-wrangling

## Summary

Wrangles Seurat objects (seuList) into the iSC.MEB iSCMEBObj container: building the multi-sample object with CreateiSCMEBObject, controlling gene/spot filtering, and deriving spot adjacency with CreateNeighbors before any dimension reduction or spatial clustering.

## When to use

Use when multiple spatially resolved transcriptomics samples are held as Seurat objects (e.g. DLPFC Visium sections in seuList) and must be converted into an iSCMEBObj with an adjacency matrix list for iSC.MEB clustering.

## When NOT to use

- Input is raw count matrices or plain R objects that are not Seurat objects — CreateiSCMEBObject expects seuList of Seurat objects, so build Seurat objects first
- Analysis targets a non-Visium platform geometry without specifying the matching platform argument in CreateNeighbors (e.g. seqFISH sections need appropriate handling, not platform='Visium')
- The goal is single-sample clustering with the original SC.MEB rather than integrated multi-sample analysis — iSC.MEB is the extension for multiple SRT datasets

## Inputs

- seuList: list of Seurat objects, one per spatial transcriptomics sample/section (e.g. DLPFC.rda or seqFISH.rda vignette data)
- platform string (e.g. 'Visium') determining how spot coordinates are used for neighbor construction
- filtering parameters: premin.spots, postmin.spots (and optionally radius.upper for CreateNeighbors)

## Outputs

- iSCMEBObj: an iSC.MEB S4 object containing filtered gene/spot data, per-section spot adjacency matrices, and slots for low-dimensional embeddings
- Per-section adjacency matrix whose dimensions equal the number of spots of the selected section

## How to apply

Download and load the .rda vignette data (e.g. DLPFC.rda) to obtain a seuList of Seurat objects, then call CreateiSCMEBObject(seuList=seuList, verbose=FALSE, premin.spots=0, postmin.spots=0); the premin.spots/postmin.spots arguments control pre- and post-filtering of spots, and setting both to 0 retains all spots for the vignette-style single-section analysis. Next construct the spot adjacency matrix list with CreateNeighbors(iSCMEBObj, platform='Visium'), which derives neighbor relationships from the Visium spot coordinates (an optional radius.upper=10 cutoff also exists per the package's filter steps). The resulting iSCMEBObj then holds per-sample low-dimensional input slots that downstream functions (runPCA with npcs=15 and pca.method='APCA', SetModelParameters, iSCMEB with K=7) consume. Correct application means the object retains the expected number of spots per section, adjacency matrices match spot counts per section, and subsequent PCA produces an embedding with the requested number of components.

## Related tools

- **iSC.MEB** (Provides CreateiSCMEBObject and CreateNeighbors to convert a seuList of Seurat objects into an iSCMEBObj with spot adjacency matrices) — https://github.com/XiaoZhangryy/iSC.MEB
- **R** (Open-source environment in which iSC.MEB is implemented and the object wrangling is executed)
- **devtools** (Used to install iSC.MEB from GitHub before object construction (install_github))

## Examples

```
iSCMEBObj <- CreateiSCMEBObject(seuList = seuList, verbose = FALSE, premin.spots=0, postmin.spots=0); iSCMEBObj <- CreateNeighbors(iSCMEBObj, platform = "Visium")
```

## Evaluation signals

- iSCMEBObj is created without error from CreateiSCMEBObject(seuList=seuList, verbose=FALSE, premin.spots=0, postmin.spots=0) and retains all spots (filtering disabled)
- CreateNeighbors returns an adjacency matrix list; for a selected Visium section, the adjacency matrix dimensions equal that section's number of spots
- Subsequent runPCA on the object yields an embedding matrix with the requested number of columns (npcs=15)
- The seuList loaded from DLPFC.rda contains the expected Seurat objects (one per DLPFC section) before object construction
- Slot extraction from iSCMEBObj provides both a PC embedding matrix and an adjacency matrix that can be saved together (e.g. to dlpfc_pcs_adjacency.rds) and re-verified on load

## Limitations

- iSC.MEB depends on 'Rcpp' and 'RcppArmadillo', requiring proper setup of Rtools (Windows) or Xcode and gfortran (Mac OS/X); Linux users may need 'ulimit -s unlimited' to avoid 'C stack usage is too close to the limit' errors
- Conda/Bioconductor-dependent packages (e.g. scater, DR.SC) may fail to install normally and need bioconda or BiocManager fallbacks
- Spot filtering via premin.spots/postmin.spots changes which sections/spots are retained; using non-zero values diverges from the vignette setup that keeps all spots
- CreateNeighbors with platform='Visium' assumes Visium spot-coordinate geometry; other platforms need their own neighbor derivation

## Evidence

- [methods] iSCMEBObj <- CreateiSCMEBObject(seuList = seuList, verbose = FALSE, premin.spots=0, postmin.spots=0): "iSCMEBObj <- CreateiSCMEBObject(seuList = seuList, verbose = FALSE, premin.spots=0, postmin.spots=0)"
- [methods] create adjacency matrix / neighbors (CreateNeighbors): "iSCMEBObj <-  CreateNeighbors(iSCMEBObj, platform = "Visium")"
- [methods] download and load the DLPFC/seqFISH data (seuList of Seurat objects): [evidence span withheld — anchor status: not_found]
- [abstract] iSC.MEB is implemented in an open-source R package: "iSC.MEB is implemented in an open-source R package"
- [readme] iSC.MEB depends on Rcpp/RcppArmadillo and system toolchains: ""iSC.MEB" depends on the 'Rcpp' and 'RcppArmadillo' package, which also requires appropriate setting of Rtools and Xcode for Windows and Mac OS/X"
