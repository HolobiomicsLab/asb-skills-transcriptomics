---
name: pairwise-cell-distance-computation
description: Use when a paired multimodal single-cell dataset (e.g., CITE-seq RNA
  counts + ADT antibody counts) needs a quantitative check that imputed RNA geometry
  matches protein-space geometry.
when_to_use_negative:
- The dataset lacks a paired protein (ADT) modality, so there is no reference geometry
  against which RNA-space distances can be correlated
- The expression matrix is unnormalized raw UMI counts — ALRA expects a normalized
  matrix and distances must be computed on comparable scales across modalities
- The goal is cell-type clustering or embedding rather than assessing modality-consistency
  of cell-cell geometry
tools:
- name: ALRA
  role: Produces the imputed RNA matrix whose completed (vs raw) Euclidean cell distances
    are correlated against ADT-space distances
  repo: github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD package required by ALRA to compute the rank-k approximation
    underlying imputation
  repo: github.com/KlugerLab/ALRA
- name: rpca-mkl
  role: Optional MKL-accelerated randomized SVD backend used via ALRA's use.mkl flag
    for faster large-matrix completion
  repo: KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:11c6a3ba8a7428ec9881a4db3114970853d7bb1933d8aefdff9316e875f34440
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALRA
  - rpca-mkl
  - rsvd
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# pairwise-cell-distance-computation

## Summary

Computes Euclidean pairwise cell-to-cell distance matrices in gene-expression space (before and after ALRA imputation) and in ADT protein space for CITE-seq data, so that rank/Spearman correlations between the two modalities' distance matrices quantify whether imputation improves RNA–protein geometric consistency.

## When to use

Use when a paired multimodal single-cell dataset (e.g., CITE-seq RNA counts + ADT antibody counts) needs a quantitative check that imputed RNA geometry matches protein-space geometry.

## When NOT to use

- The dataset lacks a paired protein (ADT) modality, so there is no reference geometry against which RNA-space distances can be correlated
- The expression matrix is unnormalized raw UMI counts — ALRA expects a normalized matrix and distances must be computed on comparable scales across modalities
- The goal is cell-type clustering or embedding rather than assessing modality-consistency of cell-cell geometry

## Inputs

- Library- and log-normalized RNA expression matrix from the CBMC CITE-seq dataset (8005 cells x 20,400 genes, GSE100866)
- Same normalized RNA matrix after ALRA imputation (rank k=25)
- Normalized ADT (antibody-derived tag) protein count matrix for the same 8005 cells

## Outputs

- Euclidean pairwise cell-distance matrix in ADT protein space
- Euclidean pairwise cell-distance matrices in gene-expression space, before and after ALRA imputation
- Before/after Spearman correlation values (approximately 0.36 raw vs 0.64 imputed), written to cbmc_distance_correlations.csv

## How to apply

Start from library- and log-normalized gene expression (cells x genes) produced by ALRA's normalize_data(), plus the normalized ADT protein matrix for the same cells. Compute all-pairs Euclidean distances between cells in the ADT space, and in gene-expression space both before and after ALRA completion (ALRA computes a rank-k approximation via randomized SVD, thresholds each gene row by its most negative value, and rescales; for CBMC the reported rank was k=25). Normalize the gene-expression distance vectors, then compute the Spearman rank correlation of RNA-space distances against ADT-space distances separately for raw and imputed matrices. The imputed correlation should be higher, reflecting ALRA's zero-preserving completion of technical dropout zeros rather than filling biological zeros.

## Related tools

- **ALRA** (Produces the imputed RNA matrix whose completed (vs raw) Euclidean cell distances are correlated against ADT-space distances) — github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD package required by ALRA to compute the rank-k approximation underlying imputation) — github.com/KlugerLab/ALRA
- **rpca-mkl** (Optional MKL-accelerated randomized SVD backend used via ALRA's use.mkl flag for faster large-matrix completion) — KlugerLab/rpca-mkl

## Examples

```
library(ALRA); A_norm <- normalize_data(A_counts); A_completed <- alra(A_norm, k=25)[[3]]; cor.raw <- cor(rna_dist_raw, adt_dist, method='spearman')
```

## Evaluation signals

- The post-imputation Spearman correlation of RNA-space vs ADT-space distances should exceed the pre-imputation value, approaching the reported 0.36 -> 0.64 improvement
- The imputed matrix must remain zero-preserving: entries corresponding to true biological zeros stay at zero after ALRA thresholding
- Distance matrices must be symmetric with zero diagonals and cover all 8005 cells in both modalities
- ALRA input orientation check: cells must be rows and genes must be columns before calling alra()
- The chosen rank should match the reported CBMC value (k=25) or be justified via choose_k() singular-value spacing diagnostics

## Limitations

- ALRA assumes the true expression matrix is non-negative, low-rank, and contains many biological zeros; violations degrade the imputed geometry
- Choosing k is difficult — it is based on spacings between singular values since the beginning of noise is hard to identify from the spectrum alone
- rpca-mkl is still under development, not on CRAN, and its pre-compiled MKL libraries only support Linux and OS X

## Evidence

- [readme] README usage: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] README algorithm: "ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene."
- [intro] intro thresholding rationale: "the entries of this matrix that correspond to the true biological zeros are symmetrically distributed around zero"
- [intro] intro SVD step: "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
- [abstract] abstract validation: "We provide theoretical justification for this denoising approach and demonstrate its advantages relative to other methods on simulated and biological datasets."
