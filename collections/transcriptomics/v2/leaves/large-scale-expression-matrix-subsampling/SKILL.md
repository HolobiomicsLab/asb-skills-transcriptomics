---
name: large-scale-expression-matrix-subsampling
description: Use when you need to derive augmentation vectors from a very large cross-tissue
  scRNA-seq matrix (e.g. the 22.2M-cell CELLxGENE corpus) and full-data centroid computation
  is infeasible, so a donor/cell-type-annotated subsample is required.
when_to_use_negative:
- The expression matrix has not yet undergone size-factor normalization to 10000 counts
  and log1p transformation — raw counts will produce distorted donor-difference centroids.
- Cells lack donor annotations, since the augmentation vectors are defined as differences
  between centroids of the same cell type from different donors.
- The goal is training the scTab classifier itself rather than computing augmentation
  vectors — model training consumes the full Merlin-format store, not the 500k-cell
  subsample.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3272
tools:
- name: NumPy
  role: Used for accumulation-based computation of per-cell-type/per-donor mean expression
    centroids over the 500,000-cell subsample
- name: scTab
  role: Provides the cellnet package and notebooks/data_augmentation notebooks in
    which subsampling and augmentation-vector calculation are implemented
  repo: https://github.com/theislab/scTab
- name: cellxgene-census
  role: Used to query and download the CELLxGENE census subset that supplies the source
    cell-by-gene matrix and annotations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:d9f28ca9ce219fd8ef3dd35901b230e944d6aa79b95e2f7829e53849dbdc3536
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - cellxgene-census
  - scTab
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# large-scale-expression-matrix-subsampling

## Summary

Subsamples 500,000 cells from the preprocessed (size-factor normalized to 10000 counts and log1p-transformed) cell-by-gene matrix together with cell type and donor annotations, to make computing donor-difference augmentation vectors computationally tractable for scTab's data augmentation pipeline.

## When to use

Use when you need to derive augmentation vectors from a very large cross-tissue scRNA-seq matrix (e.g. the 22.2M-cell CELLxGENE corpus) and full-data centroid computation is infeasible, so a donor/cell-type-annotated subsample is required.

## When NOT to use

- The expression matrix has not yet undergone size-factor normalization to 10000 counts and log1p transformation — raw counts will produce distorted donor-difference centroids.
- Cells lack donor annotations, since the augmentation vectors are defined as differences between centroids of the same cell type from different donors.
- The goal is training the scTab classifier itself rather than computing augmentation vectors — model training consumes the full Merlin-format store, not the 500k-cell subsample.

## Inputs

- Preprocessed cell-by-gene matrix (size-factor normalized to 10000 counts, log1p transformed; gene space subset to 19331 protein coding genes)
- Cell type annotations per cell (subtypes of 'native cell' from the Cell Ontology)
- Donor annotations per cell
- CELLxGENE census data (version 2023-05-15), downloaded via cellxgene-census

## Outputs

- Subsampled matrix of 500,000 cells with paired cell type and donor labels (in-memory NumPy arrays for centroid accumulation)
- Downstream pooled augmentation-vector table serialized as augmentation_vectors.parquet (Parquet format)

## How to apply

Subsample 500,000 cells from the preprocessed cell-by-gene matrix — cells must already be size-factor normalized to 10000 counts and log1p transformed — retaining their cell type and donor annotations. Group the subsampled cells by cell type and donor and compute mean expression centroids per group using NumPy accumulation, then take differences between centroids of the same cell type across different donors. These difference vectors are subsequently thresholded (zero out entries in [-0.25, 0.25]), clamped to [-1.5, 1.5], clustered with k-means (k=50), and filtered by cluster size (>2000 vectors) downstream. The rationale is that donor-to-donor differences within a cell type capture meaningful biological perturbation directions without requiring the full 22.2-million-cell corpus to be held in memory.

## Related tools

- **NumPy** (Used for accumulation-based computation of per-cell-type/per-donor mean expression centroids over the 500,000-cell subsample)
- **scTab** (Provides the cellnet package and notebooks/data_augmentation notebooks in which subsampling and augmentation-vector calculation are implemented) — https://github.com/theislab/scTab
- **cellxgene-census** (Used to query and download the CELLxGENE census subset that supplies the source cell-by-gene matrix and annotations)

## Examples

```
import pandas as pd; aug_vec = pd.read_parquet("augmentation_vectors.parquet"); assert aug_vec.map(lambda v: v == 0 or not (-0.25 < v < 0.25)).all().all()
```

## Evaluation signals

- The subsample contains exactly 500,000 cells drawn from the preprocessed (sf-10000, log1p) matrix, each carrying valid cell type and donor labels.
- All entries of the derived difference vectors lie within the clamped range [-1.5, 1.5].
- No entries of the derived vectors fall in the interval (-0.25, 0.25) — entries within [-0.25, 0.25] were set to zero.
- Each k-means cluster (k=50) retained for pooling contains more than 2000 vectors.
- The final augmentation-vector table is serialized to disk as augmentation_vectors.parquet with all entries within the clamped range.

## Limitations

- Subsampling means centroids and difference vectors are estimated from a 500,000-cell subset rather than the full 22.2-million-cell corpus, so rare cell-type/donor combinations may be underrepresented.
- Entries with small absolute differences are discarded (zeroed within [-0.25, 0.25]) to suppress noise, so subtle donor effects are not captured.
- The paper notes that 'further and more refined choices for data augmentation may be explored', so the subsampling-based vector scheme is not established as optimal.

## Evidence

- [readme] notebooks/data_augmentation: Notebooks related to data augmentation &rarr; calculation of augmentation vectors + evaluation: "notebooks/data_augmentation: Notebooks related to data augmentation &rarr; calculation of augmentation vectors + evaluation"
- [methods] Normalize each cell to have `10000` counts (size factor normalization): "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [abstract] train it using a novel data augmentation scheme across a large corpus of single-cell RNA-seq observations (22.2 million cells): "train it using a novel data augmentation scheme across a large corpus of single-cell RNA-seq observations (22.2 million cells)"
- [methods] `01_download_data.ipynb`: Subset and download data from CELLxGENE census.: "`01_download_data.ipynb`: Subset and download data from CELLxGENE census."
