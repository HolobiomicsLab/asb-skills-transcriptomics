---
name: gaussian-kernel-reference-distribution-on-grid
description: Use when computing per-gene differential expression significance from
  cell positions in a multidimensional space (t-SNE, PCA, spatial or pseudotime coordinates)
  without clustering cells.
when_to_use_negative:
- The expression data are continuous and you need differential expression based on
  expression levels rather than binary detection — the method requires binary detection
  data as input
- Two genes share the same detection pattern but different expression magnitudes —
  the hard detection threshold gives them identical D_KL and p-value, so a level-aware
  method is needed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: R package implementing the grid-based Gaussian-kernel reference distribution,
    D_KL computation, randomization-based p-values, and result visualization
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: R
  role: Language in which singleCellHaystack is implemented and in which the D_KL
    / randomization pipeline is re-implemented and validated
- name: Seurat
  role: Used upstream in the paper's methods to prepare embeddings and spatial datasets
    on which haystack is run
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:c0dd57f08dbb0e972222b1c40f61b31972b0ed378dd87182c4f85de8bc6457e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gaussian-kernel-reference-distribution-on-grid

## Summary

A core computation of singleCellHaystack: a grid is laid over a 2D+ cell space (e.g. t-SNE coordinates) and a Gaussian-kernel reference distribution Q is constructed over grid points, serving as the null density against which each gene's detection distribution is compared via Kullback-Leibler divergence.

## When to use

Use when computing per-gene differential expression significance from cell positions in a multidimensional space (t-SNE, PCA, spatial or pseudotime coordinates) without clustering cells.

## When NOT to use

- The expression data are continuous and you need differential expression based on expression levels rather than binary detection — the method requires binary detection data as input
- Two genes share the same detection pattern but different expression magnitudes — the hard detection threshold gives them identical D_KL and p-value, so a level-aware method is needed

## Inputs

- Binary gene detection matrix (genes x cells) derived from an expression matrix such as dat.expression
- 2D+ cell coordinates (e.g. dat.tsne t-SNE embedding, PCA coordinates, spatial spot coordinates, or pseudotime values)

## Outputs

- Per-gene observed D_KL values
- Per-gene log p-values from B-spline fit to randomized D_KL distributions
- Grid point coordinates (res$info$grid.coordinates) reusable for downstream clustering

## How to apply

Binarize the gene-by-cell expression matrix into a detection matrix, then build a grid over the 2D input space (e.g. dat.tsne toy coordinates) and compute a Gaussian-kernel reference distribution Q over grid points. For each gene, form P(G=T) and P(G=F) densities over the grid points from the detection pattern and compute the Kullback-Leibler divergence of each against Q via the summation formula. Generate null D_KL distributions by repeatedly (~100 times) shuffling each gene's detections across cells and recomputing D_KL. Model the mean and sd of randomized D_KL as functions of the observed D_KL with B-splines to derive per-gene log p-values; the Gaussian-kernel Q ensures the reference reflects the local density of cells so that genes detected in non-randomly positioned cell subsets yield high D_KL.

## Related tools

- **singleCellHaystack** (R package implementing the grid-based Gaussian-kernel reference distribution, D_KL computation, randomization-based p-values, and result visualization) — https://github.com/alexisvdb/singleCellHaystack
- **R** (Language in which singleCellHaystack is implemented and in which the D_KL / randomization pipeline is re-implemented and validated)
- **Seurat** (Used upstream in the paper's methods to prepare embeddings and spatial datasets on which haystack is run) — https://github.com/satijalab/seurat

## Examples

```
res <- haystack(dat.tsne, dat.expression)
```

## Evaluation signals

- Top-ranked genes from a re-implementation (e.g. gene_497 among the top DEGs) match the ranking returned by show_result_haystack(res, n=10) on the toy data
- Observed D_KL for genes detected in randomly distributed cells falls within the randomized null D_KL range, giving non-significant log p-values
- The randomization loop (e.g. ~100 shuffles of detections across cells) yields null D_KL means/sds that increase monotonically with observed D_KL as fit by the B-spline model
- The grid and reference distribution Q reproduce the local density of cells, verified by returning grid coordinates consistent with res$info$grid.coordinates
- Per-gene table of D_KL and log p-values can be written to haystack_toy_results.tsv without missing or NA entries for filtered genes

## Limitations

- The current implementation requires binary detection data; continuous expression measures such as normalized read counts are not supported
- A hard detection threshold is a weak point: genes detected in the same cell subset but with different expression levels receive identical D_KL and p-values
- P-values can be inflated because the input coordinates (PCs, t-SNE, UMAP) already capture much of the expression variability, a double use of the gene expression data
- Zero counts may reflect dropout events rather than true absence; imputation is not applied
- The detection threshold definition is left to the user, introducing arbitrariness

## Evidence

- [abstract] DKL against Q via grid: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [intro] DKL method definition: "uses Kullback–Leibler Divergence (DKL; also called relative entropy) to find genes that are expressed in subsets of cells that are non-randomly"
- [methods] haystack call: "res <- haystack(dat.tsne, dat.expression)"
- [discussion] binary input requirement: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [readme] README DKL description: [evidence span withheld — anchor status: external_doc]
