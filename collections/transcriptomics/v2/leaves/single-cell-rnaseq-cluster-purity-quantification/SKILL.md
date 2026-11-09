---
name: single-cell-rnaseq-cluster-purity-quantification
description: Use when scRNA-seq clusters need a quantitative purity score (e.g. ROGUE
  > 0.9 as pure) or when low-purity clusters (ROGUE < 0.8) must be detected and re-clustered
  into refined subtypes.
when_to_use_negative:
- The goal is assessing purity of cell populations integrated across different protocols
  and platforms — the article states ROGUE does not yet support integrated populations
  and lists this as future work
- Data are low-quality or continuous, where the 0.9 purity threshold is unsuitable
  and must instead be derived from global ROGUE values
- The task is detecting rare cell types from scratch rather than scoring purity of
  an existing clustering partition (clustering tools like GiniClust, GiniClust2, or
  RaceID3 are the appropriate alternatives)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: ROGUE
  role: Core entropy-based statistic and R package; provides matr.filter, entropy_fit,
    DetermineK, Entropy, CalculateRogue, and the rogue() entry point for computing
    cluster purity
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Dependency R package that must be installed before ROGUE
- name: devtools
  role: Used to install the ROGUE package from GitHub
- name: R
  role: Runtime environment in which the ROGUE pipeline is executed
provenance:
  source_task_ids:
  - task_007
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:f86ff2d589dc16966e86eaf4223eb6969499fc25f2bb34b1bada57957f0943ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-rnaseq-cluster-purity-quantification

## Summary

Quantifies the purity/homogeneity of scRNA-seq cell clusters using ROGUE, an entropy-based Ratio of Global Unshifted Entropy statistic implemented in the open-source R package PaulingLiu/ROGUE, and uses low ROGUE values to guide re-clustering into purer subtypes.

## When to use

Use when scRNA-seq clusters need a quantitative purity score (e.g. ROGUE > 0.9 as pure) or when low-purity clusters (ROGUE < 0.8) must be detected and re-clustered into refined subtypes.

## When NOT to use

- The goal is assessing purity of cell populations integrated across different protocols and platforms — the article states ROGUE does not yet support integrated populations and lists this as future work
- Data are low-quality or continuous, where the 0.9 purity threshold is unsuitable and must instead be derived from global ROGUE values
- The task is detecting rare cell types from scratch rather than scoring purity of an existing clustering partition (clustering tools like GiniClust, GiniClust2, or RaceID3 are the appropriate alternatives)

## Inputs

- Raw gene-by-cell scRNA-seq UMI count matrix (droplet or UMI platform, e.g. Tabula Muris brain GSE109774, ERCC spike-in matrix GSE118127, FACS-purified PBMC matrices, CRC T-cell or lung-tumor stromal matrices)
- Per-cell cluster label vector (existing clustering partition)
- Per-cell sample/batch identifier vector
- Optional platform string ('droplet' or UMI settings) and parameters min.cells, min.genes, span, r, mt.method, k

## Outputs

- Per-cluster ROGUE purity scores (expected ~1 for ERCC-only control, ~0.9-1 for pure subtypes, ~0.78 for the unsorted CRC T-cell population)
- Set of significantly entropic (S-E) variable genes from entropy_fit/DetermineK
- Re-clustered subtype partitions with per-subtype ROGUE values for previously impure clusters (e.g. 10 oligodendrocyte subtypes from 3401 cells; 11 fibroblast subtypes from 7 CAF clusters)
- Results table (e.g. rogue_values.csv, caf_rogue_subtypes.csv) with dataset, population/subtype, and ROGUE columns

## How to apply

Install tidyverse, then the ROGUE package via devtools::install_github("PaulingLiu/ROGUE"). Filter the gene-by-cell count matrix with matr.filter(expr, min.cells, min.genes), fit the expression entropy model with entropy_fit(.x, span, mt.method) and DetermineK(expr, span, r, mt.method, if.adj) to select S-E variable genes, then call the rogue() entry point (platform = 'droplet' or UMI-appropriate settings, plus labels and sample vectors) to obtain per-cluster ROGUE. Treat clusters with ROGUE < 0.8 as impure versus the ~0.9-1 purity benchmark, extract those cells, and re-cluster (e.g. Seurat normalization with negative-binomial regression, BBKNN batch correction, leiden clustering, UMAP visualization) before recomputing ROGUE with CalculateRogue(.x, platform, cutoff, k, features) to confirm each new subtype reaches ~0.9-1. The rationale is that ROGUE measures the randomness of gene expression across cells within a cluster while accounting for technical factors, giving a comparable and interpretable purity measure that manual marker-gene checking and silhouette/DendroSplit-style criteria lack.

## Related tools

- **ROGUE** (Core entropy-based statistic and R package; provides matr.filter, entropy_fit, DetermineK, Entropy, CalculateRogue, and the rogue() entry point for computing cluster purity) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Dependency R package that must be installed before ROGUE)
- **devtools** (Used to install the ROGUE package from GitHub)
- **R** (Runtime environment in which the ROGUE pipeline is executed)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE"); rogue(expr, labels, samples, platform = 'droplet', k, min.cell.n, remove.outlier.n, span, r, mt.method, filter, min.cells, min.genes)
```

## Evaluation signals

- ERCC spike-in droplet-only control (GSE118127) yields ROGUE ~1, confirming the statistic saturates on a maximally pure population
- FACS-purified PBMC populations fall in expected ranges: CD4/CD8 naive T cells at 0.94-1, CD14 monocytes and CD34+ cells at ~0.8
- The six well-defined CRC T-cell subtypes each exceed ROGUE 0.9 while the complete unsorted T-cell dataset scores ~0.78
- ROGUE-guided re-clustering of 3401 Tabula Muris brain oligodendrocytes resolves 10 subtypes with 9 of 10 (all but cluster 6) at ROGUE ~0.9-1
- Re-clustering the 7 low-purity lung-tumor CAF clusters yields 11 subtypes with higher average ROGUE, including myCAF splits at ~0.84 (BF_C01_RGS5), ~0.87 (BF_C02_ACTA2), and ~0.94 (BF_C03_GPX3), plus an apCAF cluster marked by CD74/MHC-II

## Limitations

- The recommended 0.9 purity threshold is unsuitable for low-quality or continuous data; there the threshold must be determined from global ROGUE values
- ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms; this is proposed as future work
- Existing purity approaches that ROGUE replaces (silhouette, DendroSplit, distance ratio) are not comparable among datasets and have poor interpretability, so cross-study score comparison should use ROGUE itself

## Evidence

- [abstract] ROGUE definition and purpose: "Here we present an entropy-based statistic, ROGUE, to accurately quantify the purity of identified cell clusters."
- [intro] ROGUE quantifies homogeneity accounting for technical factors: "we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population while accounting for other technical factors"
- [methods] Installation from GitHub: "devtools::install_github("PaulingLiu/ROGUE")"
- [methods] Filtering function signature: "matr.filter(expr, min.cells, min.genes)"
- [methods] CalculateRogue function signature: "CalculateRogue(.x, platform, cutoff, k, features)"
- [discussion] Re-clustering workflow rationale: "By quantifying cluster purity with ROGUE before and after clustering or re-clustering, we were able to detect low-purity clusters and perform further analysis to discover pure subtypes"
- [readme] Manuscript reproduction scripts: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
