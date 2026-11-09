---
name: cross-modality-shared-gene-subsetting
description: Use when integrating unpaired single-cell modalities (e.g. scRNA-seq
  with scATAC, smFISH, or surface protein/CyTOF data) whose native features differ,
  so modality-specific features must be converted to gene-level common features to
  compute a cross-modality cell distance for Optimal Transport.
when_to_use_negative:
- The modality pair has no cross-modality feature connections (e.g. integrating electrophysiology
  measurements with gene expression), so no shared gene space can be constructed
- Both modalities are already profiled on identical features, making gene conversion
  unnecessary and lossy
- The goal is full-feature diagonal integration without an Optimal Transport cost
  matrix — the autoencoders alone operate on the complete feature set and do not require
  this conversion
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scConfluence
  role: Consumes the common features matrix and cross-modality cost matrix to fit
    the unbalanced Inverse Optimal Transport plan; stores cm_features and registers
    cross_keys on the MuData object
  repo: https://github.com/cantinilab/scconfluence
- name: SciPy
  role: scipy.spatial.distance.cdist computes the cross-modality distance matrix on
    the connected common features
- name: Signac
  role: Computes gene activity scores from scATAC peaks to convert chromatin features
    to gene-level features
- name: Muon
  role: Hosts the multi-modal MuData object whose uns entries (cross_modality1+modality2,
    cross_keys) store the outputs
- name: Scanpy
  role: Part of the scverse preprocessing stack surrounding the gene-space subsetting
    and scaling step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:767c72a9112b5b7c19b008f286faee69df7d6ab3e67b8cc0008966822d0168cc
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-modality-shared-gene-subsetting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build the cross-modality common features matrix Y by converting heterogeneous modalities (scATAC peaks to gene activity, surface proteins to coding genes, smFISH features) to a shared gene space, then compute a correlation-based biological cost matrix between cells of different modalities for Inverse Optimal Transport in scConfluence.

## When to use

Use when integrating unpaired single-cell modalities (e.g. scRNA-seq with scATAC, smFISH, or surface protein/CyTOF data) whose native features differ, so modality-specific features must be converted to gene-level common features to compute a cross-modality cell distance for Optimal Transport.

## When NOT to use

- The modality pair has no cross-modality feature connections (e.g. integrating electrophysiology measurements with gene expression), so no shared gene space can be constructed
- Both modalities are already profiled on identical features, making gene conversion unnecessary and lossy
- The goal is full-feature diagonal integration without an Optimal Transport cost matrix — the autoencoders alone operate on the complete feature set and do not require this conversion

## Inputs

- Preprocessed MuData object with per-modality AnnData layers (scRNA counts, scATAC peak matrix, smFISH counts, surface protein matrix)
- scATAC peak-to-gene annotation for computing gene activity scores
- Protein-to-coding-gene mapping table for surface protein modalities
- List of genes shared across modalities for subsetting

## Outputs

- Common features matrix (cm_features) per modality in the shared gene space, after scaling
- Cross-modality distance/cost matrix stored as mdata.uns['cross_modality1+modality2'] (and serialized e.g. cost_matrix.npy)
- Updated mdata.uns['cross_keys'] registry listing the computed cross-modality distance keys

## How to apply

Start from the preprocessed MuData object: for scATAC data, derive gene activity scores from the open peaks (with Signac, as done for the Open Problems Multiome dataset); map each measured surface protein to its coding gene; then subset the scRNA and smFISH matrices to the genes shared across modalities to form the common features matrix. Scale the gene-converted matrices so features from different modalities are on a comparable range before any distance computation. Compute the cross-modality distance matrix with scipy.spatial.distance.cdist on the modalities' cm_features (Pearson/correlation similarity), because this cost matrix is what the unbalanced Inverse Optimal Transport plan is derived from. Store the result under mdata.uns['cross_modality1+modality2'] and register the key in mdata.uns['cross_keys'] so downstream scConfluence steps can discover it. Rationale: direct feature conversion across modalities is weakly connected and noisy, so scConfluence restricts the rIOT cost term to these shared/converted features while autoencoders handle the full feature sets.

## Related tools

- **scConfluence** (Consumes the common features matrix and cross-modality cost matrix to fit the unbalanced Inverse Optimal Transport plan; stores cm_features and registers cross_keys on the MuData object) — https://github.com/cantinilab/scconfluence
- **SciPy** (scipy.spatial.distance.cdist computes the cross-modality distance matrix on the connected common features)
- **Signac** (Computes gene activity scores from scATAC peaks to convert chromatin features to gene-level features)
- **Muon** (Hosts the multi-modal MuData object whose uns entries (cross_modality1+modality2, cross_keys) store the outputs)
- **Scanpy** (Part of the scverse preprocessing stack surrounding the gene-space subsetting and scaling step)

## Examples

```
from scipy.spatial.distance import cdist; D = cdist(mod1_cm_features, mod2_cm_features, metric='correlation')
```

## Evaluation signals

- cost_matrix.npy exists after the step and is loadable
- Cost matrix has shape (n_cells_modality1, n_cells_modality2), matching the number of cells in each modality
- All cost matrix values are finite (no NaN/inf) and non-negative, consistent with correlation distances
- mdata.uns['cross_keys'] contains the key 'modality1+modality2' and mdata.uns['cross_modality1+modality2'] holds the matrix
- The gene-converted matrices from each modality are on a comparable scale before cdist is applied (scaling step completed)

## Limitations

- Converting all modalities to the same features causes loss of biological information because across-modality feature connections are rare and noisy — this is why the cost matrix is only applied on the weakly connected common features
- Cross-modality feature connections are not always available, e.g. when integrating electrophysiology measurements with gene expression
- The default unbalanced OT uses m=0.5 transported mass; without prior information about the level of unbalancedness of the data this default may not fit rare or modality-specific cell populations

## Evidence
