---
name: kullback-leibler-divergence-scoring
description: Use when predicting DEGs from single-cell data without clustering cells,
  by scoring each gene's detection pattern against cell positions in a multidimensional
  space (t-SNE, PCA, pseudotime, or spatial coordinates) with Kullback–Leibler divergence.
when_to_use_negative:
- Gene expression is continuous (e.g. normalized read counts) and cannot reasonably
  be binarized — the current implementation requires binary detection data
- Genes differ mainly in expression level but share the same detection pattern — the
  hard detection threshold gives them identical D_KL and p-value, so a level-aware
  method is preferable
- A cluster-based DEG analysis is explicitly required, e.g. when clusters are already
  validated and per-cluster markers are the deliverable
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: R package implementing the D_KL scoring, randomization, and B-spline p-value
    calibration; use the 'binary' branch for the version described in this paper
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: R
  role: Runtime environment for loading toy data, binarizing expression, computing
    D_KL, and writing the results TSV
- name: Seurat
  role: Used in vignettes to prepare embeddings (t-SNE/PCA) and expression data fed
    into haystack
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:a51bdfd24a01584a6b38cc4a1dafcbb41bf91bc2a47917bba5e8748238bbb6cf
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# kullback-leibler-divergence-scoring

## Summary

Reconstructs the singleCellHaystack D_KL scoring and randomization p-value pipeline: per-gene detection distributions over a grid in a 2D/2D+ input space (e.g. t-SNE) are compared to a Gaussian-kernel reference Q via Kullback–Leibler divergence, and significance is calibrated by shuffling gene detections across cells and fitting B-splines to randomized D_KL statistics.

## When to use

Use when predicting DEGs from single-cell data without clustering cells, by scoring each gene's detection pattern against cell positions in a multidimensional space (t-SNE, PCA, pseudotime, or spatial coordinates) with Kullback–Leibler divergence.

## When NOT to use

- Gene expression is continuous (e.g. normalized read counts) and cannot reasonably be binarized — the current implementation requires binary detection data
- Genes differ mainly in expression level but share the same detection pattern — the hard detection threshold gives them identical D_KL and p-value, so a level-aware method is preferable
- A cluster-based DEG analysis is explicitly required, e.g. when clusters are already validated and per-cluster markers are the deliverable

## Inputs

- dat.expression: genes x cells binary detection matrix (or raw counts to binarize)
- dat.tsne (or dat.pca / spatial / pseudotime coordinates): cells x 2+ numeric embedding defining the multidimensional input space

## Outputs

- Per-gene table of D_KL values and log p-values (e.g. haystack_toy_results.tsv)
- Ranked DEG list via show_result_haystack(res, n=10)
- Grid coordinates (res$info$grid.coordinates) reusable for kmeans_haystack / hclust_haystack

## How to apply

Load the singleCellHaystack toy data (dat.expression genes x cells matrix and dat.tsne 2D coordinates) with set.seed for reproducibility, and binarize expression into a gene detection matrix. Build a grid over the input space and construct a reference distribution Q over grid points using a Gaussian kernel of cell densities. For each gene, compute the P(G=T) and P(G=F) distributions over grid points and their Kullback–Leibler divergence D_KL against Q via the summation formula. Generate null distributions by repeatedly shuffling each gene's detections across cells (e.g. ~100 randomizations) and recomputing D_KL, then model the mean and sd of randomized D_KL as functions of observed D_KL with B-splines to derive per-gene log p-values. Optionally filter genes detected in fewer than 10 cells first to cut computational time, and return results with show_result_haystack.

## Related tools

- **singleCellHaystack** (R package implementing the D_KL scoring, randomization, and B-spline p-value calibration; use the 'binary' branch for the version described in this paper) — https://github.com/alexisvdb/singleCellHaystack
- **R** (Runtime environment for loading toy data, binarizing expression, computing D_KL, and writing the results TSV)
- **Seurat** (Used in vignettes to prepare embeddings (t-SNE/PCA) and expression data fed into haystack) — https://github.com/satijalab/seurat

## Examples

```
res <- haystack(dat.tsne, dat.expression); show_result_haystack(res, n=10)
```

## Evaluation signals

- Top-ranked genes from the re-implementation (e.g. gene_497 among top DEGs) match the ranking returned by show_result_haystack(res, n=10)
- Observed D_KL for known non-random genes exceeds the mean of their ~100 shuffled-detection randomized D_KL values
- Log p-values are monotone with (observed D_KL minus B-spline-predicted randomized D_KL mean) standardized by the fitted sd
- Binarized detection matrix has the same genes x cells dimensions as dat.expression and matches the detection threshold used downstream

## Limitations

- Requires binary detection data as input; no continuous expression support implemented at time of publication
- P-values are inflated because the input coordinates (PCs, t-SNE, UMAP) derive from the same expression data used for DEG prediction
- The hard detection threshold is a weak point: genes detected in the same cell subset but with ten-fold different expression levels get identical D_KL and p-value
- Zero counts may reflect dropout events from lack of sensitivity; imputation is suggested but not applied

## Evidence

- [abstract] Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [methods] haystack call on t-SNE: "res <- haystack(dat.tsne, dat.expression)"
- [methods] gene filtering: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [discussion] binary detection requirement: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [discussion] DKL p-value inflation caveat: "singleCellHaystack suffers from the same issue, because the input coordinates (PCs, t-SNE, or UMAP coordinates) are dimensions containing a large proportion of the variability"
- [readme] README version note: [evidence span withheld — anchor status: external_doc]
