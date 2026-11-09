---
name: single-cell-rna-seq-minmax-rescaling
description: Use when an scRNA-seq expression matrix is normalized but unscaled and
  will feed a non-negative-kernel autoencoder or neural-network feature-selection/dimensionality-reduction
  step that requires bounded, non-negative input values.
when_to_use_negative:
- Data is not yet normalized — scCAN's workflow assumes an already-normalized matrix;
  rescaling raw counts directly would misrepresent expression levels
- The downstream method is not a non-negative/NN-constrained model (e.g. PCA-based
  tools like CIDR or SEURAT that accept unbounded standardized values)
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCAN
  role: R package whose pipeline implements min-max rescaling as the first step before
    the non-negative-kernel autoencoder feature filter
  repo: https://cran.r-project.org/package=scCAN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:550e3832a9e249639b100a14f8212a904236c6d8f921bd80dabce3e13c26a515
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scCAN
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# single-cell-rna-seq-minmax-rescaling

## Summary

Rescale an already-normalized scRNA-seq expression matrix (cells as rows, genes as columns) so each gene's values fall in the 0–1 range via min-max scaling, as the first step of the scCAN pipeline before non-negative-kernel autoencoder feature selection.

## When to use

Use when an scRNA-seq expression matrix is normalized but unscaled and will feed a non-negative-kernel autoencoder or neural-network feature-selection/dimensionality-reduction step that requires bounded, non-negative input values.

## When NOT to use

- Data is not yet normalized — scCAN's workflow assumes an already-normalized matrix; rescaling raw counts directly would misrepresent expression levels
- The downstream method is not a non-negative/NN-constrained model (e.g. PCA-based tools like CIDR or SEURAT that accept unbounded standardized values)

## Inputs

- Normalized scRNA-seq expression matrix with cells as rows and genes as columns (e.g. the Pollen dataset matrix)
- Optional: raw count matrix requiring log2(data + 1) transformation when max(data) > 100 (per scCAN README)

## Outputs

- Min-max rescaled expression matrix with all values in the 0–1 range, suitable for non-negative-kernel autoencoder training

## How to apply

Load the normalized expression matrix with cells as rows and genes as columns; scCAN also log2-transforms raw-count data first (if max(data) > 100, apply log2(data + 1)). Then apply min-max scaling to rescale each column's values to the range 0 to 1, as the paper's first workflow step states: 'rescale the data to a range of 0 to 1'. This bounded, non-negative scaling is a prerequisite for the subsequent non-negative-kernel autoencoder, whose encoder weights must remain non-negative while selecting informative genes. In the full scCAN workflow this rescaled matrix is passed directly to the autoencoder filter, which retains the top 5000 genes with non-zero coefficients and highest coefficient variances.

## Related tools

- **scCAN** (R package whose pipeline implements min-max rescaling as the first step before the non-negative-kernel autoencoder feature filter) — https://cran.r-project.org/package=scCAN

## Examples

```
if(max(data)>100) data <- log2(data + 1); result <- scCAN(data, r.seed = 1)
```

## Evaluation signals

- After rescaling, every value in the output matrix lies within [0, 1] (check min â¥ 0 and max â¤ 1 per gene column)
- The rescaled matrix retains the same cell-by-gene dimensions as the input normalized matrix
- Downstream non-negative-kernel autoencoder trains without negative-value errors, and the top-5000 filtered gene set with non-zero coefficients is produced
- Applying the full scCAN pipeline to a reference dataset (e.g. Pollen) yields an adjusted Rand index consistent with published cell-type labels and a cluster count matching the known number of cell types

## Limitations

- Min-max scaling is applied only after normalization; scCAN does not state behavior for raw counts, and README guidance (log2(data + 1) if max > 100) must be applied first
- No software version for scCAN is stated in the article's methods; reproducibility of exact rescaling behavior may depend on the package version installed
- Rescaling to 0–1 is designed for the non-negative autoencoder context; it may not be optimal for clustering tools that expect standardized (z-scored) data

## Evidence

- [methods] Min-Max Scaling: [evidence span withheld — anchor status: not_found]
- [methods] Feature selection follows rescaling: "the second step removes genes that are not informative"
- [readme] README log-transform guidance: "Add log transformation: `if(max(data)>100) data <- log2(data + 1)`"
- [readme] README invocation: "Generate clustering result, the input matrix has rows as samples and columns as genes: `result <- scCAN(data, r.seed = 1)`"
