---
name: single-cell-dimensionality-reduction-pca-lsi
description: Use when you need a low-dimensional cell representation of a single-cell
  count matrix (scRNA counts, protein ADT, or scATAC peak matrix) to feed a kNN graph,
  integration method, or label-consistency metric like kNN agreement.
when_to_use_negative:
- The cell-cell adjacency you need must come from the integrated scMoMaT cell factors
  themselves — use scmomat.post_process on the extracted cell factors instead of reducing
  raw modality matrices.
- Input is already a low-dimensional embedding (e.g. precomputed dimensionality reduction
  in .obsm for MultiMAP) — apply the kNN graph directly rather than re-reducing.
- The goal is cluster-specific biomarker detection via the scMoMaT second training
  stage rather than label-agreement evaluation — feature factors come from model2.extract_feature_factors(),
  not from per-modality PCA/LSI.
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scMoMaT
  role: supplies the post_process/leiden_cluster pipeline consumed after per-modality
    PCA/LSI reduction, and the label sets whose kNN agreement is evaluated
  repo: https://github.com/PeterZZQ/scMoMaT
- name: MultiMAP
  role: expects precomputed per-dataset dimensionality reduction (TFIDF_LSI for ATAC,
    PCA for RNA) in .obsm before Integration(); corroborates the modality-specific
    reduction convention
  repo: https://github.com/Teichlab/MultiMAP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:a8af415bfcab86865eff0f1463ec03607f18b62905576f569bc206f1e2ca677f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - MultiMAP
  - scMoMaT
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-dimensionality-reduction-pca-lsi

## Summary

Reduces per-modality single-cell matrices to low-dimensional representations (PCA for scRNA and protein, LSI for scATAC) so that downstream label-quality metrics such as kNN agreement, clustering, or integration can be computed on a denoised, comparable cell embedding.

## When to use

Use when you need a low-dimensional cell representation of a single-cell count matrix (scRNA counts, protein ADT, or scATAC peak matrix) to feed a kNN graph, integration method, or label-consistency metric like kNN agreement.

## When NOT to use

- The cell-cell adjacency you need must come from the integrated scMoMaT cell factors themselves — use scmomat.post_process on the extracted cell factors instead of reducing raw modality matrices.
- Input is already a low-dimensional embedding (e.g. precomputed dimensionality reduction in .obsm for MultiMAP) — apply the kNN graph directly rather than re-reducing.
- The goal is cluster-specific biomarker detection via the scMoMaT second training stage rather than label-agreement evaluation — feature factors come from model2.extract_feature_factors(), not from per-modality PCA/LSI.

## Inputs

- PBMC batch count matrices from test_pbmc (counts dictionary with nbatches, feats_name, per-modality lists; ncells x nfeatures)
- scMoMaT Leiden labels (resolution 0.9) for each cell
- Original Mimitou et al. published cell labels for each cell
- utils.preprocess()-normalized per-modality matrices

## Outputs

- Per-modality low-dimensional cell representations (PCA scores for scRNA/protein, LSI scores for scATAC)
- kNN graph per reduced representation (knn_indices, knn_dists from scmomat.post_process)
- Per-modality and averaged kNN agreement scores aggregated in knn_agreement_pbmc.csv and visualized in knn_agreement_pbmc.png

## How to apply

Load each modality's batch count matrix from the counts dictionary (shape ncells x nfeatures, with None placeholders for missing batches) after preprocessing with utils.preprocess(). For gene-expression and protein (ADT) matrices, project cells onto low-dimensional space with PCA; for scATAC peak matrices, use TFIDF-normalized LSI. The article/README workflow fixes the neighborhood size for the subsequent kNN graph at 30 (within the 30-50 range recommended for scMoMaT post-processing), and the latent dimensionality K for the scMoMaT model itself is set to 30. After computing reduced representations, build a kNN graph on each reduced representation (scmomat.post_process with n_neighbors=30) and, for the kNN-agreement application, compute per cell the fraction of its 30 nearest neighbors sharing its label, averaging across modalities and batches. Decision point: if features differ across batches, first match feature sets via gene intersection or peak remapping before reduction.

## Related tools

- **scMoMaT** (supplies the post_process/leiden_cluster pipeline consumed after per-modality PCA/LSI reduction, and the label sets whose kNN agreement is evaluated) — https://github.com/PeterZZQ/scMoMaT
- **MultiMAP** (expects precomputed per-dataset dimensionality reduction (TFIDF_LSI for ATAC, PCA for RNA) in .obsm before Integration(); corroborates the modality-specific reduction convention) — https://github.com/Teichlab/MultiMAP

## Examples

```
from sklearn.decomposition import PCA, TruncatedSVD; Z_rna = PCA(n_components=30).fit_transform(X_rna); Z_atac = TruncatedSVD(n_components=30).fit_transform(X_atac)
```

## Evaluation signals

- Script runs without error on the test_pbmc data and emits knn_agreement_pbmc.csv with per-modality and averaged agreement columns.
- The averaged kNN agreement of scMoMaT labels exceeds that of the original Mimitou et al. labels, matching the qualitative finding of Fig. 3e (lower plot).
- kNN graph is built with neighborhood size 30 (knn_indices/knn_dists shape ncells x 30), consistent with the 30-50 recommendation.
- Each modality matrix has the same feature set per modality across batches (gene intersection or remapped peaks) before reduction.
- kNN agreement values are in [0,1] since they are fractions of a cell's k nearest neighbors sharing its label.

## Limitations

- Averaged kNN agreement is a coarse, qualitative agreement metric — it does not establish per-cluster correctness of either label set.
- The modality-specific reduction choice (PCA for scRNA/protein, LSI for scATAC) is a convention; applying PCA directly to raw scATAC peaks without TFIDF is inappropriate.
- Feature mismatch across batches (genes/proteins/peaks differing per batch) must be resolved before reduction, otherwise the joint graph is invalid.

## Evidence

- [other] Reduce each modality matrix to low dimension with PCA for scRNA and protein and LSI for scATAC.: [evidence span withheld — anchor status: not_found]
- [readme] Build kNN graph with neighborhood size 30: "s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, n_neighbors, njobs = 8, r = r)"
- [readme] The data matrix is of the shape (ncells, nfeatures): "The data matrix is of the shape `(ncells, nfeatures)`, and preprocessed with `utils.preprocess()` function."
- [readme] MultiMAP expects precomputed dimensionality reduction (TFIDF_LSI for ATAC, PCA for RNA): [evidence span withheld — anchor status: external_doc]
