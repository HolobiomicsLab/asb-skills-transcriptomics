---
name: latent-space-mixing-quality-assessment
description: Use when evaluating scConfluence integration output — after model.fit()
  on a MuData object — to decide whether cell embeddings in mdata.obsm['latent'] are
  acceptably aligned across modalities (e.g. scRNA vs scATAC) before downstream clustering
  or imputation.
when_to_use_negative:
- When modalities share no cross-modality feature connections (e.g. electrophysiology
  vs gene expression), since the alignment objective relies on weakly connected features
- When the goal is paired (diagonal-on) integration of jointly profiled cells rather
  than unpaired diagonal integration
- As a substitute for benchmarking against state-of-the-art methods (Seurat v3.0/v4.3.0,
  Liger, MultiMAP, Uniport, scGLUE) in comparative evaluations
tools:
- name: scConfluence
  role: Provides model.fit(), model.get_latent(), and the sinkhorn_loss_weight / reach
    hyperparameters whose effects on latent-space mixing this assessment probes
  repo: https://github.com/cantinilab/scconfluence
- name: Scanpy
  role: Used for UMAP visualization of the extracted 16-dimensional latent embeddings
    to check modality mixing and preserved biological structure
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:f65231de9ad13a2aa21b5cbed2aa484c3f233382bc737a37181cbd6e7dae1bee
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cantinilab/scconfluence
  repo_url: https://github.com/cantinilab/scconfluence
  tools:
  - Scanpy
  - scConfluence
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# latent-space-mixing-quality-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validation procedure for scConfluence's diagonal integration: after fitting the rIOT-based alignment objective, verify that the shared 16-dimensional latent space is well mixed across modalities while per-modality biological structure and reconstruction quality are preserved.

## When to use

Use when evaluating scConfluence integration output — after model.fit() on a MuData object — to decide whether cell embeddings in mdata.obsm['latent'] are acceptably aligned across modalities (e.g. scRNA vs scATAC) before downstream clustering or imputation.

## When NOT to use

- When modalities share no cross-modality feature connections (e.g. electrophysiology vs gene expression), since the alignment objective relies on weakly connected features
- When the goal is paired (diagonal-on) integration of jointly profiled cells rather than unpaired diagonal integration
- As a substitute for benchmarking against state-of-the-art methods (Seurat v3.0/v4.3.0, Liger, MultiMAP, Uniport, scGLUE) in comparative evaluations

## Inputs

- Fitted scConfluence model (uncoupled AutoEncoders trained with IOT loss plus Sinkhorn divergence regularization)
- MuData object with modalities preprocessed (e.g. log-normalized scRNA counts, TF-IDF scATAC, CLR protein) and cross-modality distance matrices in mdata.uns
- Fitted hyperparameters: sinkhorn_loss_weight, reach, transported mass m=0.5

## Outputs

- Aligned latent embeddings in mdata.obsm['latent'] (16-dimensional shared latent space)
- latent_embeddings.npy file of the extracted embeddings
- UMAP projections of the latent embeddings colored by modality and by biological annotation

## How to apply

Extract embeddings via model.get_latent() into mdata.obsm['latent'] and save them as latent_embeddings.npy. First, assess modality mixing in a low-dimensional projection of the latent embeddings: a properly fitted objective yields a mixed latent space, and increasing the sinkhorn_loss_weight or the reach parameter (which controls unbalancedness, i.e. mass discarded from the partial transport plan with default transported mass m=0.5) should increase mixing — use this as a directional sensitivity check. Second, confirm that reconstruction quality from the uncoupled AutoEncoders (with batch index as decoder covariate) has not been excessively degraded by the IOT plus Sinkhorn divergence regularization terms. Third, verify in a UMAP of the 16-dimensional embeddings that per-modality biological structure (e.g. cell-type identities) is preserved rather than collapsed. A GPU is optional but recommended for the fitting that precedes this assessment.

## Related tools

- **scConfluence** (Provides model.fit(), model.get_latent(), and the sinkhorn_loss_weight / reach hyperparameters whose effects on latent-space mixing this assessment probes) — https://github.com/cantinilab/scconfluence
- **Scanpy** (Used for UMAP visualization of the extracted 16-dimensional latent embeddings to check modality mixing and preserved biological structure)

## Examples

```
model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent(); np.save('latent_embeddings.npy', mdata.obsm['latent'])
```

## Evaluation signals

- Latent space is visually mixed across modalities (cells from both modalities interleave in UMAP rather than separating by modality of origin)
- Increasing sinkhorn_loss_weight or reach monotonically increases modality mixing, confirming the regularization controls alignment strength as expected
- Reconstruction quality from the AutoEncoders is not excessively lost relative to the alignment gain
- Per-modality biological structure (known cell types/states) is preserved in the UMAP of the embeddings
- Embeddings are correctly written to mdata.obsm['latent'] and exported as latent_embeddings.npy

## Limitations

- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent, which can distort mixing and structure assessment
- Mini-batch optimization rarely co-samples rare populations from each modality, weakening alignment signal for those populations in the latent space
- The default transported mass m=0.5 assumes no prior knowledge of the level of unbalancedness; wrong reach/m settings can mislead the mixing-vs-reconstruction trade-off

## Evidence

- [full_text] Validation: verify the fitted objective yields a mixed latent space: [evidence span withheld — anchor status: not_found]
- [discussion] Default transported mass for unbalanced OT: [evidence span withheld — anchor status: not_found]
