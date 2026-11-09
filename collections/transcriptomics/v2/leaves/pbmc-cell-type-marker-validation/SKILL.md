---
name: pbmc-cell-type-marker-validation
description: Use when Leiden cluster labels already exist for integrated scMoMaT cell
  factors and you need cluster-specific bio-marker scores per modality, or validation
  of those scores against known markers such as GNLY/NKG7 (NK) or MS4A1/CD79A/CD37
  (B cells).
when_to_use_negative:
- Leiden clustering has not yet been performed on the post-processed cell-factor graph
  — retraining requires labels_leiden as input
- The analysis is a single-dataset, single-modality differential-expression marker
  test (e.g. a Seurat/LIGER workflow) rather than mosaic multi-omics marker extraction
  via matrix tri-factorization
- Features have not been matched across batches (gene intersection or peak remapping)
  — the counts dictionary must have identical feature sets per modality
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMoMaT
  role: Provides scmomat_retrain, model2.train, and extract_feature_factors for cluster-conditioned
    retraining and per-cluster marker score extraction
  repo: github.com/PeterZZQ/scMoMaT
- name: scmomat.leiden_cluster
  role: Produces the labels_leiden used as fixed cell factors during retraining
  repo: github.com/PeterZZQ/scMoMaT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:09264b32b07c2f99c90fc51096a55716c00e169aa6063e9985e1134e33ff1037
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scMoMaT
  - scmomat.leiden_cluster
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pbmc-cell-type-marker-validation

## Summary

Retrains the scMoMaT matrix tri-factorization model after Leiden clustering so that cluster labels become fixed cell factors, then extracts per-cluster feature-scoring matrices (bio-markers) across gene, peak/region, and protein modalities and validates them against canonical PBMC markers.

## When to use

Use when Leiden cluster labels already exist for integrated scMoMaT cell factors and you need cluster-specific bio-marker scores per modality, or validation of those scores against known markers such as GNLY/NKG7 (NK) or MS4A1/CD79A/CD37 (B cells).

## When NOT to use

- Leiden clustering has not yet been performed on the post-processed cell-factor graph — retraining requires labels_leiden as input
- The analysis is a single-dataset, single-modality differential-expression marker test (e.g. a Seurat/LIGER workflow) rather than mosaic multi-omics marker extraction via matrix tri-factorization
- Features have not been matched across batches (gene intersection or peak remapping) — the counts dictionary must have identical feature sets per modality

## Inputs

- Preprocessed PBMC counts dictionary (nbatches, feats_name, per-modality lists of (ncells, nfeatures) matrices)
- Leiden cluster labels (labels_leiden) obtained via scmomat.leiden_cluster on the post-processed KNN graph
- First-stage trained scmomat model object (model) with learned cell factors
- torch device (e.g. cuda if available)

## Outputs

- Retrained scMoMaT model (model2) with cluster-label-derived cell factors
- Per-modality marker score matrices (score_feats) for genes, regions, and proteins, one column per Leiden cluster
- Training loss trace from the T=2000 retraining iterations

## How to apply

Load the preprocessed PBMC counts dictionary and the Leiden labels from the post-processed KNN graph. Initialize the second-stage model with scmomat.scmomat_retrain(model, counts, labels_leiden, device), which encodes the cluster labels as binary cell factors held fixed during retraining while the feature factors and association matrices are re-optimized. Train the retrained model with model2.train(T=2000), keeping the default regularization lambda=0.001; each latent dimension now corresponds to one cell cluster. Extract per-modality marker score matrices with model2.extract_feature_factors(), where each column gives marker scores for one cluster. Validate the gene-modality scores: GNLY and NKG7 should rank highest in the NK cluster, and MS4A1, CD79A, CD37 should rank highest in the B cell cluster; analogous checks can be applied for regions and proteins.

## Related tools

- **scMoMaT** (Provides scmomat_retrain, model2.train, and extract_feature_factors for cluster-conditioned retraining and per-cluster marker score extraction) — github.com/PeterZZQ/scMoMaT
- **scmomat.leiden_cluster** (Produces the labels_leiden used as fixed cell factors during retraining) — github.com/PeterZZQ/scMoMaT

## Examples

```
model2 = scmomat.scmomat_retrain(model=model, counts=counts, labels=labels_leiden, device=device); losses = model2.train(T=2000); score_feats = model2.extract_feature_factors()
```

## Evaluation signals

- GNLY and NKG7 rank highest in the NK cluster's gene-modality score column
- MS4A1, CD79A, and CD37 rank highest in the B cell cluster's gene-modality score column
- The retrained model's latent dimensions align 1:1 with Leiden clusters (binary cluster-encoded cell factors held fixed during training)
- Training loss from model2.train(T=2000) converges with default lambda=0.001
- extract_feature_factors returns one score matrix per modality whose columns correspond to the number of Leiden clusters

## Limitations

- The second stage can optionally include additional modalities (e.g. motif matrices) only if the counts are updated before retraining
- Cross-modality feature relationships are not learned directly; the paper defers learning cross-modality relationships via feature representations to future work
- Dataset files beyond the provided PBMC and MOp data must be requested, so marker validation on other mosaic datasets requires obtaining those inputs first

## Evidence

- [other] retraining with cluster labels: [evidence span withheld — anchor status: not_found]
- [readme] retraining API in README: "model2 = scmomat.scmomat_retrain(model = model, counts =  counts, labels = labels_leiden, device = device)"
- [readme] marker extraction in README: "# number of iteractions is 2000
losses = model2.train(T = 2000)

# extract marker scores
score_feats = model2.extract_feature_factors()"
- [abstract] abstract claim on biomarkers: "scMoMaT is also able to uncover the cluster specific bio-markers across modalities."
