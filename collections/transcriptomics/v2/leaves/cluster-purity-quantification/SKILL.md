---
name: cluster-purity-quantification
description: Use when a scRNA-seq clustering partition must be judged for homogeneity
  — e.g. to decide whether a cluster needs re-clustering, to verify purity of identified
  subtypes (ROGUE > 0.92 threshold), or to assess purity loss from batch aggregation.
when_to_use_negative:
- Assessing purity of cell populations integrated from different protocols/platforms
  — the article states current ROGUE does not yet support this
- Low-quality or continuous datasets where the 0.9 threshold is inappropriate — the
  threshold must instead be determined from global ROGUE values
- Selecting rare cell types from raw data alone (use GiniClust-type methods); ROGUE
  quantifies purity of already-defined clusters rather than performing the initial
  rare-type detection
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: ROGUE
  role: Core entropy-based statistic and R package providing matr.filter, entropy_fit,
    DetermineK, Entropy, and CalculateRogue functions to quantify cluster purity
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Dependency R package that must be installed before ROGUE
- name: devtools
  role: Used to install ROGUE from GitHub via install_github
- name: R
  role: Runtime environment in which the ROGUE pipeline is executed
provenance:
  source_task_ids:
  - task_008
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:3cc4656c63cfd927dc0a247b7448bec318b58c072f96ad1e78368f3017ffca23
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
yes_rate: 0.4
n_claims: 10
---

# cluster-purity-quantification

## Summary

Quantify the purity/homogeneity of single-cell clusters using ROGUE (Ratio of Global Unshifted Entropy), an entropy-based statistic that measures the randomness of gene expression while accounting for technical factors, enabling detection of low-purity clusters and discovery of pure subtypes.

## When to use

Use when a scRNA-seq clustering partition must be judged for homogeneity — e.g. to decide whether a cluster needs re-clustering, to verify purity of identified subtypes (ROGUE > 0.92 threshold), or to assess purity loss from batch aggregation.

## When NOT to use

- Assessing purity of cell populations integrated from different protocols/platforms — the article states current ROGUE does not yet support this
- Low-quality or continuous datasets where the 0.9 threshold is inappropriate — the threshold must instead be determined from global ROGUE values
- Selecting rare cell types from raw data alone (use GiniClust-type methods); ROGUE quantifies purity of already-defined clusters rather than performing the initial rare-type detection

## Inputs

- Gene-by-cell scRNA-seq UMI count matrix (e.g. from 10x, GEO accession such as GSE96583 or GSE131907)
- Cell metadata / cluster assignments per cell (batch, condition, patient, or subtype labels)
- Optional: down-sampled or simulated expression matrices for robustness analyses

## Outputs

- Per-cluster ROGUE purity values (0–1 scale; ~0.99–1 for pure, low values for mixed clusters)
- Entropy-fit parameter S and selected optimal cluster number k from DetermineK
- Results tables (e.g. rogue_values.csv, rogue_pbmc_results.csv) with cluster assignments, marker genes, and ROGUE values
- Identified low-purity clusters and additional pure subtypes from ROGUE-guided re-clustering

## How to apply

Install tidyverse then ROGUE via devtools::install_github('PaulingLiu/ROGUE') in R. Preprocess the gene-by-cell UMI count matrix with matr.filter(expr, min.cells, min.genes) to retain adequately expressed genes. Estimate the entropic mean-expression/variance fit with entropy_fit(.x, span, mt.method) and run DetermineK(expr, span, r, mt.method, if.adj) to select variable genes and the optimal cluster number k. Compute per-cluster purity with CalculateRogue(.x, platform, cutoff, k, features), setting the platform argument to match the sequencing protocol (e.g. UMI-based 10x); values near 1 indicate a pure population and ROGUE > 0.9 is the recommended purity threshold for standard data. To detect hidden heterogeneity, re-cluster low-ROGUE clusters and recompute ROGUE; to assess batch effects, compare ROGUE of aggregated populations against batch- or patient-specific populations, where systematic purity reduction on aggregation indicates batch confounding.

## Related tools

- **ROGUE** (Core entropy-based statistic and R package providing matr.filter, entropy_fit, DetermineK, Entropy, and CalculateRogue functions to quantify cluster purity) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Dependency R package that must be installed before ROGUE)
- **devtools** (Used to install ROGUE from GitHub via install_github)
- **R** (Runtime environment in which the ROGUE pipeline is executed)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- Pure simulated two-replicate populations yield ROGUE ~0.99–1 across 2- to 100-fold sequencing-depth differences, while silhouette coefficients span only 0.25–0.75
- Pearson correlations of the entropy-fit parameter S across cell-number down-sampling steps exceed 0.99
- For five pure B-cell subtypes identified by ROGUE-guided clustering, per-cluster ROGUE values are > 0.92, while DEs/doublet-like and germinal center clusters show low ROGUE
- Aggregated populations across batches or across all eight individuals receive systematically lower ROGUE values than batch- or patient-specific populations for each cell type
- Results table contains ROGUE values for every cell type under both conditions, and purity-reduction patterns match the manuscript figures

## Limitations

- The recommended 0.9 purity threshold may not suit low-quality or continuous data, where it must be determined by considering the global ROGUE values
- Current ROGUE does not assess purity of integrated cell populations from different protocols and platforms; this is proposed as future work
- ROGUE values should be compared on down-sampled matrices equalized to the same cell number per cell type to remove cell-count confounding

## Evidence

- [methods] introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population: "introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population"
- [methods] CalculateRogue signature: "CalculateRogue(.x, platform, cutoff, k, features)"
- [methods] install_github instruction: "devtools::install_github("PaulingLiu/ROGUE")"
- [intro] proposed the Ratio of Global Unshifted Entropy statistic: "we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population while accounting for other technical factors"
- [readme] reproduction scripts location: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
