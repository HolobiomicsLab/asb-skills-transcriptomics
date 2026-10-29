---
name: foscttm-metric-computation
description: 'Use when evaluating or tuning a diagonal integration method''s shared
  latent embeddings: FOSCTTM quantifies cross-modality alignment by measuring each
  cell''s nearest-neighbor agreement across modalities, e.g. when sweeping the unbalanced
  OT parameter m.'
when_to_use_negative:
- Data are truly paired (multiome/cite-seq) with known cell correspondences — matching-based
  alignment metrics designed for unpaired diagonal integration are unnecessary
- Only a single modality is present — no cross-modality nearest-neighbor consistency
  can be computed
- No latent embeddings in mdata.obsm['latent'] and no cross-modality cost matrix attached
  — the metric has no input to score
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scConfluence
  role: Trains the uncoupled-autoencoder + regularized IOT model whose latent embeddings
    (model.get_latent() -> mdata.obsm['latent']) FOSCTTM scores
  repo: https://github.com/cantinilab/scconfluence
- name: Scanpy
  role: Computes the neighborhood graph and UMAP on the latent representation (sc.pp.neighbors,
    sc.tl.umap) used together with FOSCTTM to score each m value
- name: Muon
  role: Provides the MuData container holding the modalities, obsm['latent'] and uns
    entries the metric is computed on
- name: POT
  role: Supplies ot.partial.partial_wasserstein used to compute the unbalanced OT
    plan whose parameter m is being swept and scored
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:5358edc589f81615b2bca17b636deb29a28fb1666bab63eb954365d7e3b76369
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
  - Muon
  - POT
  - Scanpy
  - scConfluence
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# foscttm-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute FOSCTTM (Fraction of Samples Closer Than the True Match), a cross-modality nearest-neighbor consistency metric, to score how well scConfluence's shared latent space aligns cells profiled from different modalities. Used here as part of the hyperparameter sensitivity analysis of the unbalanced OT transported-mass parameter m on the cell-line (scCATseq) benchmark.

## When to use

Use when evaluating or tuning a diagonal integration method's shared latent embeddings: FOSCTTM quantifies cross-modality alignment by measuring each cell's nearest-neighbor agreement across modalities, e.g. when sweeping the unbalanced OT parameter m.

## When NOT to use

- Data are truly paired (multiome/cite-seq) with known cell correspondences — matching-based alignment metrics designed for unpaired diagonal integration are unnecessary
- Only a single modality is present — no cross-modality nearest-neighbor consistency can be computed
- No latent embeddings in mdata.obsm['latent'] and no cross-modality cost matrix attached — the metric has no input to score

## Inputs

- MuData object with unpaired scCATseq cell-line scRNA and scATAC modalities, with cross-modality cost matrix in mdata.uns['cross_modality1+modality2'] and mdata.uns['cross_keys']
- Latent cell embeddings in mdata.obsm['latent'] obtained via scconfluence model.get_latent()
- Grid of transported-mass values (e.g. m = 0.3, 0.4, 0.5, 0.6, 0.75)

## Outputs

- Per-m FOSCTTM values of cross-modality nearest-neighbor consistency on the latent space
- sensitivity_curves.csv with purity, FOSCTTM and graph connectivity as a function of m
- sensitivity_fig13.png reproducing the qualitative pattern of the paper's Supplementary Fig. 13

## How to apply

Fit the scConfluence model on the unpaired scCATseq MuData object (with the cross-modality cost matrix in mdata.uns['cross_modality1+modality2'] and mdata.uns['cross_keys']), then extract latent embeddings with model.get_latent() into mdata.obsm['latent']. For each value of the transported-mass parameter m on the sensitivity grid (e.g. m = 0.3, 0.4, 0.5, 0.6, 0.75), refit the model and compute FOSCTTM as cross-modality nearest-neighbor consistency on the latent representation: for each cell, check whether its true paired counterpart ranks among the closest cross-modality neighbors. Score FOSCTTM alongside cell-type purity (kNN label agreement) and graph connectivity computed with Scanpy (sc.pp.neighbors, sc.tl.umap on the 'latent' representation). Plot each metric as a function of m, compare against Supplementary Fig. 13, and accept the sweep when the default m=0.5 configuration lies in a robust, near-optimal flat region. Rationale: the paper sets m=0.5 by default because it is robust when the true level of unbalanceness (removed K562/HCT populations) is unknown.

## Related tools

- **scConfluence** (Trains the uncoupled-autoencoder + regularized IOT model whose latent embeddings (model.get_latent() -> mdata.obsm['latent']) FOSCTTM scores) — https://github.com/cantinilab/scconfluence
- **Scanpy** (Computes the neighborhood graph and UMAP on the latent representation (sc.pp.neighbors, sc.tl.umap) used together with FOSCTTM to score each m value)
- **Muon** (Provides the MuData container holding the modalities, obsm['latent'] and uns entries the metric is computed on)
- **POT** (Supplies ot.partial.partial_wasserstein used to compute the unbalanced OT plan whose parameter m is being swept and scored)

## Examples

```
import scconfluence, scanpy as sc; model = scconfluence.model.ScConfluence(mdata, unimodal_aes=aes); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent(); sc.pp.neighbors(mdata, use_rep='latent')
```

## Evaluation signals

- The metric-vs-m curves reproduce the qualitative pattern of the paper's Supplementary Fig. 13
- The default m=0.5 configuration yields robust (near-optimal, flat-region) FOSCTTM, purity and connectivity values
- FOSCTTM is reported at every point of the sensitivity grid (e.g. m = 0.3, 0.4, 0.5, 0.6, 0.75) after refitting at each m
- sensitivity_curves.csv and sensitivity_fig13.png are produced and internally consistent (figure values match the table)
- scConfluence's FOSCTTM scores are compared against state-of-the-art baselines (Seurat v3.0, Liger, MultiMAP, Uniport, scGLUE), which it outperforms across progressively more unbalanced scenarios (half K562 removed, all K562 removed, K562 and HCT removed)

## Limitations

- The metric requires cross-modality feature connections to compute the cost matrix; such connections are not always available (e.g. electrophysiology vs gene expression)
- Rare cell populations are hard to handle: they are unlikely to be simultaneously sampled in mini-batches, which can degrade embedding quality and hence metric scores
- The sensitivity sweep validates a qualitative pattern (flat region at m=0.5) rather than a single ground-truth value, since Supplementary Fig. 13 itself is referenced but not provided in the source document

## Evidence

- [other] score the shared latent space with cell-type purity (kNN label agreement), FOSCTTM (cross-modality nearest-neighbor consistency): [evidence span withheld — anchor status: not_found]
- [discussion] default m=0.5 robustness: [evidence span withheld — anchor status: not_found]
- [supplementary] Sensitivity validation criterion: [evidence span withheld — anchor status: not_found]
