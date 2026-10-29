---
name: monte-carlo-detection-shuffling-null
description: Use when computing per-gene differential expression significance from
  cell positions in an embedding (t-SNE, PCA, pseudotime, spatial 2D coordinates)
  and a per-gene parametric null is unavailable, so randomized shuffling of detection
  patterns is needed.
when_to_use_negative:
- Expression data is continuous and un-thresholded — the current implementation requires
  binary detection data, so genes differing in expression level but sharing a detection
  pattern cannot be distinguished
- Cells have already been clustered and the goal is standard cluster-based DEG calling
  between labeled groups (this method is specifically for clustering-independent DEG
  detection)
- A calibrated parametric null for the statistic already exists and does not suffer
  from p-value inflation from double use of expression data
tools:
- name: singleCellHaystack
  role: R package implementing the D_KL computation, grid construction, and randomized
    shuffling null; original binary-detection version is on the 'binary' branch
  repo: github.com/alexisvdb/singleCellHaystack
- name: R
  role: Implementation environment for the re-implementation pipeline (set.seed, matrix
    binarization, B-spline fitting, TSV export)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:3e6ac734f4cf4e7bfa633ceadb6bffb5eb7222d068afcc267dac6ebcf072024a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - singleCellHaystack
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# monte-carlo-detection-shuffling-null

## Summary

Builds a Monte Carlo null distribution for singleCellHaystack's per-gene Kullback-Leibler divergence statistic by repeatedly shuffling binary gene-detection patterns across cells in a 2D/low-dimensional input space (e.g. t-SNE), converting randomized D_KL values into calibrated log p-values without clustering cells.

## When to use

Use when computing per-gene differential expression significance from cell positions in an embedding (t-SNE, PCA, pseudotime, spatial 2D coordinates) and a per-gene parametric null is unavailable, so randomized shuffling of detection patterns is needed.

## When NOT to use

- Expression data is continuous and un-thresholded — the current implementation requires binary detection data, so genes differing in expression level but sharing a detection pattern cannot be distinguished
- Cells have already been clustered and the goal is standard cluster-based DEG calling between labeled groups (this method is specifically for clustering-independent DEG detection)
- A calibrated parametric null for the statistic already exists and does not suffer from p-value inflation from double use of expression data

## Inputs

- genes x cells binary detection matrix (derived from dat.expression by binarization)
- cells x dimensions input coordinate matrix (e.g. dat.tsne 2D t-SNE coordinates, PCA coordinates, pseudotime, or 2D spatial spot coordinates)

## Outputs

- per-gene table of D_KL values and log p-values (e.g. haystack_toy_results.tsv)
- randomized D_KL null distributions per gene (~100 shuffles) with B-spline-fitted mean/sd curves

## How to apply

Binarize the genes x cells expression matrix (e.g. dat.expression from the singleCellHaystack toy data) into a detection matrix, and construct a grid over the input coordinates (e.g. dat.tsne 2D space) with a Gaussian-kernel reference distribution Q over grid points. For each gene compute the P(G=T) and P(G=F) distributions over grid points and their D_KL against Q. Generate the null by shuffling each gene's detections across cells about ~100 times and recomputing D_KL each round. Fit the mean and sd of the randomized D_KL as functions of the observed D_KL using B-splines and derive per-gene log p-values from these fitted curves; set a seed (e.g. set.seed(1234)) for reproducibility.

## Related tools

- **singleCellHaystack** (R package implementing the D_KL computation, grid construction, and randomized shuffling null; original binary-detection version is on the 'binary' branch) — github.com/alexisvdb/singleCellHaystack
- **R** (Implementation environment for the re-implementation pipeline (set.seed, matrix binarization, B-spline fitting, TSV export))

## Examples

```
res <- haystack(dat.tsne, dat.expression)
```

## Evaluation signals

- Top-ranked genes from the re-implementation (e.g. gene_497 among top DEGs) match the ranking returned by show_result_haystack(res, n=10) on the toy data
- Randomized D_KL means/sds fit smoothly against observed D_KL via the B-spline curves; residuals of log p-values are stable across seeds
- Per-gene table contains finite D_KL and log p-values for every gene passing the detection filter (genes detected in >=10 cells)
- Re-running with set.seed(1234) reproduces the same rankings and log p-values written to haystack_toy_results.tsv

## Limitations

- Requires binary detection data; two genes with identical detection patterns but different expression levels receive identical D_KL and p-values due to the hard detection threshold
- P-values are inflated because the input coordinates (PCs, t-SNE, UMAP) are derived from the same expression data, a double-use issue the authors deferred to future updates
- Many zero counts reflect dropout from lack of sensitivity; imputation could partly address this but was not applied

## Evidence

- [abstract] DKL methodology basis: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [methods] package invocation: "res <- haystack(dat.tsne, dat.expression)"
- [discussion] binary input requirement: "The current implementation of our method requires binary detection data as input"
- [readme] DKL non-random positioning: [evidence span withheld — anchor status: external_doc]
- [discussion] p-value inflation caveat: "singleCellHaystack suffers from the same issue, because the input coordinates (PCs, t-SNE, or UMAP coordinates) are dimensions containing a large proportion"
