---
name: spike-in-control-gene-selection
description: Use when an scRNAseq dataset contains ERCC spike-in measurements that
  must be filtered and designated as the control-gene set before applying control-based
  confounding-removal methods like scPLS, RUV, or scLVM.
when_to_use_negative:
- The dataset contains no external spike-in transcripts, so all-gene-based methods
  (PCA, LMM) must be used instead of control-based ones
- Spike-ins are too sparsely or inconsistently detected across the homogeneous cell
  subset to form a reliable control set after filtering
- The goal is to infer biological factors u_i rather than remove technical confounding
  — scPLS can only infer biological effects, not the factors themselves
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Citrus
  role: R/C++ toolkit hosting the CellCycleData mESC dataset and the scPLS method
    that consumes the selected spike-in control set
  repo: http://chenmengjie.github.io/Citrus/
- name: scPLS
  role: jointly models the Target gene matrix and the selected Control (ERCC spike-in)
    set via partial least squares regression to infer and remove latent confounding
    factors
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: required dependency for installing and running Citrus/scPLS
- name: RcppArmadillo
  role: required dependency for installing and running Citrus/scPLS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:4eb1f64373d49a1357a7b41fd5c0e792c241b433e03ffe306304f7428d9bee13
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - SC3
  - scPLS
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spike-in-control-gene-selection

## Summary

Selects external RNA spike-in transcripts (e.g. ERCC spike-ins) as the control gene set used by confounding-removal methods such as scPLS, RUV, and scLVM. This skill mirrors the mESC analysis in Chen & Zhou (2017), where 32 filtered ERCC spike-ins served as controls for inferring technical confounding factors.

## When to use

Use when an scRNAseq dataset contains ERCC spike-in measurements that must be filtered and designated as the control-gene set before applying control-based confounding-removal methods like scPLS, RUV, or scLVM.

## When NOT to use

- The dataset contains no external spike-in transcripts, so all-gene-based methods (PCA, LMM) must be used instead of control-based ones
- Spike-ins are too sparsely or inconsistently detected across the homogeneous cell subset to form a reliable control set after filtering
- The goal is to infer biological factors u_i rather than remove technical confounding — scPLS can only infer biological effects, not the factors themselves

## Inputs

- scRNAseq gene-by-cell count/expression matrix containing ERCC spike-in rows (e.g. the 251-sample mESC dataset from the Citrus CellCycleData dataset)
- Cell metadata distinguishing homogeneous control cells (132 mESC cells) and known labels (culture condition / cell-cycle stage)
- Full ERCC spike-in annotation/panel list

## Outputs

- Filtered ERCC spike-in control-gene matrix (e.g. 32 filtered ERCC spike-ins x 132 control cells)
- Control set passed as the Control argument to scPLS/RUV/scLVM for confounding-factor inference

## How to apply

Start from the raw scRNAseq count matrix that includes ERCC spike-in rows alongside endogenous target genes. Filter the spike-ins to retain only those reliably detected across the homogeneous cell subset (in the mESC study, 32 of the ERCC spike-ins passed filtering out of the full panel); apply the same cell filter, e.g. keeping the 132 homogeneous control cells from the 251-sample mESC dataset. Supply the filtered spike-in matrix as the Control argument and the target gene matrix as the Target argument to scPLS(Target, Control, ...), which jointly models both sets via partial least squares regression to infer latent confounding factors. Rationale: control-only methods (RUV, scLVM) rely entirely on the quality of the spike-in control set, while scPLS bridges control-based and all-gene-based (PCA/LMM) approaches, so spike-in selection directly determines the inferred confounding factors and downstream clustering ARI.

## Related tools

- **Citrus** (R/C++ toolkit hosting the CellCycleData mESC dataset and the scPLS method that consumes the selected spike-in control set) — http://chenmengjie.github.io/Citrus/
- **scPLS** (jointly models the Target gene matrix and the selected Control (ERCC spike-in) set via partial least squares regression to infer and remove latent confounding factors) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (required dependency for installing and running Citrus/scPLS)
- **RcppArmadillo** (required dependency for installing and running Citrus/scPLS)

## Examples

```
install.packages(c("Rcpp","RcppArmadillo")); library(devtools); install_github("ChenMengjie/Citrus")  # then scPLS(Target, Control=filtered_ERCC, ...)
```

## Evaluation signals

- The filtered control set matches the reference scale: 32 filtered ERCC spike-ins retained from the mESC panel
- The control matrix is restricted to the same cells as the target matrix (132 homogeneous control cells out of 251 mESC samples)
- Downstream k-means/PCA/SC3 clustering of scPLS-corrected data using these spike-in controls yields higher ARI than uncorrected data (up to 100 vs 59) and than RUV, LMM, PCA, and scLVM in 3 of 4 approaches
- The only case where control-informed scPLS should lose is t-SNE-embedded clustering, per the published comparison
- The spike-in matrix is passed as the Control argument of scPLS, distinct from the Target gene matrix

## Limitations

- scPLS can infer biological effects but, because of model identifiability, cannot infer the biological factors u_i themselves
- The 'best' confounding-removal method varies with the downstream analysis; performance on analyses other than clustering (differential expression, lineage reconstruction) was not evaluated
- No count-based model is provided in the current framework; extension to count data is left as future work
- Which spike-ins survive filtering depends on dataset-specific detection rates (32 ERCC controls in the mESC study); the number will differ on other datasets

## Evidence

- [abstract] By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects.: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects."
- [methods] scPLS entry point signature: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [methods] CellCycleData dataset listed among Citrus datasets: "- `CellCycleData (Citrus)`"
- [readme] Citrus installation requires Rcpp and RcppArmadillo: "To use Citrus, the installation of **Rcpp** and **RcppArmadillo** is required."
- [readme] scPLS described as normalization method using control and target genes: "Method 1: **scPLS**: a normalization method to remove unwanted variation using both control and target genes."
