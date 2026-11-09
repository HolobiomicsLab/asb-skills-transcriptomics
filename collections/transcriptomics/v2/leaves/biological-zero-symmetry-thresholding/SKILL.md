---
name: biological-zero-symmetry-thresholding
description: Use when a low-rank approximation of a sparse scRNA-seq expression matrix
  has been computed and technical zeros must be completed while keeping true biological
  zeros at exactly zero.
when_to_use_negative:
- The matrix is not a sparse, dropout-heavy scRNA-seq count matrix — if there is little
  zero-inflation (e.g. bulk RNA-seq or already imputed data), the symmetry-at-zero
  rationale does not apply
- The input matrix has cells as columns — ALRA requires cells as rows and genes as
  columns, so transpose first instead of applying the thresholding to the wrong orientation
- The goal is denoising that outputs a dense matrix treating all zeros as missing
  (as DCA/MAGIC do) rather than zero-preserving imputation
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ALRA
  role: R package providing normalize_data, choose_k, and alra; its alra() function
    performs the rank-k approximation, per-gene thresholding at the magnitude of the
    most negative value, and rescaling
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD package used to compute the rank-k approximation and the top
    100 singular values for choose_k
- name: rpca-mkl
  role: Optional Intel MKL-based randomized SVD backend enabled via the use.mkl flag
    for dramatic speedups
  repo: https://github.com/KlugerLab/rpca-mkl
- name: Seurat
  role: Exposes ALRA as the RunALRA() function in Seurat v3.0
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:faec538186f8aa2d26b6676d8bcaa2d2abd1216457d31181355abf99d02a8168
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - Seurat
  - rpca-mkl
  - rsvd
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# biological-zero-symmetry-thresholding

## Summary

ALRA's final step: after computing a rank-k SVD approximation of a normalized scRNA-seq matrix, restore true biological zeros by thresholding each gene row at the magnitude of its most negative value, exploiting the fact that entries at biological-zero positions are symmetrically distributed around zero.

## When to use

Use when a low-rank approximation of a sparse scRNA-seq expression matrix has been computed and technical zeros must be completed while keeping true biological zeros at exactly zero.

## When NOT to use

- The matrix is not a sparse, dropout-heavy scRNA-seq count matrix — if there is little zero-inflation (e.g. bulk RNA-seq or already imputed data), the symmetry-at-zero rationale does not apply
- The input matrix has cells as columns — ALRA requires cells as rows and genes as columns, so transpose first instead of applying the thresholding to the wrong orientation
- The goal is denoising that outputs a dense matrix treating all zeros as missing (as DCA/MAGIC do) rather than zero-preserving imputation

## Inputs

- Library- and log-normalized scRNA-seq expression matrix (cells as rows, genes as columns), e.g. normalize_data(b_nk_example)
- Rank k chosen from singular-value spacings by ALRA's choose_k (top 100 singular values via rsvd)
- Rank-k randomized-SVD approximation of the normalized matrix

## Outputs

- Thresholded, zero-preserving completed matrix (e.g. alra_thresholded_matrix.csv)
- Per-gene table of zero counts before and after thresholding (per_gene_zero_counts.csv)
- Rescaled completed matrix returned by alra() (third element of result)

## How to apply

First normalize the count matrix with library and log normalization (cells as rows, genes as columns) via ALRA's normalize_data, then compute the rank-k approximation using randomized SVD (choose_k selects k from the top 100 singular values based on singular-value spacings versus a noise cutoff). After the low-rank approximation, work row-wise (per gene): entries at positions that are zero in the observed normalized matrix are approximately symmetrically distributed around zero, so the negative entries estimate the error distribution. For each gene, set to zero every entry smaller in magnitude than that row's most negative value — equivalently, the symmetric mirror of the 0.001 quantile of that gene's approximated values. Finally, restore any originally non-zero normalized values that fell below the threshold back to their observed values, and rescale the matrix.

## Related tools

- **ALRA** (R package providing normalize_data, choose_k, and alra; its alra() function performs the rank-k approximation, per-gene thresholding at the magnitude of the most negative value, and rescaling) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD package used to compute the rank-k approximation and the top 100 singular values for choose_k)
- **rpca-mkl** (Optional Intel MKL-based randomized SVD backend enabled via the use.mkl flag for dramatic speedups) — https://github.com/KlugerLab/rpca-mkl
- **Seurat** (Exposes ALRA as the RunALRA() function in Seurat v3.0) — https://github.com/satijalab/seurat

## Examples

```
library(ALRA); A_norm <- normalize_data(b_nk_example); k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k=k_choice$k)[[3]]
```

## Evaluation signals

- Entries of the approximated matrix at positions that were zero in the observed normalized matrix are approximately symmetrically distributed around zero within each gene row
- After thresholding, per_gene_zero_counts.csv shows biological-zero positions retained at zero while technical zeros are completed (paper reports ~97% biological-zero preservation on the simulated benchmark)
- Sanity check from the README: NCAM1 should become expressed in all NK cells and CR2 in all B cells after completion, versus 4% and 1% detected in the original data
- The threshold per gene equals the magnitude of that gene's most negative approximated value (mirror of the 0.001 quantile), and originally non-zero observed values below the threshold are restored
- k chosen by choose_k lies below 100 and coincides with the largest gap in singular-value spacings (s80..s100 noise cutoff)

## Limitations

- The symmetry-based threshold is heuristic: it relies on the assumption that true expression is non-negative and low-rank, and that error at biological-zero positions is symmetric around zero
- rpca-mkl is still under development and not on CRAN; its pre-compiled MKL libraries only support Linux and OS X, not Windows
- Results improve when using the entire dataset rather than a subset (as noted for the b_nk example)
- The automatically chosen k worked well in the paper, but authors advise inspecting the singular-value spacing plots when in doubt

## Evidence

- [intro] ALRA computes a low-rank approximation of the observed matrix via SVD, then restores biological zeros by thresholding entries using the symmetry of entries corresponding to true biological zeros around zero: "the entries of this matrix that correspond to the true biological zeros are symmetrically distributed around zero"
- [readme] Thresholding criterion per gene: "Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene. Finally, the matrix is rescaled."
- [intro] Definition of the final ALRA step: "the last step of ALRA is to restore the biological zeros in the matrix by thresholding its entries"
- [intro] Desired imputation behavior: "it should accurately impute the data, in particular preserving at zero true biological zeros while completing the technical ones"
- [readme] Upstream computation feeding the thresholding step: "ALRA first computes its rank-k approximation using randomized SVD."
