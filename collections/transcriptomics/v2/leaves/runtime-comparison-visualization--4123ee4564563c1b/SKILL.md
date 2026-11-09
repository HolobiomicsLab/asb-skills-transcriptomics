---
name: runtime-comparison-visualization
description: Use when comparing computational cost of spatial transcriptomics clustering
  methods on the same samples and K grid, e.g. showing an EM-based method like iSC.MEB
  is faster than MCMC-based BASS/BayesSpace.
when_to_use_negative:
- The goal is clustering accuracy rather than computational cost — use ARI evaluation
  of cluster labels instead of timing benchmarks.
- Only a single sample/section is available and the comparison target is iSC.MEB itself
  — SC.MEB, not iSC.MEB, is the single-sample predecessor to run.
- Methods are not run over the same K grid and same 12 DLPFC sections — timing comparisons
  across different inputs are not valid.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: iSC.MEB
  role: EM-based multi-sample spatial clustering method being benchmarked; timed across
    a K sequence via the iSCMEB function
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: SC.MEB
  role: Single-sample HMRF empirical-Bayes predecessor, timed over the same K grid
    as the speed comparison baseline
- name: BASS
  role: MCMC-based spatial clustering method timed as the slower comparison baseline
- name: BayesSpace
  role: MCMC-based spatial method timed over the same DLPFC sections and K grid as
    the slower comparison baseline
- name: R
  role: Implementation environment for all pipelines, timing loops, CSV compilation
    and plotting
- name: devtools
  role: Used to install iSC.MEB from GitHub before benchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:cac08c11df5c7ac389755f9b8280bb5a02ee16879ae7515f028b1619ca09b84c
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
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# runtime-comparison-visualization

## Summary

Benchmark wall-clock runtime of spatial clustering methods (iSC.MEB vs. SC.MEB, BASS, BayesSpace) across a sequence of cluster numbers K on the 12-section DLPFC Visium dataset, then plot runtime versus K per method.

## When to use

Use when comparing computational cost of spatial transcriptomics clustering methods on the same samples and K grid, e.g. showing an EM-based method like iSC.MEB is faster than MCMC-based BASS/BayesSpace.

## When NOT to use

- The goal is clustering accuracy rather than computational cost — use ARI evaluation of cluster labels instead of timing benchmarks.
- Only a single sample/section is available and the comparison target is iSC.MEB itself — SC.MEB, not iSC.MEB, is the single-sample predecessor to run.
- Methods are not run over the same K grid and same 12 DLPFC sections — timing comparisons across different inputs are not valid.

## Inputs

- DLPFC.rda (seuList of 12 Visium Seurat objects from Maynard et al. 2021 DLPFC dataset, from github.com/XiaoZhangryy/iSC.MEB/vignettes_data)
- Fitted iSC.MEB object (iSCMEBObj) built via CreateiSCMEBObject, CreateNeighbors, runPCA(npcs=15, pca.method='APCA'), SetModelParameters
- Per-method wall-clock timings (elapsed seconds) per K for iSC.MEB, SC.MEB, BASS, BayesSpace

## Outputs

- iscmeb_runtime_benchmark.csv (per-method, per-K elapsed times)
- Runtime-versus-K plot comparing iSC.MEB, BASS, BayesSpace and SC.MEB

## How to apply

Download DLPFC.rda from the iSC.MEB GitHub vignettes_data URL, load it into R to obtain the seuList of 12 Visium Seurat objects, and build the analysis object with CreateiSCMEBObject (premin.spots=0, postmin.spots=0), CreateNeighbors(platform='Visium'), runPCA(npcs=15, pca.method='APCA'), and SetModelParameters(maxIter_ICM=4, maxIter=20, coreNum=4). Time wall-clock execution of the iSCMEB function over a sequence of K values, recording elapsed seconds per K. Run BASS and BayesSpace — the MCMC-based methods — and SC.MEB, the single-sample HMRF empirical-Bayes predecessor, over the same DLPFC sections and the same K grid, timing each method per K. Compile all timings into a CSV (e.g. iscmeb_runtime_benchmark.csv) and plot runtime versus K for each method. Validate that iSC.MEB's times are substantially lower than BASS and BayesSpace while being comparable to SC.MEB.

## Related tools

- **iSC.MEB** (EM-based multi-sample spatial clustering method being benchmarked; timed across a K sequence via the iSCMEB function) — https://github.com/XiaoZhangryy/iSC.MEB
- **SC.MEB** (Single-sample HMRF empirical-Bayes predecessor, timed over the same K grid as the speed comparison baseline)
- **BASS** (MCMC-based spatial clustering method timed as the slower comparison baseline)
- **BayesSpace** (MCMC-based spatial method timed over the same DLPFC sections and K grid as the slower comparison baseline)
- **R** (Implementation environment for all pipelines, timing loops, CSV compilation and plotting)
- **devtools** (Used to install iSC.MEB from GitHub before benchmarking)

## Examples

```
githubURL <- "https://github.com/XiaoZhangryy/iSC.MEB/blob/main/vignettes_data/DLPFC.rda?raw=true"; download.file(githubURL,"DLPFC.rda",mode='wb'); load("DLPFC.rda"); system.time(iSCMEB(iSCMEBObj, K=7))
```

## Evaluation signals

- Per-K elapsed times for all four methods are recorded over the same 12 DLPFC sections and the same K grid, with each timing pair measured on identical inputs.
- iSC.MEB wall-clock time is substantially lower than both MCMC-based methods (BASS, BayesSpace) at each K.
- iSC.MEB runtime is comparable to SC.MEB at each K, since both are empirical-Bayes HMRF methods rather than MCMC.
- iscmeb_runtime_benchmark.csv contains one timing row per method per K with elapsed seconds, and the runtime-vs-K plot covers all four methods.
- The iSC.MEB pipeline used for timing matches the vignette settings: CreateiSCMEBObject(premin.spots=0, postmin.spots=0), CreateNeighbors(platform='Visium'), runPCA(npcs=15, pca.method='APCA'), SetModelParameters(maxIter_ICM=4, maxIter=20, coreNum=4).

## Limitations

- The runtime comparison is specific to the 12-section DLPFC dataset and the chosen K sequence; absolute timings do not generalize to other datasets or hardware.
- iSC.MEB requires Rtools/Xcode toolchains (Windows/Mac) and, on Linux, `ulimit -s unlimited` before parallel runs to avoid 'R Error: C stack usage is too close to the limit'.
- iSC.MEB depends on Rcpp/RcppArmadillo, so compilation failures on improperly configured systems can block the benchmark.

## Evidence

- [results] iSC.MEB was significantly more computationally efficient than the MCMC-based methods BASS and BayesSpace, and its speed was comparable to SC.MEB: [evidence span withheld — anchor status: not_found]
- [methods] SetModelParameters evidence: "iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4)"
- [methods] download data: "githubURL <- "https://github.com/XiaoZhangryy/iSC.MEB/blob/main/vignettes_data/DLPFC.rda?raw=true"
download.file(githubURL,"DLPFC.rda",mode='wb')"
- [methods] fitting: "iSCMEBObj <- iSCMEB(iSCMEBObj, K= 7)"
- [readme] Linux parallel caveat: [evidence span withheld — anchor status: external_doc]
