---
name: kmeans-cluster-support-filtering
description: 'Use when constructing scTab-style data augmentation: you already have
  donor-difference vectors per cell type (noise-zeroed, clamped to [-1.5, 1.5]) and
  need to consolidate them into a compact augmentation-vector set via k-means clustering
  with cluster-support filtering.'
when_to_use_negative:
- The input is raw single-cell expression vectors rather than donor-difference centroid
  vectors — clustering cells directly does not yield the donor-shift directions this
  skill extracts.
- The dataset lacks donor stratification (cell types not observed across multiple
  donors), so donor-difference centroids and the resulting difference vectors cannot
  be reliably computed.
- A precomputed augmentation_vectors.parquet from the scTab release already exists
  and does not need to be regenerated.
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: scTab
  role: Provides the data-augmentation notebooks and pipeline (notebooks/data_augmentation)
    in which the k-means clustering and support filtering are implemented
  repo: https://github.com/theislab/scTab
- name: NumPy
  role: Used for the NumPy-accumulation of per-(cell type, donor) mean-expression
    centroids from which the difference vectors fed into clustering are computed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:2133c199bcdd373f60c0ddd25e2ba95736a466965fcd20453ab038712f803293
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - cellxgene-census
  - scTab
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# kmeans-cluster-support-filtering

## Summary

Distills scTab's donor-difference augmentation vectors into a reusable set of cell-type-independent augmentation directions by k-means clustering (k=50) followed by a support filter that retains only clusters containing >2000 vectors. The pooled vectors are serialized to augmentation_vectors.parquet for use during model training.

## When to use

Use when constructing scTab-style data augmentation: you already have donor-difference vectors per cell type (noise-zeroed, clamped to [-1.5, 1.5]) and need to consolidate them into a compact augmentation-vector set via k-means clustering with cluster-support filtering.

## When NOT to use

- The input is raw single-cell expression vectors rather than donor-difference centroid vectors — clustering cells directly does not yield the donor-shift directions this skill extracts.
- The dataset lacks donor stratification (cell types not observed across multiple donors), so donor-difference centroids and the resulting difference vectors cannot be reliably computed.
- A precomputed augmentation_vectors.parquet from the scTab release already exists and does not need to be regenerated.

## Inputs

- Preprocessed cell-by-gene expression matrix (size-factor normalized to 10000 counts, log1p transformed; gene space subset to 19331 protein coding genes)
- Cell type and donor annotations for each cell
- 500,000-cell subsample of the training data
- Donor-difference vectors per cell type (entries in (-0.25, 0.25) zeroed, remaining entries clamped to [-1.5, 1.5])

## Outputs

- augmentation_vectors.parquet: Parquet-format table of pooled augmentation vectors from retained k-means clusters (all entries within [-1.5, 1.5])
- Cluster assignment + support counts for the k=50 k-means clusters (intermediate)

## How to apply

Start from difference vectors computed between per-(cell type, donor) mean-expression centroids, obtained from a 500,000-cell subsample of the preprocessed cell-by-gene matrix (size-factor normalized to 10000 counts, log1p transformed). Before clustering, zero all entries within the interval [-0.25, 0.25] to suppress noise and clamp remaining entries to [-1.5, 1.5]. Cluster the filtered vectors with k-means using k=50 to identify cell-type-independent augmentation directions; the rationale is that directions shared across cell types represent donor/batch variation rather than cell identity. Apply the support filter: retain only clusters containing more than 2000 vectors, discarding low-support clusters as noise. Pool the vectors of retained clusters into the final augmentation-vector table and write it to disk as a Parquet file named augmentation_vectors.parquet, to be sampled and added to gene expression vectors during training.

## Related tools

- **scTab** (Provides the data-augmentation notebooks and pipeline (notebooks/data_augmentation) in which the k-means clustering and support filtering are implemented) — https://github.com/theislab/scTab
- **NumPy** (Used for the NumPy-accumulation of per-(cell type, donor) mean-expression centroids from which the difference vectors fed into clustering are computed)

## Evaluation signals

- Each retained cluster contains more than 2000 vectors; clusters with support <= 2000 are dropped before pooling.
- All stored vector entries lie within the clamped range [-1.5, 1.5], and no entries fall within the suppressed interval (-0.25, 0.25).
- Exactly k=50 k-means clusters were formed, and only retained clusters were pooled into the output set.
- The final table is serialized to disk in Parquet format under the exact filename augmentation_vectors.parquet.

## Limitations

- The article notes that 'further and more refined choices for data augmentation may be explored' — the k=50 cluster count and >2000-vector support threshold are heuristic choices, not optimized parameters.
- Vectors are precomputed from a 500,000-cell subsample, so rare augmentation directions present only in small cell-type/donor groups may be missed.
- The skill presupposes scTab preprocessing (size-factor normalization to 10000 counts, log1p transformation) and the specified gene space of 19331 protein coding genes.

## Evidence

- [other] Cluster the resulting vectors with k-means using k=50 to identify cell-type-independent augmentation directions.: [evidence span withheld — anchor status: not_found]
- [abstract] Novel data augmentation scheme: "we propose scTab, an automated cell type prediction model specific to tabular data, and train it using a novel data augmentation scheme"
- [intro] Rationale for augmentation: "model generalizability can be improved by artificially increasing the training data size through data augmentation"
- [readme] Augmentation notebooks in repo: "``notebooks/data_augmentation``: Notebooks related to data augmentation &rarr; calculation of augmentation vectors + evaluation"
- [methods] Noise suppression and clamping thresholds: [evidence span withheld — anchor status: not_found]
