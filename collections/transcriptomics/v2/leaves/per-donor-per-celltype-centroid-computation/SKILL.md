---
name: per-donor-per-celltype-centroid-computation
description: 'Use when precomputing scTab data augmentation: per-cell-type, per-donor
  mean expression centroids must be derived from a size-factor-normalized, log1p CxG
  matrix to build donor-difference augmentation vectors.'
when_to_use_negative:
- Input is raw-count or unnormalized data — centroids would mix library-size effects
  instead of reflecting size-factor-normalized (10,000 counts), log1p expression.
- The analysis needs per-cell resolution or within-group variance; per-(cell type,
  donor) means discard single-cell heterogeneity.
- Augmentation vectors are already precomputed (augmentation_vectors.parquet exists)
  and only sampling of stored vectors during training is required.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scTab
  role: Reference implementation; notebooks/data_augmentation contains the centroid
    and augmentation-vector calculation code
  repo: https://github.com/theislab/scTab
- name: NumPy
  role: Accumulation-based computation of mean expression centroids per (cell type,
    donor) group
- name: cellxgene-census
  role: Queries and downloads the size-factor-normalized, log1p-transformed CxG census
    matrix with cell type and donor annotations
  repo: https://chanzuckerberg.github.io/cellxgene-census/python-api.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:a66ab9c2ce127d3f251d8c603f2dd197c53efee75f75d5bb6c8f23b08a9406a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - cellxgene-census
  - scTab
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# per-donor-per-celltype-centroid-computation

## Summary

Computes mean gene-expression centroids per (cell type, donor) pair from a size-factor-normalized (10,000 counts) and log1p-transformed scRNA-seq cell-by-gene matrix; these group means are the basis from which scTab derives donor-difference augmentation vectors.

## When to use

Use when precomputing scTab data augmentation: per-cell-type, per-donor mean expression centroids must be derived from a size-factor-normalized, log1p CxG matrix to build donor-difference augmentation vectors.

## When NOT to use

- Input is raw-count or unnormalized data — centroids would mix library-size effects instead of reflecting size-factor-normalized (10,000 counts), log1p expression.
- The analysis needs per-cell resolution or within-group variance; per-(cell type, donor) means discard single-cell heterogeneity.
- Augmentation vectors are already precomputed (augmentation_vectors.parquet exists) and only sampling of stored vectors during training is required.

## Inputs

- Preprocessed cell-by-gene matrix from CELLxGENE census (CxG, version 2023-05-15), size-factor normalized to 10,000 counts and log1p transformed
- Cell type annotation per cell (CELLxGENE cell ontology terms, subtype of 'native cell')
- Donor annotation per cell
- Gene feature list (19,331 protein-coding genes, notebooks/store_creation/features.parquet)

## Outputs

- Matrix/table of mean expression centroids, one row per unique (cell type, donor) group over the 19,331-gene space
- Per-group cell counts validating group sizes in the 500,000-cell subsample
- Downstream: donor-difference augmentation vector table serialized to disk in Parquet format as augmentation_vectors.parquet

## How to apply

Start from the preprocessed cell-by-gene matrix in which every cell is size-factor normalized to 10,000 counts and log1p transformed and whose gene space is the 19,331 protein-coding genes (features.parquet); subsample 500,000 cells together with their cell type and donor annotations. Enumerate the unique (cell type, donor) pairs in the subsample — the study only includes cell types observed in at least 30 donors, which makes donor-to-donor comparisons statistically reliable. For each pair, accumulate per-group sums over the gene matrix with NumPy (np.add.at on group-indexed rows) and divide by per-group cell counts (np.bincount) to yield one mean expression centroid per group; accumulation avoids materializing per-group matrix copies. Rationale: pairing centroids of the same cell type across different donors yields full-gene-space difference vectors that capture donor-specific expression shifts, and these become the augmentation vectors added to training cells. The centroids therefore feed the downstream pipeline — zeroing entries in [-0.25, 0.25], clamping to [-1.5, 1.5], k-means with k=50, retaining clusters with more than 2,000 vectors, and serializing the pooled set as augmentation_vectors.parquet.

## Related tools

- **scTab** (Reference implementation; notebooks/data_augmentation contains the centroid and augmentation-vector calculation code) — https://github.com/theislab/scTab
- **NumPy** (Accumulation-based computation of mean expression centroids per (cell type, donor) group)
- **cellxgene-census** (Queries and downloads the size-factor-normalized, log1p-transformed CxG census matrix with cell type and donor annotations) — https://chanzuckerberg.github.io/cellxgene-census/python-api.html

## Examples

```
sums = np.zeros((n_groups, X.shape[1])); np.add.at(sums, group_idx, X); centroids = sums / np.bincount(group_idx)[:, None]
```

## Evaluation signals

- Centroid count equals the number of unique (cell type, donor) pairs present in the 500,000-cell subsample, and each centroid has 19,331 gene entries.
- Per gene, the sum of deviations (cell expression − its group centroid) over the subsample is 0, confirming each centroid is the exact group mean.
- Centroid entries are non-negative, since they are means of log1p-transformed, size-factor-normalized counts.
- Downstream validation: after k-means (k=50), every retained cluster contains more than 2,000 difference vectors.
- All entries in augmentation_vectors.parquet lie within the clamped range [-1.5, 1.5], and no entries fall in the zeroed interval (-0.25, 0.25).

## Limitations

- Centroids are estimated from a 500,000-cell subsample rather than the full 22.2-million-cell training corpus, so donor means carry sampling variance.
- Entries in (-0.25, 0.25) are treated as noise and zeroed and remaining values are clamped to [-1.5, 1.5], so small-magnitude donor shifts are deliberately suppressed.
- Donor-difference pairing is only meaningful for cell types observed in many donors (the study filters to at least 30 donors per cell type), and the paper notes that more refined augmentation choices remain to be explored.

## Evidence

- [other] Subsample 500,000 cells from the preprocessed cell-by-gene matrix (cells already size-factor normalized to 10000 counts and log1p transformed): [evidence span withheld — anchor status: not_found]
- [other] Group the subsampled cells by cell type and donor and compute mean expression centroids per group using NumPy accumulation.: [evidence span withheld — anchor status: not_found]
- [methods] 1. Normalize each cell to have `10000` counts (size factor normalization): "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [methods] * Gene space subset to `19331` protein coding genes (see `notebooks/store_creation/features.parquet` for full list): "* Gene space subset to `19331` protein coding genes (see `notebooks/store_creation/features.parquet` for full list)"
- [methods] Each cell type has to be observed in at least `30` donors to reliably quantify whether the classifier can generalize to new donors.: "Each cell type has to be observed in at least `30` donors to reliably quantify whether the classifier can generalize to new donors."
- [abstract] we show that the proposed data augmentation schema improves model generalization: "we show that the proposed data augmentation schema improves model generalization"
- [readme] Notebooks related to data augmentation &rarr; calculation of augmentation vectors + evaluation: "Notebooks related to data augmentation &rarr; calculation of augmentation vectors + evaluation"
- [discussion] In the context of cellular representation learning, further and more refined choices for data augmentation may be explored.: "In the context of cellular representation learning, further and more refined choices for data augmentation may be explored."
