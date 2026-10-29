---
name: cortical-neuron-cell-type-annotation
description: Use when scMoMaT cell factors from the MOp_5batches dataset (6 matrices
  + 2 GxR.npz pseudo-scRNA-seq inputs) have been Leiden-clustered and clusters must
  be annotated to cortical cell types via marker scores.
when_to_use_negative:
- The cell factors have not yet been post-processed into a KNN graph and Leiden-clustered
  — retraining requires labels=labels_leiden
- The task is batch integration alone without cluster interpretation — first-stage
  cell factors suffice and the marker-detection retraining stage is unnecessary
- The dataset is not the MOp cortical data and no cortical marker gene panel (Rorb,
  Calb1, Tshz2, Sulf1, Foxp2, Pvalb, Sst, Npy, Vip) is applicable
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scMoMaT
  role: Performs mosaic integration via matrix tri-factorization and, in the retrained
    second stage (scmomat_retrain), extracts cluster-specific bio-marker feature scores
    used for cell-type annotation
  repo: https://github.com/PeterZZQ/scMoMaT
- name: UMAP
  role: Visualizes the Leiden clusters in 2D (n_components=2, min_dist=0.20, metric='precomputed')
    to confirm clusters separate into transcriptionally coherent cortical populations
  repo: https://github.com/PeterZZQ/scMoMaT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:b4a83a575b07119c60c65b43aad79346a0a971473e8fa6f76d1450a76092f377
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - UMAP
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cortical-neuron-cell-type-annotation

## Summary

Annotates Leiden clusters from the scMoMaT mosaic integration of the 5-batch mouse brain cortex (MOp) data to cortical cell types using retrained cluster-specific marker gene scores (e.g., Rorb, Tshz2, Calb1).

## When to use

Use when scMoMaT cell factors from the MOp_5batches dataset (6 matrices + 2 GxR.npz pseudo-scRNA-seq inputs) have been Leiden-clustered and clusters must be annotated to cortical cell types via marker scores.

## When NOT to use

- The cell factors have not yet been post-processed into a KNN graph and Leiden-clustered — retraining requires labels=labels_leiden
- The task is batch integration alone without cluster interpretation — first-stage cell factors suffice and the marker-detection retraining stage is unnecessary
- The dataset is not the MOp cortical data and no cortical marker gene panel (Rorb, Calb1, Tshz2, Sulf1, Foxp2, Pvalb, Sst, Npy, Vip) is applicable

## Inputs

- Trained first-stage scmomat model on MOp_5batches counts (6 data matrices plus 2 pseudo-scRNA-seq inputs derived from the GxR.npz connection matrix)
- Leiden cluster labels (labels_leiden) computed with scmomat.leiden_cluster at resolution=0.9
- Feature names per modality (feats_name dictionary, e.g., {'rna': np.array([...])})

## Outputs

- Cluster-specific marker scores from model2.extract_feature_factors()
- Top-20 marker genes per Leiden cluster with cell-type annotations (mop5batches_marker_scores.csv)
- Cell-type annotation table reproducing Fig. 4e (cluster 3 = L4/5 excitatory/Rorb, cluster 7 = near-projecting excitatory/Tshz2, cluster 1 = L2/3 excitatory/Calb1)

## How to apply

After first-stage scMoMaT training (scmomat_model with K=30, T=4000), cell-factor extraction, scmomat.post_process (n_neighbors=30, njobs=8) and scmomat.leiden_cluster (resolution=0.9), run the second stage with scmomat.scmomat_retrain(model, counts, labels=labels_leiden) for T=2000 iterations and call model2.extract_feature_factors() to obtain cluster-specific marker scores. Interpret each cluster by inspecting its top marker genes against known cortical markers: Rorb for L4/5 excitatory, Tshz2 for near-projecting excitatory, Calb1 for L2/3 excitatory, Sulf1/Foxp2 for L6, and Pvalb/Sst vs Npy/Vip for inhibitory subclasses. Save the top-20 marker genes per cluster with annotations (e.g., mop5batches_marker_scores.csv) to reproduce the Fig. 4e marker-based cell-type annotation table. Rationale: scMoMaT learns cluster-specific bio-markers across modalities during retraining, so the learned feature scores serve as evidence-backed bio-markers rather than a single-modality guess.

## Related tools

- **scMoMaT** (Performs mosaic integration via matrix tri-factorization and, in the retrained second stage (scmomat_retrain), extracts cluster-specific bio-marker feature scores used for cell-type annotation) — https://github.com/PeterZZQ/scMoMaT
- **UMAP** (Visualizes the Leiden clusters in 2D (n_components=2, min_dist=0.20, metric='precomputed') to confirm clusters separate into transcriptionally coherent cortical populations) — https://github.com/PeterZZQ/scMoMaT

## Examples

```
model2 = scmomat.scmomat_retrain(model=model, counts=counts, labels=labels_leiden, device=device); losses = model2.train(T=2000); score_feats = model2.extract_feature_factors()
```

## Evaluation signals

- Training losses of both the first-stage (T=4000) and retrained (T=2000) models decrease monotonically over iterations
- UMAP embedding of the post-processed graph shows clusters separating into transcriptionally coherent cortical populations
- Top marker genes reproduce expected cortical patterns: Rorb in the L4/5 excitatory cluster, Tshz2 in the near-projecting excitatory cluster, Calb1 in the L2/3 excitatory cluster
- Inhibitory subclass markers partition as expected (Pvalb/Sst vs Npy/Vip) and L6 markers Sulf1/Foxp2 appear in the L6 cluster
- The resulting cluster-to-cell-type annotation table matches the marker-based annotation of Fig. 4d-e

## Limitations

- Annotation depends on the Leiden resolution choice (0.9 here); a different resolution changes cluster granularity and the marker-to-cluster mapping
- Marker interpretation relies on prior knowledge of cortical marker genes; clusters without canonical markers cannot be confidently annotated
- The connection matrix GxR.npz used to build the pseudo-scRNA-seq inputs must be provided in advance; it is dataset-specific
- Cross-modality feature relationships are not learned in this framework — the paper defers considering feature representations in the integration framework to future work

## Evidence

- [results] marker-based cell-type annotation of MOp clusters: [evidence span withheld — anchor status: not_found]
- [methods] retraining for marker extraction: "labels_leiden = scmomat.leiden_cluster(X = None, knn_indices = knn_indices, knn_dists = knn_dists, resolution = resolution)"
- [methods] second-stage feature factors: "score_feats = model2.extract_feature_factors()"
- [abstract] biomarker purpose per abstract: "scMoMaT is also able to uncover the cluster specific bio-markers across modalities. These multi-modal bio-markers are used to interpret and annotate the clusters to cell types."
- [methods] pseudo-count generation prerequisite: "The connection matrix (`GxR.npz`) that is used to generate pseudo-count is also provided in advance"
- [intro] biomarker modalities per intro: "scMoMaT learns cell type specific bio-markers across modalities, including marker genes (from the gene expression modality), marker motifs or regions"
