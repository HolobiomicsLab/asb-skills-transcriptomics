---
name: global-structure-preservation-assessment
description: Use when a 2D embedding (ivis or a comparable method) of high-dimensional
  single-cell data must be verified for preservation of global inter-cluster geometry,
  e.g. after embedding scRNA-seq (hippocampus, melanoma SCP11, retinal bipolar) or
  CyTOF (Levine-32-dim) data.
when_to_use_negative:
- The data is already a low-dimensional embedding and no original high-dimensional
  distance matrix is available for Mantel-test comparison
- The goal is supervised cell-type prediction rather than assessing structural fidelity
  of a visualisation embedding
- The dataset is a toy example where global structure is trivially inspectable (e.g.
  the iris or Smiley dataset) and formal subsampled PCC estimation is unnecessary
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ivis
  role: Generates the 2D siamese-network (triplet-loss) embedding whose global-structure
    preservation is being assessed
  repo: https://github.com/beringresearch/ivis
- name: Python
  role: Implementation environment for MinMaxScaler preprocessing, PCA-50 reduction,
    ivis runs, Mantel/PCC computation, and matplotlib scatter plots
- name: t-SNE
  role: Baseline dimensionality-reduction method against which ivis's global-structure
    preservation is compared
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:6ce5c65a2cf09ab02c3078ad649f27c749857261352c8fd016eb22b2fd58a600
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - t-SNE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# global-structure-preservation-assessment

## Summary

A validation procedure for judging whether a low-dimensional embedding (e.g. an ivis embedding of scRNA-seq or CyTOF data) preserves the global structure of the original high-dimensional space, using Mantel-test Pearson correlation between distance matrices plus qualitative and classifier-based checks.

## When to use

Use when a 2D embedding (ivis or a comparable method) of high-dimensional single-cell data must be verified for preservation of global inter-cluster geometry, e.g. after embedding scRNA-seq (hippocampus, melanoma SCP11, retinal bipolar) or CyTOF (Levine-32-dim) data.

## When NOT to use

- The data is already a low-dimensional embedding and no original high-dimensional distance matrix is available for Mantel-test comparison
- The goal is supervised cell-type prediction rather than assessing structural fidelity of a visualisation embedding
- The dataset is a toy example where global structure is trivially inspectable (e.g. the iris or Smiley dataset) and formal subsampled PCC estimation is unnecessary

## Inputs

- High-dimensional expression matrix (e.g. scRNA-seq counts from melanoma SCP11, hippocampus, or retinal bipolar datasets; CyTOF Levine-32-dim) after MinMaxScaler [0,1] scaling and, for melanoma, PCA reduction to n=50
- The corresponding 2D embedding coordinates (e.g. ivis output)
- Annotated cell-type or cluster labels for the observations

## Outputs

- Pearson correlation coefficients (PCC continuum from n=50 stratified subsamples of 1000 observations) between original and embedding distance matrices via Mantel test
- SVM classifier accuracy (train 75% / evaluate 25%) on embedding coordinates
- Scatter plots (PNG) of the 2D embedding coloured by cell type
- Qualitative comparison against published embedding figures

## How to apply

First preprocess the expression matrix exactly as the embedding was built: scale values to the [0, 1] range with sklearn's MinMaxScaler, and for the melanoma scRNA-seq dataset (SCP11; 4,645 x 23,686) reduce the feature space to n=50 principal components before running ivis (embedding_dims=2, k=15, model='maaten' for datasets under 500,000 observations). To quantify global-structure preservation, compute distance matrices in the original (PCA) space and in the embedding space and correlate them with the Mantel test using the Pearson correlation coefficient; to generate a continuum of PCC values, draw random stratified subsamples (n=50) of 1000 observations rather than relying on a single correlation estimate. As an orthogonal functional check, train a Support Vector Machine classifier on 75% of each subsample and evaluate on the remaining 25% to assess embedding accuracy. Finally, scatter-plot the 2D embedding coloured by annotated cell type (matplotlib) and confirm clusters are distinct and not artificially packed, and that biologically meaningful relationships (e.g. hippocampal dentate gyrus -> CA3 -> CA1 information flow) appear as inter-cluster proximities.

## Related tools

- **ivis** (Generates the 2D siamese-network (triplet-loss) embedding whose global-structure preservation is being assessed) — https://github.com/beringresearch/ivis
- **Python** (Implementation environment for MinMaxScaler preprocessing, PCA-50 reduction, ivis runs, Mantel/PCC computation, and matplotlib scatter plots)
- **t-SNE** (Baseline dimensionality-reduction method against which ivis's global-structure preservation is compared)

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(X); emb = Ivis(embedding_dims=2, k=15).fit_transform(X_scaled)
```

## Evaluation signals

- Mantel-test Pearson correlation between original and embedding distance matrices yields a continuum of PCC values across n=50 random stratified subsamples of 1000 observations
- SVM classifier trained on 75% of each subsample achieves consistent accuracy on the held-out 25% in embedding space
- Cell types form distinct, non-artificially-packed clusters whose relative positions match published figures (e.g. retinal bipolar segregation)
- Generated embedding scatter-plot PNG files exist for every dataset processed
- Biologically expected inter-cluster proximities are present, e.g. hippocampal flow from dentate gyrus to CA3 and CA1 via mutual cell proximities

## Limitations

- t-SNE, the common baseline, tends to yield spurious clusters with clean boundaries in the presence of random noise, which can mislead visual assessment
- Comparative results are referenced via Supplementary Figures rather than in-text hyperparameter settings, complicating exact replication of ivis runs
- Assessment of biological structure (e.g. hippocampal information flow) is qualitative when labels permit only approximate inter-cluster distance coherence checks against the original PCA space

## Evidence

- [methods] Mantel test with PCC subsampling: "The level of correlation between the original distance matrix and the distance matrices in the embedding spaces was then assessed using the `Mantel test"
- [methods] Subsampling procedure: "Random stratified subsamples (n=50) of 1000 observations were used to generate a continuum of PCC values for each embedding technique."
- [methods] SVM classifier check: "Accuracy was calculated by training a Support Vector Machine classifier on 75% of each subsample and evaluating the classifier performance on the remaining 25%"
- [discussion] Global-structure claim across datasets: "Using two synthetic and three CyTOF datasets, we have shown that ivis reduces dimensionality whilst preserving the “global” structure in a dataset"
- [discussion] t-SNE failure mode motivating the check: "in the presence of random noise, the algorithm tends to yield spurious clusters with clean boundaries, potentially hindering accurate interpretation and discovery"
- [readme] README example with scaling and Ivis invocation: "X_scaled = MinMaxScaler().fit_transform(X)

model = Ivis(embedding_dims=2, k=15)

embeddings = model.fit_transform(X_scaled)"
