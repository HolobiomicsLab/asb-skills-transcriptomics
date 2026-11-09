---
name: chunked-estep-vectorized-computation
description: Use when fitting scPLS to large scRNAseq count matrices (thousands of
  cells × tens of thousands of genes) where the naive full-matrix EM becomes CPU-prohibitive
  and a scalable block-wise EM is needed.
when_to_use_negative:
- The dataset is small enough that the naive full-matrix EM already runs in acceptable
  time — chunking adds parameter complexity without benefit
- The goal is to infer the biological factors u_i themselves — scPLS can only infer
  biological effects (Lambda_u u_i), not the factors, due to model identifiability
- The data require a count-based model — the article states that extending the framework
  to modeling count data is left as future work
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: scPLS
  role: Core method whose Chunk and chunk.size parameters invoke the block-wise (chunked)
    E-step of the EM algorithm
  repo: http://chenmengjie.github.io/Citrus/
- name: Citrus
  role: R/C++ toolkit that distributes scPLS and its chunked EM implementation
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: R-to-C++ binding used by Citrus to implement the vectorized/chunked EM computation
    efficiently
- name: RcppArmadillo
  role: Linear algebra backend (Armadillo) underlying Citrus's C++ EM computations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:3407841eea50d104704f0fe7e95ace2cd836fbaf3797dddca7c587e8bf9227ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - scPLS
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# chunked-estep-vectorized-computation

## Summary

A block-wise expectation maximization strategy used by scPLS (in the Citrus R package) in which the E-step is evaluated on chunks of cells, making confounding-factor inference scalable to thousands of cells and tens of thousands of genes without changing the estimation result.

## When to use

Use when fitting scPLS to large scRNAseq count matrices (thousands of cells × tens of thousands of genes) where the naive full-matrix EM becomes CPU-prohibitive and a scalable block-wise EM is needed.

## When NOT to use

- The dataset is small enough that the naive full-matrix EM already runs in acceptable time — chunking adds parameter complexity without benefit
- The goal is to infer the biological factors u_i themselves — scPLS can only infer biological effects (Lambda_u u_i), not the factors, due to model identifiability
- The data require a count-based model — the article states that extending the framework to modeling count data is left as future work

## Inputs

- Target gene expression matrix from scRNAseq data (cells × genes)
- Control gene expression matrix (cells × control genes)
- Chunk flag (TRUE/FALSE) and chunk.size parameter (e.g. 500 or 1000 cells per block)
- Number of latent factors k1 and k2, plus scPLS tuning parameters (iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, tol, center)

## Outputs

- Inferred confounding effects per cell (equivalent to naive EM estimates)
- CPU-time and estimation-error comparison table (e.g. chunked_em_benchmark.csv) across (n, p) grid and chunk sizes

## How to apply

Fit the scPLS model on the Target and Control gene matrices using the scPLS function in Citrus, which exposes the block-wise EM via the Chunk and chunk.size parameters. When Chunk is set to TRUE, the E-step is computed on blocks of cells of size chunk.size (e.g. 500 or 1000) rather than on the full expression matrix, exploiting C++/Armadillo implementations via Rcpp and RcppArmadillo. Benchmark by recording estimation error of inferred confounding effects and elapsed CPU time against the naive EM across simulation settings (e.g. n=200/400 cells, p=2000/4000 genes). Accept the chunked E-step only if its estimation error matches the naive EM within Monte Carlo tolerance at every (n, p) setting and chunk size, while reducing CPU time, consistent with the paper's claim of a scalable block-wise EM.

## Related tools

- **scPLS** (Core method whose Chunk and chunk.size parameters invoke the block-wise (chunked) E-step of the EM algorithm) — http://chenmengjie.github.io/Citrus/
- **Citrus** (R/C++ toolkit that distributes scPLS and its chunked EM implementation) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (R-to-C++ binding used by Citrus to implement the vectorized/chunked EM computation efficiently)
- **RcppArmadillo** (Linear algebra backend (Armadillo) underlying Citrus's C++ EM computations)

## Examples

```
install.packages(c("Rcpp","RcppArmadillo","genlasso","gplots")); install_github("ChenMengjie/Citrus"); res <- scPLS(Target, Control, k1, k2, Chunk=TRUE, chunk.size=500)
```

## Evaluation signals

- Chunked E-step estimation error matches the naive full-matrix EM within Monte Carlo tolerance for every simulated (n, p) setting (n=200/400 cells, p=2000/4000 genes)
- Match holds at both chunk.size=500 and chunk.size=1000, confirming chunk size does not bias inference
- CPU time for the chunked variant is reduced relative to the naive EM, consistent with the paper's scalability claim
- Aggregated benchmark writes a complete comparison table (chunked_em_benchmark.csv) covering all (n, p) settings and chunk sizes with no missing rows
- Installed Citrus loads successfully with its declared dependencies Rcpp, RcppArmadillo, genlasso, and gplots

## Limitations

- scPLS infers biological effects (Lambda_u u_i) but not the biological factors u_i because of model identifiability
- The scPLS framework is not a count-based model; extension to count data is future work
- The 'best' confounding-effect removal method may vary depending on the downstream analysis; scPLS performance in other analysis settings (e.g. differential expression, lineage reconstruction) has not been evaluated
- No software version numbers for Citrus/scPLS are stated in the article

## Evidence

- [intro] with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
- [methods] scPLS function signature exposing Chunk and chunk.size: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [readme] Citrus implementation dependencies: "Citrus relies on the following R packages: **Rcpp**, **RcppArmadillo**, **genlasso**, **gplots**. All packagess are hosted on CRAN."
- [readme] scPLS as normalization method in Citrus: "* Method 1: **scPLS**: a normalization method to remove unwanted variation using both control and target genes."
- [discussion] Identifiability limitation of scPLS: "because of model identifiability, scPLS can only be used to infer the biological effects"
