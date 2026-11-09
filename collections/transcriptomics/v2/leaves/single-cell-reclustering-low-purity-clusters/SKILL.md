---
name: single-cell-reclustering-low-purity-clusters
description: Use when an scRNA-seq clustering partition contains clusters of questionable
  homogeneity and you need to quantify cluster purity with ROGUE and re-cluster low-purity
  clusters into purer subtypes.
when_to_use_negative:
- Clusters already have high ROGUE (e.g. the recommended ~0.9 purity threshold is
  met); re-clustering is only warranted for low-purity clusters
- Input is already a summarized feature/purity table rather than a raw or filtered
  count matrix — ROGUE's matr.filter and entropy_fit require expression counts
- Data are integrated populations from different protocols/platforms, which ROGUE
  does not yet assess for purity
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: ROGUE
  role: Entropy-based metric and R package used to quantify cluster purity (CalculateRogue),
    filter the matrix (matr.filter), fit the entropy model (entropy_fit), select significantly
    entropic variable genes (DetermineK), and guide re-clustering decisions
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Dependency package that must be installed before installing ROGUE; used for
    data manipulation in the R workflow
- name: devtools
  role: Used to install the ROGUE package from GitHub via install_github
- name: R
  role: Runtime environment in which the ROGUE package and re-clustering workflow
    are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:d4003ca1d8c6f7049fa9693c3c2173ea894587760cc8a772e3dd99b36e6ec10d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# single-cell-reclustering-low-purity-clusters

## Summary

ROGUE-guided re-clustering of low-purity single-cell clusters: compute the entropy-based ROGUE statistic per cluster, re-cluster clusters with low ROGUE (e.g. lung-tumor CAF clusters), and validate that the refined partition raises average cluster purity and reveals biologically distinct subtypes.

## When to use

Use when an scRNA-seq clustering partition contains clusters of questionable homogeneity and you need to quantify cluster purity with ROGUE and re-cluster low-purity clusters into purer subtypes.

## When NOT to use

- Clusters already have high ROGUE (e.g. the recommended ~0.9 purity threshold is met); re-clustering is only warranted for low-purity clusters
- Input is already a summarized feature/purity table rather than a raw or filtered count matrix — ROGUE's matr.filter and entropy_fit require expression counts
- Data are integrated populations from different protocols/platforms, which ROGUE does not yet assess for purity

## Inputs

- scRNA-seq raw count matrix (gene x cell) with original cluster assignments, e.g. Kim et al. lung-tumor stromal data with 7 CAF clusters
- Marker gene lists for subtype annotation (e.g. RGS5/ACTA2/GPX3 for myCAFs, CD74/MHC-II for apCAFs)

## Outputs

- Re-clustered partition with higher average ROGUE (e.g. 11 subtypes from 7 clusters)
- Per-cluster ROGUE purity table before and after re-clustering (e.g. caf_rogue_subtypes.csv)
- Annotated subtype labels with UMAP embedding

## How to apply

Load the raw count matrix with existing cluster assignments and filter genes/cells using ROGUE's matr.filter (min.cells, min.genes). Fit the entropy model with entropy_fit and DetermineK to select S-E (significantly entropic) variable genes for downstream clustering. Re-cluster the low-purity clusters (e.g. the seven lung-tumor CAF clusters from Kim et al.) after normalization with Seurat negative-binomial regression and batch correction with BBKNN, using leiden clustering and UMAP visualization. Compute per-cluster purity with CalculateRogue on both the original and re-clustered partitions and compare: re-clustering is successful when the average ROGUE increases and subtypes can be annotated by markers (myCAF: RGS5/ACTA2/GPX3; apCAF: CD74/MHC-II).

## Related tools

- **ROGUE** (Entropy-based metric and R package used to quantify cluster purity (CalculateRogue), filter the matrix (matr.filter), fit the entropy model (entropy_fit), select significantly entropic variable genes (DetermineK), and guide re-clustering decisions) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Dependency package that must be installed before installing ROGUE; used for data manipulation in the R workflow)
- **devtools** (Used to install the ROGUE package from GitHub via install_github)
- **R** (Runtime environment in which the ROGUE package and re-clustering workflow are executed)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- Re-clustering yields the expected number of purer subtypes (e.g. 11 subtypes from the original 7 low-purity CAF clusters) with higher average ROGUE
- Per-cluster ROGUE values match reported benchmarks, e.g. original myCAF cluster AF_C02_COL4A1 ROGUE = 0.81 splits into BF_C01_RGS5 (0.84), BF_C02_ACTA2 (0.87), BF_C03_GPX3 (0.94)
- The refined subtypes are annotatable by expected markers: myCAF subclusters express RGS5/ACTA2/GPX3; the apCAF cluster shows high CD74 and MHC class-II gene expression
- DetermineK identifies a non-empty set of S-E (significantly entropic) variable genes suitable for downstream clustering
- Output table (e.g. caf_rogue_subtypes.csv) contains ROGUE values for both the original and re-clustered partitions enabling before/after comparison

## Limitations

- The recommended 0.9 purity threshold may not be suitable for low-quality or continuous data; the threshold should be determined from global ROGUE values
- Current ROGUE does not assess purity of integrated cell populations from different protocols and platforms; this is proposed as future work
- Roles of some resolved subtypes may remain biologically unclear (e.g. apCAFs in the lung-cancer microenvironment require further investigation)

## Evidence

- [abstract] ROGUE quantifies purity and guides re-clustering: "Here we present an entropy-based statistic, ROGUE, to accurately quantify the purity of identified cell clusters."
- [abstract] Re-clustering discovers pure subtypes: "Applying this metric to fibroblast, B cell and brain data, we identify additional subtypes"
- [discussion] Workflow: quantify purity before and after re-clustering: "By quantifying cluster purity with ROGUE before and after clustering or re-clustering, we were able to detect low-purity clusters and perform further analysis to discover pure subtypes"
- [methods] Filter step: "matr.filter(expr, min.cells, min.genes)"
- [methods] Purity computation step: "CalculateRogue(.x, platform, cutoff, k, features)"
- [discussion] Threshold caveat: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
- [readme] Install from GitHub: "devtools::install_github("PaulingLiu/ROGUE")"
