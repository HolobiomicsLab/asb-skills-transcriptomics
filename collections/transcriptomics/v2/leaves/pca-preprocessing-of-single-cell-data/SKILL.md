---
name: pca-preprocessing-of-single-cell-data
description: Use when a scRNA-seq expression matrix has thousands of features (e.g.
  ~23,686 genes) and must be scaled and compressed to PCA-50 prior to training ivis
  (k=15, model='maaten') for 2D embedding.
when_to_use_negative:
- CyTOF data such as the Levine-32-dim dataset, which the paper preprocesses with
  an arcsinh transformation (X = np.arcsinh(X/5)) instead of MinMaxScaler plus PCA-50
- Datasets already reduced to a low-dimensional feature space where PCA-50 compression
  is unnecessary
- Situations where absolute (unscaled) feature magnitudes must be preserved for downstream
  interpretation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: sklearn MinMaxScaler
  role: scales feature values to the [0, 1] range prior to dimensionality reduction,
    as required by ivis
- name: Principal Component Analysis (PCA)
  role: reduces the wide melanoma feature space (23,686 features) to n=50 principal
    components before ivis training
- name: ivis
  role: trains a siamese neural network (triplet loss, k=15, model='maaten') on the
    preprocessed data to produce a 2D embedding
  repo: https://github.com/beringresearch/ivis
- name: matplotlib
  role: scatter-plots the 2D embedding colored by annotated cell type to verify cluster
    segregation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:193baad812032dc8a49b4aa589dd535166ee23337579d5ec18e33fa81001413b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Principal Component Analysis (PCA)
  - Python
  - ivis
  - matplotlib
  - sklearn MinMaxScaler
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pca-preprocessing-of-single-cell-data

## Summary

Preprocess high-dimensional single-cell RNA-seq expression matrices by scaling feature values to the [0, 1] range with sklearn's MinMaxScaler and reducing very wide feature spaces (e.g. the 4,645 x 23,686 melanoma dataset) to 50 principal components before ivis dimensionality reduction.

## When to use

Use when a scRNA-seq expression matrix has thousands of features (e.g. ~23,686 genes) and must be scaled and compressed to PCA-50 prior to training ivis (k=15, model='maaten') for 2D embedding.

## When NOT to use

- CyTOF data such as the Levine-32-dim dataset, which the paper preprocesses with an arcsinh transformation (X = np.arcsinh(X/5)) instead of MinMaxScaler plus PCA-50
- Datasets already reduced to a low-dimensional feature space where PCA-50 compression is unnecessary
- Situations where absolute (unscaled) feature magnitudes must be preserved for downstream interpretation

## Inputs

- scRNA-seq expression matrix (e.g. melanoma SCP11: 4,645 observations x 23,686 features from the Broad Single Cell Portal)
- hippocampus and retinal bipolar scRNA-seq expression matrices
- annotated cell-type labels for validation plotting

## Outputs

- Min-max scaled expression matrix in the [0, 1] range
- PCA-reduced feature matrix with n=50 principal components
- 2D ivis embedding (per cell) suitable for matplotlib scatter plots colored by cell type

## How to apply

First scale feature values to the [0, 1] range with sklearn's MinMaxScaler, since ivis expects preprocessed values in this range. If the feature space is very wide (e.g. the metastatic melanoma dataset from the Broad Single Cell Portal, SCP11, with 4,645 observations x 23,686 features), further reduce it to n=50 principal components using PCA. Train ivis on the preprocessed matrix using recommended hyperparameters for datasets under 500,000 observations (k=15, model='maaten') to obtain a 2D embedding. Scatter-plot the embedding colored by annotated cell type to confirm cell types form distinct clusters that preserve global relationships (e.g. hippocampus embedding capturing information flow from dentate gyrus to CA3 and CA1 nuclei).

## Related tools

- **sklearn MinMaxScaler** (scales feature values to the [0, 1] range prior to dimensionality reduction, as required by ivis)
- **Principal Component Analysis (PCA)** (reduces the wide melanoma feature space (23,686 features) to n=50 principal components before ivis training)
- **ivis** (trains a siamese neural network (triplet loss, k=15, model='maaten') on the preprocessed data to produce a 2D embedding) — https://github.com/beringresearch/ivis
- **matplotlib** (scatter-plots the 2D embedding colored by annotated cell type to verify cluster segregation)

## Examples

```
from sklearn.preprocessing import MinMaxScaler; from sklearn.decomposition import PCA; X = MinMaxScaler().fit_transform(X); X = PCA(n_components=50).fit_transform(X)
```

## Evaluation signals

- Confirm scaled values fall within the [0, 1] range after MinMaxScaler (X = MinMaxScaler().fit_transform(X))
- Confirm the PCA-reduced melanoma matrix has exactly 50 components and 4,645 observations
- Verify the ivis run used the recommended under-500k hyperparameters (k=15, model='maaten')
- Check scatter plots show distinct, non-artificially-packed cell-type clusters matching published segregation patterns (e.g. retinal bipolar cell types, dentate gyrus→CA3→CA1 flow)
- Report Pearson-correlation-based coherence of inter-cluster distances against the original PCA space where labels permit (Mantel test)

## Limitations

- The PCA-50 reduction step was applied specifically to the melanoma feature space; other datasets in the paper (hippocampus, retinal bipolar) did not use it
- t-SNE-style visualization methods can yield spurious clusters with clean boundaries in the presence of random noise, so cluster interpretation must be checked against annotated labels
- ivis results are robust to hyperparameter settings per the paper, but recommended settings are documented only in Supplementary Figures rather than the main text

## Evidence

- [methods] The Melanoma featurespace was further reduced to n=50 using Principal Component Analysis.: "The Melanoma featurespace was further reduced to n=50 using Principal Component Analysis."
- [methods] Prior to preprocessing, values in both datasets were scaled to [0, 1] range.: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [methods] Melanoma (SCP11) dataset dimensions from methods: "`Melanoma <https://portals.broadinstitute.org/single_cell/study/SCP11/melanoma-intra-tumor-heterogeneity>`__ (4,645 x 23,686)"
- [readme] README example showing MinMaxScaler before Ivis: "X_scaled = MinMaxScaler().fit_transform(X)"
