---
name: peak-to-gene-activity-conversion
description: 'Use when integrating scATAC (peaks) with transcriptomic modalities (scRNA/smFISH)
  in scConfluence: chromatin peaks must first be converted to gene activity scores
  to build the cross-modality common features matrix before computing the correlation-based
  cost matrix.'
when_to_use_negative:
- When the modality pair has no weakly connected features (e.g., electrophysiology
  vs gene expression), since scConfluence relies on cross-modality feature connections
- When the scATAC data is already provided as gene activity scores (e.g., the Open
  problems Multiome dataset, where gene activities were already computed by the authors
  with Signac)
- When doing paired/multiomic integration on the same cells; scConfluence targets
  unpaired (diagonal) integration
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: scConfluence
  role: Consumes the common features matrix to compute the cross-modality cost matrix
    and unbalanced OT plan
  repo: https://github.com/cantinilab/scconfluence
- name: Signac
  role: Computes gene activity scores from scATAC peaks (as done for the Open Problems
    Multiome dataset)
- name: Muon
  role: TF-IDF normalization of scATAC cells and storage of results in the MuData
    object
- name: SciPy
  role: scipy.spatial.distance.cdist computes the cross-modality distance matrix on
    cm_features
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:33dff836d1aa875f9cc6da3e5dae2d4b3f897447463b22cac1dec890fe9475ab
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# peak-to-gene-activity-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts scATAC chromatin peak measurements into gene-level activity scores (e.g., with Signac) so that scATAC cells can be compared on shared gene features with scRNA/smFISH cells, enabling scConfluence's cross-modality common features matrix Y and correlation-based biological cost matrix.

## When to use

Use when integrating scATAC (peaks) with transcriptomic modalities (scRNA/smFISH) in scConfluence: chromatin peaks must first be converted to gene activity scores to build the cross-modality common features matrix before computing the correlation-based cost matrix.

## When NOT to use

- When the modality pair has no weakly connected features (e.g., electrophysiology vs gene expression), since scConfluence relies on cross-modality feature connections
- When the scATAC data is already provided as gene activity scores (e.g., the Open problems Multiome dataset, where gene activities were already computed by the authors with Signac)
- When doing paired/multiomic integration on the same cells; scConfluence targets unpaired (diagonal) integration

## Inputs

- Preprocessed MuData object with a QC-filtered, TF-IDF-normalized scATAC peak matrix
- scRNA count matrix (normalized/log1p, gene-subset)
- smFISH or surface-protein count matrix with protein-to-coding-gene mapping

## Outputs

- Gene-level activity matrix per scATAC cell (common features matrix Y)
- Scaled gene-converted cm_features matrices per modality
- Cross-modality distance/cost matrix stored in mdata.uns['cross_modality1+modality2'] with key registered in mdata.uns['cross_keys']

## How to apply

Start from a preprocessed MuData object whose scATAC modality has been QC-filtered and TF-IDF normalized with Muon's tfidf. Compute gene activities from the peaks (for the Open Problems Multiome dataset this was done with Signac). For other modality pairs, map each measured surface protein to its coding gene, then subset the scRNA and smFISH matrices to the genes shared across modalities to form the common features matrix. Scale the gene-converted matrices so features from different modalities are on a comparable range. Compute the cross-modality distance with scipy.spatial.distance.cdist on the modalities' cm_features, store it under mdata.uns['cross_modality1+modality2'], and register the key in mdata.uns['cross_keys']; scConfluence then uses this cost matrix (Pearson similarity) to find an unbalanced Optimal Transport plan.

## Related tools

- **scConfluence** (Consumes the common features matrix to compute the cross-modality cost matrix and unbalanced OT plan) — https://github.com/cantinilab/scconfluence
- **Signac** (Computes gene activity scores from scATAC peaks (as done for the Open Problems Multiome dataset))
- **Muon** (TF-IDF normalization of scATAC cells and storage of results in the MuData object)
- **SciPy** (scipy.spatial.distance.cdist computes the cross-modality distance matrix on cm_features)

## Examples

```
from scipy.spatial.distance import cdist; D = cdist(mdata['modality1'].obsm['cm_features'], mdata['modality2'].obsm['cm_features'], metric='correlation'); mdata.uns['cross_modality1+modality2'] = D
```

## Evaluation signals

- cost_matrix.npy exists and has shape (n_cells_modality1, n_cells_modality2)
- All entries of the cost matrix are finite and non-negative correlation-distance values
- The scATAC and transcriptomic matrices are subset to exactly the same set of shared genes before cdist
- mdata.uns['cross_keys'] contains the registered key 'cross_modality1+modality2' matching the stored matrix
- Gene-converted matrices were scaled so features from different modalities are on a comparable range

## Limitations

- Feature conversion across modalities is rare and noisy and can cause loss of biological information; scConfluence mitigates but relies on these weakly connected features
- Cross-modality feature connections are unavailable for some modality pairs (e.g., electrophysiology vs gene expression)
- Rare cell populations are hard to handle and under-represented in mini-batch optimization

## Evidence

- [discussion] TF-IDF normalization of scATAC cells: [evidence span withheld — anchor status: not_found]
