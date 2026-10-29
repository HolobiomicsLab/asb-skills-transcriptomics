---
name: mosaic-multiomics-data-handling
description: Use when you already have Leiden cluster labels from scMoMaT's post-processed
  cell-factor KNN graph and need cluster-specific marker genes, marker regions, or
  marker proteins (e.g. from ASAP-PBMC RNA/ATAC/protein counts) to annotate clusters.
when_to_use_negative:
- You have not yet run first-stage cell-factor training and post-processing/clustering
  (scmomat.post_process + leiden_cluster) — retraining requires cluster labels as
  fixed binary cell factors
- Your goal is only cell embedding/batch integration without cluster-specific bio-markers
  — the first-stage model's extract_cell_factors() alone suffices
- Batches lack matched feature sets per modality (genes intersected or peaks remapped)
  — the counts dictionary requires identical features within each modality before
  retraining
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scMoMaT
  role: Provides scmomat.scmomat_retrain and extract_feature_factors for the second-stage
    retraining that produces cluster-specific marker score matrices
  repo: github.com/PeterZZQ/scMoMaT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:bb95905280458b28080311081a8c0c86d1a30d7d754fe6f4c89ed759e66f0e4d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mosaic-multiomics-data-handling

## Summary

Post-clustering retraining stage of scMoMaT: after Leiden clustering of the integrated cell embedding, the matrix tri-factorization model is retrained with cluster labels encoded as fixed binary cell factors so that feature factors and association matrices yield per-cluster, multi-modal bio-marker score matrices.

## When to use

Use when you already have Leiden cluster labels from scMoMaT's post-processed cell-factor KNN graph and need cluster-specific marker genes, marker regions, or marker proteins (e.g. from ASAP-PBMC RNA/ATAC/protein counts) to annotate clusters.

## When NOT to use

- You have not yet run first-stage cell-factor training and post-processing/clustering (scmomat.post_process + leiden_cluster) — retraining requires cluster labels as fixed binary cell factors
- Your goal is only cell embedding/batch integration without cluster-specific bio-markers — the first-stage model's extract_cell_factors() alone suffices
- Batches lack matched feature sets per modality (genes intersected or peaks remapped) — the counts dictionary requires identical features within each modality before retraining

## Inputs

- Fitted first-stage scmomat model with learned cell factors (from scmomat_model + train_func on the counts dictionary)
- Preprocessed counts dictionary (nbatches, feats_name, per-modality lists ordered by batch ID, matrices preprocessed with utils.preprocess())
- Leiden cluster labels (labels_leiden) from scmomat.leiden_cluster on the post-processed KNN graph
- torch device specification (CPU or GPU)

## Outputs

- Retrained scMoMaT model (model2) with re-optimized feature factors and association matrices
- Per-modality marker score matrices (score_feats) for genes, regions, and proteins, one column per Leiden cluster

## How to apply

Load the preprocessed PBMC counts dictionary (nbatches, feats_name, per-modality lists aligned by batch ID with None placeholders) together with labels_leiden obtained from scmomat.leiden_cluster on the post-processed KNN graph. Initialize the second-stage model with scmomat.scmomat_retrain(model, counts, labels_leiden, device), which encodes each Leiden cluster label as a binary cell factor that is held fixed during retraining, so re-optimization concentrates on the feature factors and association matrices. Train the retraining model for T = 2000 iterations with the default regularization lambda = 0.001 (optionally updating counts to include additional modalities such as motif deviation matrices). Extract per-modality marker score matrices with model2.extract_feature_factors(), where each latent dimension/column corresponds to one cell cluster and scores quantify the importance of a feature (gene, region, protein) for that cluster. Rationale: because cluster memberships are frozen, the learned feature factors directly represent cell-type-specific bio-markers usable to interpret and annotate clusters across modalities.

## Related tools

- **scMoMaT** (Provides scmomat.scmomat_retrain and extract_feature_factors for the second-stage retraining that produces cluster-specific marker score matrices) — github.com/PeterZZQ/scMoMaT

## Examples

```
model2 = scmomat.scmomat_retrain(model=model, counts=counts, labels=labels_leiden, device=device); losses = model2.train(T=2000); score_feats = model2.extract_feature_factors()
```

## Evaluation signals

- Losses = model2.train(T = 2000) converges without divergence at default lambda = 0.001
- score_feats from extract_feature_factors() has one column per Leiden cluster, with per-modality blocks matching feats_name dimensions (genes, regions, proteins)
- On ASAP-PBMC, gene-modality scores rank GNLY and NKG7 highest in the NK cluster
- On ASAP-PBMC, gene-modality scores rank MS4A1, CD79A, and CD37 highest in the B cluster
- Model structure invariant: cluster labels encoded as binary cell factors remain fixed after retraining (only feature factors and association matrices are re-optimized)

## Limitations

- Bio-marker extraction depends on Leiden cluster resolution (e.g. resolution = 0.9 in the README example); different resolutions change the number of marker-score columns and cluster definitions
- Cross-modality feature-to-feature relationships are not learned by this framework — the article defers learning cross-modality relationships via feature representations to future work
- The [Optional] updated counts for retraining (e.g. adding motif deviation matrices) require features generated by other methods to be supplied by the user

## Evidence

- [methods] retraining encodes cluster labels as fixed binary cell factors: [evidence span withheld — anchor status: not_found]
- [methods] feature scoring matrices derived from retrained factors: [evidence span withheld — anchor status: not_found]
- [readme] marker score extraction invocation: "model2 = scmomat.scmomat_retrain(model = model, counts =  counts, labels = labels_leiden, device = device)"
- [abstract] multi-modal bio-marker purpose: "scMoMaT is also able to uncover the cluster specific bio-markers across modalities. These multi-modal bio-markers are used to interpret and annotate the clusters to cell types."
