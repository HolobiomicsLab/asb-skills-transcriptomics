---
name: hmrf-spatial-clustering-model-fitting
description: Use when multiple spatially resolved transcriptomics sections (e.g. Visium
  DLPFC or seqFISH embryo sections) need joint spatial-domain clustering with simultaneous
  batch-effect estimation over a grid of cluster numbers K.
when_to_use_negative:
- Data are not spatially resolved or no spot/cell adjacency can be defined (CreateNeighbors
  cannot be applied), since the model is a hidden Markov random field over spatial
  neighborhoods.
- Only a single sample with no batch structure is being analyzed — the single-sample
  empirical-Bayes HMRF predecessor SC.MEB is the appropriate tool.
- Input is raw count matrices without dimensionality reduction; the model requires
  low-dimensional representations (e.g. npcs=15 ACA/APCA embeddings), not full-gene
  expression.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: iSC.MEB
  role: R package implementing the hidden-Markov-random-field empirical-Bayes model;
    its iSCMEB function performs ICM-EM fitting and grid search over beta, with SelectModel
    for MBIC cluster-number selection
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: SC.MEB
  role: Single-sample empirical-Bayes HMRF predecessor, used as the runtime/behavior
    baseline when iSC.MEB is fitted to multiple sections
- name: R
  role: Runtime environment in which the iSC.MEB package (dependent on Rcpp/RcppArmadillo)
    is installed via devtools and the fitting pipeline is executed
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:12decac5e691752340a9292d1bca872d319aff4a01379c04d7fdebc4d59b1bc0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - BASS
  - BayesSpace
  - R
  - SC.MEB
  - devtools
  - iSC.MEB
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# hmrf-spatial-clustering-model-fitting

## Summary

Fits a hidden-Markov-random-field spatial clustering model with empirical Bayes (iSC.MEB) to jointly estimate cluster labels and batch-related embeddings across multiple SRT sections, using ICM-EM inference and MBIC cluster-number selection.

## When to use

Use when multiple spatially resolved transcriptomics sections (e.g. Visium DLPFC or seqFISH embryo sections) need joint spatial-domain clustering with simultaneous batch-effect estimation over a grid of cluster numbers K.

## When NOT to use

- Data are not spatially resolved or no spot/cell adjacency can be defined (CreateNeighbors cannot be applied), since the model is a hidden Markov random field over spatial neighborhoods.
- Only a single sample with no batch structure is being analyzed — the single-sample empirical-Bayes HMRF predecessor SC.MEB is the appropriate tool.
- Input is raw count matrices without dimensionality reduction; the model requires low-dimensional representations (e.g. npcs=15 ACA/APCA embeddings), not full-gene expression.

## Inputs

- seuList: list of Seurat objects for multiple SRT sections (e.g. DLPFC.rda or seqFISH.rda downloaded from the iSC.MEB vignettes_data repo)
- iSCMEBObj containing APCA low-dimensional embeddings (npcs=15) and spatial adjacency matrices from CreateNeighbors
- Model configuration from SetModelParameters (maxIter_ICM, maxIter, coreNum)
- Candidate cluster-number grid (e.g. K = 16:19)

## Outputs

- Fitted cluster labels per spot, extractable with idents() (e.g. saved to iscmeb_cluster_labels.rds with the MBIC-selected K)
- Estimated batch-related embeddings u_ti per sample and spot
- MBIC selection curve over K and the selected optimal K
- Updated iSCMEBObj with an 'iSCMEB' reduction usable by CalculateTSNE and SpaHeatMap

## How to apply

Build an iSCMEBObj from a seuList of Seurat objects (CreateiSCMEBObject with premin.spots=0, postmin.spots=0), add platform-specific spatial adjacency with CreateNeighbors(platform='Visium'), and reduce dimensions with runPCA(npcs=15, pca.method='APCA') since the HMRF model operates on low-dimensional embeddings, not raw counts. Configure inference via SetModelParameters (maxIter_ICM=4, maxIter=20, coreNum=4); during the ICM-EM fit the model alternates between estimating the cluster label y_ti by maximizing its posterior given the batch embedding u_ti, and estimating u_ti by maximizing its posterior given y_ti. Call iSCMEB over a candidate K grid (e.g. K=16:19), which grid-searches the smoothness parameter beta at each K. Select K with SelectModel(criteria='MBIC', c_penalty=...), tuning the c_penalty to the regime (5–10 when K<10, 0.5–2 for many clusters, 0.2 in the seqFISH vignette) and inspecting the selection curve with SelectKPlot(criteria='MBIC').

## Related tools

- **iSC.MEB** (R package implementing the hidden-Markov-random-field empirical-Bayes model; its iSCMEB function performs ICM-EM fitting and grid search over beta, with SelectModel for MBIC cluster-number selection) — https://github.com/XiaoZhangryy/iSC.MEB
- **SC.MEB** (Single-sample empirical-Bayes HMRF predecessor, used as the runtime/behavior baseline when iSC.MEB is fitted to multiple sections)
- **R** (Runtime environment in which the iSC.MEB package (dependent on Rcpp/RcppArmadillo) is installed via devtools and the fitting pipeline is executed)

## Examples

```
iSCMEBObj <- iSCMEB(iSCMEBObj, K= 7); iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2)
```

## Evaluation signals

- SelectModel(criteria='MBIC') returns a well-defined optimal K and the SelectKPlot(criteria='MBIC') curve shows a clear minimum over the K grid
- Fitted labels are retrievable via idents() and are spatially contiguous when plotted with SpaHeatMap(plot_type='Scatter')
- The ICM-EM fit converges within the configured limits (maxIter_ICM=4, maxIter=20) across the K grid without parallel errors (coreNum=4)
- If manual annotations are available, ARI computed against them is reasonable and comparable to the paper's reported cell/domain detection accuracy
- Saved output (iscmeb_cluster_labels.rds) contains both the label vector and the MBIC-selected K for reproducibility

## Limitations

- The c_penalty for MBIC is dataset-dependent and must be tuned: 5–10 when K < 10, 0.5–2 for many clusters, and 0.2 was used in the seqFISH vignette; a poorly tuned penalty can select the wrong K.
- Installation requires Rcpp/RcppArmadillo with correctly configured Rtools (Windows) or Xcode and gfortran (Mac); on Linux, `ulimit -s unlimited` must be set to avoid the 'C stack usage is too close to the limit' error during parallel fitting.
- Runtime is comparable to SC.MEB but the MCMC-free empirical-Bayes fitting is the efficiency rationale — scaling behavior should not be assumed for very large K grids or very many sections.

## Evidence

- [abstract] we introduce an extension to SC.MEB, denoted as integrated spatial clustering with hidden Markov random field using empirical Bayes (iSC.MEB): "we introduce an extension to SC.MEB, denoted as integrated spatial clustering with hidden Markov random field using empirical Bayes (iSC.MEB)"
- [abstract] permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets: "permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets"
- [methods] iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4): "iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4)"
- [methods] iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2): "iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2)"
- [results] In the ICM step of the ICM-EM algorithm, iSC.MEB alternates between estimating the cluster label yti by maximizing its posterior given the batch-related embedding uti, and estimating the batch-related embedding uti by maximizing its posterior given the label yti.: [evidence span withheld — anchor status: not_found]
