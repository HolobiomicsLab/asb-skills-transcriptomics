---
name: spatial-neighborhood-graph-construction
description: Use when fitting spatial-domain clustering models (iSC.MEB, SC.MEB, BayesSpace,
  SpaGCN) on SRT sections and an explicit spatial adjacency/neighbor graph must be
  built from spot coordinates before model fitting.
when_to_use_negative:
- Input is a non-spatial expression matrix with no spot/cell coordinate information
  — the neighbor graph cannot be defined.
- Data already contains a precomputed spatial adjacency graph compatible with the
  MRF model.
- Analysis is purely non-spatial single-cell clustering (e.g., standard Louvain on
  PCA), where a spatial graph adds the wrong prior.
edam_operation: http://edamontology.org/operation_0279
tools:
- name: iSC.MEB
  role: provides the CreateNeighbors function that builds L2-distance-based spatial
    adjacency matrices (radius.upper=10) as MRF input
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: R
  role: implementation language; iSC.MEB is an open-source R package run via Rscript
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: Seurat
  role: source object format — the seuList input is a list of Seurat objects holding
    spatial coordinates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:54313651bccf298b3a5623c0b3882e360f1ed94e3700ff72b35490efa5bc2acb
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# spatial-neighborhood-graph-construction

## Summary

Constructs spatial adjacency matrices capturing spot/cell neighborhood structure from multi-sample spatial transcriptomics data (e.g., six-section mouse embryo seqFISH), providing the graph input required by hidden Markov random field clustering in iSC.MEB.

## When to use

Use when fitting spatial-domain clustering models (iSC.MEB, SC.MEB, BayesSpace, SpaGCN) on SRT sections and an explicit spatial adjacency/neighbor graph must be built from spot coordinates before model fitting.

## When NOT to use

- Input is a non-spatial expression matrix with no spot/cell coordinate information — the neighbor graph cannot be defined.
- Data already contains a precomputed spatial adjacency graph compatible with the MRF model.
- Analysis is purely non-spatial single-cell clustering (e.g., standard Louvain on PCA), where a spatial graph adds the wrong prior.

## Inputs

- seuList: list of Seurat objects with spatial spot/cell coordinates (e.g., seqFISH.rda six mouse embryo sections)
- iSCMEBObj created via CreateiSCMEBObject
- platform geometry specification ('Visium' lattice or coordinate-based)

## Outputs

- Spatial adjacency/neighbor matrices attached to the iSCMEBObj
- Graph structure consumed downstream by iSC.MEB model fitting (iSCMEB with K=16:19) and SelectModel (criteria='MBIC', c_penalty=0.2)

## How to apply

Load the list of Seurat objects (e.g., seuList from seqFISH.rda) and build an iSCMEBObj with CreateiSCMEBObject, then call CreateNeighbors to compute pairwise spot distances and adjacency matrices. For the high-resolution seqFISH embryo sections, use L2 (Euclidean) distance with radius.upper=10 rather than the Visium hex-lattice default, so that each spot links only to spatially proximal neighbors. The resulting adjacency matrices define the Markov random field over which iSC.MEB's empirical-Bayes hidden MRF later smooths low-dimensional representations (runPCA, npcs=15, pca.method='APCA') and performs clustering. Choose the neighborhood radius to match platform geometry: lattice platforms can use platform='Visium', while arbitrary-coordinate data (seqFISH) requires the distance-threshold approach.

## Related tools

- **iSC.MEB** (provides the CreateNeighbors function that builds L2-distance-based spatial adjacency matrices (radius.upper=10) as MRF input) — https://github.com/XiaoZhangryy/iSC.MEB
- **R** (implementation language; iSC.MEB is an open-source R package run via Rscript) — https://github.com/XiaoZhangryy/iSC.MEB
- **Seurat** (source object format — the seuList input is a list of Seurat objects holding spatial coordinates)

## Examples

```
iSCMEBObj <- CreateNeighbors(iSCMEBObj, radius.upper=10)
```

## Evaluation signals

- CreateNeighbors returns adjacency matrices for every section in the six-section seqFISH dataset without dropping spots (premin.spots=0, postmin.spots=0 were used at object creation).
- For seqFISH, the neighborhood is distance-threshold based (radius.upper=10, L2 distance), not the Visium lattice default — verify the correct platform/threshold configuration was used.
- Downstream iSC.MEB fitting over K=16:19 converges with SetModelParameters(maxIter_ICM=4, maxIter=20, coreNum=4), which fails if the adjacency graph is malformed.
- Integrated clustering quality matches the reported benchmark (iSC.MEB integrated ARI/NMI of 0.52/0.69 vs BASS 0.45/0.66 against celltype_mapped_refined labels), indicating the graph correctly encodes spatial structure.

## Limitations

- The radius threshold (radius.upper=10) and platform parameter must match the physical spot spacing of the platform; using the 'Visium' default on arbitrary-coordinate data such as seqFISH mis-defines neighborhoods.
- iSC.MEB depends on Rcpp/RcppArmadillo compilation; system setup (Rtools on Windows, Xcode/gfortran on macOS) is required, and Linux parallel runs may need `ulimit -s unlimited` to avoid 'C stack usage is too close to the limit' errors.
- Conda environments may fail to install dependencies like scater; BiocManager/CRAN or bioconda (bioconductor-scater) installs are recommended workarounds.

## Evidence

- [methods] CreateNeighbors call: "iSCMEBObj <-  CreateNeighbors(iSCMEBObj, radius.upper=10)"
- [abstract] multi-sample SRT integration scope: "permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets"
- [readme] package purpose: "iSC.MEB is a package for integrating and analyzing multiple spatially resolved transcriptomics (SRT) datasets"
- [readme] dependency chain requiring R compilation setup: ""iSC.MEB" depends on the 'Rcpp' and 'RcppArmadillo' package"
