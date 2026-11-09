---
name: multi-sample-srt-data-integration
description: Use when two or more SRT sections (e.g. Visium DLPFC sections or seqFISH
  mouse embryo sections) across batches, individuals, or conditions must be jointly
  clustered while correcting batch effects.
when_to_use_negative:
- Only a single SRT section is available — the predecessor SC.MEB covers single-sample
  empirical-Bayes spatial clustering
- Input is already-integrated expression matrices without spatial coordinates, since
  adjacency matrices from spot positions are required
- Datasets have no shared cell/domain clusters at all, outside the 'partially shared
  clusters' context the package targets
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: iSC.MEB
  role: R package implementing the integrated hidden Markov random field with empirical
    Bayes for multi-sample spatial clustering and batch-effect estimation
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: SC.MEB
  role: Predecessor single-sample empirical-Bayes hidden-Markov-random-field method
    that iSC.MEB extends
- name: Seurat
  role: Provides the per-sample Seurat objects (seuList) consumed by CreateiSCMEBObject
- name: R
  role: Language in which iSC.MEB is implemented and run; depends on Rcpp and RcppArmadillo
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:5fe3f6056f354efea25b20aae2b470b61e89c3d795f4423498a4883180d6717b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - R
  - SC.MEB
  - Seurat
  - iSC.MEB
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# multi-sample-srt-data-integration

## Summary

Integrate multiple spatially resolved transcriptomics (SRT) datasets with iSC.MEB, which simultaneously estimates batch effects and performs spatial clustering via a hidden Markov random field fitted with empirical Bayes on low-dimensional representations of each sample.

## When to use

Use when two or more SRT sections (e.g. Visium DLPFC sections or seqFISH mouse embryo sections) across batches, individuals, or conditions must be jointly clustered while correcting batch effects.

## When NOT to use

- Only a single SRT section is available — the predecessor SC.MEB covers single-sample empirical-Bayes spatial clustering
- Input is already-integrated expression matrices without spatial coordinates, since adjacency matrices from spot positions are required
- Datasets have no shared cell/domain clusters at all, outside the 'partially shared clusters' context the package targets

## Inputs

- seuList: list of Seurat objects, one per SRT section (e.g. DLPFC.rda or seqFISH.rda from the iSC.MEB vignettes_data)
- Per-sample spot coordinates (e.g. Visium platform) for adjacency construction
- Optional manual layer/cell-type annotations for ARI evaluation

## Outputs

- iSCMEBObj with fitted cluster labels across all samples (accessed with idents())
- Batch-effect embeddings (uti) and integrated low-dimensional representation per spot
- MBIC selection curve over candidate K values
- Integrated Seurat object via IntegrateSpaData (e.g. seuInt <- IntegrateSpaData(iSCMEBObj, 'mouse'))

## How to apply

Assemble per-sample Seurat objects into a seuList and build an iSCMEBObj with CreateiSCMEBObject (e.g. premin.spots=0, postmin.spots=0 to retain all spots in the vignette). Build spot adjacency matrices per sample with CreateNeighbors using the platform name (e.g. 'Visium'), then reduce each sample's log-transformed expression to a shared low-dimensional space with runPCA (npcs=15, pca.method='APCA', the default approximate PCA used to speed initial-value calculation). Fit the integrated hidden-Markov-random-field model with empirical Bayes via the iSCMEB function over candidate cluster numbers, configuring ICM-EM settings through SetModelParameters (maxIter_ICM=4, maxIter=20, coreNum=4) and selecting K with SelectModel using MBIC (c_penalty tuned, e.g. 0.2 in the seqFISH vignette). The model alternates ICM updates of cluster labels yti given batch embeddings uti and of uti given yti, with beta grid-searched during fitting. Judge success by MBIC-selected K, spatially coherent clusters (SpaHeatMap), and ARI against manual annotations.

## Related tools

- **iSC.MEB** (R package implementing the integrated hidden Markov random field with empirical Bayes for multi-sample spatial clustering and batch-effect estimation) — https://github.com/XiaoZhangryy/iSC.MEB
- **SC.MEB** (Predecessor single-sample empirical-Bayes hidden-Markov-random-field method that iSC.MEB extends)
- **Seurat** (Provides the per-sample Seurat objects (seuList) consumed by CreateiSCMEBObject)
- **R** (Language in which iSC.MEB is implemented and run; depends on Rcpp and RcppArmadillo)

## Examples

```
iSCMEBObj <- CreateiSCMEBObject(seuList=seuList, verbose=FALSE, premin.spots=0, postmin.spots=0); iSCMEBObj <- CreateNeighbors(iSCMEBObj, platform="Visium"); iSCMEBObj <- runPCA(iSCMEBObj, npcs=15, pca.method="APCA"); iSCMEBObj <- iSCMEB(iSCMEBObj, K=7); iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty=0.2)
```

## Evaluation signals

- runPCA output embedding matrix has npcs=15 columns per sample; adjacency matrix dimensions equal the number of spots of each section
- SelectModel with criteria='MBIC' returns a selected K consistent with the minimum of the MBIC selection curve (inspect with SelectKPlot)
- Fitted cluster labels form spatially contiguous domains on each section (SpaHeatMap scatter plot) rather than salt-and-pepper noise
- ARI against manual DLPFC layer annotations is reported and non-trivially above chance
- IntegrateSpaData returns an integrated Seurat object whose cell count equals the summed spots across input samples

## Limitations

- Installation requires C++ compilation toolchains: Rcpp/RcppArmadillo with Rtools (Windows), Xcode and gfortran (Mac), and 'ulimit -s unlimited' on Linux to avoid 'C stack usage is too close to the limit' errors during parallel computation
- Conda users may need bioconda (e.g. bioconductor-scater) or BiocManager installs for dependent packages such as scater and DR.SC
- c_penalty for MBIC must be tuned: large values (5–10) when K < 10, 0.5–2 for many clusters, 0.2 used in the seqFISH vignette — mis-setting it can skew cluster-number selection

## Evidence

- [abstract] iSC.MEB extends SC.MEB to simultaneously estimate batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets: "permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets"
- [intro] Multi-sample scope of iSC.MEB: "iSC.MEB is a package for integrating and analyzing multiple spatially resolved transcriptomics (SRT) datasets"
- [results] runPCA options and APCA default: [evidence span withheld — anchor status: not_found]
- [results] ICM-EM alternating updates of labels and embeddings: [evidence span withheld — anchor status: not_found]
- [readme] Installation from GitHub: "install.packages("devtools")
library(devtools)
install_github("XiaoZhangryy/iSC.MEB")"
- [readme] Linux C-stack fix for parallel computation: [evidence span withheld — anchor status: external_doc]
