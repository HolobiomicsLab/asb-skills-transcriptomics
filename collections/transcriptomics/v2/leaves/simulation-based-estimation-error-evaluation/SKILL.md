---
name: simulation-based-estimation-error-evaluation
description: Use when evaluating whether a scalable algorithmic variant (e.g. chunked
  E-step with Chunk=TRUE, chunk.size=500 or 1000) reproduces the accuracy of a reference
  implementation while reducing runtime, or when benchmarking confounding-factor inference
  on simulated scRNAseq grids.
when_to_use_negative:
- Downstream-analysis evaluation beyond confounding-effect estimation — the paper
  notes the 'best' confounding removal method depends on the downstream question and
  only clustering-type settings were evaluated
- When the goal is inference of biological factors u_i themselves — scPLS can only
  infer biological effects (Lambda_u u_i), not the factors, due to model identifiability
- When a count-based error model is required — the paper's framework does not model
  count data, which is left as future work
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPLS
  role: Method under benchmark; fit with naive EM vs chunked E-step (Chunk, chunk.size
    parameters) to compare estimation error and CPU time
  repo: http://chenmengjie.github.io/Citrus/
- name: Citrus
  role: R/C++ toolkit whose scPLS implementation exposes the Chunk and chunk.size
    parameters used in the benchmark
  repo: https://github.com/ChenMengjie/Citrus
- name: RcppArmadillo
  role: C++ linear algebra backend required to install and run the compiled scPLS/Citrus
    code
- name: Rcpp
  role: R-to-C++ binding layer required to install Citrus
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:62cb93f0b1c535db077b9990d8a457d983e47cac75f2737317de11c1de65279d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - scPLS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# simulation-based-estimation-error-evaluation

## Summary

Benchmark the accuracy and CPU cost of an inference method (here scPLS confounding-effect estimation via naive vs. block-wise chunked EM) on simulated scRNAseq data across a grid of cell/gene counts, recording estimation error of the inferred confounding effects and elapsed CPU time for comparison.

## When to use

Use when evaluating whether a scalable algorithmic variant (e.g. chunked E-step with Chunk=TRUE, chunk.size=500 or 1000) reproduces the accuracy of a reference implementation while reducing runtime, or when benchmarking confounding-factor inference on simulated scRNAseq grids.

## When NOT to use

- Downstream-analysis evaluation beyond confounding-effect estimation — the paper notes the 'best' confounding removal method depends on the downstream question and only clustering-type settings were evaluated
- When the goal is inference of biological factors u_i themselves — scPLS can only infer biological effects (Lambda_u u_i), not the factors, due to model identifiability
- When a count-based error model is required — the paper's framework does not model count data, which is left as future work

## Inputs

- Simulated scRNAseq expression matrices from the scPLS generative model at (n, p) in {200, 400} cells x {2000, 4000} genes
- Control gene set and target gene set indices used by scPLS
- Chunk size values to benchmark (e.g. chunk.size=500 and chunk.size=1000)

## Outputs

- Estimation error of inferred confounding effects per (n, p) setting per implementation (naive EM vs chunked EM)
- Elapsed CPU time per fit
- Aggregated comparison table written to chunked_em_benchmark.csv

## How to apply

Simulate scRNAseq expression data under the paper's simulation grid (n=200 and 400 cells; p=2000 and 4000 genes) using the scPLS generative model with control and target gene sets. Fit each dataset with the naive EM implementation of scPLS, recording estimation error of the inferred confounding effects and elapsed CPU time, then refit with the chunked E-step variant (Chunk=TRUE, chunk.size=500 and chunk.size=1000). Aggregate results across all (n, p) settings and chunk sizes into a single estimation-error vs CPU-time comparison table (e.g. chunked_em_benchmark.csv). The chunked variant is accepted when its estimation error matches the naive EM within Monte Carlo tolerance at every (n, p) setting and both chunk sizes while reducing CPU time, consistent with the claim that the block-wise EM is scalable to thousands of cells and tens of thousands of genes.

## Related tools

- **scPLS** (Method under benchmark; fit with naive EM vs chunked E-step (Chunk, chunk.size parameters) to compare estimation error and CPU time) — http://chenmengjie.github.io/Citrus/
- **Citrus** (R/C++ toolkit whose scPLS implementation exposes the Chunk and chunk.size parameters used in the benchmark) — https://github.com/ChenMengjie/Citrus
- **RcppArmadillo** (C++ linear algebra backend required to install and run the compiled scPLS/Citrus code)
- **Rcpp** (R-to-C++ binding layer required to install Citrus)

## Examples

```
library(Citrus); fit <- scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk=TRUE, chunk.size=500, center)
```

## Evaluation signals

- Chunked E-step estimation error matches naive EM within Monte Carlo tolerance for every (n, p) setting
- Chunked variant reduces CPU time relative to naive EM at both chunk.size=500 and chunk.size=1000
- Results aggregated across all four (n, p) settings ({200, 400} cells x {2000, 4000} genes) appear in the comparison table
- Output CSV (chunked_em_benchmark.csv) contains both estimation-error and CPU-time columns per configuration
- Fits run successfully with Chunk=TRUE in the Citrus scPLS signature, confirming the parameter is accepted

## Limitations

- Acceptable chunk-size values and error tolerance are dataset-dependent; the paper's grid is limited to n in {200, 400} and p in {2000, 4000}
- scPLS infers biological effects but not the biological factors themselves due to model identifiability, so 'ground truth' error is defined on effects only
- No count-based generative model is provided in the framework; extending to count data is left as future work

## Evidence

- [intro] Block-wise EM scalability claim: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
- [intro] Joint PLS modeling rationale: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding factors."
- [methods] scPLS function signature exposing Chunk parameters: "scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)"
- [abstract] Simulation-based validation framing: "With extensive simulations and comparisons with other methods, we demonstrate the effectiveness of scPLS."
- [readme] Citrus implementation context: "Citrus is a toolkit for single cell sequencing analysis under development. Citrus is written in c++ with easy-to-use R wrappers."
