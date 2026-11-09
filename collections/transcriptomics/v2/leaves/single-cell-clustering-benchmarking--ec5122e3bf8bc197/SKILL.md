---
name: single-cell-clustering-benchmarking
description: Use when comparing normalization/confounding-removal methods (e.g. scPLS
  vs RUV, LMM, PCA, scLVM) on scRNA-seq data with known cell labels, and you need
  ARI-based clustering performance per downstream method.
when_to_use_negative:
- When the goal is inference of the biological factors (u_i) themselves — scPLS can
  only infer biological effects (Lambda_u u_i), not the factors, due to model identifiability
- When evaluation is needed for downstream analyses other than clustering (e.g. differential
  expression, lineage reconstruction), which is beyond the scope of this evaluation
- When a single 'best' confounding-removal method must hold for all questions — the
  best method may vary depending on the downstream analysis of interest (e.g. scPLS
  lost under tSNE)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3523
tools:
- name: Citrus
  role: R/C++ toolkit providing the scPLS normalization method and the CellCycleData
    mESC dataset used in the benchmark
  repo: http://chenmengjie.github.io/Citrus/
- name: scPLS
  role: Confounding-effect-removal method being benchmarked; jointly models control
    (ERCC spike-ins) and target genes via partial least squares regression
  repo: http://chenmengjie.github.io/Citrus/
- name: SC3
  role: One of the four downstream clustering approaches used to evaluate corrected
    matrices
- name: t-SNE
  role: Embedding-based clustering approach used to evaluate corrected matrices
- name: Rcpp
  role: Required dependency for installing and running Citrus/scPLS C++ code
- name: RcppArmadillo
  role: Required dependency for installing and running Citrus/scPLS C++ code
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:fbda1d4c256862630d7ef5839146c5d1b02fe6ecfce65bec0753f77a8da61ddb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Leiden
  - Python 3.8
  - Rcpp
  - RcppArmadillo
  - SC3
  - scAce
  - scPLS
  - t-SNE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-clustering-benchmarking

## Summary

Benchmark confounding-effect-removal methods for single-cell RNA-seq by clustering corrected expression matrices with four downstream approaches (k-means, PCA, t-SNE, SC3) and scoring them with adjusted Rand Index against known labels. Used to judge whether scPLS-corrected data best reveals known biological clusters, e.g. the two culture-condition clusters in the 132 homogeneous control cells of the 251-sample mESC dataset.

## When to use

Use when comparing normalization/confounding-removal methods (e.g. scPLS vs RUV, LMM, PCA, scLVM) on scRNA-seq data with known cell labels, and you need ARI-based clustering performance per downstream method.

## When NOT to use

- When the goal is inference of the biological factors (u_i) themselves — scPLS can only infer biological effects (Lambda_u u_i), not the factors, due to model identifiability
- When evaluation is needed for downstream analyses other than clustering (e.g. differential expression, lineage reconstruction), which is beyond the scope of this evaluation
- When a single 'best' confounding-removal method must hold for all questions — the best method may vary depending on the downstream analysis of interest (e.g. scPLS lost under tSNE)

## Inputs

- Single-cell RNA-seq expression matrix (e.g. 251-sample mESC dataset, 132 homogeneous control cells) from the Citrus CellCycleData dataset
- Control gene set (e.g. 32 filtered ERCC spike-ins)
- Known cell labels (cell-cycle stage / culture-condition clusters) for ARI computation
- Corrected expression matrices from each confounding-removal method (scPLS, RUV, scLVM, LMM, PCA)

## Outputs

- ARI values per correction method per clustering approach (kmeans, PCA, tSNE, SC3)
- Methods-by-clustering-approach comparison table written to ari_table.csv
- Ranked identification of which corrected data best reveals the known clusters

## How to apply

Load the single-cell dataset (e.g. the 251-cell mESC CellCycleData from the Citrus package, focusing on the 132 homogeneous control cells) and apply each correction method — scPLS with the 32 filtered ERCC spike-ins as the control set, RUV and scLVM (control-gene-based), and LMM and PCA (all-gene-based) — to obtain corrected expression matrices. Cluster each corrected matrix with four downstream approaches: k-means, PCA-based clustering, t-SNE-embedded clustering, and SC3. Compute the adjusted Rand Index of each clustering against the known labels (e.g. cell-cycle stage or culture-condition clusters) using the mclust ARI implementation. Assemble the ARI values into a methods-by-clustering-approach table (e.g. ari_table.csv) and compare. In the reference result, scPLS reached ARI up to 100 versus 59 uncorrected and won in kmeans, PCA and SC3 but lost under tSNE, illustrating that the 'best' method may vary with the downstream analysis.

## Related tools

- **Citrus** (R/C++ toolkit providing the scPLS normalization method and the CellCycleData mESC dataset used in the benchmark) — http://chenmengjie.github.io/Citrus/
- **scPLS** (Confounding-effect-removal method being benchmarked; jointly models control (ERCC spike-ins) and target genes via partial least squares regression) — http://chenmengjie.github.io/Citrus/
- **SC3** (One of the four downstream clustering approaches used to evaluate corrected matrices)
- **t-SNE** (Embedding-based clustering approach used to evaluate corrected matrices)
- **Rcpp** (Required dependency for installing and running Citrus/scPLS C++ code)
- **RcppArmadillo** (Required dependency for installing and running Citrus/scPLS C++ code)

## Examples

```
install.packages("Rcpp"); install.packages("RcppArmadillo"); library(devtools); install_github("ChenMengjie/Citrus")
```

## Evaluation signals

- ari_table.csv contains one ARI value for every combination of correction method (scPLS, RUV, LMM, PCA, scLVM, uncorrected) and clustering approach (kmeans, PCA, tSNE, SC3)
- scPLS-corrected ARI on the mESC control cells reaches up to 100 (vs 59 uncorrected) under kmeans/PCA/SC3
- Reproduced ranking should match: scPLS best in three of four approaches, losing only under tSNE
- ARI is computed with the mclust implementation against the known cell-cycle-stage labels
- All methods are run on the same corrected matrices from the same 132 homogeneous control cells with 32 filtered ERCC spike-ins as controls

## Limitations

- The 'best' confounding-effect-removal method may vary depending on the downstream analysis or question of interest; performance of scPLS in other analysis settings has not been evaluated
- Evaluation of scPLS for downstream analyses other than clustering (e.g. differential expression, lineage reconstruction) is not performed
- No count-based model is provided in the scPLS framework; it is not designed for direct count modeling
- No software version numbers are stated for the tools involved

## Evidence

- [results] scPLS outperforms RUV, LMM, PCA and scLVM in three of the four clustering approaches (kmeans, PCA, SC3), with ARI values up to 100 (vs 59 uncorrected), losing only under tSNE.: [evidence span withheld — anchor status: not_found]
- [discussion] the “best” confounding effects removing method may vary depending on the question of interest. Therefore, it would be important to evaluate the performance of scPLS in other analysis settings in: "the “best” confounding effects removing method may vary depending on the question of interest"
- [methods] Load the mESC single-cell RNA-seq dataset (251 cells, 132 homogeneous control cells) via the Citrus package's CellCycleData dataset.: "- `CellCycleData (Citrus)`"
- [readme] Citrus is a toolkit for single cell sequencing analysis under development. Citrus is written in c++ with easy-to-use R wrappers. To use Citrus, the installation of Rcpp and RcppArmadillo is required.: "Citrus is written in c++ with easy-to-use R wrappers. To use Citrus, the installation of **Rcpp** and **RcppArmadillo** is required."
- [abstract] By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects.: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects."
