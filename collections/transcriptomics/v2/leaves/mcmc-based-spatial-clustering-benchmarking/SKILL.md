---
name: mcmc-based-spatial-clustering-benchmarking
description: Use when comparing computational efficiency of spatial transcriptomics
  clustering methods (e.g. iSC.MEB vs MCMC-based BASS/BayesSpace) across multiple
  tissue sections and a sequence of K cluster numbers.
when_to_use_negative:
- The goal is accuracy (e.g. ARI) rather than computational efficiency benchmarking
  — use the clustering-evaluation workflow instead
- Data are not multi-sample SRT sections (e.g. single-cell non-spatial data), where
  spatial adjacency and HMRF methods do not apply
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: iSC.MEB
  role: Empirical-Bayes HMRF multi-sample spatial clustering method whose fitting
    time is benchmarked via the iSCMEB function across K values
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: SC.MEB
  role: Single-sample HMRF empirical-Bayes predecessor run over the same K grid as
    the comparable-speed baseline
- name: BASS
  role: MCMC-based spatial clustering method run on the same 12 DLPFC sections as
    the slow baseline
- name: BayesSpace
  role: MCMC-based spatial clustering method run on the same sections and K grid as
    the slow baseline
- name: Seurat
  role: Format of the input seuList Visium objects loaded from DLPFC.rda
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:5425f8217089ea31a6a128a509fb16218941c9a3c6aac7a1f455d97fb2e37c35
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
  - devtools
  - iSC.MEB
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mcmc-based-spatial-clustering-benchmarking

## Summary

Benchmark the wall-clock runtime of spatial clustering methods (iSC.MEB, SC.MEB vs MCMC-based BASS and BayesSpace) across a grid of cluster numbers K on the 12-section DLPFC Visium dataset, to demonstrate the computational advantage of empirical-Bayes HMRF fitting over MCMC sampling.

## When to use

Use when comparing computational efficiency of spatial transcriptomics clustering methods (e.g. iSC.MEB vs MCMC-based BASS/BayesSpace) across multiple tissue sections and a sequence of K cluster numbers.

## When NOT to use

- The goal is accuracy (e.g. ARI) rather than computational efficiency benchmarking — use the clustering-evaluation workflow instead
- Data are not multi-sample SRT sections (e.g. single-cell non-spatial data), where spatial adjacency and HMRF methods do not apply

## Inputs

- DLPFC.rda containing seuList: a list of 12 Visium Seurat objects (Maynard et al., 2021 DLPFC dataset)
- iSCMEBObj with Visium adjacency and 15-component APCA-reduced dimensions
- A sequence of K cluster numbers to iterate over

## Outputs

- iscmeb_runtime_benchmark.csv with wall-clock seconds per method per K per section
- Runtime versus K plot comparing iSC.MEB, SC.MEB, BASS, and BayesSpace
- Validated conclusion that iSC.MEB is faster than MCMC-based methods and comparable to SC.MEB

## How to apply

Download DLPFC.rda from the iSC.MEB GitHub vignettes_data URL and load the seuList of 12 Visium Seurat objects (Maynard et al., 2021). Build the analysis object with CreateiSCMEBObject(premin.spots=0, postmin.spots=0), add Visium adjacency via CreateNeighbors(platform='Visium'), reduce dimensions with runPCA(npcs=15, pca.method='APCA'), and set fitting parameters with SetModelParameters(maxIter_ICM=4, maxIter=20, coreNum=4). Time the iSCMEB fit over a sequence of K values, recording elapsed seconds per K; then run BASS, BayesSpace, and SC.MEB pipelines on the same 12 sections over the same K grid with identical timing. Compile all timings into a CSV (e.g. iscmeb_runtime_benchmark.csv) and plot runtime versus K per method. The decision point is validating that iSC.MEB's empirical-Bayes/ICM fitting (bounded by maxIter_ICM=4, maxIter=20) is substantially faster than MCMC-based BASS and BayesSpace and comparable to SC.MEB.

## Related tools

- **iSC.MEB** (Empirical-Bayes HMRF multi-sample spatial clustering method whose fitting time is benchmarked via the iSCMEB function across K values) — https://github.com/XiaoZhangryy/iSC.MEB
- **SC.MEB** (Single-sample HMRF empirical-Bayes predecessor run over the same K grid as the comparable-speed baseline)
- **BASS** (MCMC-based spatial clustering method run on the same 12 DLPFC sections as the slow baseline)
- **BayesSpace** (MCMC-based spatial clustering method run on the same sections and K grid as the slow baseline)
- **Seurat** (Format of the input seuList Visium objects loaded from DLPFC.rda)

## Examples

```
iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4); system.time(iSCMEB(iSCMEBObj, K=7))
```

## Evaluation signals

- Wall-clock time per K is recorded for every method (iSC.MEB, SC.MEB, BASS, BayesSpace) on the same 12 DLPFC sections
- iSC.MEB elapsed times are substantially lower than BASS and BayesSpace at every K in the grid
- iSC.MEB runtime is comparable to SC.MEB, matching the paper's reported comparison
- All timings are compiled into iscmeb_runtime_benchmark.csv with a runtime-versus-K plot per method
- Fitting uses the specified parameters (maxIter_ICM=4, maxIter=20, coreNum=4, npcs=15 APCA) so timings are reproducible

## Limitations

- Runtime conclusions hold for the 12-section DLPFC Visium data and the chosen K grid; timings depend on hardware and coreNum (4 cores used)
- iSC.MEB depends on Rcpp/RcppArmadillo compilation; on Linux, C_stack must be set unlimited (ulimit -s unlimited) to avoid 'C stack usage is too close to the limit' errors
- The benchmark compares speed, not clustering accuracy; the DLPFC.rda is a filtered subset of the full DLPFC data

## Evidence

- [results] On the 12 DLPFC samples, iSC.MEB was significantly more computationally efficient than the MCMC-based methods BASS and BayesSpace, and its speed was comparable to SC.MEB.: [evidence span withheld — anchor status: not_found]
- [methods] the analysis of the DLPFC dataset [Maynard K. et al, 2021](https://www.nature.com/articles/s41593-020-00787-0), which contains the spatial topography of gene expression from 12 human postmortem DLPFC: "the analysis of the DLPFC dataset [Maynard K. et al, 2021](https://www.nature.com/articles/s41593-020-00787-0), which contains the spatial topography of gene expression from 12 human postmortem DLPFC"
- [methods] iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4): "iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4)"
- [readme] For parallel computation on Linux, users must use the following system command to set the C_stack unlimited in case of the error `R Error: C stack usage is too close to the limit`.: [evidence span withheld — anchor status: external_doc]
