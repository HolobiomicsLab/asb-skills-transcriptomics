---
name: protein-to-coding-gene-mapping
description: Use when integrating modalities measured on different feature spaces
  (e.g., scRNA-seq vs surface proteins or smFISH) and you need a common gene-level
  feature matrix to compute cross-modality cell distances.
when_to_use_negative:
- When the modalities already share the same native feature space (e.g., two scRNA-seq
  datasets), no protein-to-gene conversion is needed
- When no cross-modality feature connections exist at all (e.g., electrophysiology
  measurements vs gene expression), since scConfluence relies on connections between
  features of different modalities
- When the goal is paired multimodal integration (matched cells across modalities)
  rather than diagonal integration of unpaired samples
tools:
- name: scConfluence
  role: Framework whose common-feature (cm_features) representation the mapped gene
    matrices feed into for cross-modality distance computation
  repo: https://github.com/cantinilab/scconfluence
- name: Muon
  role: Handles the multimodal MuData object and per-modality normalization (e.g.,
    Center Log Ratio for protein data) around the mapping step
- name: SciPy
  role: scipy.spatial.distance.cdist computes the cross-modality correlation distance
    matrix on the mapped gene-level features
- name: Scanpy
  role: scRNA-side normalization (normalize_total to 10,000 counts then log1p) so
    gene matrices are comparable before subsetting to shared genes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:c8bbbd424e5b10fc189c739b0660aeb1fc343a9bfc48e7cb84c1c8f314e5bf70
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# protein-to-coding-gene-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts measured surface-protein features (e.g., CITE-seq/CyTOF antibodies, smFISH targets) into their coding genes so protein-based modalities can be expressed on a gene feature space shared with scRNA-seq, enabling cross-modality cell-to-cell comparison in scConfluence.

## When to use

Use when integrating modalities measured on different feature spaces (e.g., scRNA-seq vs surface proteins or smFISH) and you need a common gene-level feature matrix to compute cross-modality cell distances.

## When NOT to use

- When the modalities already share the same native feature space (e.g., two scRNA-seq datasets), no protein-to-gene conversion is needed
- When no cross-modality feature connections exist at all (e.g., electrophysiology measurements vs gene expression), since scConfluence relies on connections between features of different modalities
- When the goal is paired multimodal integration (matched cells across modalities) rather than diagonal integration of unpaired samples

## Inputs

- Preprocessed MuData object with per-modality AnnData (scRNA-seq counts, surface-protein abundances or smFISH measurements)
- Protein-to-gene mapping (e.g., antibody target names resolved to HGNC coding gene symbols)

## Outputs

- Gene-level common features matrices (cm_features) per modality, subset to the intersection of genes shared across modalities
- Cross-modality cost/distance matrix stored as mdata.uns['cross_modality1+modality2'] (shape n_cells_modality1 x n_cells_modality2, finite non-negative correlation distances), registered in mdata.uns['cross_keys']

## How to apply

Starting from the preprocessed MuData object, map each measured surface protein to its coding gene, then subset the scRNA and protein/smFISH matrices to the genes shared across modalities to build the common features matrix (cm_features). Scale the gene-converted matrices so features from different modalities are on a comparable range before comparison. On these connected features, compute the cross-modality distance matrix with scipy.spatial.distance.cdist applied to the modalities' cm_features (Pearson/correlation distance, as scConfluence uses Pearson similarity for the cost matrix), and store it under mdata.uns['cross_modality1+modality2'] with the key registered in mdata.uns['cross_keys']. The rationale is to obtain weakly connected gene-level features that feed the unbalanced Optimal Transport step without forcing a full feature-space conversion, which would lose biological information.

## Related tools

- **scConfluence** (Framework whose common-feature (cm_features) representation the mapped gene matrices feed into for cross-modality distance computation) — https://github.com/cantinilab/scconfluence
- **Muon** (Handles the multimodal MuData object and per-modality normalization (e.g., Center Log Ratio for protein data) around the mapping step)
- **SciPy** (scipy.spatial.distance.cdist computes the cross-modality correlation distance matrix on the mapped gene-level features)
- **Scanpy** (scRNA-side normalization (normalize_total to 10,000 counts then log1p) so gene matrices are comparable before subsetting to shared genes)

## Examples

```
from scipy.spatial.distance import cdist; mdata.uns['scrna+smfish'] = cdist(rna.obsm['cm_features'], fish.obsm['cm_features']); mdata.uns['cross_keys'].append('scrna+smfish')
```

## Evaluation signals

- cost_matrix.npy exists and has shape (n_cells_modality1, n_cells_modality2)
- All entries of the cost matrix are finite and non-negative (valid correlation distances)
- The intersection of mapped protein coding genes and scRNA/smFISH genes is non-empty; both cm_features matrices are subset to exactly the same shared gene set
- Gene-converted matrices from different modalities are scaled to a comparable range before computing the distance matrix
- mdata.uns['cross_keys'] contains the new 'modality1+modality2' key matching the stored distance matrix entry

## Limitations

- Feature conversion across modalities is lossy: cross-modality feature connections are rare and noisy, so converting all modalities to the same features can cause loss of biological information
- Relies on cross-modality feature connections, which are unavailable for some modality pairs (e.g., electrophysiology vs gene expression)
- Rare cell populations are hard to handle and may be under-represented in downstream unbalanced OT estimation, especially with mini-batch optimization

## Evidence

- [methods] Map each measured surface protein to its coding gene, then subset the scRNA and smFISH matrices to the genes shared across modalities to build the common features matrix.: [evidence span withheld — anchor status: not_found]
- [methods] Compute the cross-modality distance matrix on the connected features with scipy.spatial.distance.cdist: [evidence span withheld — anchor status: not_found]
