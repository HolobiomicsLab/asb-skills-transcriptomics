---
name: converted-feature-matrix-scaling
description: Use when modalities were converted to shared gene-level features (scATAC
  peaks via gene activities, protein names mapped to coding genes, scRNA/smFISH subset
  to shared genes) and need to be put on a comparable range before computing the cross-modality
  cost matrix.
when_to_use_negative:
- No cross-modality feature connections exist (e.g., integrating electrophysiology
  measurements with gene expression), so there is no converted common-feature space
  to scale
- Modalities already share native gene-level features on the same scale and no peak-to-gene
  or protein-to-gene conversion was performed
- The goal is dimensionality reduction on the complete feature set — that is handled
  by the uncoupled autoencoders, not by scaling the converted features
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scConfluence
  role: Method that consumes the scaled common-feature matrices to compute the cross-modality
    cost matrix used by regularized Inverse Optimal Transport
  repo: https://github.com/cantinilab/scconfluence
- name: SciPy
  role: scipy.spatial.distance.cdist computes the correlation-based cross-modality
    distance on the scaled cm_features
- name: Scanpy
  role: Normalization utilities (normalize_total, log1p, scaling) within the scverse
    workflow preceding feature conversion
- name: Muon
  role: MuData object handling and modality normalization (TF-IDF, CLR) upstream of
    building the common features matrix
- name: Signac
  role: Computes gene activity scores from scATAC peaks to convert chromatin accessibility
    into gene-level common features
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:6274f6bddae7d33a1ee5c73c92d7b58df58bff1a337b1381a8ae961fa90da7ac
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
  - Scanpy
  - SciPy
  - Signac
  - scConfluence
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# converted-feature-matrix-scaling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scales the gene-converted common features matrices (e.g., scATAC gene activity scores, surface proteins mapped to coding genes) so features from different modalities occupy a comparable range before the cross-modality Pearson/correlation distance matrix is computed for Inverse Optimal Transport in scConfluence.

## When to use

Use when modalities were converted to shared gene-level features (scATAC peaks via gene activities, protein names mapped to coding genes, scRNA/smFISH subset to shared genes) and need to be put on a comparable range before computing the cross-modality cost matrix.

## When NOT to use

- No cross-modality feature connections exist (e.g., integrating electrophysiology measurements with gene expression), so there is no converted common-feature space to scale
- Modalities already share native gene-level features on the same scale and no peak-to-gene or protein-to-gene conversion was performed
- The goal is dimensionality reduction on the complete feature set — that is handled by the uncoupled autoencoders, not by scaling the converted features

## Inputs

- Preprocessed MuData object with modalities (scRNA, scATAC, smFISH, surface protein)
- Gene activity scores derived from scATAC peaks (computed with Signac)
- Surface protein-to-coding-gene mappings
- Per-modality converted common-feature matrices (cm_features) restricted to shared genes

## Outputs

- Scaled common features matrices comparable across modalities
- Cross-modality distance matrix stored in mdata.uns['cross_modality1+modality2'] with shape (n_cells_modality1, n_cells_modality2)
- Registration of the cost matrix key in mdata.uns['cross_keys']

## How to apply

First build the common features matrix by deriving gene activity scores from scATAC peaks (with Signac, as done for the Open Problems Multiome dataset) and mapping each measured surface protein to its coding gene, then subsetting the scRNA and smFISH matrices to the genes shared across modalities. Scale the gene-converted matrices so features from different modalities are on a comparable range — without this step, correlation-style distances on the connected features would be dominated by modality-specific feature ranges. Compute the cross-modality distance on the scaled cm_features with scipy.spatial.distance.cdist and store it under mdata.uns['cross_modality1+modality2'], registering it in mdata.uns['cross_keys']. The scaled, connected features are the weak feature connections that scConfluence's regularized Inverse Optimal Transport (rIOT) uses, complementing the uncoupled autoencoders trained on complete features.

## Related tools

- **scConfluence** (Method that consumes the scaled common-feature matrices to compute the cross-modality cost matrix used by regularized Inverse Optimal Transport) — https://github.com/cantinilab/scconfluence
- **SciPy** (scipy.spatial.distance.cdist computes the correlation-based cross-modality distance on the scaled cm_features)
- **Scanpy** (Normalization utilities (normalize_total, log1p, scaling) within the scverse workflow preceding feature conversion)
- **Muon** (MuData object handling and modality normalization (TF-IDF, CLR) upstream of building the common features matrix)
- **Signac** (Computes gene activity scores from scATAC peaks to convert chromatin accessibility into gene-level common features)

## Examples

```
from scipy.spatial.distance import cdist; D = cdist(mod1_cm_features, mod2_cm_features)
```

## Evaluation signals

- cost_matrix.npy exists and has shape (n_cells_modality1, n_cells_modality2)
- All entries of the cost matrix are finite and non-negative correlation-distance values
- The distance matrix is stored under mdata.uns['cross_modality1+modality2'] and its key is registered in mdata.uns['cross_keys']
- Per-feature scale statistics of the converted matrices are comparable across modalities after scaling
- Downstream unbalanced OT plan is computed from this cost matrix (Pearson similarity) via ot.partial.partial_wasserstein with default transported mass m=0.5

## Limitations

- Converting all modalities to the same features prior to integration causes loss of biological information because across-modality feature connections are rare and noisy
- The cost matrix relies on cross-modality feature connections, which are not always available (e.g., electrophysiology vs gene expression)
- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent

## Evidence
