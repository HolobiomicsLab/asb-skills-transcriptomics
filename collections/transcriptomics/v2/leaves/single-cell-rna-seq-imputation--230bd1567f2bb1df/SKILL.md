---
name: single-cell-rna-seq-imputation
description: Use when a droplet-based scRNA-seq expression matrix has a large fraction
  of false zeros (dropout) and downstream analyses such as cell-cell distance or correlation
  computation are degraded by undetected transcripts.
when_to_use_negative:
- When all zeros should be treated as missing data and filled everywhere — methods
  like MAGIC/DCA output a matrix where every gene is expressed in every cell, whereas
  ALRA preserves biological zeros
- When the input is not an scRNA-seq-like non-negative, low-rank expression matrix
  with many zeros — ALRA's thresholding relies on true biological zeros being symmetrically
  distributed around zero after SVD
- When the matrix orientation is genes x rows — ALRA requires cells as rows and genes
  as columns
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ALRA
  role: 'Core imputation method: computes rank-k approximation via randomized SVD,
    thresholds each gene by the magnitude of its most negative value, and rescales'
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD prerequisite package used by ALRA to compute the rank-k approximation
    (install.packages('rsvd'))
- name: rpca-mkl
  role: Optional MKL-based randomized SVD backend enabling dramatic speedups of alra()
    via the use.mkl flag
  repo: https://github.com/KlugerLab/rpca-mkl
- name: Seurat
  role: Single-cell toolkit in which ALRA is integrated as RunALRA() (Seurat v3.0);
    alraSeurat2.R provides a wrapper for Seurat v2 objects
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:691bdb77a47b819959117b189b0e059548b681e3158530622007f6b22cae2c5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - Seurat
  - rpca-mkl
  - rsvd
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-cell-rna-seq-imputation

## Summary

Zero-preserving imputation of scRNA-seq count matrices using ALRA (Adaptively thresholded Low-Rank Approximation), which computes a rank-k randomized SVD approximation and thresholds per-gene entries to complete technical (dropout) zeros while keeping true biological zeros at zero.

## When to use

Use when a droplet-based scRNA-seq expression matrix has a large fraction of false zeros (dropout) and downstream analyses such as cell-cell distance or correlation computation are degraded by undetected transcripts.

## When NOT to use

- When all zeros should be treated as missing data and filled everywhere — methods like MAGIC/DCA output a matrix where every gene is expressed in every cell, whereas ALRA preserves biological zeros
- When the input is not an scRNA-seq-like non-negative, low-rank expression matrix with many zeros — ALRA's thresholding relies on true biological zeros being symmetrically distributed around zero after SVD
- When the matrix orientation is genes x rows — ALRA requires cells as rows and genes as columns

## Inputs

- scRNA-seq expression count matrix (cells x genes) with a high fraction of zeros, e.g., the CBMC dataset from GEO GSE100866 (8005 cells x 20,400 genes after filtering cells with >90% mouse UMI fraction)
- Library- and log-normalized matrix (via normalize_data()) with cells as rows and genes as columns
- Optional: a protein/ADT count matrix from CITE-seq for distance-correlation validation

## Outputs

- Completed (imputed) normalized expression matrix, result.completed[[3]], with technical zeros imputed and biological zeros preserved at zero
- Chosen rank k from choose_k() based on singular value spacings
- Downstream metrics, e.g., before/after Spearman correlations of gene-space vs. ADT-space cell distances (cbmc_distance_correlations.csv)

## How to apply

Pass ALRA a normalized matrix with cells as rows and genes as columns — first apply library and log normalization via normalize_data(). Choose the rank k using choose_k(), which picks k from the spacings between singular values (the paper's automatic choice worked well; for the CITE-seq CBMC dataset k=25 is reported). Run alra() on the normalized matrix: it computes the rank-k approximation via randomized SVD (rsvd; optionally faster with rpca-mkl via use.mkl=TRUE), thresholds each gene row by the magnitude of its most negative value to restore biological zeros, and rescales; the completed matrix is returned as the third list element. Evaluate by checking that marker genes missing from known cell populations are recovered (e.g., NCAM1 in NK cells, CR2 in B cells in the b_nk example) and, for CITE-seq data, that Spearman correlation between gene-expression-space and ADT protein-space pairwise cell distances improves (CBMC: ~0.36 raw to ~0.64 after ALRA).

## Related tools

- **ALRA** (Core imputation method: computes rank-k approximation via randomized SVD, thresholds each gene by the magnitude of its most negative value, and rescales) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD prerequisite package used by ALRA to compute the rank-k approximation (install.packages('rsvd')))
- **rpca-mkl** (Optional MKL-based randomized SVD backend enabling dramatic speedups of alra() via the use.mkl flag) — https://github.com/KlugerLab/rpca-mkl
- **Seurat** (Single-cell toolkit in which ALRA is integrated as RunALRA() (Seurat v3.0); alraSeurat2.R provides a wrapper for Seurat v2 objects) — https://github.com/satijalab/seurat

## Examples

```
library(ALRA); A_norm <- normalize_data(b_nk_example); k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k=k_choice$k)[[3]]
```

## Evaluation signals

- Spearman correlation between normalized gene-expression distances and ADT (protein) distances should increase after imputation (CBMC expectation: ~0.36 raw vs ~0.64 imputed, imputed > raw)
- Marker recovery: NCAM1 should be expressed in all NK cells (only 4% in original b_nk example data) and CR2 in all B cells (only 1%) after completion
- True biological zeros must remain exactly zero in the completed matrix
- The chosen k (e.g., k=25 for CBMC) should coincide with the beginning of noise in the singular value spacing plot from choose_k()
- Result must be non-negative and rescaled back to the original normalized scale

## Limitations

- The automatic k from choose_k() may warrant manual inspection in some cases; k is chosen from singular value spacings because identifying the beginning of noise from the spectrum alone is hard
- The rpca-mkl accelerated backend is not on CRAN, is still under development, and its pre-compiled MKL libraries only support Linux and OS X (Windows requires compiling MKL from source)
- Results improve when using the entire dataset rather than a subset (see the ALRA-paper repo for full-dataset codes)

## Evidence

- [readme] rank-k approximation via randomized SVD then per-gene thresholding: "Given a scRNA-seq expression matrix, ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene."
- [readme] cells as rows, genes as columns input requirement: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [abstract] zero-preserving rationale from abstract: "We present a method based on low-rank matrix approximation which imputes these values while preserving biologically non-expressed genes (true biological zeros) at zero expression levels."
- [intro] ALRA exploits non-negativity and low-rank structure: "we present Adaptively thresholded Low-Rank Approximation (ALRA), a method for imputation of scRNA-seq data. ALRA takes advantage of the non-negativity and low-rank structure of an expression matrix"
- [intro] dropout motivates imputation: [evidence span withheld — anchor status: not_found]
