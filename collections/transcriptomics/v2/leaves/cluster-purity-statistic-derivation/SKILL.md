---
name: cluster-purity-statistic-derivation
description: Use when you need a quantitative purity score for an identified scRNA-seq
  cell cluster — e.g. to decide whether to re-cluster — from a gene-by-cell UMI count
  matrix annotated with a platform (droplet or full-length).
when_to_use_negative:
- Assessing purity of integrated cell populations from different protocols and platforms
  — the article states current ROGUE does not support this (proposed as future work)
- Low-quality or continuous data where the 0.9 purity threshold does not apply without
  considering global ROGUE values
- Detecting rare cell types or performing the clustering itself — use methods like
  GiniClust, GiniClust2, or RaceID3/StemID2 instead; ROGUE quantifies purity of already-identified
  clusters
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: ROGUE
  role: R package implementing the entropy model, DetermineK, and CalculateRogue functions
    that produce the purity statistic
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: R dependency package that must be installed before installing ROGUE
- name: devtools
  role: Used to install ROGUE from GitHub via devtools::install_github("PaulingLiu/ROGUE")
- name: R
  role: Runtime environment for the ROGUE implementation and analysis scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:44e3d4f7f7937641d5a051adc1ca2ad75200bbac18607566855271e64cef0d95
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
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cluster-purity-statistic-derivation

## Summary

Derives the ROGUE (Ratio of Global Unshifted Entropy) cluster-purity statistic by fitting an entropy-based S-E model (LOESS of ln E(X) vs average expression) to a scRNA-seq expression matrix, extracting per-gene entropy-reduction values (ds), and aggregating them as ROGUE = 1 - sum(ds)/(sum(ds) + K) to quantify whether a cell cluster is a pure population.

## When to use

Use when you need a quantitative purity score for an identified scRNA-seq cell cluster — e.g. to decide whether to re-cluster — from a gene-by-cell UMI count matrix annotated with a platform (droplet or full-length).

## When NOT to use

- Assessing purity of integrated cell populations from different protocols and platforms — the article states current ROGUE does not support this (proposed as future work)
- Low-quality or continuous data where the 0.9 purity threshold does not apply without considering global ROGUE values
- Detecting rare cell types or performing the clustering itself — use methods like GiniClust, GiniClust2, or RaceID3/StemID2 instead; ROGUE quantifies purity of already-identified clusters

## Inputs

- gene-by-cell scRNA-seq UMI/raw count expression matrix (CSV or R matrix)
- cell metadata with cluster labels and sample ids
- platform annotation: 'droplet' or 'full-length'
- LOESS span parameter (tunable, used in SE_fun/entropy_fit/DetermineK)
- multiple-testing method (mt.method) and adjustment flag (if.adj)
- gene retention cutoff and matr.filter parameters (min.cells, min.genes)

## Outputs

- per-gene entropy-reduction statistics (ds) with multiple-testing-adjusted significance
- scalar ROGUE purity score per cell population/cluster (1 = completely pure)
- fitted LOESS entropy model (ln E(X) vs average-expression-derived entropy) and estimated K constant

## How to apply

Compute single-gene sample entropy under a Poisson-Gamma/negative-binomial model with Entropy(expr, r), yielding shifted entropy S = ln E(X) + a. Fit a LOESS regression (tunable span) of ln E(X) against average-expression-derived entropy via SE_fun/entropy_fit to obtain the residual entropy-reduction statistic ds per gene, applying multiple-testing correction controlled by mt.method/if.adj. Estimate the platform-specific constant K with DetermineK (K=45 for platform='droplet', K=500 for platform='full-length'). Aggregate retained genes (respecting the cutoff and matr.filter min.cells/min.genes) with CalculateRogue so that a population with no significant ds for any gene receives ROGUE = 1 (completely pure); lower values indicate heterogeneity. Optionally, before installing ROGUE, install the 'tidyverse' dependency package.

## Related tools

- **ROGUE** (R package implementing the entropy model, DetermineK, and CalculateRogue functions that produce the purity statistic) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (R dependency package that must be installed before installing ROGUE)
- **devtools** (Used to install ROGUE from GitHub via devtools::install_github("PaulingLiu/ROGUE"))
- **R** (Runtime environment for the ROGUE implementation and analysis scripts)

## Examples

```
install.packages("tidyverse"); devtools::install_github("PaulingLiu/ROGUE"); CalculateRogue(expr, platform="droplet", cutoff=1, mt.method="BH", if.adj=TRUE)
```

## Evaluation signals

- On a test matrix with cell labels, sample ids, and platform annotation, module outputs must numerically match the reference ROGUE R package results on the same data
- A population with no significant ds for any gene must yield ROGUE = 1 (completely pure subtype/state)
- Check that K is set to 45 when platform='droplet' and 500 when platform='full-length', and that the cutoff correctly limits retained genes/features
- Re-clustering a low-ROGUE cluster should yield subpopulations with higher ROGUE values, consistent with the discovery of pure subtypes in fibroblast, B cell and brain data
- The metric should be broadly applicable across droplet and full-length scRNA-seq datasets, as demonstrated on simulated and real datasets

## Limitations

- Current ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms; this is proposed as future work
- The recommended 0.9 purity threshold may not be suitable for low-quality or continuous data, where it must be determined from global ROGUE values
- Existing approaches (silhouette, DendroSplit, distance ratio) for cluster number determination are not comparable among datasets — ROGUE is positioned as a complement for purity, not a full clustering solution

## Evidence

- [abstract] abstract: "Here we present an entropy-based statistic, ROGUE, to accurately quantify the purity of identified cell clusters."
- [methods] methods-loess: "entropy_fit(.x, span, mt.method)"
- [methods] methods-calc: "CalculateRogue(.x, platform, cutoff, k, features)"
- [readme] readme-install: "Before installing ROGUE, the “tidyverse” package should be installed first"
- [methods] methods-repo: "devtools::install_github("PaulingLiu/ROGUE")"
- [discussion] discussion-threshold: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
