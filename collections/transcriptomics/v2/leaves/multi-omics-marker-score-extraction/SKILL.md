---
name: multi-omics-marker-score-extraction
description: Use when Leiden clusters from the post-processed scMoMaT cell-factor
  KNN graph are available and cluster-specific bio-markers across RNA/ATAC/protein
  modalities are needed to interpret and annotate clusters.
when_to_use_negative:
- Cross-modality feature relationships are the primary goal — the paper defers learning
  cross-modality relationships via feature representations to future work
- Batches lack matched feature sets per modality (features must first be matched via
  gene intersection or peak remapping)
- You need integration metrics only (e.g., batch mixing) without cell-type interpretation
  — the first-stage cell factors suffice
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scMoMaT
  role: Provides scmomat_retrain and extract_feature_factors for second-stage training
    and marker score extraction
  repo: https://github.com/PeterZZQ/scMoMaT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:6b125e6f63cd67b4c607dbca72354c2e0be4184ff709497114138e143ea473fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# multi-omics-marker-score-extraction

## Summary

Retrains the scMoMaT matrix tri-factorization model with Leiden cluster labels held fixed as binary cell factors, then extracts per-cluster feature scoring matrices (marker genes, marker regions/motifs, marker proteins) across modalities for cell-type annotation.

## When to use

Use when Leiden clusters from the post-processed scMoMaT cell-factor KNN graph are available and cluster-specific bio-markers across RNA/ATAC/protein modalities are needed to interpret and annotate clusters.

## When NOT to use

- Cross-modality feature relationships are the primary goal — the paper defers learning cross-modality relationships via feature representations to future work
- Batches lack matched feature sets per modality (features must first be matched via gene intersection or peak remapping)
- You need integration metrics only (e.g., batch mixing) without cell-type interpretation — the first-stage cell factors suffice

## Inputs

- Fitted first-stage scmomat model object (learned cell factors)
- Preprocessed counts dictionary (nbatches, feats_name, per-modality lists with None placeholders for missing batches)
- Leiden cluster labels (labels_leiden) from the post-processed cell-factor KNN graph
- Torch device (CPU or cuda)

## Outputs

- Retrained scMoMaT model (model2) with re-optimized feature factors and association matrices
- Per-modality marker score matrices from model2.extract_feature_factors() (genes, regions, proteins), each column = per-cluster marker scores
- Cluster-specific bio-markers used to interpret and annotate clusters to cell types

## How to apply

After first-stage training and Leiden clustering (labels_leiden), retrain the model with `scmomat.scmomat_retrain(model, counts, labels_leiden, device)`, which encodes cluster labels as binary cell factors held fixed during retraining while feature factors and association matrices are re-optimized. Train the retrained model for T = 2000 iterations with the default regularization lambda = 0.001. Optionally update counts to include additional training modalities (e.g., motif deviation matrices such as chromVAR outputs). Extract per-modality marker score matrices with `model2.extract_feature_factors()`, where each column corresponds to one cell cluster and each row to a feature (gene, region, or protein). Interpret clusters by checking that known markers dominate their expected clusters.

## Related tools

- **scMoMaT** (Provides scmomat_retrain and extract_feature_factors for second-stage training and marker score extraction) — https://github.com/PeterZZQ/scMoMaT

## Examples

```
model2 = scmomat.scmomat_retrain(model=model, counts=counts, labels=labels_leiden, device=device); losses = model2.train(T=2000); score_feats = model2.extract_feature_factors()
```

## Evaluation signals

- Gene-modality scores should rank GNLY and NKG7 highest in the NK cluster
- Gene-modality scores should rank MS4A1, CD79A, and CD37 highest in the B cluster
- Number of columns in each extracted marker score matrix equals the number of Leiden clusters; rows match feats_name per modality
- Retraining loss (model2.train(T=2000)) converges with lambda at its default 0.001
- Derived markers are consistent across modalities (marker genes, marker motifs/regions, marker proteins) for the same cluster

## Limitations

- Marker scores depend on the Leiden resolution used for labels_leiden; poor clustering propagates into misleading marker matrices
- Latent dimension count (K, recommended 20–30) constrains how many clusters receive distinct marker columns
- Feature sets must be identical within each modality across batches; unmatched features (unremapped peaks) yield invalid marker scores

## Evidence

- [abstract] cluster-specific biomarker retraining from abstract: "scMoMaT is also able to uncover the cluster specific bio-markers across modalities. These multi-modal bio-markers are used to interpret and annotate the clusters to cell types."
- [readme] retraining pipeline from README: "model2 = scmomat.scmomat_retrain(model = model, counts =  counts, labels = labels_leiden, device = device)"
- [readme] feature factor extraction from README: "score_feats = model2.extract_feature_factors()"
- [intro] biomarker modality scope from intro: "scMoMaT learns cell type specific bio-markers across modalities, including marker genes (from the gene expression modality), marker motifs or regions"
