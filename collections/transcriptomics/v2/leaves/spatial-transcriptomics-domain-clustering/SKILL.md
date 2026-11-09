---
name: spatial-transcriptomics-domain-clustering
description: Use when integrating and spatially clustering multiple SRT sections (e.g.,
  12-sample human DLPFC 10x Visium) into known domain counts, or when benchmarking
  clustering accuracy (ARI/NMI) against methods like BASS, SC.MEB, BayesSpace, SpaGCN,
  Louvain.
when_to_use_negative:
- Data is a single-section SRT dataset where simpler single-sample methods like SC.MEB
  or BayesSpace suffice — iSC.MEB is designed for multi-sample integration
- Input data lacks spatial coordinates or a supported platform adjacency (e.g., non-Visium
  platforms without a CreateNeighbors strategy)
- The goal is differential expression or gene-level analysis rather than domain/cell
  clustering (use doDEG downstream instead)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: iSC.MEB
  role: 'Primary R package: fits the hidden Markov random field empirical Bayes model
    for integrated multi-sample spatial clustering'
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: SC.MEB
  role: Single-sample predecessor method used as a comparison baseline in ARI/NMI
    benchmarking
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: Seurat
  role: Provides the Seurat object containers (seuList) holding per-section expression
    and spatial metadata ingested by CreateiSCMEBObject
- name: mclust
  role: Supplies adjustedRandIndex for computing ARI between predicted cluster labels
    and manual annotations
- name: BASS
  role: Comparison method for both clustering accuracy and multi-sample data integration
    performance
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:90df765c66cf3ea02aa29de4b6d45bce6d0ce5cba1b41d6d90653741a7de9b8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - BASS
  - R
  - SC.MEB
  - Seurat
  - iSC.MEB
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# spatial-transcriptomics-domain-clustering

## Summary

Benchmark and perform multi-sample spatial domain/cell clustering on spatially resolved transcriptomics data using iSC.MEB, a hidden Markov random field empirical Bayes extension of SC.MEB, and evaluate accuracy against manual annotations with ARI/NMI.

## When to use

Use when integrating and spatially clustering multiple SRT sections (e.g., 12-sample human DLPFC 10x Visium) into known domain counts, or when benchmarking clustering accuracy (ARI/NMI) against methods like BASS, SC.MEB, BayesSpace, SpaGCN, Louvain.

## When NOT to use

- Data is a single-section SRT dataset where simpler single-sample methods like SC.MEB or BayesSpace suffice — iSC.MEB is designed for multi-sample integration
- Input data lacks spatial coordinates or a supported platform adjacency (e.g., non-Visium platforms without a CreateNeighbors strategy)
- The goal is differential expression or gene-level analysis rather than domain/cell clustering (use doDEG downstream instead)

## Inputs

- List of Seurat objects (seuList) from multiple SRT sections, e.g. the filtered 12-section DLPFC 10x Visium subset (DLPFC.rda)
- Manual domain/layer annotations per section (e.g., layer_guess_reordered) as ground truth for ARI/NMI
- Platform specification for adjacency construction (e.g., 'Visium')

## Outputs

- iSCMEBObj with spatial cluster labels per spot (retrievable via idents())
- Per-sample and integrated ARI/NMI table (e.g., dlpfc_ari_nmi_results.csv; iSC.MEB integrated ARI/NMI expected at 0.48/0.57)
- Spatial scatter plots of clusters via SpaHeatMap

## How to apply

Download the filtered multi-section Seurat object list (e.g., DLPFC.rda from the iSC.MEB vignettes_data repo) and load it into R. Build the iSCMEBObj with CreateiSCMEBObject (premin.spots=0, postmin.spots=0), build Visium adjacency matrices with CreateNeighbors, and run adjusted PCA (APCA) with 15 principal components via runPCA. Fit the hidden Markov random field empirical Bayes model with iSCMEB using the expected number of domains (K=7 for the 7 DLPFC cortical layers) and finalize labels with SelectModel (criteria='MBIC', c_penalty=0.2). For benchmarking, run SC.MEB, BASS, BayesSpace, SpaGCN, and Louvain on the same sections with standard settings, then compute per-sample and pooled-spot ARI (mclust::adjustedRandIndex) and NMI against manual annotations (e.g., layer_guess_reordered). Rationale: iSC.MEB simultaneously estimates batch effect while clustering low-dimensional representations of multiple SRT datasets, which is why it supports integrated comparison across batches, individuals, and conditions.

## Related tools

- **iSC.MEB** (Primary R package: fits the hidden Markov random field empirical Bayes model for integrated multi-sample spatial clustering) — https://github.com/XiaoZhangryy/iSC.MEB
- **SC.MEB** (Single-sample predecessor method used as a comparison baseline in ARI/NMI benchmarking) — https://github.com/XiaoZhangryy/iSC.MEB
- **Seurat** (Provides the Seurat object containers (seuList) holding per-section expression and spatial metadata ingested by CreateiSCMEBObject)
- **mclust** (Supplies adjustedRandIndex for computing ARI between predicted cluster labels and manual annotations)
- **BASS** (Comparison method for both clustering accuracy and multi-sample data integration performance)

## Examples

```
iSCMEBObj <- CreateiSCMEBObject(seuList = seuList); iSCMEBObj <- CreateNeighbors(iSCMEBObj, platform = "Visium"); iSCMEBObj <- runPCA(iSCMEBObj, npcs = 15, pca.method = "APCA"); iSCMEBObj <- iSCMEB(iSCMEBObj, K = 7); iSCMEBObj <- SelectModel(iSCMEBObj)
```

## Evaluation signals

- Integrated ARI/NMI for iSC.MEB on the 12 DLPFC samples should reach approximately 0.48/0.57 with the highest median ARI/NMI among compared methods
- Per-sample ARI computed via mclust::adjustedRandIndex against layer_guess_reordered annotations should be non-trivially positive (>0) for each section
- CreateiSCMEBObject must ingest all 12 Seurat objects and CreateNeighbors must build a valid Visium adjacency matrix before runPCA (npcs=15, APCA) succeeds
- SelectModel should converge on cluster labels consistent with K=7 cortical layers when run with criteria='MBIC'
- Output CSV contains one ARI/NMI row per section plus integrated values across all pooled spots

## Limitations

- iSC.MEB installation requires Rcpp, RcppArmadillo, and platform compilers (Rtools on Windows, Xcode and gfortran on macOS); misconfigured systems fail to install
- On Linux, parallel computation requires 'ulimit -s unlimited' to avoid 'R Error: C stack usage is too close to the limit'; some Bioconductor dependencies (e.g., scater) may need conda or BiocManager installs
- The number of clusters K must be specified (K=7 for DLPFC); the benchmark compares only a fixed set of methods (BASS, SC.MEB, BayesSpace, SpaGCN, Louvain) on a filtered 12-section DLPFC subset

## Evidence

- [abstract] permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets: "permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets"
- [methods] download DLPFC.rda: "githubURL <- "https://github.com/XiaoZhangryy/iSC.MEB/blob/main/vignettes_data/DLPFC.rda?raw=true"
download.file(githubURL,"DLPFC.rda",mode='wb')"
- [methods] runPCA APCA npcs=15: "iSCMEBObj <-  runPCA(iSCMEBObj, npcs = 15, pca.method = "APCA")"
- [methods] SelectModel MBIC c_penalty 0.2: "iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2)"
- [readme] README install and DLPFC vignette: "iSC.MEB is a package for integrating and analyzing multiple spatially resolved transcriptomics (SRT) datasets, developed by the Jin Liu's lab."
