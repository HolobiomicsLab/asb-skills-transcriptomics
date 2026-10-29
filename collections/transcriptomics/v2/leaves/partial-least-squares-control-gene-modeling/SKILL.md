---
name: partial-least-squares-control-gene-modeling
description: Use when scRNAseq expression matrices contain unmeasured technical confounders
  (e.g. ERCC spike-ins available as controls), and a corrected matrix is needed for
  downstream clustering/analysis.
when_to_use_negative:
- You need direct estimates of biological factors u_i — scPLS can only infer biological
  effects (Lambda_u u_i), not the factors themselves, due to model identifiability
- Downstream analysis is not clustering (e.g. differential expression, lineage reconstruction)
  — the paper states performance of scPLS in other analysis settings was not evaluated
- Data are counts that require a count-based model — extending the framework to count
  modeling is explicitly left as future work
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3308
tools:
- name: Citrus
  role: R/C++ toolkit that implements scPLS and provides the CellCycleData mESC dataset
    used as input
  repo: http://chenmengjie.github.io/Citrus/
- name: scPLS
  role: Core method jointly modeling control and target gene sets via partial least
    squares regression to infer confounding factors
  repo: http://chenmengjie.github.io/Citrus/
- name: SC3
  role: Clustering approach used to score corrected matrices via ARI
- name: t-SNE
  role: Embedding-based clustering approach used in the ARI benchmark
- name: Rcpp
  role: Dependency required to install and run the C++-backed Citrus implementation
- name: RcppArmadillo
  role: Dependency required to install and run the C++-backed Citrus implementation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:399877a9e8713ee26d9ed71806d0968ccf09d5e0429c91ad6fba60db5027242d
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# partial-least-squares-control-gene-modeling

## Summary

Apply scPLS (single cell partial least squares) to infer and remove hidden technical confounding factors in scRNAseq data by jointly modeling a control gene set (e.g. ERCC spike-ins) and a target gene set via partial least squares regression, then benchmark the corrected data with clustering (k-means, PCA, t-SNE, SC3) scored by adjusted Rand Index.

## When to use

Use when scRNAseq expression matrices contain unmeasured technical confounders (e.g. ERCC spike-ins available as controls), and a corrected matrix is needed for downstream clustering/analysis.

## When NOT to use

- You need direct estimates of biological factors u_i — scPLS can only infer biological effects (Lambda_u u_i), not the factors themselves, due to model identifiability
- Downstream analysis is not clustering (e.g. differential expression, lineage reconstruction) — the paper states performance of scPLS in other analysis settings was not evaluated
- Data are counts that require a count-based model — extending the framework to count modeling is explicitly left as future work

## Inputs

- Target gene expression matrix from single-cell RNA-seq (e.g. 132 homogeneous mESC control cells from the 251-cell Citrus CellCycleData dataset)
- Filtered control gene set (e.g. 32 filtered ERCC spike-ins)
- Number of latent factors k1 and k2 and scPLS tuning parameters (iter, alpha, kappa, diagH, penalty options)
- Known cell labels for ARI evaluation (e.g. cell-cycle-stage or culture-condition labels)

## Outputs

- Confounder-corrected target gene expression matrix
- Inferred latent confounding effects (Lambda_u u_i)
- ARI values per correction method per clustering approach (methods-by-clustering table, e.g. ari_table.csv)

## How to apply

Load the target gene expression matrix and a filtered control set — in the mESC benchmark, 32 filtered ERCC spike-ins from the 132 homogeneous control cells of the 251-cell Citrus CellCycleData dataset. Call scPLS(Target, Control, k1, k2, ...) to jointly infer latent confounding factors via partial least squares regression with a block-wise EM algorithm, bridging all-gene methods (PCA, LMM) and control-gene-only methods (RUV, scLVM), each of which is a special case. Run RUV, scLVM, LMM and PCA on the same data as comparators. Cluster each corrected matrix with k-means, PCA-based clustering, t-SNE-embedded clustering and SC3, then compute adjusted Rand Index against known labels (mclust ARI; values on the mESC data reached up to 100 for scPLS vs 59 uncorrected) and tabulate the methods-by-approach ARI comparison.

## Related tools

- **Citrus** (R/C++ toolkit that implements scPLS and provides the CellCycleData mESC dataset used as input) — http://chenmengjie.github.io/Citrus/
- **scPLS** (Core method jointly modeling control and target gene sets via partial least squares regression to infer confounding factors) — http://chenmengjie.github.io/Citrus/
- **SC3** (Clustering approach used to score corrected matrices via ARI)
- **t-SNE** (Embedding-based clustering approach used in the ARI benchmark)
- **Rcpp** (Dependency required to install and run the C++-backed Citrus implementation)
- **RcppArmadillo** (Dependency required to install and run the C++-backed Citrus implementation)

## Examples

```
install_github("ChenMengjie/Citrus"); data(CellCycleData); corrected <- scPLS(Target, Control_ERCC32, k1, k2)
```

## Evaluation signals

- Adjusted Rand Index (mclust ARI) of corrected-data clustering against known labels: scPLS should reach ARI up to 100 vs 59 on uncorrected mESC control-cell data, winning k-means, PCA and SC3 but not t-SNE
- ARI table assembled as methods (scPLS, RUV, LMM, PCA, scLVM) x clustering approaches (kmeans, PCA, tSNE, SC3) and written to ari_table.csv without missing cells
- scPLS runs converge via the block-wise EM algorithm on thousands of cells and tens of thousands of genes without errors
- Citrus installs cleanly after Rcpp and RcppArmadillo, and the package vignette reproduces the scPLS workflow

## Limitations

- Because of model identifiability, scPLS can only infer biological effects (Lambda_u u_i), not the biological factors themselves
- The best confounding-removal method varies with the downstream question; scPLS performance for non-clustering analyses was not evaluated
- No count-based model is provided; extending the framework to count data is left as future work
- Under t-SNE-embedded clustering, scPLS lost to at least one comparator on the mESC benchmark

## Evidence

- [abstract] scPLS jointly models control and target sets: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects."
- [intro] block-wise EM scalability: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
- [methods] scPLS function signature: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [intro] Citrus availability and implementation: "Our method is implemented as a part of the Citrus project and is freely available at: http://chenmengjie.github.io/Citrus/."
- [readme] scPLS description in README: "scPLS: a normalization method to remove unwanted variation using both control and target genes."
- [discussion] identifiability limitation: "because of model identifiability, scPLS can only be used to infer the biological effects (i.e."
