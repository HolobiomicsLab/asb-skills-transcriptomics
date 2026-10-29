---
name: binary-gene-detection-binarization
description: Use when preparing an expression matrix (e.g. dat.expression, genes x
  cells) as input to singleCellHaystack's haystack() or DKL computation, which accepts
  only binary detection data per cell.
when_to_use_negative:
- Your analysis depends on expression magnitude, not just detection — two genes with
  the same detection pattern but ten-fold different levels get identical DKL and p-values
  under this hard threshold.
- Your downstream method supports continuous expression (e.g. normalized read counts);
  the article notes continuous-measure support was planned but not yet available in
  this implementation.
- You need p-values corrected for double use of gene expression data (e.g. when coordinates
  were derived from the same expression data) — singleCellHaystack p-values are known
  to be inflated in that setting.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: R package whose haystack() consumes the binarized detection matrix together
    with cell coordinates to compute per-gene D_KL, randomized p-values, and gene
    rankings
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: R
  role: implementation environment for loading dat.expression/dat.tsne, binarizing
    counts, and running the DKL pipeline
- name: Seurat
  role: used to load single-cell/spatial datasets (e.g. via SeuratData) whose expression
    matrices are binarized before haystack analysis
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:1fa4c42985418acba7e04522034305cec021edfccd64a7325584e2412162942b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# binary-gene-detection-binarization

## Summary

Convert a single-cell expression matrix (genes x cells) into a binary gene detection matrix (1 = detected, 0 = not detected), because singleCellHaystack's Kullback-Leibler divergence approach requires binary detection data rather than continuous read counts.

## When to use

Use when preparing an expression matrix (e.g. dat.expression, genes x cells) as input to singleCellHaystack's haystack() or DKL computation, which accepts only binary detection data per cell.

## When NOT to use

- Your analysis depends on expression magnitude, not just detection — two genes with the same detection pattern but ten-fold different levels get identical DKL and p-values under this hard threshold.
- Your downstream method supports continuous expression (e.g. normalized read counts); the article notes continuous-measure support was planned but not yet available in this implementation.
- You need p-values corrected for double use of gene expression data (e.g. when coordinates were derived from the same expression data) — singleCellHaystack p-values are known to be inflated in that setting.

## Inputs

- genes x cells expression/count matrix (e.g. dat.expression from the singleCellHaystack toy data)
- detection threshold criterion (default: non-zero counts)
- cell coordinates in a multidimensional space (t-SNE/UMAP/PCA, 2D spatial coordinates, or pseudotime)

## Outputs

- binary gene detection matrix (same genes x cells dimensions as the input)
- gene-filtered matrix after removing genes detected in < 10 cells
- detection matrix ready for haystack() DKL scoring and downstream TSV output (haystack_toy_results.tsv)

## How to apply

Take the genes x cells expression matrix (e.g. the singleCellHaystack toy data dat.expression) and threshold each entry to produce a detection matrix where 1 marks a cell in which the gene is detected and 0 otherwise; the detection cutoff is left to the user and can be adapted per dataset. Before binarizing, filter genes detected in fewer than 10 non-zero-count cells to remove very lowly expressed genes and reduce computation time. Pair the detection matrix with cell coordinates in a multidimensional space (e.g. dat.tsne 2D t-SNE coordinates, first 20 principal components, spatial spot coordinates, or pseudotime), then pass both to haystack(). Rationale: singleCellHaystack computes P(G=T) and P(G=F) distributions over grid points and their D_KL against a reference Q, so only the detection pattern, not expression magnitude, drives the result. Set a seed (e.g. set.seed(1234)) so the subsequent randomization-based null distribution and p-values are reproducible.

## Related tools

- **singleCellHaystack** (R package whose haystack() consumes the binarized detection matrix together with cell coordinates to compute per-gene D_KL, randomized p-values, and gene rankings) — https://github.com/alexisvdb/singleCellHaystack
- **R** (implementation environment for loading dat.expression/dat.tsne, binarizing counts, and running the DKL pipeline)
- **Seurat** (used to load single-cell/spatial datasets (e.g. via SeuratData) whose expression matrices are binarized before haystack analysis) — https://github.com/satijalab/seurat

## Examples

```
set.seed(1234); dat.detection <- dat.expression > 0; res <- haystack(dat.tsne, dat.expression)
```

## Evaluation signals

- The detection matrix has exactly the same genes x cells dimensions as the input expression matrix, and all entries are binary (0/1).
- No gene in the detection matrix is detected in fewer than 10 cells after filtering (article threshold).
- Two genes sharing the same detection pattern produce identical D_KL values and p-values — a consequence of the hard-threshold binarization.
- The top-ranked genes from the re-implementation (e.g. gene_497 among the top DEGs) match the ranking returned by show_result_haystack(res, n=10) on the toy data.
- Randomized D_KL null distributions (≈100 randomizations of detection labels across cells) are reproducible given the same set.seed(1234).

## Limitations

- The use of a hard threshold for detection is a weak point: genes with different expression levels but the same detection pattern receive identical D_KL and p-values.
- Many zero counts may be dropout events from lack of sensitivity rather than true absence; imputation could partly address this but was not applied.
- The definition of detection is left to the user, which introduces potential arbitrariness across datasets.
- Because input coordinates (PCs, t-SNE, UMAP) are derived from the same expression data, returned p-values are inflated due to double use of the data.
- The current (binary-branch) implementation requires binary detection data only; continuous normalized read counts are not yet supported.

## Evidence

- [abstract] singleCellHaystack predicts DEGs without explicit cell clustering, using Kullback-Leibler divergence to find genes expressed in non-randomly positioned cell subsets: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [discussion] The method requires binary detection data; thresholding is a prerequisite for haystack().: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [methods] Genes detected in fewer than 10 cells are filtered before analysis.: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [discussion] The detection threshold is user-defined and dataset-dependent.: "The definition of detection is left to the user and could be modified according to the characteristics of each dataset."
- [discussion] Hard-threshold detection is acknowledged as a weak point of the method.: "we are aware that the use of a hard threshold for detection is a weak point of our method."
- [readme] The updated singleCellHaystack operates on binary-like activity patterns in any input space.: [evidence span withheld — anchor status: external_doc]
