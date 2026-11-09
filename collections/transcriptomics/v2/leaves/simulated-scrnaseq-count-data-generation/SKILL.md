---
name: simulated-scrnaseq-count-data-generation
description: Use when you need to benchmark gene/feature-selection methods on synthetic
  scRNA-seq count data with known informative genes and controlled subpopulation rarity.
when_to_use_negative:
- Input is real (non-simulated) scRNA-seq data where the goal is selecting variable
  genes on actual expression matrices rather than benchmarking against known ground
  truth
- The question is assessing purity of already-defined cell clusters (use ROGUE's CalculateRogue/Entropy
  on a real cluster, not the NB/ZINB simulation benchmark)
- The data are not raw UMI count matrices from NB/ZINB-like generative processes (e.g.
  log-transformed qPCR-style data)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ROGUE
  role: R package providing the S-E entropy-based variable-gene selection (entropy_fit,
    matr.filter) applied to each simulated NB/ZINB dataset and whose scripts generate
    the simulations
  repo: https://github.com/PaulingLiu/ROGUE
- name: R
  role: Runtime environment in which the 1600 NB/ZINB simulations and all method evaluations
    are executed
- name: devtools
  role: Used to install ROGUE from GitHub before running the simulation and benchmark
    scripts
- name: GiniClust
  role: Competing Gini-index-based feature-selection method evaluated on the identical
    simulated datasets
  repo: https://github.com/lanjiangboston/GiniClust
- name: RaceID3
  role: Competing rare-cell feature-selection method evaluated on the same synthetic
    matrices
  repo: https://github.com/dgrun/RaceID3_StemID2_package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:9fa970cbb6812b285290d9b61aaddb328e0b89c7616d6513a534b4f7f7476cb3
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# simulated-scrnaseq-count-data-generation

## Summary

Generates synthetic scRNA-seq count matrices from negative-binomial (NB) and zero-inflated negative-binomial (ZINB) generative models so that feature-selection methods such as the S-E entropy model, HVG, Gini, M3Drop, SCTransform, Fano factor, and RaceID3 can be compared with ground-truth informative genes.

## When to use

Use when you need to benchmark gene/feature-selection methods on synthetic scRNA-seq count data with known informative genes and controlled subpopulation rarity.

## When NOT to use

- Input is real (non-simulated) scRNA-seq data where the goal is selecting variable genes on actual expression matrices rather than benchmarking against known ground truth
- The question is assessing purity of already-defined cell clusters (use ROGUE's CalculateRogue/Entropy on a real cluster, not the NB/ZINB simulation benchmark)
- The data are not raw UMI count matrices from NB/ZINB-like generative processes (e.g. log-transformed qPCR-style data)

## Inputs

- Simulation parameters: NB and ZINB generative models, 20,000 genes x 2000 cells per matrix, impure subpopulation fractions of 50%, 20%, 10%, 1%
- Reproduction scripts from the ./scripts directory of the PaulingLiu/ROGUE repository
- R environment with tidyverse and devtools, ROGUE installed from GitHub

## Outputs

- 1600 simulated NB and ZINB scRNA-seq count matrices (20,000 genes x 2000 cells each) with embedded impure subpopulations
- Per-method, per-condition AUC scores aggregated into auc_benchmark.csv
- AUC distribution plots per feature-selection method (as reported in Fig. 1d)

## How to apply

Simulate 1600 scRNA-seq count matrices under the negative-binomial and zero-inflated negative-binomial (ZINB) generative models, each with dimensions 20,000 genes x 2000 cells, embedding impure subpopulations at fractions of 50%, 20%, 10%, and 1%. Use the reproduction scripts in the ./scripts directory of the PaulingLiu/ROGUE repository, which the paper states contain the code for producing all quantitative results (Fig. 1d AUC benchmark). Apply the S-E model (ROGUE entropy-based variable-gene selection, via the entropy_fit and matr.filter routines) and the six competing feature-selection methods on the identical simulated datasets so every method is evaluated on the same synthetic matrices. Compute AUC per method and subpopulation-fraction condition, aggregate per-dataset scores, and save the aggregated table as auc_benchmark.csv. Install the R environment first: tidyverse, then ROGUE via devtools::install_github("PaulingLiu/ROGUE").

## Related tools

- **ROGUE** (R package providing the S-E entropy-based variable-gene selection (entropy_fit, matr.filter) applied to each simulated NB/ZINB dataset and whose scripts generate the simulations) — https://github.com/PaulingLiu/ROGUE
- **R** (Runtime environment in which the 1600 NB/ZINB simulations and all method evaluations are executed)
- **devtools** (Used to install ROGUE from GitHub before running the simulation and benchmark scripts)
- **GiniClust** (Competing Gini-index-based feature-selection method evaluated on the identical simulated datasets) — https://github.com/lanjiangboston/GiniClust
- **RaceID3** (Competing rare-cell feature-selection method evaluated on the same synthetic matrices) — https://github.com/dgrun/RaceID3_StemID2_package

## Examples

```
install.packages("tidyverse"); if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- Each simulated matrix has the expected dimensions of 20,000 genes x 2000 cells, with 1600 total matrices spanning NB and ZINB models and 50%, 20%, 10%, 1% impure subpopulation fractions
- All six competing methods and the S-E model are evaluated on the identical synthetic datasets (same matrix set per method)
- The aggregated auc_benchmark.csv shows the S-E model attains the highest average AUC across NB and ZINB conditions, reproducing Fig. 1d
- AUC distributions per method and per subpopulation-fraction condition are plottable and non-empty for all 1600 datasets

## Limitations

- Simulations are restricted to NB and ZINB generative models with specific subpopulation fractions (50%, 20%, 10%, 1%); other count-generating processes are not covered
- The manuscript's reproduction scripts must be located in the ./scripts directory of the ROGUE repo; alternative parameterizations require adapting those scripts
- Conclusions about method ranking hold for the simulated ground-truth setting and may not transfer to real scRNA-seq datasets

## Evidence

- [methods] Simulations use NB/ZINB generative models with controlled subpopulation fractions: "matr.filter(expr, min.cells, min.genes)"
- [methods] The paper's quantitative results, including the Fig. 1d AUC benchmark, are produced by repository scripts: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)"
- [methods] ROGUE is implemented as an open-source R package installed via devtools: "devtools::install_github("PaulingLiu/ROGUE")"
- [intro] The S-E entropy model is the statistical foundation whose performance is assessed on the simulated data: "we present an entropy-based model to measure the randomness of gene expression in single cells"
- [readme] The ROGUE package and its cluster-purity statistic are the basis for the skill's tooling: "Here, we proposed the concept of cluster purity and introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population."
