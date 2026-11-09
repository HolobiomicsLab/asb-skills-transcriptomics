---
name: correlation-distance-cost-matrix-computation
description: Use when two modalities have been converted to a shared feature space
  (e.g. peaks→gene activity, proteins→coding genes) and a cell-by-cell cost matrix
  is needed for unbalanced OT alignment.
when_to_use_negative:
- Modality pairs with no cross-modality feature connections (e.g. electrophysiology
  measurements vs gene expression), since the cost matrix relies on shared features
- Modality-specific or unconvertible features whose conversion to the common space
  would lose biological information; prefer keeping complete feature sets in the autoencoders
- The input is already a distance/cost matrix between cells and only the OT plan needs
  to be computed
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scConfluence
  role: Method whose cross_modality distance computation consumes the cm_features
    and stores the cost matrix in the MuData object
  repo: https://github.com/cantinilab/scconfluence
- name: SciPy
  role: scipy.spatial.distance.cdist is used to compute the cross-modality correlation
    distance matrix
- name: Muon
  role: Provides tfidf normalization and MuData container where cost matrices are
    stored in .uns
- name: Signac
  role: Computes gene activities from scATAC peaks to build the common features
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:d81d8134b3c7ea772a9b80a812915c4e5216aa9d5376143e6eb6df6c7fba8a24
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# correlation-distance-cost-matrix-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute the cross-modality cost matrix for scConfluence by applying scipy.spatial.distance.cdist to the scaled common-feature matrices (e.g. gene activity scores from scATAC and gene-mapped protein counts), storing the correlation-based distances for downstream unbalanced Optimal Transport.

## When to use

Use when two modalities have been converted to a shared feature space (e.g. peaks→gene activity, proteins→coding genes) and a cell-by-cell cost matrix is needed for unbalanced OT alignment.

## When NOT to use

- Modality pairs with no cross-modality feature connections (e.g. electrophysiology measurements vs gene expression), since the cost matrix relies on shared features
- Modality-specific or unconvertible features whose conversion to the common space would lose biological information; prefer keeping complete feature sets in the autoencoders
- The input is already a distance/cost matrix between cells and only the OT plan needs to be computed

## Inputs

- Preprocessed MuData object with scRNA and scATAC (or smFISH / protein) modalities
- Gene activity scores derived from scATAC peaks (Signac)
- Gene-converted, scaled common-feature matrices (cm_features) per modality

## Outputs

- cost_matrix.npy of shape (n_cells_modality1, n_cells_modality2) with finite, non-negative correlation distances, stored in mdata.uns['cross_modality1+modality2']
- Registered key in mdata.uns['cross_keys']

## How to apply

First build the common features matrix Y by converting each modality onto shared features: derive gene activity scores from scATAC peaks with Signac, map each surface protein to its coding gene, and subset scRNA and smFISH matrices to genes shared across modalities. Scale the gene-converted matrices so features from different modalities are on a comparable range. Then compute the cross-modality distance matrix with scipy.spatial.distance.cdist applied to the modalities' cm_features, storing it under mdata.uns['cross_modality1+modality2'] and registering the key in mdata.uns['cross_keys']. scConfluence uses the Pearson similarity to compute this cost matrix; correlation distances are non-negative and finite, which is required for the subsequent unbalanced OT plan computed with ot.partial.partial_wasserstein (default m=0.5).

## Related tools

- **scConfluence** (Method whose cross_modality distance computation consumes the cm_features and stores the cost matrix in the MuData object) — https://github.com/cantinilab/scconfluence
- **SciPy** (scipy.spatial.distance.cdist is used to compute the cross-modality correlation distance matrix)
- **Muon** (Provides tfidf normalization and MuData container where cost matrices are stored in .uns)
- **Signac** (Computes gene activities from scATAC peaks to build the common features)

## Examples

```
from scipy.spatial.distance import cdist; mdata.uns['cross_modality1+modality2'] = cdist(cm_features_modality1, cm_features_modality2); mdata.uns['cross_keys'].append('cross_modality1+modality2')
```

## Evaluation signals

- cost_matrix.npy exists and has shape (n_cells_modality1, n_cells_modality2)
- All entries of the cost matrix are finite and non-negative (correlation distances)
- mdata.uns['cross_keys'] contains the registered key 'cross_modality1+modality2'
- Common features matrices were scaled so features from different modalities are on a comparable range before computing distances

## Limitations

- Relies on cross-modality feature connections, which are not always available (e.g. electrophysiology vs gene expression)
- Feature conversion to a common space can cause loss of biological information since across-modality feature connections are rare and noisy

## Evidence

- [methods] cdist on cm_features stored in mdata.uns: [evidence span withheld — anchor status: not_found]
- [discussion] unbalanced OT default mass: [evidence span withheld — anchor status: not_found]
