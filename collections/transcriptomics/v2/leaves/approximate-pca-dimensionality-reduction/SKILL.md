---
name: approximate-pca-dimensionality-reduction
description: Use when an iSCMEBObj built from multiple Seurat objects (e.g. DLPFC
  Visium sections) needs a top-PC embedding (typically npcs=15) before spatial clustering
  with the hidden Markov random field model.
when_to_use_negative:
- The input is not an iSCMEBObj (e.g. raw FASTQ files or an unprocessed count matrix)
  — CreateiSCMEBObject preprocessing and adjacency construction must precede runPCA.
- The goal is batch-effect-corrected integrated embeddings rather than the initial
  low-dimensional representation — IntegrateSpaData operates on the fitted iSCMEBObj,
  not on runPCA output alone.
- A 2-D visualization embedding is needed rather than PCs for clustering — CalculateTSNE(reduction="iSCMEB",
  n_comp=2) is the appropriate step after model fitting.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: iSC.MEB
  role: R package providing runPCA with three PCA methods (classical, approximate/weighted)
    for dimensionality reduction of multi-sample SRT data prior to spatial clustering
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: R
  role: Open-source language in which iSC.MEB and its runPCA function are implemented
    and run
- name: Seurat
  role: Source format of the input seuList of Seurat objects loaded from DLPFC.rda
    that feed the iSCMEBObj used by runPCA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:b82f960aa17174185142093249fb6b88dcd6b7abfc8622295a0dd655c8660013
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - R
  - Seurat
  - iSC.MEB
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# approximate-pca-dimensionality-reduction

## Summary

Generate a low-dimensional embedding of the combined log-transformed expression matrix of multi-sample SRT data using iSC.MEB's runPCA, which offers classical PCA, approximate PCA (APCA), and weighted PCA — with APCA as the default to speed up the initial value calculation.

## When to use

Use when an iSCMEBObj built from multiple Seurat objects (e.g. DLPFC Visium sections) needs a top-PC embedding (typically npcs=15) before spatial clustering with the hidden Markov random field model.

## When NOT to use

- The input is not an iSCMEBObj (e.g. raw FASTQ files or an unprocessed count matrix) — CreateiSCMEBObject preprocessing and adjacency construction must precede runPCA.
- The goal is batch-effect-corrected integrated embeddings rather than the initial low-dimensional representation — IntegrateSpaData operates on the fitted iSCMEBObj, not on runPCA output alone.
- A 2-D visualization embedding is needed rather than PCs for clustering — CalculateTSNE(reduction="iSCMEB", n_comp=2) is the appropriate step after model fitting.

## Inputs

- iSCMEBObj object created by CreateiSCMEBObject from a seuList of Seurat objects (e.g. DLPFC.rda) with adjacency matrices built via CreateNeighbors
- Combined log-transformed expression matrix across the samples
- Parameters: npcs (e.g. 15), pca.method ('PCA', 'APCA', or 'WPCA'; default APCA)

## Outputs

- iSCMEBObj updated with a PC embedding matrix with npcs columns (e.g. 15 components) in its low-dimensional representation slot
- Per-section PC embedding matrices extractable for downstream iSC.MEB clustering (e.g. saved to an RDS file)

## How to apply

After constructing the iSCMEBObj with CreateiSCMEBObject(seuList=seuList, premin.spots=0, postmin.spots=0) and building the spot adjacency matrix with CreateNeighbors(iSCMEBObj, platform="Visium"), call runPCA(iSCMEBObj, npcs=15, pca.method="APCA") to obtain the 15-component low-dimensional representation used as input to the iSC.MEB model. Choose the method deliberately: classical PCA, approximate PCA (APCA, the default), and weighted PCA are all available; APCA is chosen to speed up the initial value calculation. For a single section, extract the PC embedding matrix (npcs columns) and the section's adjacency matrix from the iSCMEBObj slots for downstream use or saving (e.g. to an RDS file). The number of PCs (npcs) is a key parameter — the vignette uses npcs=15 for the DLPFC analysis.

## Related tools

- **iSC.MEB** (R package providing runPCA with three PCA methods (classical, approximate/weighted) for dimensionality reduction of multi-sample SRT data prior to spatial clustering) — https://github.com/XiaoZhangryy/iSC.MEB
- **R** (Open-source language in which iSC.MEB and its runPCA function are implemented and run)
- **Seurat** (Source format of the input seuList of Seurat objects loaded from DLPFC.rda that feed the iSCMEBObj used by runPCA)

## Examples

```
iSCMEBObj <- runPCA(iSCMEBObj, npcs = 15, pca.method = "APCA")
```

## Evaluation signals

- The updated iSCMEBObj contains a PC embedding matrix with exactly npcs columns (15 in the vignette).
- The extracted per-section adjacency matrix dimensions equal the number of spots of the selected section.
- pca.method is one of the three supported options ('PCA', 'APCA', 'WPCA'); omitting it yields the APCA default.
- The saved RDS file (e.g. dlpfc_pcs_adjacency.rds) exists and contains both the embedding and adjacency matrices.
- Downstream iSC.MEB model fitting (iSCMEB(iSCMEBObj, K=7)) runs without error on the computed PCs.

## Limitations

- APCA is an approximation chosen to speed up the initial value calculation, not an exact PCA decomposition.
- runPCA requires prior gene/spot filtering and adjacency construction via CreateiSCMEBObject and CreateNeighbors; it cannot be run standalone on raw seuList data.
- Installation of iSC.MEB requires Rcpp/RcppArmadillo and system setup (Rtools on Windows, Xcode on macOS); on Linux, 'ulimit -s unlimited' may be needed to avoid 'R Error: C stack usage is too close to the limit'.

## Evidence

- [methods] The runPCA function generates top PCs with three optional methods — classical PCA, approximate PCA, and weighted PCA — and the approximate PCA is the default, chosen to speed up the initial value calculation.: [evidence span withheld — anchor status: not_found]
- [methods] iSCMEBObj <-  runPCA(iSCMEBObj, npcs = 15, pca.method = "APCA"): "iSCMEBObj <-  runPCA(iSCMEBObj, npcs = 15, pca.method = "APCA")"
- [abstract] permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets: "permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets"
- [readme] For parallel computation on Linux, users must use the following system command to set the C_stack unlimited in case of the error `R Error: C stack usage is too close to the limit`.: [evidence span withheld — anchor status: external_doc]
