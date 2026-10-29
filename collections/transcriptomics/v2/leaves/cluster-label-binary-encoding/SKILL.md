---
name: cluster-label-binary-encoding
description: Use when Leiden cluster labels from the scMoMaT post-processed KNN graph
  (scmomat.post_process + leiden_cluster, e.g. resolution 0.9) must be converted into
  fixed binary cell factors for the 2nd-stage retraining that yields cluster-specific
  bio-markers.
when_to_use_negative:
- Cluster labels are not yet available — this encoding presupposes a completed Leiden
  clustering of the post-processed first-stage cell factors
- Goal is only mosaic integration without biomarker detection — the first training
  stage (scmomat_model + extract_cell_factors) suffices and retraining is unnecessary
- Batches lack matched feature sets across modalities (gene intersection or peak remapping
  not done) — retraining on unmatched counts is invalid per the README data requirements
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: scMoMaT
  role: Provides scmomat.scmomat_retrain, which encodes Leiden labels as fixed binary
    cell factors, and model2.extract_feature_factors for marker scores
  repo: https://github.com/PeterZZQ/scMoMaT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:6b8f5c9ef99a633456bc98577783113357a6351a3f18e9ebdb555a67d7702ea2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - scMoMaT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# cluster-label-binary-encoding

## Summary

Encodes the Leiden cluster labels (derived from the post-processed cell-factor KNN graph) as binary cell factors that are held fixed during scMoMaT's second-stage retraining, so the model re-optimizes feature factors and association matrices to produce per-cluster marker score matrices across RNA, ATAC, and protein modalities.

## When to use

Use when Leiden cluster labels from the scMoMaT post-processed KNN graph (scmomat.post_process + leiden_cluster, e.g. resolution 0.9) must be converted into fixed binary cell factors for the 2nd-stage retraining that yields cluster-specific bio-markers.

## When NOT to use

- Cluster labels are not yet available — this encoding presupposes a completed Leiden clustering of the post-processed first-stage cell factors
- Goal is only mosaic integration without biomarker detection — the first training stage (scmomat_model + extract_cell_factors) suffices and retraining is unnecessary
- Batches lack matched feature sets across modalities (gene intersection or peak remapping not done) — retraining on unmatched counts is invalid per the README data requirements

## Inputs

- labels_leiden: Leiden cluster label array from scmomat.leiden_cluster (resolution ~0.9)
- counts: preprocessed counts dictionary (nbatches, feats_name, per-modality lists with None placeholders for missing batches)
- model: first-stage scmomat_model whose cell factors were clustered
- device: torch.device (cuda or cpu)

## Outputs

- model2: retrained scMoMaT model with fixed binary cluster-label cell factors, re-optimized feature factors and association matrices
- score_feats from model2.extract_feature_factors(): per-modality cluster-specific marker score matrices (genes, regions, proteins), one column per cluster

## How to apply

After clustering the first-stage cell factors with scmomat.leiden_cluster (X = None, using knn_indices and knn_dists from scmomat.post_process), initialize the second-stage model with scmomat.scmomat_retrain(model = model, counts = counts, labels = labels_leiden, device = device); inside this call the cluster labels are encoded as binary cell factors that scMoMaT holds fixed while training. Train for T = 2000 iterations with the default regularization lambda = 0.001, which re-optimizes the feature factors and association matrices while each latent dimension corresponds to one cell cluster. Then call model2.extract_feature_factors() to obtain per-modality marker score matrices (genes, regions, proteins), where each column gives per-cluster marker scores. Optionally extend counts to include additional training modalities such as motif deviation matrices before retraining.

## Related tools

- **scMoMaT** (Provides scmomat.scmomat_retrain, which encodes Leiden labels as fixed binary cell factors, and model2.extract_feature_factors for marker scores) — https://github.com/PeterZZQ/scMoMaT

## Examples

```
model2 = scmomat.scmomat_retrain(model=model, counts=counts, labels=labels_leiden, device=device); losses = model2.train(T=2000); score_feats = model2.extract_feature_factors()
```

## Evaluation signals

- Training loss decreases over the T = 2000 retraining iterations without the binary cell factors changing (they are held fixed)
- model2.extract_feature_factors() returns one marker-score matrix per modality with a column per Leiden cluster
- Gene-modality scores are biologically coherent: GNLY and NKG7 score highest in the NK cluster; MS4A1, CD79A, CD37 score highest in the B cluster
- Marker scores span the modalities actually present in counts (rna, atac/regions, protein), matching feats_name

## Limitations

- Quality of the encoded binary factors depends entirely on the first-stage Leiden clustering (resolution parameter, e.g. 0.9); mis-clustered cells propagate directly into marker scores
- Cross-modality relationships via feature representations are deferred to future work, so the association matrices learned here do not fully capture cross-modality feature structure
- Additional modalities (e.g. motif deviation matrices from chromVAR) can only be added by updating counts before retraining, as noted in the README

## Evidence

- [readme] retrain initializes fixed binary cluster factors: "model2 = scmomat.scmomat_retrain(model = model, counts =  counts, labels = labels_leiden, device = device)"
- [readme] second stage trains with cluster result: "2nd stage training with cluster result, obtained feature factors"
- [methods] Leiden clustering supplies the labels: "labels_leiden = scmomat.leiden_cluster(X = None, knn_indices = knn_indices, knn_dists = knn_dists, resolution = resolution)"
- [readme] feature scoring after retraining: "# extract marker scores
score_feats = model2.extract_feature_factors()"
