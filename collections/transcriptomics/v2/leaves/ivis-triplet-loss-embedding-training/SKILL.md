---
name: ivis-triplet-loss-embedding-training
description: Use when a high-dimensional single-cell expression matrix (e.g. melanoma
  4,645 x 23,686 from SCP11, hippocampus, retinal bipolar) needs a 2D embedding that
  preserves both local cell-type clusters and global inter-cluster relationships.
when_to_use_negative:
- Data is low-dimensional or already an embedding — ivis is designed for high-dimensional
  single-cell expression matrices
- You need annotated spurious-cluster behavior analysis with clean boundaries — t-SNE/SIMLR
  comparisons are separate skills; ivis is chosen to avoid spurious clusters under
  random noise
- Dataset exceeds the validated hyperparameter regime (>500,000 observations) without
  adjusting the recommended k/model settings
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ivis
  role: Siamese neural network trained with triplet loss producing the 2D embedding
  repo: https://github.com/beringresearch/ivis
- name: Python
  role: Execution environment; ivis is exposed via Python (and R) interfaces
  repo: https://github.com/beringresearch/ivis
- name: t-SNE
  role: Comparison baseline that yields spurious clusters in the presence of random
    noise
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:7cdf5fdee0a41a2f218f1d8cc28d985bde3b28575b3b1dc7afad1f1710a71ce0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - t-SNE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ivis-triplet-loss-embedding-training

## Summary

Train the ivis siamese neural network with a triplet-loss function to produce 2D structure-preserving embeddings of high-dimensional scRNA-seq (and CyTOF) datasets, using MinMaxScaler [0,1] scaling and PCA-50 reduction as preprocessing and k=15, model='maaten' for datasets under 500,000 observations.

## When to use

Use when a high-dimensional single-cell expression matrix (e.g. melanoma 4,645 x 23,686 from SCP11, hippocampus, retinal bipolar) needs a 2D embedding that preserves both local cell-type clusters and global inter-cluster relationships.

## When NOT to use

- Data is low-dimensional or already an embedding — ivis is designed for high-dimensional single-cell expression matrices
- You need annotated spurious-cluster behavior analysis with clean boundaries — t-SNE/SIMLR comparisons are separate skills; ivis is chosen to avoid spurious clusters under random noise
- Dataset exceeds the validated hyperparameter regime (>500,000 observations) without adjusting the recommended k/model settings

## Inputs

- scRNA-seq expression matrix (e.g. melanoma SCP11, 4,645 observations x 23,686 features from the Broad Single Cell Portal; hippocampus; retinal bipolar; 10X neural subset)
- Cell-type annotation labels for coloring/validation
- Optionally CyTOF matrices (Levine 32-dim, 265,627 cells x 32 markers) after arcsinh transformation X/5

## Outputs

- 2D ivis embedding (n_observations x 2) as a PNG scatter plot colored by cell type
- Trained ivis parametric model supporting transform() of new points

## How to apply

First preprocess the expression matrix: scale feature values to the [0,1] range with sklearn's MinMaxScaler, and for very wide feature spaces such as the metastatic melanoma dataset (4,645 x 23,686) reduce to n=50 principal components with PCA. Train ivis with the recommended hyperparameters for datasets under 500,000 observations: k=15 and model='maaten', requesting embedding_dims=2; ivis samples triplets with positives from the KNN vector and negatives from outside the KNN vector, so a k of 15 controls neighborhood granularity. Scatter-plot the 2D embedding colored by annotated cell type with matplotlib, checking that cell types form distinct, non-artificially-packed clusters. Validate by confirming global structure is preserved — e.g. the hippocampus embedding should capture information flow from dentate gyrus to CA3 and CA1 — and, where labels permit, report Pearson-correlation-based coherence of inter-cluster distances against the original PCA space (Mantel test on distance matrices).

## Related tools

- **ivis** (Siamese neural network trained with triplet loss producing the 2D embedding) — https://github.com/beringresearch/ivis
- **Python** (Execution environment; ivis is exposed via Python (and R) interfaces) — https://github.com/beringresearch/ivis
- **t-SNE** (Comparison baseline that yields spurious clusters in the presence of random noise)

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(X); embeddings = Ivis(embedding_dims=2, k=15, model='maaten').fit_transform(X_scaled)
```

## Evaluation signals

- Each generated PNG embedding file exists and cells form distinct, non-artificially-packed cell-type clusters
- Qualitative reproduction of published segregation patterns (e.g. retinal bipolar cell types matching the paper's figure)
- Hippocampus embedding captures flow of sensory information from dentate gyrus to CA3 and CA1 via mutual cell proximities
- Pearson-correlation-based coherence (Mantel test) of inter-cluster distances against the original PCA-50 space where labels permit
- Preprocessing invariants: values scaled to [0,1] before training and melanoma feature space reduced to n=50 PCA components

## Limitations

- t-SNE yields spurious clusters with clean boundaries under random noise — a motivation for ivis, but embedding quality must still be checked against known cell types
- Mantel-test distance correlations require cell-type labels; unlabeled data limits quantitative validation
- Hyperparameters for large runs are referenced only via supplementary material; recommended settings apply to datasets under 500,000 observations

## Evidence

- [methods] Scale data to [0, 1] with MinMaxScaler: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [methods] PCA reduction of Melanoma to n=50: "The Melanoma featurespace was further reduced to n=50 using Principal Component Analysis."
- [discussion] Triplet sampling: "ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively"
- [methods] Mantel test validation: "The level of correlation between the original distance matrix and the distance matrices in the embedding spaces was then assessed using the `Mantel test"
- [discussion] Hippocampus global structure finding: "ivis embeddings of the hippocampus dataset captured distinct nuclei clusters that correspond to known cell types and anatomical regions"
- [readme] README example invocation: "X_scaled = MinMaxScaler().fit_transform(X)

model = Ivis(embedding_dims=2, k=15)"
