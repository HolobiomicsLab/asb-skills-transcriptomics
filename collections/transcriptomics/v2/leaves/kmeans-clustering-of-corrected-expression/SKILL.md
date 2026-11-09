---
name: kmeans-clustering-of-corrected-expression
description: 'Use when evaluating how well a confounder-correction method preserves
  simulated cell-type structure: you have corrected expression matrices plus true
  cell labels and want a clustering-accuracy (ARI) comparison across methods.'
when_to_use_negative:
- You need to evaluate downstream analyses other than clustering (e.g. differential
  expression, lineage reconstruction) — the paper explicitly states evaluating other
  downstream analyses is beyond its scope
- Your data lack ground-truth cell labels, so no ARI against known groupings can be
  computed
- You need to recover the biological factors u_i themselves — scPLS can only infer
  biological effects, not the factors, due to model identifiability
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPLS
  role: Produces the confounder-corrected expression matrices that k-means is applied
    to; primary method under comparison
  repo: https://github.com/ChenMengjie/Citrus
- name: Citrus
  role: Toolkit providing the scPLS implementation and simulation utilities (with
    CellCycleData control-gene resource) used to generate the corrected matrices
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: R/C++ integration dependency required to install and run Citrus/scPLS
- name: RcppArmadillo
  role: Linear-algebra backend for the Rcpp-based scPLS block-wise EM algorithm
- name: genlasso
  role: R dependency of Citrus used in the scPLS penalty fitting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:82c83c4188364df6affc640adcac952afbdc5d2e16502d6f813b5531581bcd9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChenMengjie/Citrus
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - genlasso
  - scPLS
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# k-means clustering of confounder-corrected expression

## Summary

Applies k-means clustering to expression matrices after latent confounder removal (e.g. by scPLS, PCA, LMM, RUV, scLVM) and scores cluster recovery against known cell labels with the Adjusted Rand Index, the benchmark used in the scPLS paper (Scientific Reports, 2017) to compare confounding-removal methods.

## When to use

Use when evaluating how well a confounder-correction method preserves simulated cell-type structure: you have corrected expression matrices plus true cell labels and want a clustering-accuracy (ARI) comparison across methods.

## When NOT to use

- You need to evaluate downstream analyses other than clustering (e.g. differential expression, lineage reconstruction) — the paper explicitly states evaluating other downstream analyses is beyond its scope
- Your data lack ground-truth cell labels, so no ARI against known groupings can be computed
- You need to recover the biological factors u_i themselves — scPLS can only infer biological effects, not the factors, due to model identifiability

## Inputs

- Confounder-corrected expression matrix (cells x genes) per method (scPLS, PCA, LMM, RUV, scLVM)
- Vector of true simulated cell labels for the same cells
- Simulation design parameters: scenario (I or II), dropout condition, PDE level (20%-100%)

## Outputs

- Per-condition ARI values for each correction method
- PVE (proportion of variance explained) of estimated latent factors
- Aggregated results table written to simulation_evaluation_results.csv

## How to apply

First obtain confounder-corrected expression matrices, e.g. via scPLS(Target, Control, k1, k2, ...) from the Citrus package (implemented in Rcpp/RcppArmadillo with a block-wise EM algorithm), or via the PCA, LMM, RUV and scLVM baselines. Run k-means on each corrected matrix using the number of clusters matching the simulated cell groups. Compute the Adjusted Rand Index (ARI) of the k-means partition against the true simulated cell labels, repeating across the proportion of differential expression (PDE) sweep from 20% to 100%, for scenario I and scenario II, each with and without dropout events. Optionally estimate the proportion of variance explained (PVE) of inferred latent factors to check confounder-estimation accuracy. Assemble ARI and PVE values across all methods and conditions into a results table (e.g. simulation_evaluation_results.csv); correctness is judged by scPLS- or best-method-corrected data attaining comparable or higher ARI, reproducing the paper's simulation finding.

## Related tools

- **scPLS** (Produces the confounder-corrected expression matrices that k-means is applied to; primary method under comparison) — https://github.com/ChenMengjie/Citrus
- **Citrus** (Toolkit providing the scPLS implementation and simulation utilities (with CellCycleData control-gene resource) used to generate the corrected matrices) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (R/C++ integration dependency required to install and run Citrus/scPLS)
- **RcppArmadillo** (Linear-algebra backend for the Rcpp-based scPLS block-wise EM algorithm)
- **genlasso** (R dependency of Citrus used in the scPLS penalty fitting)

## Examples

```
corrected <- scPLS(Target, Control, k1, k2); cl <- kmeans(corrected, centers = k, nstart = 25)
```

## Evaluation signals

- ARI of k-means on scPLS-corrected data should equal or exceed that of PCA-, LMM-, RUV- and scLVM-corrected data across the PDE 20%-100% sweep in both scenarios, with and without dropout
- PVE of inferred latent factors should match the true simulated factor variance (accurate PVE)
- Clustering must be computed on the corrected expression matrix, not the raw counts — verify the input matrix is post-correction
- Results table simulation_evaluation_results.csv should contain one ARI entry per method x scenario x dropout x PDE condition
- ARI should be computed with mclust-style adjusted Rand Index against the true simulated cell labels, not raw accuracy

## Limitations

- scPLS can only infer biological effects (Lambda_u u_i), not the biological factors u_i, because of model identifiability
- The 'best' confounding-removal method may vary depending on the downstream analysis; performance of scPLS for analyses other than clustering was not evaluated
- No count-based model is provided; the framework was not extended to modeling count data
- Citrus requires Rcpp and RcppArmadillo installation before use

## Evidence

- [methods] scPLS signature: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [abstract] effectiveness via simulations: "With extensive simulations and comparisons with other methods, we demonstrate the effectiveness of scPLS."
- [intro] Citrus availability: "Our method is implemented as a part of the Citrus project and is freely available at: http://chenmengjie.github.io/Citrus/."
- [discussion] scope limited to evaluated analyses: "While evaluating the performance of various confounding effects removing methods for other downstream analysis is beyond the scope of the present study"
- [readme] Citrus clustering method: "Now a normalized method and a clustering method are available in the distributed version."
