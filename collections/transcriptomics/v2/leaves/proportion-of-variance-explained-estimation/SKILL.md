---
name: proportion-of-variance-explained-estimation
description: Use when benchmarking confounding-factor inference on simulated scRNAseq
  data with known latent factors and you need to quantify latent-factor estimation
  accuracy, e.g. as a secondary panel alongside ARI across a PDE 20-100% sweep.
when_to_use_negative:
- The dataset is real (not simulated), so no ground-truth latent factors exist and
  PVE accuracy cannot be computed against true factors
- The downstream question is cell-type clustering quality, where the Adjusted Rand
  Index on corrected data is the primary metric
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: scPLS
  role: Jointly models control and target gene sets by partial least squares regression
    with a block-wise EM algorithm to estimate confounding factors whose PVE is evaluated
  repo: http://chenmengjie.github.io/Citrus/
- name: Citrus
  role: Toolkit providing the scPLS implementation, simulation utilities and the CellCycleData
    control-gene resource used to generate benchmark data
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: C++ backend dependency required to run the scPLS block-wise EM implementation
- name: RcppArmadillo
  role: Armadillo linear-algebra backend dependency required to run the scPLS implementation
- name: genlasso
  role: R dependency of Citrus installed alongside the scPLS pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:0b52bb2458d964ff65281f6fdfca9ca17958a0df5c750152fde893e8dd1bd559
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - genlasso
  - scPLS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# proportion-of-variance-explained-estimation

## Summary

Compute the proportion of variance explained (PVE) by estimated confounding factors to quantify how accurately each method (scPLS, PCA, LMM, RUV, scLVM) recovers latent factors in simulated scRNAseq benchmarks, complementing ARI-based clustering evaluation.

## When to use

Use when benchmarking confounding-factor inference on simulated scRNAseq data with known latent factors and you need to quantify latent-factor estimation accuracy, e.g. as a secondary panel alongside ARI across a PDE 20-100% sweep.

## When NOT to use

- The dataset is real (not simulated), so no ground-truth latent factors exist and PVE accuracy cannot be computed against true factors
- The downstream question is cell-type clustering quality, where the Adjusted Rand Index on corrected data is the primary metric

## Inputs

- Simulated single-cell RNA-seq expression matrices with known latent confounders (scenario I and II, dropout and non-dropout, PDE 20%-100%) generated with Citrus simulation utilities and the CellCycleData control-gene resource
- Estimated confounding factors from scPLS, PCA, LMM, RUV and scLVM correction runs

## Outputs

- PVE values quantifying latent-factor estimation accuracy per method, per scenario, per dropout condition, per PDE level
- Rows appended to simulation_evaluation_results.csv combining the PVE-accuracy panel with ARI comparisons

## How to apply

For each simulated dataset (scenario I and scenario II, with and without dropout, sweeping the proportion of differential expression from 20% to 100%), infer confounding factors with scPLS (via the exported scPLS(Target, Control, ...) function from Citrus, implemented in Rcpp/RcppArmadillo with a block-wise EM algorithm) and with PCA, LMM, RUV and scLVM. After correction, compute the proportion of variance explained (PVE) by each method's estimated confounding factors and compare it against the true simulated latent factors to quantify estimation accuracy. Assemble the PVE-accuracy panel together with the k-means ARI results into a single results table written to simulation_evaluation_results.csv. Validation: scPLS should attain accurate PVE (comparable or better than baselines) across the full PDE sweep in both scenarios, consistent with the paper's claim of scPLS effectiveness from extensive simulations.

## Related tools

- **scPLS** (Jointly models control and target gene sets by partial least squares regression with a block-wise EM algorithm to estimate confounding factors whose PVE is evaluated) — http://chenmengjie.github.io/Citrus/
- **Citrus** (Toolkit providing the scPLS implementation, simulation utilities and the CellCycleData control-gene resource used to generate benchmark data) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (C++ backend dependency required to run the scPLS block-wise EM implementation)
- **RcppArmadillo** (Armadillo linear-algebra backend dependency required to run the scPLS implementation)
- **genlasso** (R dependency of Citrus installed alongside the scPLS pipeline)

## Examples

```
install.packages("Rcpp"); install.packages("RcppArmadillo"); install_github("ChenMengjie/Citrus"); scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)
```

## Evaluation signals

- PVE of scPLS-estimated factors matches or exceeds that of PCA, LMM, RUV and scLVM across the PDE 20%-100% sweep in both scenarios
- PVE-accuracy panel and ARI table assemble into one consistent simulation_evaluation_results.csv
- Results hold in both the standard and dropout settings, matching the paper's validation claim
- scPLS runs complete without Rcpp/RcppArmadillo build errors and its block-wise EM converges (tol criterion met)

## Limitations

- PVE accuracy is only computable in simulations where true latent confounders are known; on real data there is no ground truth
- scPLS can only infer biological effects (Lambda_u u_i), not the biological factors u_i, because of model identifiability
- The framework models continuous expression, not counts; extending to count data is left as future work

## Evidence

- [abstract] By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects.: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects."
- [methods] scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center): "scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)"
- [intro] with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
- [readme] Citrus is a toolkit for single cell sequencing analysis under development. Citrus is written in c++ with easy-to-use R wrappers.: "Citrus is a toolkit for single cell sequencing analysis under development. Citrus is written in c++ with easy-to-use R wrappers."
- [discussion] because of model identifiability, scPLS can only be used to infer the biological effects: "because of model identifiability, scPLS can only be used to infer the biological effects"
