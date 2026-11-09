---
name: unbalanced-sinkhorn-divergence-regularization
description: Use when integrating unpaired single-cell modalities (e.
when_to_use_negative:
- No cross-modality feature connections are available for a modality pair (e.g. electrophysiology
  vs gene expression), since the coupling cost matrix depends on weakly connected
  features
- Data are paired (same cells profiled across modalities); scConfluence targets diagonal
  integration of unpaired data, and joint integration of paired with unpaired data
  is not addressed
- The goal requires handling rare cell populations that rarely co-occur in mini-batches;
  mini-batch optimization makes rare populations much less likely to be simultaneously
  sampled from each modality
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scConfluence
  role: Implements the uncoupled AutoEncoders, the unbalanced OT plan (ot.partial.partial_wasserstein,
    m=0.5), the IOT loss and Sinkhorn divergence regularizer trained via model.fit;
    latent embeddings extracted with model.get_latent()
  repo: https://github.com/cantinilab/scconfluence
- name: Scanpy
  role: Normalization of scRNA counts (normalize_total to 10,000 then log1p) feeding
    the encoder inputs used in the alignment objective
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:a116f21736480ff001479c45ac10ca1175866403ae5729da94d36d091fde34d9
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# unbalanced-sinkhorn-divergence-regularization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs scConfluence's fixed alignment objective: gradients of the IOT loss (penalizing distances between embeddings coupled by an unbalanced OT plan computed from weakly connected features) and of an unbalanced Sinkhorn divergence regularizer are back-propagated through uncoupled modality autoencoders to align latent cell embeddings across modalities.

## When to use

Use when integrating unpaired single-cell modalities (e.g. scRNA-seq with scATAC-seq, surface protein, or CyTOF in a MuData object) where cross-modality distance matrices exist from weakly connected features and a shared low-dimensional latent space must be learned despite modality-specific cell populations.

## When NOT to use

- No cross-modality feature connections are available for a modality pair (e.g. electrophysiology vs gene expression), since the coupling cost matrix depends on weakly connected features
- Data are paired (same cells profiled across modalities); scConfluence targets diagonal integration of unpaired data, and joint integration of paired with unpaired data is not addressed
- The goal requires handling rare cell populations that rarely co-occur in mini-batches; mini-batch optimization makes rare populations much less likely to be simultaneously sampled from each modality

## Inputs

- Preprocessed MuData object with cross-modality distance matrices in mdata.uns keyed by cross_keys (cost from Pearson similarity on common/weakly connected features)
- Per-modality normalized matrices (log-normalized scRNA with 100 PCs, TF-IDF-normalized scATAC, CLR-normalized protein)
- Batch index annotation for use as decoder covariate

## Outputs

- Fitted uncoupled AutoEncoder per modality (via model.fit, saved to 'results')
- Aligned latent cell embeddings in mdata.obsm['latent'] (16-dimensional), exportable as latent_embeddings.npy
- Training objective values: IOT distance term and weighted Sinkhorn divergence regularizer

## How to apply

Load the preprocessed MuData object and use the cross-modality distance matrices computed from connected features (stored in mdata.uns under cross_keys) as the coupling cost between modalities. Initialize one uncoupled AutoEncoder per modality (scconfluence.model.AutoEncoder) mapping each modality into a shared 16-dimensional latent space, with the first 100 principal components as input for scRNA and batch index as decoder covariate. Compute the unbalanced OT plan between latent embeddings of each modality pair from the converted feature cost matrix using ot.partial.partial_wasserstein with default transported mass m=0.5, then train with model.fit(save_path='results'), minimizing the IOT distance term penalizing plan-coupled embedding distances plus the unbalanced Sinkhorn divergence regularizer weighted by sinkhorn_loss_weight, with the reach parameter controlling how much mass is discarded (unbalancedness). Gradients of both terms are back-propagated through both modality encoders; increasing sinkhorn_loss_weight or reach increases latent mixing, so tune them to balance alignment against preservation of per-modality reconstruction and biological structure.

## Related tools

- **scConfluence** (Implements the uncoupled AutoEncoders, the unbalanced OT plan (ot.partial.partial_wasserstein, m=0.5), the IOT loss and Sinkhorn divergence regularizer trained via model.fit; latent embeddings extracted with model.get_latent()) — https://github.com/cantinilab/scconfluence
- **Scanpy** (Normalization of scRNA counts (normalize_total to 10,000 then log1p) feeding the encoder inputs used in the alignment objective)

## Examples

```
import scconfluence; model = scconfluence.model.AutoEncoder(mdata, modality_keys=..., cross_keys=...); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent()
```

## Evaluation signals

- The fitted objective yields a mixed latent space: increasing sinkhorn_loss_weight or reach should increase cell-type mixing across modalities in mdata.obsm['latent']
- Reconstruction quality is not excessively degraded relative to the alignment gain (decoder with ZINB likelihood on raw counts, batch index covariate)
- Per-modality biological structure is preserved in a UMAP of the 16-dimensional latent embeddings (e.g. B cell vs Monocyte separation in scRNA-scATAC-CyTOF data)
- The transport plan discards mass consistent with the unbalancedness setting (reach; default transported mass m=0.5), leaving mass unassigned for modality-specific cell types
- Cross-modality cost stored in mdata.uns under cross_keys is actually consumed as the coupling cost, not recomputed from raw features

## Limitations

- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent where rare populations are unlikely to be simultaneously sampled from each modality
- The method relies on cross-modality feature connections, which are not always available (e.g. electrophysiology vs gene expression)
- Integration of paired with unpaired multimodal data is not yet addressed by scConfluence
- Hidden layer sizes must be adapted to dataset dimensionality (Supplementary Table 5 not provided in the document); latent dimension is fixed at 16

## Evidence

- [methods] Gradients of both losses back-propagated through encoders: [evidence span withheld — anchor status: not_found]
- [discussion] Unbalanced OT plan with m=0.5: [evidence span withheld — anchor status: not_found]
