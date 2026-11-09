---
name: cross-modality-feature-connection-computation
description: Use when integrating unpaired single-cell modalities (e.g. scRNA-seq
  with smFISH or scATAC-seq) where features are only weakly connected across modalities
  and a feature-level cost matrix is needed for rIOT-based alignment.
when_to_use_negative:
- When integrating modality pairs with no cross-modality feature connections at all
  (e.g. electrophysiology measurements with gene expression), since the method 'relies
  on connections between features of different modalities'.
- When data are paired (same cells measured across modalities) rather than unpaired/diagonal
  integration.
- When the goal is a modality-specific feature conversion rather than preserving each
  modality's complete feature set with uncoupled autoencoders.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scConfluence
  role: Consumes the cross-modality cost matrix (via mdata.uns cross_keys) during
    uncoupled-autoencoder + Sinkhorn (rIOT) training
  repo: https://github.com/cantinilab/scconfluence
- name: Muon
  role: Hosts the multi-modal MuData object whose uns field stores the cross-modality
    matrix
- name: scipy
  role: scipy.spatial.distance.cdist computes the cross-modality distance matrix on
    weakly connected gene features
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:393a24cc9f56455247d6947a147d42d7492203026321db1f4a87cd64fabfa22a
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
  - Seurat
  - Uniport
  - scConfluence
  - scGLUE
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-modality-feature-connection-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the cross-modality distance/cost matrix linking features (e.g. genes) shared between single-cell modalities, which scConfluence uses to align unpaired modalities via regularized Inverse Optimal Transport on weakly connected features.

## When to use

Use when integrating unpaired single-cell modalities (e.g. scRNA-seq with smFISH or scATAC-seq) where features are only weakly connected across modalities and a feature-level cost matrix is needed for rIOT-based alignment.

## When NOT to use

- When integrating modality pairs with no cross-modality feature connections at all (e.g. electrophysiology measurements with gene expression), since the method 'relies on connections between features of different modalities'.
- When data are paired (same cells measured across modalities) rather than unpaired/diagonal integration.
- When the goal is a modality-specific feature conversion rather than preserving each modality's complete feature set with uncoupled autoencoders.

## Inputs

- MuData object with multiple modalities (e.g. scRNA-seq and smFISH AnnData modalities)
- Set of weakly connected features shared across modalities (e.g. genes measured in both scRNA and smFISH)
- Feature expression matrices per modality (normalized counts used for the Pearson similarity or cdist computation)

## Outputs

- Cross-modality feature distance/cost matrix
- Registration of the cost matrix in mdata.uns under the cross_keys entry for scConfluence consumption

## How to apply

Load the modalities (e.g. Smartseq2 scRNA-seq and smFISH from mouse somatosensory cortex) into a MuData object and identify the weakly connected gene features shared between them. Compute the cross-modality distance matrix on these shared features — for the cortex scRNA-smFISH case, with scipy.spatial.distance.cdist — and register it in mdata.uns under the cross_keys entry so scConfluence can consume it. Wherever the paper details the cost computation, the Pearson similarity is used to compute the cost matrix between common feature matrices. This matrix supplies the across-modality feature linkage that the uncoupled autoencoders + Sinkhorn (rIOT) training relies on, since scConfluence deliberately avoids converting all modalities to the same feature space. Tune downstream sinkhorn_loss_weight/reach if modality embeddings separate, but the cost matrix itself is a prerequisite input to model fitting.

## Related tools

- **scConfluence** (Consumes the cross-modality cost matrix (via mdata.uns cross_keys) during uncoupled-autoencoder + Sinkhorn (rIOT) training) — https://github.com/cantinilab/scconfluence
- **Muon** (Hosts the multi-modal MuData object whose uns field stores the cross-modality matrix)
- **scipy** (scipy.spatial.distance.cdist computes the cross-modality distance matrix on weakly connected gene features)

## Examples

```
from scipy.spatial.distance import cdist; D = cdist(rna_shared_features, fish_shared_features); mdata.uns['cross_keys'] = D
```

## Evaluation signals

- The distance matrix is registered in mdata.uns under the expected cross_keys entry and matches the number of weakly connected shared features.
- After scConfluence training, modality embeddings do not separate in the shared latent space (tune sinkhorn_loss_weight/reach otherwise).
- Imputed spatial patterns for held-out smFISH gene groups reproduce the measured patterns (average/median Spearman correlations match the scConfluence-vs-baseline ranking).
- Imputed patterns for Scgn, Synpr and Olah qualitatively match Fig. 4d-f of the paper.

## Limitations

- scConfluence relies on cross-modality feature connections, which are not always available (e.g. electrophysiology vs gene expression).
- Across-modality feature connections are rare and noisy, so converting all modalities to the same features prior to integration causes loss of biological information — hence the cost-matrix-on-shared-features design.
- Hidden layer sizes for the autoencoders are dataset-dependent (Supplementary Table 5 referenced but not provided in the document).

## Evidence

- [methods] cdist workflow step: [evidence span withheld — anchor status: not_found]
