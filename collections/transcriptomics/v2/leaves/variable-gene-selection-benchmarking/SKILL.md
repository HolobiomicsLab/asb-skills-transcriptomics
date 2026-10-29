---
name: variable-gene-selection-benchmarking
description: Use when you need to quantitatively compare feature/gene-selection methods
  on synthetic scRNA-seq data with known informative genes, e.g. validating an entropy-based
  selector against HVG, Gini, M3Drop, SCTransform, Fano, or RaceID3 via AUC.
when_to_use_negative:
- Input is already a filtered/curated feature table or a precomputed gene ranking
  — the benchmark requires raw simulated count matrices with known informative-gene
  ground truth
- The goal is cluster-purity assessment of real single-cell populations rather than
  benchmarking gene-selection methods on synthetic data (use the ROGUE statistic directly,
  e.g. CalculateRogue, instead)
- Data are integrated multi-protocol/multi-platform populations, which ROGUE's authors
  explicitly flag as not yet supported purity-assessment targets
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ROGUE (R package)
  role: Provides the S-E entropy-based variable-gene selection routines (matr.filter,
    entropy_fit) evaluated in the benchmark
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Dependency package that must be installed before ROGUE; used for aggregation
    and plotting of the AUC results table
  repo: https://github.com/PaulingLiu/ROGUE
- name: devtools
  role: Used to install ROGUE from GitHub via devtools::install_github
  repo: https://github.com/PaulingLiu/ROGUE
- name: GiniClust
  role: Source of the competing Gini-index feature-selection method benchmarked against
    the S-E model
  repo: https://github.com/lanjiangboston/GiniClust
- name: RaceID3_StemID2_package
  role: Source of the competing RaceID3 gene-selection method evaluated on the identical
    simulated datasets
  repo: https://github.com/dgrun/RaceID3_StemID2_package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:ea40c68d9688ed108dbb0caa63dcb29aede3963259d594c95f18f16c0789a63f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - GiniClust
  - R
  - ROGUE (R package)
  - RaceID3_StemID2_package
  - devtools
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# variable-gene-selection-benchmarking

## Summary

Benchmark entropy-based S-E variable-gene selection (ROGUE's entropy_fit/matr.filter) against six competing feature-selection methods — HVG, Gini, M3Drop, SCTransform, Fano factor, RaceID3 — on scRNA-seq count matrices simulated from negative-binomial (NB) and zero-inflated negative-binomial (ZINB) generative models, scoring each method by average AUC for identifying informative genes.

## When to use

Use when you need to quantitatively compare feature/gene-selection methods on synthetic scRNA-seq data with known informative genes, e.g. validating an entropy-based selector against HVG, Gini, M3Drop, SCTransform, Fano, or RaceID3 via AUC.

## When NOT to use

- Input is already a filtered/curated feature table or a precomputed gene ranking — the benchmark requires raw simulated count matrices with known informative-gene ground truth
- The goal is cluster-purity assessment of real single-cell populations rather than benchmarking gene-selection methods on synthetic data (use the ROGUE statistic directly, e.g. CalculateRogue, instead)
- Data are integrated multi-protocol/multi-platform populations, which ROGUE's authors explicitly flag as not yet supported purity-assessment targets

## Inputs

- Simulated scRNA-seq count matrices from NB and ZINB generative models (e.g. 20,000 genes x 2000 cells, 1600 matrices)
- Embedded impure-subpopulation fractions (50%, 20%, 10%, 1%) defining ground-truth informative genes
- Reproduction scripts from the ./scripts directory of the PaulingLiu/ROGUE repository

## Outputs

- auc_benchmark.csv — aggregated AUC scores per method and subpopulation-fraction condition
- Per-method AUC distribution plots across NB and ZINB simulation conditions
- Per-dataset feature rankings from each of the seven compared gene-selection methods

## How to apply

Simulate scRNA-seq count matrices under NB and ZINB generative models (here 1600 datasets, each 20,000 genes x 2000 cells, with impure subpopulations embedded at fractions of 50%, 20%, 10%, and 1%) using the reproduction scripts in the ./scripts directory of the PaulingLiu/ROGUE repository. Apply the S-E entropy model to every simulated matrix using the ROGUE R package's matr.filter(expr, min.cells, min.genes) and entropy_fit(.x, span, mt.method) routines. Run each of the six competing methods (HVG, Gini, M3Drop, SCTransform, Fano factor, RaceID3) on the identical simulated datasets so all methods are evaluated on exactly the same synthetic matrices. Compute AUC per method and per subpopulation-fraction condition, aggregate per-dataset scores into a results table, and export it as auc_benchmark.csv. Plot AUC distributions per method and judge success by whether the S-E model attains the highest average AUC across NB and ZINB conditions, as reported in Fig. 1d.

## Related tools

- **ROGUE (R package)** (Provides the S-E entropy-based variable-gene selection routines (matr.filter, entropy_fit) evaluated in the benchmark) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Dependency package that must be installed before ROGUE; used for aggregation and plotting of the AUC results table) — https://github.com/PaulingLiu/ROGUE
- **devtools** (Used to install ROGUE from GitHub via devtools::install_github) — https://github.com/PaulingLiu/ROGUE
- **GiniClust** (Source of the competing Gini-index feature-selection method benchmarked against the S-E model) — https://github.com/lanjiangboston/GiniClust
- **RaceID3_StemID2_package** (Source of the competing RaceID3 gene-selection method evaluated on the identical simulated datasets) — https://github.com/dgrun/RaceID3_StemID2_package

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- auc_benchmark.csv contains AUC scores for all seven methods across NB and ZINB conditions and the four subpopulation fractions (50%, 20%, 10%, 1%)
- The S-E model attains the highest average AUC across all simulated NB and ZINB conditions, matching the Fig. 1d result
- All methods are evaluated on the same 1600 simulated datasets (equal evaluation sets — no method sees different synthetic matrices)
- Simulated matrices match the reported dimensions of 20,000 genes x 2000 cells with embedded impure subpopulations
- AUC distribution plots reproduce the reported separation of the S-E model from HVG, Gini, M3Drop, SCTransform, Fano, and RaceID3

## Limitations

- Dropout-based variable-gene selection approaches hinder capturing the global expression distribution, and ad hoc dimensionality-reduction-weight approaches are computationally intensive (orders of magnitude more time than HVG and M3Drop)
- Gini-based selection is designed specifically for rare cell-type identification, limiting its generality in the benchmark
- The 0.9 purity threshold may not be suitable for low-quality or continuous data, where the threshold must be determined from global ROGUE values
- Benchmark conclusions depend on the NB/ZINB simulation settings (dataset dimensions, subpopulation fractions) and may not transfer to other generative regimes

## Evidence

- [methods] Reproduction scripts: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)"
- [intro] S-E model rationale: "we present an entropy-based model to measure the randomness of gene expression in single cells"
- [intro] Competing-method limitations: "such ad hoc approaches are computationally intensive, requiring potentially orders of magnitude more time than methods like HVG and M3Drop"
- [readme] Installation command: "devtools::install_github("PaulingLiu/ROGUE")"
- [discussion] Threshold caveat: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
