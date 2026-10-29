---
name: bspline-null-statistics-smoothing-pvalue-estimation
description: 'Use when re-implementing or validating singleCellHaystack''s per-gene
  significance pipeline: you have observed D_KL values per gene plus randomized null
  D_KL replicates and need smoothed p-values via B-splines instead of per-gene empirical
  tests.'
when_to_use_negative:
- Expression data is continuous counts/read counts — the method requires binary detection
  data (a gene is either detected or not in each cell); continuous-measure support
  is planned but not available in the described version
- You need p-values corrected for double use of gene expression data (coordinates
  derived from the same expression matrix) — the authors note this inflation issue
  is deferred to future updates
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: R package whose binary-branch implementation performs grid construction, Gaussian-kernel
    Q estimation, randomized D_KL generation, and B-spline-based log p-value calculation
  repo: github.com/alexisvdb/singleCellHaystack
- name: R
  role: Implementation environment; the re-implementation of the D_KL and randomization
    pipeline with set.seed (e.g. 1234) for reproducibility
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:7d1752ba5751610ec3d76e727a6b443cc8a8d6f30082e4567210ac084a37fbdb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# bspline-null-statistics-smoothing-pvalue-estimation

## Summary

A statistical sub-step of singleCellHaystack that converts randomized Kullback-Leibler divergence null distributions into per-gene significance by modeling the mean and SD of randomized D_KL as B-spline functions of the observed D_KL, then deriving log p-values from the fitted curves. This enables clustering-independent DEG calling for genes non-randomly positioned in a multidimensional space.

## When to use

Use when re-implementing or validating singleCellHaystack's per-gene significance pipeline: you have observed D_KL values per gene plus randomized null D_KL replicates and need smoothed p-values via B-splines instead of per-gene empirical tests.

## When NOT to use

- Expression data is continuous counts/read counts — the method requires binary detection data (a gene is either detected or not in each cell); continuous-measure support is planned but not available in the described version
- You need p-values corrected for double use of gene expression data (coordinates derived from the same expression matrix) — the authors note this inflation issue is deferred to future updates

## Inputs

- Binary gene detection matrix (genes x cells), e.g. dat.expression binarized
- 2D or higher-dimensional cell coordinates (e.g. dat.tsne t-SNE coordinates)
- Randomized null D_KL replicates per gene (~100 permutations of detections across cells)

## Outputs

- Per-gene table of observed D_KL values and log p-values (e.g. haystack_toy_results.tsv)
- Ranked list of DEGs matching show_result_haystack(res, n=10)

## How to apply

Load the gene detection matrix (genes x cells) and 2D coordinates (e.g. dat.tsne), then for each gene compute P(G=T) and P(G=F) distributions over a Gaussian-kernel grid and their D_KL against the reference distribution Q. Shuffle each gene's detections across cells (~100 randomizations) and recompute D_KL to build null replicates. Fit the mean and standard deviation of the randomized D_KL as B-spline curves over the observed D_KL, so lowly and highly variable genes borrow strength across the full range. Convert each gene's observed D_KL to a log p-value by evaluating how extreme it lies relative to the fitted null mean/sd curves. Validate by comparing the top-ranked genes (e.g. gene_497 among the top DEGs on the toy data) against show_result_haystack(res, n=10).

## Related tools

- **singleCellHaystack** (R package whose binary-branch implementation performs grid construction, Gaussian-kernel Q estimation, randomized D_KL generation, and B-spline-based log p-value calculation) — github.com/alexisvdb/singleCellHaystack
- **R** (Implementation environment; the re-implementation of the D_KL and randomization pipeline with set.seed (e.g. 1234) for reproducibility)

## Examples

```
res <- haystack(dat.tsne, dat.expression)
```

## Evaluation signals

- Top-ranked genes from the re-implementation (e.g. gene_497 among the top DEGs) match the ranking returned by show_result_haystack(res, n=10)
- Randomized null D_KL mean/sd curves should be monotone, smooth functions of the observed D_KL (B-spline fits behave smoothly across the D_KL range)
- Per-gene log p-values in the output TSV are finite and ordered consistently with observed D_KL extremity relative to the fitted null
- Re-running with set.seed(1234) reproduces the same ranking and log p-values
- Genes with identical detection patterns across cells yield identical D_KL and p-values (an expected invariance of the detection-based method)

## Limitations

- Use of a hard detection threshold is a weak point: two genes detected in the same subset of cells but with ten-fold different expression levels get identical D_KL and p-value
- P-values may be inflated because the input coordinates (PCs, t-SNE, UMAP) are dimensions containing a large proportion of the variability in the original expression data (double use of data)
- Many zero counts reflect dropout events; imputation could partly address this but is not applied
- The detection threshold definition is left to the user, introducing potential arbitrariness

## Evidence

- [abstract] KLD-to-pvalue method basis: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [methods] Concrete R invocation of the pipeline whose null/p-value machinery is being reconstructed: "res <- haystack(dat.tsne, dat.expression)"
- [discussion] Binary input requirement constraining the null randomization step: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [readme] Branch caveat for reproducing the original (binary) algorithm: "The version on CRAN
is also this updated version. For the older version described
[here](https://doi.org/10.1038/s41467-020-17900-3), please use branch
“binary”."
