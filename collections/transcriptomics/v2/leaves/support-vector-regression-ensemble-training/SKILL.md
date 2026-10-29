---
name: support-vector-regression-ensemble-training
description: Use when a reference atlas (e.g. mTCA.seurat.slim.qs) has a low-dimensional
  embedding and you must train the mapping from gene-set score matrices to UMAP_1/2
  coordinates.
when_to_use_negative:
- Query cells likely contain novel cell states absent from the reference — fixed-model
  projections may be misleading; compare with unsupervised clustering instead
- Projecting across platforms (e.g. Smart-seq2 to 10x) — performance declines due
  to differing UMI distributions and sequencing depth
- Reference is larger than what can be subsampled to 4000–10,000 cells without losing
  rare populations, given the ~O(Nc^2) reference building cost
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ProjectSVR
  role: Provides FitEnsembleSVM and ComputeModuleScore to fit the SVR ensemble on
    gene-set scores vs. UMAP coordinates
  repo: https://github.com/JarningGau/ProjectSVR
- name: AUCell
  role: Computes rank-based gene activity scores (0–1) used as SVR input features,
    robust to normalization
- name: e1071
  role: Underlying svm engine for each SVR sub-model (defaults C=1, epsilon=0.1, radial
    kernel)
- name: Seurat
  role: Handles the reference atlas object and its UMAP_1/2 embedding used as regression
    targets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:39abad73b0339a3351fbc80d10e8b3e7319380936c9e05039e87d9c1e2679720
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/JarningGau/ProjectSVR
  tools:
  - AUCell
  - ProjectSVR
  - Seurat
  - e1071
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# support-vector-regression-ensemble-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains the ProjectSVR ensemble of t=10 SVR sub-models (e1071 defaults: C=1, epsilon=0.1, radial kernel), each fitted on m=8000 randomly subsampled reference cells, to regress AUCell gene-set scores onto reference UMAP_1/2 coordinates; per-dimension predictions are aggregated by the median.

## When to use

Use when a reference atlas (e.g. mTCA.seurat.slim.qs) has a low-dimensional embedding and you must train the mapping from gene-set score matrices to UMAP_1/2 coordinates.

## When NOT to use

- Query cells likely contain novel cell states absent from the reference — fixed-model projections may be misleading; compare with unsupervised clustering instead
- Projecting across platforms (e.g. Smart-seq2 to 10x) — performance declines due to differing UMI distributions and sequencing depth
- Reference is larger than what can be subsampled to 4000–10,000 cells without losing rare populations, given the ~O(Nc^2) reference building cost

## Inputs

- Seurat reference atlas object (e.g. mTCA.seurat.slim.qs) with UMAP_1/2 embeddings
- Gene set score matrix for reference cells computed by ProjectSVR::ComputeModuleScore (method='AUCell'), e.g. from cNMF-derived K70 gene sets
- Ensemble hyperparameters: m (subsample size, default 8000) and t (number of sub-models, default 10)

## Outputs

- Trained ProjectSVR ensemble model object (exportable as .rds, cf. model.mTCA.rds on zenodo.org/record/8350732)
- Per-UMAP-dimension median-aggregated predictions for cells
- Held-out evaluation metrics (correlation/error between predicted and reference UMAP coordinates)

## How to apply

Load the reference atlas (e.g. mTCA.seurat.slim.qs from zenodo.org/record/8350746) and its K70 cNMF-derived gene sets, then compute the gene-set score matrix for all reference cells with ProjectSVR::ComputeModuleScore using method='AUCell'. Fit the ensemble with ProjectSVR::FitEnsembleSVM on the gene-set score matrix and the UMAP_1/2 embeddings: each of the t=10 sub-models is an independent SVR (e1071 svm defaults C=1, epsilon=0.1, radial kernel) trained on m=8000 randomly subsampled cells, kept within the 4000–10,000 subsampling range that balances performance and efficiency because reference building time is ~O(Nc^2). One independent SVR is trained per UMAP dimension, and per-dimension predictions from the t sub-models are aggregated by taking the median output for each embedding coordinate. Judge success by comparing predicted vs. reference UMAP coordinates on held-out reference cells (correlation/error metrics) to confirm accuracy comparable to state-of-the-art mapping methods.

## Related tools

- **ProjectSVR** (Provides FitEnsembleSVM and ComputeModuleScore to fit the SVR ensemble on gene-set scores vs. UMAP coordinates) — https://github.com/JarningGau/ProjectSVR
- **AUCell** (Computes rank-based gene activity scores (0–1) used as SVR input features, robust to normalization)
- **e1071** (Underlying svm engine for each SVR sub-model (defaults C=1, epsilon=0.1, radial kernel))
- **Seurat** (Handles the reference atlas object and its UMAP_1/2 embedding used as regression targets)

## Examples

```
model <- ProjectSVR::FitEnsembleSVM(gene.set.scores, ref.umap, m = 8000, t = 10)
```

## Evaluation signals

- Each sub-model is trained on exactly m=8000 randomly subsampled cells, and the ensemble contains t=10 SVR sub-models
- Each SVR uses e1071 svm defaults: C=1, epsilon=0.1, radial kernel
- One independent SVR per UMAP dimension (UMAP_1 and UMAP_2), not a multi-output model
- Aggregated prediction per coordinate equals the median across the 10 sub-models, not the mean
- Held-out predicted coordinates correlate with reference UMAP coordinates at accuracy consistent with ProjectSVR's claim of comparability to state-of-the-art reference mapping methods

## Limitations

- Independent SVR models per UMAP dimension neglect potential correlations among embedding dimensions, limiting the model's ability to learn joint structures
- Reference building time is ~O(Nc^2), requiring subsampling to 4000–10,000 cells for scalability
- Multi-output SVR / multi-target regression is not implemented (proposed future work)
