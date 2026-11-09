---
name: negative-binomial-zinb-expression-modeling
description: Use when a feature-selection or variable-gene method needs fair performance
  comparison on controlled synthetic scRNA-seq data, especially where subpopulations
  at fractions of 50%, 20%, 10%, and 1% must be embedded to test sensitivity to rare
  cell types.
when_to_use_negative:
- Data are real, unlabelled scRNA-seq matrices where the goal is cluster purity assessment
  (ROGUE value per cluster) rather than synthetic ground-truth benchmarking
- The analysis goal is detection of rare cell types in a single dataset (e.g. GiniClust's
  design purpose) rather than method-level AUC comparison across simulated conditions
- Only one feature-selection method is of interest and there is no comparator set
  — the skill's value comes from evaluating all seven methods on identical synthetic
  datasets
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: ROGUE (R package)
  role: Provides entropy_fit and matr.filter routines used to apply the S–E model
    to each simulated dataset; installed via devtools from GitHub
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Required dependency installed before ROGUE; used for data manipulation and
    plotting of AUC distributions
- name: devtools
  role: Installs the ROGUE package from GitHub
- name: GiniClust
  role: Competing Gini-index-based feature-selection method evaluated on the same
    simulated matrices
  repo: https://github.com/lanjiangboston/GiniClust
- name: GiniClust2
  role: Competing cluster-aware weighted consensus clustering method in the benchmark
  repo: https://github.com/dtsoucas/GiniClust2
- name: RaceID3
  role: Competing rare-cell feature-selection/clustering method evaluated on the same
    simulated datasets
  repo: https://github.com/dgrun/RaceID3_StemID2_package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:cce5f63e0f22b502d38c1a9df1d755af01e6e13dc0403777df7a60880a1d84cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - GiniClust
  - GiniClust2
  - R
  - ROGUE (R package)
  - RaceID3
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# negative-binomial-zinb-expression-modeling

## Summary

Simulate large scRNA-seq count matrices from negative-binomial (NB) and zero-inflated negative-binomial (ZINB) generative models with embedded impure subpopulations, in order to benchmark feature-selection methods (S–E, HVG, Gini, M3Drop, SCTransform, Fano factor, RaceID3) by AUC as done in the ROGUE paper.

## When to use

Use when a feature-selection or variable-gene method needs fair performance comparison on controlled synthetic scRNA-seq data, especially where subpopulations at fractions of 50%, 20%, 10%, and 1% must be embedded to test sensitivity to rare cell types.

## When NOT to use

- Data are real, unlabelled scRNA-seq matrices where the goal is cluster purity assessment (ROGUE value per cluster) rather than synthetic ground-truth benchmarking
- The analysis goal is detection of rare cell types in a single dataset (e.g. GiniClust's design purpose) rather than method-level AUC comparison across simulated conditions
- Only one feature-selection method is of interest and there is no comparator set — the skill's value comes from evaluating all seven methods on identical synthetic datasets

## Inputs

- Simulated scRNA-seq count matrices from NB and ZINB generative models (20,000 genes × 2000 cells)
- Subpopulation proportion conditions: 50%, 20%, 10%, and 1% impure subpopulations
- ROGUE R package routines: entropy_fit, matr.filter, DetermineK, CalculateRogue, Entropy
- Competing feature-selection methods: HVG, GiniClust (Gini), M3Drop, SCTransform, Fano factor, RaceID3

## Outputs

- Aggregated AUC benchmark table (auc_benchmark.csv) with per-method, per-condition scores
- AUC distribution plots per method (corresponding to Fig. 1d of the manuscript)
- Per-dataset AUC scores for each method and subpopulation-fraction condition

## How to apply

Generate 1600 simulated scRNA-seq count matrices (20,000 genes × 2000 cells) split across NB and ZINB generative models, embedding impure subpopulations at fractions of 50%, 20%, 10%, and 1%. Run the entropy-based S–E model on each matrix using the ROGUE R package's entropy_fit and matr.filter routines (preceded by matr.filter(expr, min.cells, min.genes) for expression filtering). Run the six competing methods (HVG, Gini, M3Drop, SCTransform, Fano factor, RaceID3) on the identical simulated matrices so every method sees the same data. Compute per-dataset AUC per method and subpopulation-fraction condition, aggregate into a results table, and plot AUC distributions. The rationale is that simulation from NB/ZINB generative models gives known ground truth about which genes are informative, allowing AUC comparison that is impossible on unlabelled real data; reproduction scripts live in the ./scripts directory of PaulingLiu/ROGUE.

## Related tools

- **ROGUE (R package)** (Provides entropy_fit and matr.filter routines used to apply the S–E model to each simulated dataset; installed via devtools from GitHub) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Required dependency installed before ROGUE; used for data manipulation and plotting of AUC distributions)
- **devtools** (Installs the ROGUE package from GitHub)
- **GiniClust** (Competing Gini-index-based feature-selection method evaluated on the same simulated matrices) — https://github.com/lanjiangboston/GiniClust
- **GiniClust2** (Competing cluster-aware weighted consensus clustering method in the benchmark) — https://github.com/dtsoucas/GiniClust2
- **RaceID3** (Competing rare-cell feature-selection/clustering method evaluated on the same simulated datasets) — https://github.com/dgrun/RaceID3_StemID2_package

## Examples

```
install.packages("tidyverse"); if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- The S–E model attains the highest average AUC across all NB and ZINB conditions, matching the reported Fig. 1d result
- All seven methods are evaluated on the same 1600 simulated count matrices (no dataset leakage between methods)
- auc_benchmark.csv contains per-method, per-subpopulation-fraction (50%/20%/10%/1%) AUC entries for both NB and ZINB generative models
- Simulated matrices have the specified dimensions (20,000 genes × 2000 cells) with impure subpopulations at the specified fractions
- Competing methods reproduce their documented behavior (e.g. Gini selects rare-cell genes, HVG/M3Drop run orders of magnitude faster than ad hoc approaches)

## Limitations

- Results are conditional on the NB/ZINB generative assumptions; performance on real datasets may differ from the simulated AUC ranking
- The manuscript notes existing methods have design mismatches — e.g. scran may select many lowly expressed genes and Gini is designed for rare cell-type identification — so benchmark AUC partly reflects scope, not just quality
- ROGUE purity assessment (and by extension downstream selection) is calibrated for standard data; for low-quality or continuous data thresholds must be determined from global ROGUE values rather than the default 0.9 cutoff
- The 0.9 purity threshold may not be suitable for low-quality or continuous data

## Evidence

- [abstract] ROGUE validated on simulated and real datasets: "We demonstrate that our ROGUE metric is broadly applicable, and enables accurate, sensitive and robust assessment of cluster purity on a wide range of simulated and real datasets."
- [methods] Reproduction scripts location: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)"
- [methods] Expression filtering routine: "matr.filter(expr, min.cells, min.genes)"
- [intro] Speed comparison motivating HVG/M3Drop baselines: "such ad hoc approaches are computationally intensive, requiring potentially orders of magnitude more time than methods like HVG and M3Drop"
- [readme] Installation dependency from README: "Before installing ROGUE, the “tidyverse” package should be installed first"
