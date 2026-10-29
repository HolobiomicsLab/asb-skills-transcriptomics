---
name: mosaic-dataset-label-annotation-matching
description: Use when re-annotated cell labels from an integration method (e.g., scMoMaT
  Leiden labels at resolution 0.9) must be compared against a paper's original published
  labels on the same mosaic dataset to argue annotation quality.
when_to_use_negative:
- When the two label sets come from different cell sets or batches that cannot be
  aligned cell-for-cell — kNN agreement requires labels defined on identical cells
- When the goal is quantitative correspondence to a ground-truth cell type annotation
  rather than agreement with local per-matrix neighborhood structure (this metric
  measures self-consistency, not external accuracy)
- When a modality lacks a usable low-dimensional representation (e.g., too few features
  after peak remapping) — agreement must be computed per modality, so dropping modality
  averages silently changes the metric
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMoMaT
  role: Produces the evaluated cell labels via matrix tri-factorization, post-processed
    kNN graph, and Leiden clustering; the baseline against which original labels are
    compared
  repo: https://github.com/PeterZZQ/scMoMaT
- name: LIGER
  role: Prior horizontal/diagonal integration method cited for context on integration
    tasks; not used in this kNN-agreement evaluation
  repo: https://github.com/welch-lab/liger
- name: MultiMAP
  role: Alternative multi-omics integration method; relevant as a competing label
    source, not part of this evaluation workflow
  repo: https://github.com/Teichlab/MultiMAP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:0c8fa158176b4d05a556653740c103e1c141774516cae2e7caf88eca0a6f6f41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - LIGER
  - MultiMAP
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mosaic-dataset-label-annotation-matching

## Summary

Evaluates whether cell type labels produced by a mosaic-integration method (e.g., scMoMaT Leiden clusters) agree with per-batch cell–cell variation better than originally published labels, by computing kNN agreement in each input data matrix's own reduced space (PCA for scRNA/protein, LSI for scATAC).

## When to use

Use when re-annotated cell labels from an integration method (e.g., scMoMaT Leiden labels at resolution 0.9) must be compared against a paper's original published labels on the same mosaic dataset to argue annotation quality.

## When NOT to use

- When the two label sets come from different cell sets or batches that cannot be aligned cell-for-cell — kNN agreement requires labels defined on identical cells
- When the goal is quantitative correspondence to a ground-truth cell type annotation rather than agreement with local per-matrix neighborhood structure (this metric measures self-consistency, not external accuracy)
- When a modality lacks a usable low-dimensional representation (e.g., too few features after peak remapping) — agreement must be computed per modality, so dropping modality averages silently changes the metric

## Inputs

- Mosaic batch count matrices (e.g., test_pbmc) stored as counts dictionary with nbatches, feats_name, and per-modality lists (None placeholders for missing modalities)
- Label set 1: integration-method-derived labels (scMoMaT Leiden clustering at resolution 0.9)
- Label set 2: original published cell type labels (Mimitou et al.)

## Outputs

- knn_agreement_pbmc.csv — per-modality and averaged kNN agreement values for both label sets
- knn_agreement_pbmc.png — side-by-side bar plot of per-modality and averaged agreement
- Scalar conclusion: which label set has higher averaged kNN agreement

## How to apply

Load the mosaic batch count matrices (e.g., test_pbmc) together with both label sets — the method's Leiden labels (resolution 0.9) and the original published labels (e.g., Mimitou et al.). For each input modality matrix, reduce to low dimension independently: PCA for scRNA and protein matrices, LSI (TF-IDF based) for scATAC. Build a kNN graph per modality with neighborhood size 30 (within the 30–50 range recommended for scMoMaT post-processing) and compute kNN agreement as the fraction of each cell's 30 nearest neighbors sharing its label under each label set. Average agreement across modalities and batches, aggregate into a CSV (e.g., knn_agreement_pbmc.csv), and plot per-modality and averaged agreement side by side (knn_agreement_pbmc.png). The label set with higher mean kNN agreement is judged to better reflect per-matrix cell–cell structure; scMoMaT labels exceeding the original labels reproduces the qualitative finding of Fig. 3e.

## Related tools

- **scMoMaT** (Produces the evaluated cell labels via matrix tri-factorization, post-processed kNN graph, and Leiden clustering; the baseline against which original labels are compared) — https://github.com/PeterZZQ/scMoMaT
- **LIGER** (Prior horizontal/diagonal integration method cited for context on integration tasks; not used in this kNN-agreement evaluation) — https://github.com/welch-lab/liger
- **MultiMAP** (Alternative multi-omics integration method; relevant as a competing label source, not part of this evaluation workflow) — https://github.com/Teichlab/MultiMAP

## Examples

```
s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, n_neighbors=30, njobs=8, r=None); labels_leiden = scmomat.leiden_cluster(X=None, knn_indices=knn_indices, knn_dists=knn_dists, resolution=0.9)
```

## Evaluation signals

- Script runs without error on the provided test_pbmc data
- Averaged kNN agreement value for scMoMaT labels in knn_agreement_pbmc.csv exceeds that of the original Mimitou et al. labels, matching the Fig. 3e (lower plot) finding
- Per-modality agreement values are all in [0, 1] and each cell's agreement is the fraction of its 30 nearest neighbors sharing its label
- Both label sets evaluated on the same per-modality reduced spaces (PCA for scRNA/protein, LSI for scATAC) with identical neighborhood size 30, so the comparison is apples-to-apples
- Plotted per-modality and averaged agreement appear side by side for both label sets in knn_agreement_pbmc.png

## Limitations

- kNN agreement measures internal consistency with per-matrix cell–cell variation, not correctness of cell type assignment against external ground truth
- Results depend on neighborhood size choice; 30 is used as the low end of the 30–50 range recommended for scMoMaT post-processing
- Incorporating feature representations for cross-modality relationship learning was deferred to future work, so modality-specific reductions are computed independently rather than jointly

## Evidence

- [results] kNN agreement computed per modality with neighborhood size 30: [evidence span withheld — anchor status: not_found]
- [readme] Post-processing with 30–50 neighbors: "# neighborhood size (30~50)
n_neighbors = 30"
- [readme] Leiden clustering at resolution 0.9: "labels_leiden = scmomat.leiden_cluster(X = None, knn_indices = knn_indices, knn_dists = knn_dists, resolution = resolution)"
- [abstract] scMoMaT labels of higher quality than original annotations: "integrated cell embedding combined with learned bio-markers lead to cell type annotations of higher quality or resolution compared to their original annotations"
