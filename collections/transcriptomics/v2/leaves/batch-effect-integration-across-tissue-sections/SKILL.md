---
name: batch-effect-integration-across-tissue-sections
description: Use when analyzing multiple SRT tissue sections (e.g. six mouse embryo
  seqFISH sections or twelve human DLPFC sections) where cross-sample batch effects
  must be estimated simultaneously with spatial domain clustering, and integrated
  clustering metrics are needed.
when_to_use_negative:
- Single-section SRT data with no batches to integrate — the SC.MEB model (which iSC.MEB
  extends) is the appropriate one-sample choice
- Non-spatial single-cell data without coordinates, since CreateNeighbors requires
  spatial locations to build the adjacency matrix
- Benchmarks where only pre-corrected (batch-removed) expression matrices are available,
  since iSC.MEB is designed to estimate batch effects itself from the multi-section
  input
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: iSC.MEB
  role: 'Core package: fits the integrated spatial clustering hidden Markov random
    field with empirical Bayes over multiple SRT sections'
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: SC.MEB
  role: Single-sample predecessor method whose extension iSC.MEB implements; used
    as a benchmark comparison on the same seqFISH data
- name: Seurat
  role: Section-level input containers (Seurat objects in seuList) consumed by CreateiSCMEBObject
- name: mclust
  role: adjustedRandIndex used to compute per-section and integrated ARI against celltype_mapped_refined
    labels
- name: BASS
  role: Primary benchmark method for multi-sample spatial clustering to compare integrated
    ARI/NMI against
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:3330e4a8321e263534ca0a2a2a48835f36652dd80bc860df688f41205e8c9cdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - BASS
  - BayesSpace
  - R
  - SC.MEB
  - Seurat
  - SpaGCN
  - iSC.MEB
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# batch-effect-integration-across-tissue-sections

## Summary

Jointly estimate batch effects and perform spatial clustering on low-dimensional representations of multiple spatially resolved transcriptomics (SRT) sections using iSC.MEB, a hidden Markov random field model fitted with empirical Bayes. It enables domain detection across sections while integrating samples that may share only partially overlapping clusters.

## When to use

Use when analyzing multiple SRT tissue sections (e.g. six mouse embryo seqFISH sections or twelve human DLPFC sections) where cross-sample batch effects must be estimated simultaneously with spatial domain clustering, and integrated clustering metrics are needed.

## When NOT to use

- Single-section SRT data with no batches to integrate — the SC.MEB model (which iSC.MEB extends) is the appropriate one-sample choice
- Non-spatial single-cell data without coordinates, since CreateNeighbors requires spatial locations to build the adjacency matrix
- Benchmarks where only pre-corrected (batch-removed) expression matrices are available, since iSC.MEB is designed to estimate batch effects itself from the multi-section input

## Inputs

- List of Seurat objects (seuList), one per SRT tissue section (e.g. seqFISH.rda or DLPFC.rda)
- Custom gene list shared across sections
- Spatial coordinates per section for adjacency construction

## Outputs

- iSCMEBObj with fitted cluster labels per spot across all sections
- Selected number of clusters K (via MBIC with c_penalty=0.2)
- Integrated/per-section ARI and NMI (mclust::adjustedRandIndex) against reference labels
- Integrated Seurat object via IntegrateSpaData for downstream visualization (SpaHeatMap, doHeatmap)

## How to apply

Load the preprocessed sections as a list of Seurat objects (e.g. seqFISH.rda from the iSC.MEB GitHub vignettes_data directory) and build an iSCMEBObj with CreateiSCMEBObject using a custom gene list. Construct L2-distance-based adjacency matrices with CreateNeighbors (radius.upper=10) and reduce dimensionality with runPCA using npcs=15 and pca.method='APCA'. Configure fitting via SetModelParameters (maxIter_ICM=4, maxIter=20, coreNum=4), fit the hidden Markov random field over a candidate cluster range (e.g. K=16:19 for seqFISH), and select the optimal K with SelectModel using criteria='MBIC' and c_penalty=0.2. Rationale: iSC.MEB extends SC.MEB so batch effects are estimated jointly with clustering rather than corrected as a separate step, which is what enables integrated multi-sample domain detection.

## Related tools

- **iSC.MEB** (Core package: fits the integrated spatial clustering hidden Markov random field with empirical Bayes over multiple SRT sections) — https://github.com/XiaoZhangryy/iSC.MEB
- **SC.MEB** (Single-sample predecessor method whose extension iSC.MEB implements; used as a benchmark comparison on the same seqFISH data)
- **Seurat** (Section-level input containers (Seurat objects in seuList) consumed by CreateiSCMEBObject)
- **mclust** (adjustedRandIndex used to compute per-section and integrated ARI against celltype_mapped_refined labels)
- **BASS** (Primary benchmark method for multi-sample spatial clustering to compare integrated ARI/NMI against)

## Examples

```
iSCMEBObj <- CreateiSCMEBObject(seuList = seuList, verbose = FALSE, premin.spots = 0, postmin.spots = 0); iSCMEBObj <- runPCA(iSCMEBObj, npcs = 15, pca.method = "APCA")
```

## Evaluation signals

- Integrated ARI/NMI computed with mclust::adjustedRandIndex against celltype_mapped_refined labels; on the six-section seqFISH dataset expect iSC.MEB to recover approximately (0.52, 0.69) versus BASS's (0.45, 0.66)
- iSC.MEB should attain the highest median ARI and NMI among benchmarked methods (SC.MEB, BASS, BayesSpace, SpaGCN) across the six sections
- SelectModel with criteria='MBIC' and c_penalty=0.2 should select a stable K within the searched range (e.g. 16:19 for seqFISH)
- Fitted cluster labels should be spatially coherent per section, verifiable with SpaHeatMap scatter plots of the cluster item
- CreateNeighbors must produce valid L2-distance-based adjacency matrices (radius.upper=10) for all six sections before fitting

## Limitations

- Compilation dependencies Rcpp and RcppArmadillo require Rtools (Windows) or Xcode and gfortran (Mac OS/X) to be set up; without proper setup installation fails
- On Linux, parallel computation can fail with 'R Error: C stack usage is too close to the limit'; the ulimit -s unlimited system command must be run first
- In conda environments some dependent packages (e.g. scater) may fail to install and must be pulled from bioconda or Bioconductor separately
- Benchmark scope covered seqFISH (6 mouse embryo sections) and DLPFC (12 human postmortem sections) with Visium/seqFISH-style data; results depend on the preprocessing (APCA, npcs=15) used in the vignette

## Evidence

- [abstract] permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets: "permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets"
- [intro] iSC.MEB is a package for integrating and analyzing multiple spatially resolved transcriptomics (SRT) datasets: "iSC.MEB is a package for integrating and analyzing multiple spatially resolved transcriptomics (SRT) datasets"
- [methods] The seqFISH dataset contains the high-resolution spatial map from 6 mouse embryo tissue sections generated by a modified version of the seqFISH: "The seqFISH dataset contains the high-resolution spatial map from 6 mouse embryo tissue sections generated by a modified version of the seqFISH"
- [readme] iSC.MEB can be used to compare and contrast experimental datasets in a variety of contexts, for instance: Across experimental batches, Across individuals: "iSC.MEB can be used to compare and contrast experimental datasets in a variety of contexts, for instance: Across experimental batches"
- [readme] For parallel computation on Linux, users must use the following system command to set the C_stack unlimited in case of the error `R Error: C stack usage is too close to the limit`.: [evidence span withheld — anchor status: external_doc]
