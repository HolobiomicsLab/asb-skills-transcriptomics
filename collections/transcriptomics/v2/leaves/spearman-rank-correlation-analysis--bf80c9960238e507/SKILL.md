---
name: spearman-rank-correlation-analysis
description: Use when a multimodal single-cell dataset (e.g., CITE-seq RNA + ADT)
  is available and you need to test whether an imputation or transformation in one
  modality makes its pairwise cell distances more consistent with distances in the
  other modality.
when_to_use_negative:
- When only one modality is present (no ADT/protein measurements) and there is no
  second distance structure to correlate against
- When the analysis goal is cluster/label agreement rather than concordance of pairwise
  cell-distance orderings — Spearman on distance vectors answers the latter, not the
  former
- When distance distributions are already on the same scale and a direct distance
  comparison (e.g., Procrustes or RMSE) is preferred over rank-based correlation
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: ALRA
  role: Provides the imputed RNA matrix whose distance-vs-ADT Spearman correlation
    is evaluated; alra() and normalize_data() from the R package
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD backend required by ALRA to compute the rank-k approximation
    used before distance computation
- name: rpca-mkl
  role: Optional MKL-accelerated randomized SVD (fastRPCA R wrapper) used via ALRA's
    use.mkl flag for speedup
  repo: https://github.com/KlugerLab/rpca-mkl
- name: Seurat
  role: Single-cell toolkit exposing ALRA as RunALRA() (v3.0); can supply normalized
    RNA and ADT assays for distance computation
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:34c533070f600277e9733c710ac3fd47d0c7ccf7447145c7d34edacf18515c73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - Seurat
  - rpca-mkl
  - rsvd
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# spearman-rank-correlation-analysis

## Summary

Quantifies agreement between pairwise cell-distance structures computed in two feature spaces (e.g., imputed/log-normalized gene expression vs. ADT protein counts from CITE-seq) using the Spearman rank correlation. In the ALRA paper this is the metric demonstrating that ALRA imputation improves RNA-to-protein distance concordance on the CBMC dataset (Spearman ~0.36 raw → ~0.64 after ALRA).

## When to use

Use when a multimodal single-cell dataset (e.g., CITE-seq RNA + ADT) is available and you need to test whether an imputation or transformation in one modality makes its pairwise cell distances more consistent with distances in the other modality.

## When NOT to use

- When only one modality is present (no ADT/protein measurements) and there is no second distance structure to correlate against
- When the analysis goal is cluster/label agreement rather than concordance of pairwise cell-distance orderings — Spearman on distance vectors answers the latter, not the former
- When distance distributions are already on the same scale and a direct distance comparison (e.g., Procrustes or RMSE) is preferred over rank-based correlation

## Inputs

- Normalized RNA expression matrix (cells x genes, e.g., 8005 cells x 20,400 genes for CBMC from GSE100866) before and after ALRA imputation
- ADT (protein) count matrix from CITE-seq (antibody-derived tags for the same cells)
- Pairwise Euclidean distance matrices/vectors computed in ADT space and gene-expression space

## Outputs

- Spearman correlation of gene-space vs ADT-space distances before imputation (~0.36 for CBMC)
- Spearman correlation of gene-space vs ADT-space distances after ALRA imputation (~0.64 for CBMC)
- CSV file (e.g., cbmc_distance_correlations.csv) containing the before/after Spearman correlations

## How to apply

Compute all-pairs Euclidean distances between cells in the ADT (protein) space, and separately in gene-expression space both before and after ALRA imputation (library- and log-normalized via normalize_data(), rank k=25 for CBMC, completed via alra() with randomized SVD from the rsvd package). Normalize the gene-expression distance vectors, then compute the Spearman rank correlation between gene-space distances and ADT-space distances for the raw and imputed matrices. The Spearman statistic is appropriate because only the rank ordering of the ~N(N-1)/2 pairwise distances matters, making it robust to scale differences between modalities. Report before/after correlations (e.g., to cbmc_distance_correlations.csv); correct application requires the imputed correlation to exceed the raw one, consistent with the reported 0.36 → 0.64 improvement.

## Related tools

- **ALRA** (Provides the imputed RNA matrix whose distance-vs-ADT Spearman correlation is evaluated; alra() and normalize_data() from the R package) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD backend required by ALRA to compute the rank-k approximation used before distance computation)
- **rpca-mkl** (Optional MKL-accelerated randomized SVD (fastRPCA R wrapper) used via ALRA's use.mkl flag for speedup) — https://github.com/KlugerLab/rpca-mkl
- **Seurat** (Single-cell toolkit exposing ALRA as RunALRA() (v3.0); can supply normalized RNA and ADT assays for distance computation) — https://github.com/satijalab/seurat

## Examples

```
library(ALRA); A_norm <- normalize_data(rna_counts); A_completed <- alra(A_norm, k=25)[[3]]; cor(as.vector(dist(A_norm)), as.vector(dist(t(adt_norm))), method='spearman')
```

## Evaluation signals

- The Spearman correlation computed on the ALRA-imputed matrix must exceed the raw (pre-imputation) correlation — for CBMC, imputed should approach ~0.64 versus raw ~0.36
- The correlation is computed on the full set of pairwise cell distances from identical cell sets in both modalities (same cell order)
- Distance vectors are normalized before correlation so the statistic reflects rank concordance, not scale
- Results are written to a defined artifact (cbmc_distance_correlations.csv) with two comparable correlation values

## Limitations

- Spearman concordance is a global summary; improved correlation does not guarantee per-cell-type distance fidelity
- Rank k matters: the reported CBMC improvement used k=25 (as reported for CBMC); a poorly chosen k can degrade distance consistency
- ALRA assumes the true expression matrix is non-negative, low-rank, and contains many biological zeros; violations undermine the imputation that the correlation evaluates

## Evidence

- [other] ALRA improved the Spearman correlation between normalized gene-expression distances and protein (ADT) distances in the CITE-seq CBMC dataset from 0.36 to 0.64.: [evidence span withheld — anchor status: not_found]
- [readme] ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene.: "ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene."
- [readme] Please be sure to pass ALRA a matrix where the cells are rows and genes are columns.: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as install.packages('rsvd').: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
- [intro] Compute a low-rank approximation of the observed matrix using singular value decomposition (SVD): "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
