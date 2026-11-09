---
name: auc-distribution-comparison-across-methods
description: Use when several gene/feature-selection methods must be ranked on the
  same synthetic datasets with known ground truth, e.g. simulated 20,000 gene x 2,000
  cell NB/ZINB count matrices with embedded impure subpopulations.
when_to_use_negative:
- Evaluation data lack ground-truth informative genes or simulated subpopulation labels,
  so AUC cannot be defined
- Methods were run on differently preprocessed or filtered matrices, breaking the
  paired same-dataset comparison
- The goal is cluster purity quantification of real data (use ROGUE's CalculateRogue
  on actual clusters, not this simulation benchmark)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_2229
tools:
- name: ROGUE
  role: Implements the S–E entropy model via entropy_fit and matr.filter for the method
    under test on each simulated dataset
  repo: https://github.com/PaulingLiu/ROGUE
- name: R
  role: Environment for running reproduction scripts, simulation, AUC computation,
    and aggregation
- name: tidyverse
  role: Dependency for data aggregation and plotting of AUC distributions
- name: devtools
  role: Used to install the ROGUE R package from GitHub
- name: GiniClust
  role: Competing Gini-based feature-selection method evaluated on the same simulated
    matrices
  repo: https://github.com/lanjiangboston/GiniClust
- name: RaceID3
  role: Competing outlier-based method evaluated on the same simulated matrices
  repo: https://github.com/dgrun/RaceID3_StemID2_package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:ed1af8917e8ac0cda4afabb45b3ad0d87861945dbe24772c61f16190a2b58b26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - GiniClust
  - R
  - ROGUE
  - RaceID3
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# auc-distribution-comparison-across-methods

## Summary

Benchmarks competing feature/gene-selection methods by computing per-dataset AUC on identical simulated inputs (here NB and ZINB scRNA-seq count matrices), aggregating scores, and comparing AUC distributions to confirm which method (e.g. the S–E entropy model) attains the highest average AUC as in ROGUE Fig. 1d.

## When to use

Use when several gene/feature-selection methods must be ranked on the same synthetic datasets with known ground truth, e.g. simulated 20,000 gene x 2,000 cell NB/ZINB count matrices with embedded impure subpopulations.

## When NOT to use

- Evaluation data lack ground-truth informative genes or simulated subpopulation labels, so AUC cannot be defined
- Methods were run on differently preprocessed or filtered matrices, breaking the paired same-dataset comparison
- The goal is cluster purity quantification of real data (use ROGUE's CalculateRogue on actual clusters, not this simulation benchmark)

## Inputs

- 1600 simulated scRNA-seq count matrices (20,000 genes x 2,000 cells) from NB and ZINB generative models with impure subpopulation fractions of 50%, 20%, 10%, and 1%
- Ground-truth labels of informative/variable genes per simulated dataset
- Feature-selection method outputs (gene rankings or scores) from S–E/ROGUE, HVG, Gini, M3Drop, SCTransform, Fano factor, and RaceID3

## Outputs

- Per-method, per-condition AUC scores for each simulated dataset
- Aggregated benchmark table (auc_benchmark.csv)
- AUC distribution plots per method (paper Fig. 1d-style comparison)

## How to apply

Simulate evaluation datasets under matched generative models (NB and ZINB), embedding impure subpopulations at known fractions (50%, 20%, 10%, 1%) so AUC against ground-truth informative genes is well-defined. Run every competing method (S–E/ROGUE, HVG, Gini, M3Drop, SCTransform, Fano factor, RaceID3) on the identical simulated count matrices so comparisons are paired. For ROGUE, apply entropy_fit and matr.filter to select variable genes per dataset. Compute per-dataset, per-condition AUC using the known simulated variable genes as positives, then aggregate into a results table and plot AUC distributions per method; export the table (auc_benchmark.csv) and check that the claimed method has the highest average AUC (here the S–E model, Fig. 1d).

## Related tools

- **ROGUE** (Implements the S–E entropy model via entropy_fit and matr.filter for the method under test on each simulated dataset) — https://github.com/PaulingLiu/ROGUE
- **R** (Environment for running reproduction scripts, simulation, AUC computation, and aggregation)
- **tidyverse** (Dependency for data aggregation and plotting of AUC distributions)
- **devtools** (Used to install the ROGUE R package from GitHub)
- **GiniClust** (Competing Gini-based feature-selection method evaluated on the same simulated matrices) — https://github.com/lanjiangboston/GiniClust
- **RaceID3** (Competing outlier-based method evaluated on the same simulated matrices) — https://github.com/dgrun/RaceID3_StemID2_package

## Examples

```
devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- Every method is evaluated on the identical 1600 simulated NB/ZINB datasets (20,000 genes x 2,000 cells) — the paired design must hold
- S–E model attains the highest average AUC across subpopulation-fraction conditions (50%/20%/10%/1%), matching Fig. 1d
- auc_benchmark.csv is produced with per-dataset AUC aggregated per method and condition
- AUC values fall within [0,1] for all methods and conditions
- AUC distribution plots per method reproduce the paper's comparative ordering

## Limitations

- Conclusions are tied to NB/ZINB simulation assumptions with specified subpopulation proportions and abundance levels; ranking may differ on real data
- The recommended 0.9 purity threshold may not suit low-quality or continuous data — thresholds should be set from global ROGUE values
- ROGUE does not yet assess purity of integrated cell populations from different protocols/platforms

## Evidence

- [methods] matr.filter: "matr.filter(expr, min.cells, min.genes)"
- [methods] entropy_fit: "entropy_fit(.x, span, mt.method)"
- [abstract] introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population: "introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population"
- [readme] The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts): "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)"
- [discussion] the 0.9 purity threshold may not be suitable for low-quality or continuous data: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
