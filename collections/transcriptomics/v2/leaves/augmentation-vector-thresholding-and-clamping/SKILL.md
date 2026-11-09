---
name: augmentation-vector-thresholding-and-clamping
description: Use when building scTab-style training data from donor- and cell-type-annotated,
  size-factor-normalized (10000 counts, log1p) scRNA-seq matrices and you need precomputed
  augmentation vectors for donor-difference data augmentation.
when_to_use_negative:
- Input data lacks reliable donor identity or cell type labels, so cross-donor difference
  vectors cannot be computed per cell type
- The expression matrix has not been size-factor normalized to 10000 counts and log1p
  transformed, which would make donor-difference vectors incomparable across datasets
- A cell type is observed in too few donors to reliably estimate mean-expression centroids
  per donor
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scTab
  role: Defines the donor-difference augmentation scheme; the resulting vectors are
    added to gene expression vectors during scTab model training
  repo: https://github.com/theislab/scTab
- name: NumPy
  role: Accumulates per (cell type, donor) mean expression centroids and performs
    the entrywise zero-thresholding and clamping of difference vectors
- name: cellxgene-census
  role: Queries the CELLxGENE corpus to retrieve the cell-by-gene matrix with cell
    type and donor annotations used as input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:8dd1d61b7c57cf39645f3e771818d4bac0dee9a04a7795638857dc1cb9edd839
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - cellxgene-census
  - scTab
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# augmentation-vector-thresholding-and-clamping

## Summary

Precomputes scTab's donor-difference augmentation vectors: mean-expression centroids are differenced across donors within each cell type, noise is suppressed by zeroing entries in [-0.25, 0.25], and surviving entries are clamped to [-1.5, 1.5]. Clustering and pooling then yield the augmentation-vector set added to gene expression vectors during model training.

## When to use

Use when building scTab-style training data from donor- and cell-type-annotated, size-factor-normalized (10000 counts, log1p) scRNA-seq matrices and you need precomputed augmentation vectors for donor-difference data augmentation.

## When NOT to use

- Input data lacks reliable donor identity or cell type labels, so cross-donor difference vectors cannot be computed per cell type
- The expression matrix has not been size-factor normalized to 10000 counts and log1p transformed, which would make donor-difference vectors incomparable across datasets
- A cell type is observed in too few donors to reliably estimate mean-expression centroids per donor

## Inputs

- Cell-by-gene matrix of scRNA-seq data, size-factor normalized to 10000 counts and log1p transformed (e.g. from CELLxGENE census version 2023-05-15)
- Cell type annotation per cell (Cell Ontology terms, filtered to subtypes of 'native cell' with at least 5000 cells per type)
- Donor annotation per cell (each cell type observed in at least 30 donors)

## Outputs

- augmentation_vectors.parquet — pooled table of clamped, thresholded augmentation vectors over the 19331 protein coding genes, used to perturb gene expression vectors during model training

## How to apply

Subsample 500,000 cells from the preprocessed cell-by-gene matrix (size-factor normalized to 10000 counts and log1p transformed) together with their cell type and donor labels. Group cells by cell type and donor and compute mean expression centroids per group using NumPy accumulation. Compute difference vectors between centroids of the same cell type from two different donors, set all entries within [-0.25, 0.25] to zero to suppress noise, and clamp remaining entries to [-1.5, 1.5]. Cluster the vectors with k-means using k=50 to identify cell-type-independent augmentation directions, retain only clusters containing more than 2000 vectors, and pool the retained clusters' vectors into the final augmentation-vector set.

## Related tools

- **scTab** (Defines the donor-difference augmentation scheme; the resulting vectors are added to gene expression vectors during scTab model training) — https://github.com/theislab/scTab
- **NumPy** (Accumulates per (cell type, donor) mean expression centroids and performs the entrywise zero-thresholding and clamping of difference vectors)
- **cellxgene-census** (Queries the CELLxGENE corpus to retrieve the cell-by-gene matrix with cell type and donor annotations used as input)

## Examples

```
aug = np.clip(diff_vecs, -1.5, 1.5); aug[np.abs(aug) <= 0.25] = 0.0
```

## Evaluation signals

- Every retained k-means cluster contains more than 2000 vectors
- All stored vector entries lie within the clamped range [-1.5, 1.5]
- Entries inside the suppressed interval [-0.25, 0.25] are absent (zeroed) in the output
- Clustering was run with k=50 before retention and the final set is the pool of vectors from retained clusters only
- Output file augmentation_vectors.parquet exists and matches the gene space of 19331 protein coding genes

## Limitations

- The article notes that further and more refined choices for data augmentation may be explored in the context of cellular representation learning
- Difference-vector magnitudes depend on the 10000-count size-factor normalization and log1p transform; applying these vectors to differently normalized data is not covered
- Suppression and clamping thresholds (-0.25/0.25 and 1.5) are fixed heuristics tuned for this pipeline and may not transfer to other normalization schemes

## Evidence

- [methods] thresholding and clamping thresholds: [evidence span withheld — anchor status: not_found]
- [methods] k-means clustering and retention rule: [evidence span withheld — anchor status: not_found]
- [methods] centroid computation: [evidence span withheld — anchor status: not_found]
- [abstract] rationale for augmentation: "we show that the proposed data augmentation schema improves model generalization"
- [readme] where the augmentation-vector code lives: "``data_augmentation``: Notebooks related to data augmentation &rarr; calculation of augmentation vectors +"
