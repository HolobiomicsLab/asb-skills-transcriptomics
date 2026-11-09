---
name: scrna-seq-preprocessing-and-filtering
description: Use when an scRNA-seq expression matrix (gene x cell) is about to be
  imputed (e.g., with scISR) or k-means clustered and contains zero-only genes, sparsely
  expressed genes (<2 cells, <1% non-zero entries), or value ranges exceeding 100.
when_to_use_negative:
- Input is already a feature table or dimensionality-reduced embedding rather than
  a raw gene-by-cell expression matrix
- The dataset's value range is at or below 100 — applying an extra log2 rescale would
  be unnecessary and could distort values
- Genes already pass the filters (expressed in ≥2 cells and ≥1% non-zero entries);
  re-filtering would be a no-op
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: Downstream imputation method applied to the filtered matrix; performs hyper-geometric
    dropout identification followed by subspace regression imputation
  repo: https://github.com/duct317/scISR
- name: irlba
  role: PCA (prcomp_irlba, n = 50) on the filtered matrix prior to k-means clustering
    for ARI assessment
- name: mclust
  role: Provides adjustedRandIndex to evaluate k-means clustering quality on raw vs.
    imputed data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:6f85f4ca1bffd391c71dfa74bfdd1f0a6ff97d826b20ec72c68e8b9ff089c82f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - DrImpute
  - MAGIC
  - SAVER
  - Splatter
  - irlba
  - mclust
  - scISR
  - scImpute
  - scScope
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# scrna-seq-preprocessing-and-filtering

## Summary

Preprocess and filter scRNA-seq count/expression matrices before imputation or clustering, following the scISR paper's protocol: remove uninformative genes, drop all-zero genes, and rescale wide-range data with log2. These steps stabilize downstream clustering and prevent imputation on non-informative features.

## When to use

Use when an scRNA-seq expression matrix (gene x cell) is about to be imputed (e.g., with scISR) or k-means clustered and contains zero-only genes, sparsely expressed genes (<2 cells, <1% non-zero entries), or value ranges exceeding 100.

## When NOT to use

- Input is already a feature table or dimensionality-reduced embedding rather than a raw gene-by-cell expression matrix
- The dataset's value range is at or below 100 — applying an extra log2 rescale would be unnecessary and could distort values
- Genes already pass the filters (expressed in ≥2 cells and ≥1% non-zero entries); re-filtering would be a no-op

## Inputs

- Gene-by-cell scRNA-seq expression/count matrix (e.g., Goolam dataset via data('Goolam'); raw <- Goolam$data)
- Known cell-type label vector (e.g., Goolam$label) used later for ARI evaluation

## Outputs

- Filtered expression matrix with sparse/zero-only genes removed
- Log2-rescaled matrix when the original value range exceeds 100
- Imputation-ready matrix suitable for scISR or competitor imputation methods

## How to apply

First remove genes expressed in fewer than two cells and genes with less than one percent non-zero-valued entries, since such genes carry no signal for dropout identification or regression. Additionally drop genes that are zero across all cells (as in the README example: raw[rowSums(raw != 0) > 0, ]). If the range of values in the dataset is larger than 100, rescale the data using a log transformation with base 2 to bring the dynamic range into a workable scale. After filtering, the matrix can feed into imputation methods (scISR, scImpute, SAVER, MAGIC, DrImpute, scScope) and then k-means clustering with the true number of cell types k, evaluated by Adjusted Rand Index against known labels.

## Related tools

- **scISR** (Downstream imputation method applied to the filtered matrix; performs hyper-geometric dropout identification followed by subspace regression imputation) — https://github.com/duct317/scISR
- **irlba** (PCA (prcomp_irlba, n = 50) on the filtered matrix prior to k-means clustering for ARI assessment)
- **mclust** (Provides adjustedRandIndex to evaluate k-means clustering quality on raw vs. imputed data)

## Examples

```
raw_filer <- raw[rowSums(raw != 0) > 0, ]; imputed <- scISR(data = raw_filer)
```

## Evaluation signals

- After filtering, no gene in the matrix has fewer than two cells expressing it and no gene has less than 1% non-zero entries
- If log2 rescaling was applied, the matrix value range should no longer exceed 100
- Filtered raw matrix should yield a lower ARI than the imputed matrix in k-means clustering (ARI computed with adjustedRandIndex against true labels)
- Downstream k-means must use the true number of cell types k with nstart = 2000, iter.max = 2000 on 50-component PCA embedding
- Filtering should not error on datasets like Goolam; the scISR call on the filtered matrix should return an imputed matrix of the same cell dimensionality

## Limitations

- The paper notes rescaling via log2 (base 2) is applied only when the value range exceeds 100; applying it conditionally is part of the protocol
- The paper defers key robustness evidence to supplementary figures (S19, S21, S24), and simulated dataset generation parameters are only partially specified, limiting full reproducibility
- No version is stated for the R stats package or the scISR software

## Evidence

- [results] gene filtering before imputation/clustering: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [results] log2 rescaling rule: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [readme] README all-zero gene filter and PCA setup: "# Filter genes that have only zeros from raw data
raw_filer <- raw[rowSums(raw != 0) > 0, ]"
- [readme] downstream clustering evaluation context: "pca_raw <- irlba::prcomp_irlba(t(raw_filer), n = 50)$x"
