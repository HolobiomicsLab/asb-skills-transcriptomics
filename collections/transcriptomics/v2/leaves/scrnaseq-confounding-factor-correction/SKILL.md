---
name: scrnaseq-confounding-factor-correction
description: Use when a scRNAseq expression matrix shows unwanted variation (e.g.
  ERCC spike-in confounding or cell-cycle effects) and downstream clustering must
  recover known biological groupings from corrected data.
when_to_use_negative:
- 'You need to directly infer the biological factors themselves (u_i): scPLS can only
  infer the biological effects (Lambda_u u_i), not the factors, because of model identifiability'
- 'You require a count-based model of raw scRNAseq counts: extending the framework
  to modeling count data is left as future work in the paper'
- 'Your downstream analysis is not clustering (e.g. differential expression, lineage
  reconstruction): the paper''s evaluation of correction methods covers clustering
  only and notes performance in other settings was not evaluated'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scPLS
  role: 'Primary method: joint PLS-regression inference and removal of latent confounding
    factors using control and target gene sets'
  repo: http://chenmengjie.github.io/Citrus/
- name: Citrus
  role: R/C++ toolkit hosting the scPLS() implementation, with Chunk/chunk.size parameters
    for the block-wise EM
  repo: http://chenmengjie.github.io/Citrus/
- name: SC3
  role: One of the four downstream clustering approaches used to score corrected matrices
    by ARI
- name: t-SNE
  role: Embedding-based clustering approach in the ARI comparison; the one approach
    where scPLS loses to comparators
- name: Rcpp
  role: Required dependency for installing and running the C++-backed Citrus package
  repo: http://chenmengjie.github.io/Citrus/
- name: RcppArmadillo
  role: Required dependency for Citrus's C++ linear-algebra backend
  repo: http://chenmengjie.github.io/Citrus/
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:749e05d0bf771a04125dae92bee7814e3d692ca16db383a36479e12f013195f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - SC3
  - scPLS
  - t-SNE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# scrnaseq-confounding-factor-correction

## Summary

Infer and remove hidden confounding factors (technical or cell-cycle effects) from scRNAseq expression matrices using scPLS, which jointly models a control gene set (e.g. ERCC spike-ins) and a target gene set via partial least squares regression, implemented in the Citrus R/C++ toolkit with a block-wise EM algorithm.

## When to use

Use when a scRNAseq expression matrix shows unwanted variation (e.g. ERCC spike-in confounding or cell-cycle effects) and downstream clustering must recover known biological groupings from corrected data.

## When NOT to use

- You need to directly infer the biological factors themselves (u_i): scPLS can only infer the biological effects (Lambda_u u_i), not the factors, because of model identifiability
- You require a count-based model of raw scRNAseq counts: extending the framework to modeling count data is left as future work in the paper
- Your downstream analysis is not clustering (e.g. differential expression, lineage reconstruction): the paper's evaluation of correction methods covers clustering only and notes performance in other settings was not evaluated

## Inputs

- scRNAseq target gene expression matrix (cells x genes), e.g. mESC 251 cells / 132 homogeneous control cells from Citrus CellCycleData
- Control gene set, e.g. 32 filtered ERCC spike-ins
- Known cell labels (e.g. cell-cycle stage) for ARI evaluation
- scPLS parameters: Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center

## Outputs

- Corrected expression matrices with inferred confounding effects removed (one per method: scPLS, RUV, LMM, PCA, scLVM)
- Methods-by-clustering-approach ARI comparison table (e.g. ari_table.csv), ARI values up to 100 for scPLS vs 59 uncorrected on mESC
- Chunked-EM benchmark table of estimation error and CPU time across (n, p) settings and chunk sizes (e.g. chunked_em_benchmark.csv)

## How to apply

Load the expression data (e.g. the 251-cell mESC dataset with 132 homogeneous control cells via Citrus's CellCycleData) and identify a control set — here 32 filtered ERCC spike-ins. Call Citrus's exported scPLS() entry point with the target gene matrix and control set to jointly infer latent confounding factors through partial least squares regression; for large datasets (thousands of cells, tens of thousands of genes) set Chunk=TRUE with chunk.size=500 or 1000 to run the block-wise EM. Fit the same data with comparators: RUV and scLVM (control-gene-based) and LMM and PCA (all-gene-based). Evaluate each corrected matrix by four clustering approaches — k-means, PCA-based, t-SNE-embedded, and SC3 — and compute adjusted Rand Index against known labels (e.g. cell-cycle stage) with the mclust ARI implementation, assembling a methods-by-approach table.

## Related tools

- **scPLS** (Primary method: joint PLS-regression inference and removal of latent confounding factors using control and target gene sets) — http://chenmengjie.github.io/Citrus/
- **Citrus** (R/C++ toolkit hosting the scPLS() implementation, with Chunk/chunk.size parameters for the block-wise EM) — http://chenmengjie.github.io/Citrus/
- **SC3** (One of the four downstream clustering approaches used to score corrected matrices by ARI)
- **t-SNE** (Embedding-based clustering approach in the ARI comparison; the one approach where scPLS loses to comparators)
- **Rcpp** (Required dependency for installing and running the C++-backed Citrus package) — http://chenmengjie.github.io/Citrus/
- **RcppArmadillo** (Required dependency for Citrus's C++ linear-algebra backend) — http://chenmengjie.github.io/Citrus/

## Examples

```
install_github("ChenMengjie/Citrus"); corrected <- scPLS(Target = expr_matrix, Control = ercc_spikes, Chunk = TRUE, chunk.size = 500)
```

## Evaluation signals

- ARI of corrected-data clustering against known labels exceeds the uncorrected baseline (paper: scPLS ARI up to 100 vs 59 uncorrected on the mESC control cells)
- scPLS outperforms RUV, LMM, PCA and scLVM in at least three of four clustering approaches (kmeans, PCA, SC3); a loss under tSNE is consistent with the published result
- With Chunk=TRUE and chunk.size=500 or 1000, estimation error matches the naive EM within Monte Carlo tolerance for every (n, p) setting while CPU time decreases
- All corrected matrices are scored by the same four downstream approaches and assembled into a methods-by-approach ARI table (e.g. ari_table.csv)

## Limitations

- The 'best' confounding-effect-removal method may vary depending on the downstream analysis or question of interest; scPLS lost to comparators under tSNE clustering
- scPLS can only infer biological effects (Lambda_u u_i), not the biological factors (u_i), due to model identifiability
- No count-based model is provided; count-data modeling is future work, and performance for downstream analyses beyond clustering (differential expression, lineage reconstruction, etc.) was not evaluated

## Evidence

- [intro] Joint PLS modeling rationale: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding factors."
- [intro] Scalability of the block-wise EM: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
- [methods] scPLS function signature with Chunk/chunk.size parameters: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [discussion] Caveat on method choice: "the “best” confounding effects removing method may vary depending on the question of interest."
- [readme] Citrus toolkit description from README: "Citrus is a toolkit for single cell sequencing analysis under development. 
Citrus is written in c++ with easy-to-use R wrappers."
- [readme] scPLS described in README: "Method 1: **scPLS**: a normalization method to remove unwanted variation using both control and target genes."
