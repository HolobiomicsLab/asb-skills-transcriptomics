---
name: cpu-time-benchmarking-of-em-variants
description: Use when a fitted model's E-step is too slow on large scRNAseq data and
  you need to verify that a chunked EM variant matches the naive EM's estimation error
  while lowering CPU time.
when_to_use_negative:
- The goal is inference of biological factors (u_i) themselves — scPLS can only infer
  biological effects (Lambda_u u_i) due to model identifiability
- Data are small enough that the naive EM already runs quickly, so chunking provides
  no runtime benefit to validate
- The downstream question requires evaluating confounding removal for analyses other
  than those the method was evaluated on (e.g. differential expression, lineage reconstruction)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: Citrus
  role: R/C++ toolkit exposing the scPLS EM variants via Chunk and chunk.size parameters
    used in the benchmark
  repo: http://chenmengjie.github.io/Citrus/
- name: scPLS
  role: Normalization method whose naive and chunked block-wise EM implementations
    are being timed and compared
  repo: http://chenmengjie.github.io/Citrus/
- name: RcppArmadillo
  role: Required dependency for Citrus's C++ backends that execute the EM algorithm
- name: Rcpp
  role: Required dependency providing the R-to-C++ wrappers for Citrus
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:cfea824f394dac1d8255ed94d6b1d77e336934e9183436948ea91c7ccbfeb5d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - scPLS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# CPU-time benchmarking of EM variants

## Summary

Benchmark the block-wise (chunked) EM implementation of scPLS against the naive EM by recording estimation error of inferred confounding effects and elapsed CPU time across simulated (n, p) grid settings, validating that chunking preserves accuracy while reducing runtime.

## When to use

Use when a fitted model's E-step is too slow on large scRNAseq data and you need to verify that a chunked EM variant matches the naive EM's estimation error while lowering CPU time.

## When NOT to use

- The goal is inference of biological factors (u_i) themselves — scPLS can only infer biological effects (Lambda_u u_i) due to model identifiability
- Data are small enough that the naive EM already runs quickly, so chunking provides no runtime benefit to validate
- The downstream question requires evaluating confounding removal for analyses other than those the method was evaluated on (e.g. differential expression, lineage reconstruction)

## Inputs

- Simulated scRNAseq expression matrices under the scPLS generative model with control and target gene sets (n=200/400 cells, p=2000/4000 genes)
- Naive scPLS EM fit results (confounding-effect estimates and CPU times)
- Chunked scPLS E-step fits with Chunk=TRUE at chunk.size=500 and chunk.size=1000

## Outputs

- Estimation-error and CPU-time comparison table across all (n, p) settings and chunk sizes
- chunked_em_benchmark.csv

## How to apply

Simulate scRNAseq expression data under the scPLS generative model with control and target gene sets on the paper's simulation grid (n=200 and 400 cells; p=2000 and 4000 genes). Fit each dataset first with the naive EM implementation of scPLS, recording estimation error of the inferred confounding effects and elapsed CPU time; then refit with Chunk=TRUE and chunk.size=500, repeating with chunk.size=1000. Aggregate estimation error and CPU time across all (n, p) settings and chunk sizes into a single comparison table (e.g., the paper's Table 2) written to a CSV such as chunked_em_benchmark.csv. Accept the chunked E-step if its estimation error matches the naive EM within Monte Carlo tolerance for every (n, p) setting at both chunk sizes while reducing CPU time. Rationale: the block-wise EM is what makes scPLS scalable to thousands of cells and tens of thousands of genes, and Citrus implements it in C++ with R wrappers requiring Rcpp and RcppArmadillo.

## Related tools

- **Citrus** (R/C++ toolkit exposing the scPLS EM variants via Chunk and chunk.size parameters used in the benchmark) — http://chenmengjie.github.io/Citrus/
- **scPLS** (Normalization method whose naive and chunked block-wise EM implementations are being timed and compared) — http://chenmengjie.github.io/Citrus/
- **RcppArmadillo** (Required dependency for Citrus's C++ backends that execute the EM algorithm)
- **Rcpp** (Required dependency providing the R-to-C++ wrappers for Citrus)

## Examples

```
scPLS(Target, Control, k1, k2, Chunk=TRUE, chunk.size=500)
```

## Evaluation signals

- Chunked E-step estimation error matches naive EM within Monte Carlo tolerance for every (n, p) setting
- Chunked runs show reduced CPU time relative to naive EM at both chunk.size=500 and chunk.size=1000
- Results aggregated consistently across all grid cells (n=200/400, p=2000/4000) in the benchmark table
- Benchmark CSV (chunked_em_benchmark.csv) reproduces the structure of the paper's Table 2 comparison

## Limitations

- Performance of scPLS in downstream analysis settings other than those evaluated in the paper has not been assessed; the 'best' confounding-removal method may vary by question of interest
- Extending the framework to count-based models is left as future work, so benchmarks assume the Gaussian/PLS generative model
- No software version numbers are stated for the tools, so exact reproducibility of CPU times may vary across builds

## Evidence

- [intro] block-wise EM scalability claim: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
- [methods] scPLS signature with Chunk parameters: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [readme] Citrus implementation stack: "Citrus is written in c++ with easy-to-use R wrappers. To use Citrus, the installation of **Rcpp** and **RcppArmadillo** is required."
