---
name: entropy-based-feature-scoring
description: Use when selecting informative/variable genes from raw UMI count matrices
  (20,000 genes x 2,000 cells scale) prior to clustering, or when benchmarking feature-selection
  methods against HVG, Gini, M3Drop, SCTransform, Fano, or RaceID3.
when_to_use_negative:
- Gini-based scoring is preferred when the goal is rare cell-type identification;
  the article notes 'Gini is designed for rare cell-type identification'
- The 0.9 purity threshold is unsuitable for low-quality or continuous data — ROGUE
  thresholds must be derived from global ROGUE values instead
- Assessing purity of integrated cell populations from different protocols/platforms
  — the article lists this as unimplemented future work for ROGUE
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: ROGUE
  role: 'R package implementing the entropy-based S-E model: entropy_fit, matr.filter,
    DetermineK, CalculateRogue, and Entropy functions used for feature scoring'
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: required dependency installed before ROGUE
- name: devtools
  role: used to install ROGUE from GitHub
- name: GiniClust
  role: competing Gini-index feature-selection method benchmarked against the S-E
    model
  repo: lanjiangboston/GiniClust
- name: GiniClust2
  role: competing cluster-aware method in the feature-selection comparison
  repo: dtsoucas/GiniClust2
- name: RaceID3
  role: competing outlier/rare-cell feature-selection method benchmarked against the
    S-E model
  repo: dgrun/RaceID3_StemID2_package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:d8add6f924156ab7162dc0dc562f348454c07156c60001bd083bd48cd9754e53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - GiniClust
  - GiniClust2
  - R
  - ROGUE
  - RaceID3
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# entropy-based-feature-scoring

## Summary

Scores genes for informativeness in scRNA-seq data using an entropy-based model (the S-E model implemented in the ROGUE package), fitting per-gene entropy curves to identify variable genes that enable accurate, sensitive cluster-purity assessment and benchmark-beating feature selection.

## When to use

Use when selecting informative/variable genes from raw UMI count matrices (20,000 genes x 2,000 cells scale) prior to clustering, or when benchmarking feature-selection methods against HVG, Gini, M3Drop, SCTransform, Fano, or RaceID3.

## When NOT to use

- Gini-based scoring is preferred when the goal is rare cell-type identification; the article notes 'Gini is designed for rare cell-type identification'
- The 0.9 purity threshold is unsuitable for low-quality or continuous data — ROGUE thresholds must be derived from global ROGUE values instead
- Assessing purity of integrated cell populations from different protocols/platforms — the article lists this as unimplemented future work for ROGUE

## Inputs

- gene-by-cell scRNA-seq UMI count matrix (e.g. 20,000 genes x 2,000 cells)
- simulated NB or ZINB count matrices with embedded impure subpopulations at 50%, 20%, 10%, and 1% fractions

## Outputs

- per-gene entropy-fit scores / ranked variable-gene features
- ROGUE purity statistic per cluster (CalculateRogue output)
- AUC per feature-selection method aggregated into auc_benchmark.csv and AUC distribution plots (Fig. 1d)

## How to apply

Install ROGUE from GitHub via devtools::install_github('PaulingLiu/ROGUE') after installing tidyverse. Run the ROGUE pipeline per dataset: first apply matr.filter(expr, min.cells, min.genes) to retain adequately expressed genes, then fit per-gene entropy with entropy_fit(.x, span, mt.method) using DetermineK for k estimation and Entropy for per-gene entropy. Aggregate filtered/fit results and rank genes by the entropy-based statistic; for cluster-purity assessment feed the scored features into CalculateRogue(.x, platform, cutoff, k, features) to obtain the ROGUE value. The rationale, per the AUC benchmark (Fig. 1d), is that the S-E model consistently achieves the highest average AUC versus HVG, Gini, M3Drop, SCTransform, Fano factor, and RaceID3 on NB/ZINB-simulated data with subpopulation fractions of 50%, 20%, 10%, and 1%.

## Related tools

- **ROGUE** (R package implementing the entropy-based S-E model: entropy_fit, matr.filter, DetermineK, CalculateRogue, and Entropy functions used for feature scoring) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (required dependency installed before ROGUE)
- **devtools** (used to install ROGUE from GitHub)
- **GiniClust** (competing Gini-index feature-selection method benchmarked against the S-E model) — lanjiangboston/GiniClust
- **GiniClust2** (competing cluster-aware method in the feature-selection comparison) — dtsoucas/GiniClust2
- **RaceID3** (competing outlier/rare-cell feature-selection method benchmarked against the S-E model) — dgrun/RaceID3_StemID2_package

## Examples

```
devtools::install_github("PaulingLiu/ROGUE"); SE_genes <- entropy_fit(matr.filter(expr, min.cells, min.genes), span = 0.55); ROGUE <- CalculateRogue(.x, platform = "UMI", cutoff = 0.9)
```

## Evaluation signals

- S-E model attains the highest average AUC across all simulated NB and ZINB conditions (Fig. 1d benchmark)
- Benchmark covers 1600 simulated count matrices per generative model with subpopulation fractions of 50%, 20%, 10%, and 1%
- Per-dataset AUC scores aggregated into auc_benchmark.csv with per-method AUC distributions plotted
- ROGUE metric judged accurate, sensitive, and robust on simulated and real datasets per abstract claim
- Selected features yield higher ROGUE purity values after re-clustering of low-purity clusters

## Limitations

- The recommended 0.9 purity threshold may not suit low-quality or continuous data; thresholds must be derived from global ROGUE values
- ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms (proposed future work)
- Dropout-based approaches hinder capturing the global expression distribution — the S-E model addresses this but depends on fitting span and k parameters

## Evidence

- [methods] entropy_fit routine: "entropy_fit(.x, span, mt.method)"
- [methods] filter routine: "matr.filter(expr, min.cells, min.genes)"
- [readme] installation: "devtools::install_github("PaulingLiu/ROGUE")"
- [abstract] ROGUE purpose: "Here we present an entropy-based statistic, ROGUE, to accurately quantify the purity of identified cell clusters."
- [methods] benchmark script location: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)"
- [discussion] purity threshold caveat: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
