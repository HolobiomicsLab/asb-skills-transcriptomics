---
name: optimal-transport-embedding-alignment
description: Use when integrating unpaired single-cell modalities (e.g. scRNA-seq
  with scATAC-seq, surface protein, morphology, or CyTOF) where cross-modality feature
  connections are weak, and a modality-mixed latent embedding is the desired output.
when_to_use_negative:
- Modalities lack any cross-modality feature connections (e.g. electrophysiology measurements
  vs gene expression), since scConfluence relies on connections between features of
  different modalities
- Data are paired (same cells measured in all modalities) — diagonal integration is
  designed for unpaired cells from the same biological condition
- Goal is detecting very rare cell populations, which are hard to align because they
  are rarely co-sampled from each modality in mini-batches
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scConfluence
  role: Provides the uncoupled AutoEncoders, unbalanced OT plan (ot.partial.partial_wasserstein,
    m=0.5), IOT loss with Sinkhorn divergence regularization, and get_latent() extraction
    of aligned embeddings
  repo: https://github.com/cantinilab/scconfluence
- name: Scanpy
  role: Used to compute the neighbor graph on the latent representation (sc.pp.neighbors
    with use_rep='latent') and UMAP embedding to validate modality mixing and preserved
    biological structure
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:15168f62197a69d32f2e49b5bc3ee916795fbd7adc263f6efaae9c757239a660
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# optimal-transport-embedding-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align unpaired single-cell modalities into a shared 16-dimensional latent space by back-propagating the IOT loss and unbalanced Sinkhorn divergence regularization from an unbalanced Optimal Transport plan computed on weakly connected cross-modality features, as implemented in scConfluence.

## When to use

Use when integrating unpaired single-cell modalities (e.g. scRNA-seq with scATAC-seq, surface protein, morphology, or CyTOF) where cross-modality feature connections are weak, and a modality-mixed latent embedding is the desired output.

## When NOT to use

- Modalities lack any cross-modality feature connections (e.g. electrophysiology measurements vs gene expression), since scConfluence relies on connections between features of different modalities
- Data are paired (same cells measured in all modalities) — diagonal integration is designed for unpaired cells from the same biological condition
- Goal is detecting very rare cell populations, which are hard to align because they are rarely co-sampled from each modality in mini-batches

## Inputs

- Preprocessed MuData object with one modality AnnData per omics layer and a cross-modality distance/cost matrix stored in mdata.uns referenced by cross_keys
- Per-modality QC-filtered and normalized matrices (log-normalized scRNA with top PCs, TF-IDF normalized scATAC, CLR-normalized protein)

## Outputs

- Modality-mixed latent embeddings in mdata.obsm['latent'] (16 dimensions, aligned to mdata.obs_names)
- Saved embedding array latent_embeddings.npy and fitted model checkpoint under save_path='results'

## How to apply

Compute a cross-modality cost matrix (e.g. Pearson similarity on connected features, or scipy cdist for morphology-gene pairs) and store it in mdata.uns with a cross_keys entry. Initialize one uncoupled scconfluence.model.AutoEncoder per modality mapping to a shared latent dimension of 16, then build a scconfluence.model.ScConfluence model and train with .fit(save_path='results'). During training, compute the unbalanced OT plan between latent embeddings of each modality pair via ot.partial.partial_wasserstein with default transported mass m=0.5, and minimize the IOT distance term penalizing plan-coupled embedding distances plus a Sinkhorn divergence regularizer weighted by sinkhorn_loss_weight, with the reach parameter controlling unbalancedness (mass discarded from the plan); gradients flow through both modality encoders. Extract aligned embeddings with model.get_latent() into mdata.obsm['latent'] aligned to mdata.obs_names.

## Related tools

- **scConfluence** (Provides the uncoupled AutoEncoders, unbalanced OT plan (ot.partial.partial_wasserstein, m=0.5), IOT loss with Sinkhorn divergence regularization, and get_latent() extraction of aligned embeddings) — https://github.com/cantinilab/scconfluence
- **Scanpy** (Used to compute the neighbor graph on the latent representation (sc.pp.neighbors with use_rep='latent') and UMAP embedding to validate modality mixing and preserved biological structure)

## Examples

```
import scconfluence, mudata as md; mdata = md.read('data.h5mu'); model = scconfluence.model.ScConfluence(autoencoders); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent()
```

## Evaluation signals

- The fitted objective yields a mixed latent space in which modalities are aligned (increasing sinkhorn_loss_weight or reach should increase mixing)
- Reconstruction quality is not excessively degraded despite the alignment loss
- Per-modality biological structure is preserved in a UMAP of the latent embeddings (e.g. tufted vs untufted IT neurons organized along the apical-dendrite manifold with graded Fezf2 expression)
- scConfluence embeddings benchmark competitively against Seurat v3.0/v4.3.0, Liger, MultiMAP, Uniport, and scGLUE on scRNA-surface protein and scRNA-scATAC integration problems
- Latent embeddings are aligned to mdata.obs_names and written to mdata.obsm['latent'] with latent dimension 16

## Limitations

- scConfluence relies on cross-modality feature connections, which are not always available (e.g. integrating electrophysiology with gene expression)
- Rare cell populations are much less likely to be simultaneously sampled from each modality in mini-batches, yet mini-batch optimization is necessary to scale to millions of cells
- Integration of paired with unpaired multimodal data is not yet addressed by scConfluence; hidden layer sizes were adapted per dataset (Supplementary Table 5), latent dimension fixed at 16

## Evidence

- [methods] Unbalanced OT plan computed via ot.partial.partial_wasserstein with default transported mass m=0.5: [evidence span withheld — anchor status: not_found]
