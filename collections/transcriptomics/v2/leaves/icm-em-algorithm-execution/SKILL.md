---
name: icm-em-algorithm-execution
description: Use when multiple spatially resolved transcriptomics datasets (e.g. Visium
  or seqFISH sections) have APCA low-dimensional embeddings and spatial adjacency
  matrices already built, and you need joint spatial clustering with batch-effect
  estimation and a data-driven choice of cluster number.
when_to_use_negative:
- You have a single SRT sample with no batch structure — the simpler SC.MEB empirical
  Bayes model is the predecessor designed for that setting
- Input data has not yet been reduced to low-dimensional embeddings and spatial neighbors
  (no APCA/PCA representation or adjacency matrix) — the model operates on these,
  not raw counts
- You need marker-gene differential expression rather than clustering; those come
  after integration via doDEG on the IntegrateSpaData output
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: iSC.MEB
  role: R package implementing the iSCMEB hidden Markov random field model with ICM-EM
    inference and MBIC model selection; fitted via the iSCMEB() and SelectModel()
    functions
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: SC.MEB
  role: Empirical Bayes hidden Markov random field predecessor that iSC.MEB extends
    to multi-sample batch-effect estimation
- name: R
  role: Open-source language in which iSC.MEB is implemented
- name: Rcpp
  role: C++ dependency required to install iSC.MEB
- name: RcppArmadillo
  role: Linear-algebra dependency required to install iSC.MEB
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:d1bb397aaaf4adf029540f8c4eb6ab6ef5c1c657c849029aaaa17b0862e94023
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - R
  - Rcpp
  - RcppArmadillo
  - SC.MEB
  - iSC.MEB
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# icm-em-algorithm-execution

## Summary

Fit the iSC.MEB hidden-Markov-random-field model with empirical Bayes over a grid of candidate cluster numbers, using ICM-EM inference that alternates cluster-label and batch-embedding posterior updates, then select K by MBIC. This estimates spatial cluster labels while simultaneously correcting batch effects across multiple SRT samples.

## When to use

Use when multiple spatially resolved transcriptomics datasets (e.g. Visium or seqFISH sections) have APCA low-dimensional embeddings and spatial adjacency matrices already built, and you need joint spatial clustering with batch-effect estimation and a data-driven choice of cluster number.

## When NOT to use

- You have a single SRT sample with no batch structure — the simpler SC.MEB empirical Bayes model is the predecessor designed for that setting
- Input data has not yet been reduced to low-dimensional embeddings and spatial neighbors (no APCA/PCA representation or adjacency matrix) — the model operates on these, not raw counts
- You need marker-gene differential expression rather than clustering; those come after integration via doDEG on the IntegrateSpaData output

## Inputs

- iSCMEBObj created by CreateiSCMEBObject from a seuList of Seurat objects
- Spatial adjacency matrices built by CreateNeighbors (platform e.g. "Visium", radius.upper=10)
- APCA low-dimensional embeddings from runPCA(npcs = 15, pca.method = "APCA")

## Outputs

- Fitted cluster labels per spot (retrievable with idents()) for the MBIC-selected K
- MBIC selection curve over the candidate K grid (SelectKPlot)
- Estimated batch-related embeddings uti and smoothness parameter beta
- Saved cluster labels, e.g. iscmeb_cluster_labels.rds

## How to apply

Load the iSCMEBObj containing the prior APCA embeddings (npcs = 15) and adjacency matrices from CreateNeighbors, then configure ICM-EM settings with SetModelParameters (maxIter_ICM = 4, maxIter = 20, coreNum = 4). Fit the model with the iSCMEB function over a grid of candidate cluster numbers (e.g. K = 16:19), which internally grid-searches the smoothness parameter beta; during each ICM step the algorithm alternates estimating the cluster label yti by maximizing its posterior given the batch-related embedding uti, and estimating uti by maximizing its posterior given yti. Select the optimal K with SelectModel(criteria = "MBIC", c_penalty = ...), tuning c_penalty large (5–10) when K < 10 and smaller (0.5–2) for many clusters — 0.2 is used in the seqFISH vignette — and inspect the selection with SelectKPlot(criteria = "MBIC"). Extract the fitted labels with idents() and evaluate with ARI and SpaHeatMap spatial scatter plots.

## Related tools

- **iSC.MEB** (R package implementing the iSCMEB hidden Markov random field model with ICM-EM inference and MBIC model selection; fitted via the iSCMEB() and SelectModel() functions) — https://github.com/XiaoZhangryy/iSC.MEB
- **SC.MEB** (Empirical Bayes hidden Markov random field predecessor that iSC.MEB extends to multi-sample batch-effect estimation)
- **R** (Open-source language in which iSC.MEB is implemented)
- **Rcpp** (C++ dependency required to install iSC.MEB)
- **RcppArmadillo** (Linear-algebra dependency required to install iSC.MEB)

## Examples

```
iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4); iSCMEBObj <- iSCMEB(iSCMEBObj, K = 16:19); iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2)
```

## Evaluation signals

- MBIC selection curve over the candidate K grid (e.g. K = 16:19) shows a clear minimum at the selected K
- Cluster labels extracted with idents() cover all spots in all samples with no missing assignments
- ARI against known annotations (e.g. DLPFC layers) confirms accurate cell/domain detection
- SpaHeatMap scatter plot of item="cluster" shows spatially contiguous domains rather than speckled labels
- Fitting converges within the configured maxIter = 20 EM iterations and maxIter_ICM = 4 ICM steps

## Limitations

- c_penalty for MBIC must be tuned: large values (5–10) when K < 10, 0.5–2 for many clusters; a mis-set penalty biases cluster-number selection
- Depends on Rcpp/RcppArmadillo compilation, requiring Rtools (Windows) or Xcode (Mac OS/X); on Linux, 'ulimit -s unlimited' may be needed to avoid 'C stack usage is too close to the limit'
- Requires prior dimensionality reduction (APCA, npcs = 15) and adjacency construction; running iSC.MEB directly on raw counts is out of scope

## Evidence

- [results] In the ICM step of the ICM-EM algorithm, iSC.MEB alternates between estimating the cluster label yti ... and estimating the batch-related embedding uti: [evidence span withheld — anchor status: not_found]
- [abstract] permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets: "permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets"
- [methods] iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4): "iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4)"
- [methods] iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2): "iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2)"
- [readme] iSC.MEB depends on the 'Rcpp' and 'RcppArmadillo' package, which also requires appropriate setting of Rtools and Xcode: ""iSC.MEB" depends on the 'Rcpp' and 'RcppArmadillo' package, which also requires appropriate setting of Rtools and Xcode for Windows and Mac OS/X, respectively."
