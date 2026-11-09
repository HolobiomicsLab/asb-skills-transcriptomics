---
name: optimal-transport-latent-space-tuning
description: Use when training scConfluence on unpaired single-cell multiomics data
  (e.g., scRNA-scATAC MuData objects) and the latent embeddings must be checked for
  modality mixing versus cell-type structure before benchmarking or downstream clustering.
when_to_use_negative:
- No cross-modality feature connections are available (e.g., integrating electrophysiology
  with gene expression), since scConfluence relies on connections between features
  of different modalities
- Data is paired multimodal (same cells profiled in both modalities) — scConfluence
  targets diagonal integration of unpaired data and does not yet jointly integrate
  paired and unpaired data
- The target population is extremely rare, since rare populations are unlikely to
  be co-sampled in mini-batches, degrading both scConfluence and state-of-the-art
  methods
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scConfluence
  role: Method combining uncoupled autoencoders with regularized Inverse Optimal Transport;
    provides the fit/get_latent API whose sinkhorn_loss_weight and reach are tuned
  repo: https://github.com/cantinilab/scconfluence
- name: Muon
  role: Prepares the MuData input (TF-IDF for scATAC, QC filtering) consumed by the
    ScConfluence model
- name: SciPy
  role: Computes the cross-modality cost matrix via scipy.spatial.distance.cdist on
    connected features
- name: Scanpy
  role: Log-normalizes scRNA counts (normalize_total to 10,000 then log1p) before
    autoencoder input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:82b676406c78b905259d78733143e0f41d1f2051c03145ca03e15b65732fa227
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
  - Liger
  - MultiMAP
  - Muon
  - Scanpy
  - SciPy
  - Seurat v4.3.0
  - Uniport
  - scConfluence
  - scGLUE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# optimal-transport-latent-space-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tuning the unbalanced optimal-transport hyperparameters of scConfluence (sinkhorn_loss_weight and reach, with partial transported mass m=0.5) during model training so that the shared latent space both mixes modalities and preserves biological structure in diagonal integration tasks.

## When to use

Use when training scConfluence on unpaired single-cell multiomics data (e.g., scRNA-scATAC MuData objects) and the latent embeddings must be checked for modality mixing versus cell-type structure before benchmarking or downstream clustering.

## When NOT to use

- No cross-modality feature connections are available (e.g., integrating electrophysiology with gene expression), since scConfluence relies on connections between features of different modalities
- Data is paired multimodal (same cells profiled in both modalities) — scConfluence targets diagonal integration of unpaired data and does not yet jointly integrate paired and unpaired data
- The target population is extremely rare, since rare populations are unlikely to be co-sampled in mini-batches, degrading both scConfluence and state-of-the-art methods

## Inputs

- MuData object with preprocessed scRNA-seq (log-normalized) and scATAC-seq (TF-IDF via Muon) modalities
- Cross-modality cost matrix computed from connected features, stored in mdata.uns with mdata.uns['cross_keys']
- Per-modality scconfluence AutoEncoder instances

## Outputs

- Trained ScConfluence model checkpoints saved to 'results'
- Latent embeddings in mdata.obsm['latent'] (latent dimension 16 in the paper)
- Benchmark metric table (cell-type purity, label-transfer accuracy, connectivity, FOSCTTM) e.g., benchmark_metrics.csv

## How to apply

Instantiate one scconfluence AutoEncoder per modality, build the ScConfluence model, and call model.fit(save_path='results') while tuning sinkhorn_loss_weight and reach so that latent mixing and structure preservation are both maintained; extract embeddings with model.get_latent() into mdata.obsm['latent']. The OT plan is computed unbalanced via ot.partial.partial_wasserstein with default transported mass m=0.5, which the paper found robust when the degree of unbalancedness is unknown. The cross-modality cost matrix (e.g., computed with scipy.spatial.distance.cdist on connected features and stored under mdata.uns with mdata.uns['cross_keys']) drives the transport, so verify it is finite and correctly keyed before tuning. Judge tuning success with cell-type purity, label-transfer accuracy, connectivity of the latent embedding, and FOSCTTM — especially under unbalanced population scenarios where robustness matters most.

## Related tools

- **scConfluence** (Method combining uncoupled autoencoders with regularized Inverse Optimal Transport; provides the fit/get_latent API whose sinkhorn_loss_weight and reach are tuned) — https://github.com/cantinilab/scconfluence
- **Muon** (Prepares the MuData input (TF-IDF for scATAC, QC filtering) consumed by the ScConfluence model)
- **SciPy** (Computes the cross-modality cost matrix via scipy.spatial.distance.cdist on connected features)
- **Scanpy** (Log-normalizes scRNA counts (normalize_total to 10,000 then log1p) before autoencoder input)

## Examples

```
model = scconfluence.ScConfluence(autoencoders=aes, unmapped_data=mdata); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent(mdata)
```

## Evaluation signals

- CSV/benchmark table contains all 6 methods x 4 unbalancing scenarios rows with finite values for cell-type purity, label-transfer accuracy, connectivity, and FOSCTTM
- Qualitative ranking reproduces Fig. 2b: scConfluence outperforms Seurat, Liger, MultiMAP, Uniport, and scGLUE in every scenario, with greater robustness as scenarios become less balanced
- mdata.uns['cross_keys'] is set and the cost matrix contains no NaN/inf entries before fit()
- Latent embeddings land in mdata.obsm['latent'] with the expected dimensionality (16) for all cells in each modality

## Limitations

- If prior information about the level of unbalancedness is unavailable, the default transported mass m=0.5 is used; mis-estimated unbalancedness can degrade alignment
- Rare cell populations are hard to handle: they are under-represented in parameter estimation, and mini-batch gradient descent rarely co-samples them from each modality
- A GPU is not required but strongly recommended by the README; hyperparameter sensitivity results (Supplementary Figs 13-18) are referenced but not included in the provided context

## Evidence

- [other] Tune sinkhorn_loss_weight and reach to maintain both latent mixing and structure preservation: [evidence span withheld — anchor status: not_found]
- [discussion] Default partial transported mass m=0.5 for unbalanced OT: [evidence span withheld — anchor status: not_found]
