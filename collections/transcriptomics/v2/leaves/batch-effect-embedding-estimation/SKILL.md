---
name: batch-effect-embedding-estimation
description: Use when integrating multiple spatially resolved transcriptomics datasets
  (e.g. DLPFC or seqFISH sections) whose low-dimensional embeddings contain batch
  effects that must be estimated jointly with spatial clustering.
when_to_use_negative:
- Single-sample SRT data with no batch structure — the plain SC.MEB empirical-Bayes
  hidden-Markov-random-field model suffices instead of the integrated iSC.MEB extension
- Datasets where only raw counts (not low-dimensional representations) are available
  and no dimensionality reduction (e.g. runPCA with pca.method = 'APCA', npcs = 15)
  has been performed yet
- Analyses needing cell-type annotation of counts rather than joint batch-effect and
  spatial-domain estimation
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
tools:
- name: iSC.MEB
  role: Implements the hidden-Markov-random-field empirical-Bayes model whose ICM-EM
    fitting alternately estimates cluster labels and batch-related embeddings; used
    via the iSC.MEB() function call
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: R
  role: Open-source language in which the iSC.MEB package (depending on Rcpp and RcppArmadillo)
    is implemented and run
  repo: https://github.com/XiaoZhangryy/iSC.MEB
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:f50940d22be26c03b48c0338cd6c4f28ffc145d7bffb6014bb063d346a94e83e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - R
  - iSC.MEB
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# batch-effect-embedding-estimation

## Summary

Estimate batch-effect embeddings jointly with spatial cluster labels by fitting the iSC.MEB hidden-Markov-random-field model with empirical Bayes, using the ICM-EM algorithm from the iSC.MEB R package. The batch-related embeddings are updated iteratively against cluster labels so multiple SRT datasets can be integrated despite batch effects.

## When to use

Use when integrating multiple spatially resolved transcriptomics datasets (e.g. DLPFC or seqFISH sections) whose low-dimensional embeddings contain batch effects that must be estimated jointly with spatial clustering.

## When NOT to use

- Single-sample SRT data with no batch structure — the plain SC.MEB empirical-Bayes hidden-Markov-random-field model suffices instead of the integrated iSC.MEB extension
- Datasets where only raw counts (not low-dimensional representations) are available and no dimensionality reduction (e.g. runPCA with pca.method = 'APCA', npcs = 15) has been performed yet
- Analyses needing cell-type annotation of counts rather than joint batch-effect and spatial-domain estimation

## Inputs

- iSCMEBObj created from a seuList of Seurat objects (e.g. DLPFC.rda or seqFISH.rda) with APCA embeddings (npcs = 15)
- Spatial adjacency/neighbor matrices built with CreateNeighbors (platform = 'Visium' or radius.upper = 10 for seqFISH-like platforms)

## Outputs

- Batch-related embeddings uti estimated jointly with cluster labels, stored in the iSCMEBObj iSCMEB reduction
- Fitted cluster labels yti and MBIC-selected K (saved e.g. as iscmeb_cluster_labels.rds)
- MBIC selection curve (inspectable via SelectKPlot)

## How to apply

Start from an iSCMEBObj holding per-sample APCA low-dimensional embeddings (npcs = 15) and spatial adjacency matrices from CreateNeighbors. Configure the ICM-EM algorithm with SetModelParameters (e.g. maxIter_ICM = 4, maxIter = 20, coreNum = 4) so the ICM step alternates between estimating the cluster label yti by maximizing its posterior given the batch-related embedding uti, and estimating uti by maximizing its posterior given yti; the EM part supplies the empirical-Bayes update of hyperparameters. Fit over a grid of candidate cluster numbers (e.g. K = 16:19) so the smoothness parameter beta is grid-searched, then select the model with SelectModel(criteria = 'MBIC', c_penalty = 0.2). Correctness is judged by MBIC selection stability across K, spatial coherence of cluster labels, and, where truth is available, ARI.

## Related tools

- **iSC.MEB** (Implements the hidden-Markov-random-field empirical-Bayes model whose ICM-EM fitting alternately estimates cluster labels and batch-related embeddings; used via the iSC.MEB() function call) — https://github.com/XiaoZhangryy/iSC.MEB
- **R** (Open-source language in which the iSC.MEB package (depending on Rcpp and RcppArmadillo) is implemented and run) — https://github.com/XiaoZhangryy/iSC.MEB

## Examples

```
iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum = 4); iSCMEBObj <- iSCMEB(iSCMEBObj, K = 7)
```

## Evaluation signals

- SelectModel with criteria = 'MBIC' returns a stable, well-defined minimum over the candidate K grid (e.g. K = 16:19); inspect the curve with SelectKPlot(criteria = 'MBIC')
- Fitted cluster labels (idents()) show spatially coherent domains when plotted with SpaHeatMap(plot_type = 'Scatter')
- Where reference annotations exist, ARI computed on the fitted clusters is acceptably high
- The ICM step's alternation converges within maxIter_ICM = 4 and maxIter = 20 without excessive runtime given coreNum = 4
- Downstream integration (IntegrateSpaData on the fitted iSCMEBObj) yields a merged Seurat object whose clusters are consistent across batches

## Limitations

- Requires appropriate system setup (Rtools on Windows, Xcode plus gfortran on macOS; on Linux 'ulimit -s unlimited' to avoid 'C stack usage is too close to the limit') because iSC.MEB depends on Rcpp/RcppArmadillo
- The c_penalty for MBIC needs tuning: large values (5–10) when K < 10, 0.5–2 for many clusters, 0.2 used in the seqFISH vignette — mis-tuning can skew cluster-number selection
- Conda environments may fail to install dependencies such as scater; Bioconductor/CRAN fallbacks (BiocManager, DR.SC) may be needed

## Evidence

- [abstract] permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets: "permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets"
- [methods] the function `iSC.MEB` implements the model `iSCMEB`: "This vignette provides an introduction to the R package `iSC.MEB`, where the function `iSC.MEB` implements the model `iSCMEB`"
- [methods] iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4): "iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4)"
- [readme] install the 'devtools' package: "To install the the packages "iSC.MEB", firstly, install the 'devtools' package. Besides, "iSC.MEB" depends on the 'Rcpp' and 'RcppArmadillo' package"
- [other] In the ICM step of the ICM-EM algorithm, iSC.MEB alternates between estimating the cluster label yti by maximizing its posterior given the batch-related embedding uti: [evidence span withheld — anchor status: not_found]
