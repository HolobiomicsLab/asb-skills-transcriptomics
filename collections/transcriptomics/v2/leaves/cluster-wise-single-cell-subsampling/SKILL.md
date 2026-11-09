---
name: cluster-wise-single-cell-subsampling
description: Use when constructing cell-type-specific gene regulatory networks from
  untreated scRNA-seq expression profiles and single full-cluster inference would
  be unstable — i.e., when subsampled PCR fits must be aggregated across clusters
  before in silico drug-perturbation ranking.
when_to_use_negative:
- The drug acts as an agonist or has an unknown/multi-target mechanism — scRank primarily
  focuses on inhibitor drugs, and agonist modeling relies on a simplifying assumption
  of maximal out-edge weights, while multi-target perturbation requires methodologies
  not yet developed
- Disease-treatment paired datasets are available and simpler DEG-count or separability
  methods (e.g., Augur) would suffice without GRN construction
- The input is not single-cell expression data with cell-type annotations but already
  a feature table or bulk expression matrix
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scRank
  role: Provides the CreateScRank and Constr_net functions that implement cluster-wise
    cell subsampling, feature selection, and PCR-based co-expression network construction
  repo: https://github.com/ZJUFanLab/scRank
- name: rTensor
  role: Performs tensor component analysis to aggregate subsample-level networks into
    one normalized GRN per cell type
  repo: https://github.com/rikenbit/rTensor
- name: tensorly
  role: Alternative Python module for tensor decomposition within Constr_net when
    use_py is enabled
- name: devtools
  role: Used to install the scRank package from GitHub
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:841728a5603e065be3e53936c598bf8f4b7ffb980aeaefe0f3747d7d268cf7b3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - devtools
  - rTensor
  - scRank
  - tensorly
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Cluster-wise single-cell subsampling

## Summary

Within the scRank GRN-construction stage (Constr_net), cells are repeatedly subsampled inside each cell-type cluster and fitted with principal-component regression to derive gene-gene adjacency weights, which are then aggregated into one normalized cell-type-specific GRN via tensor decomposition. This stabilizes co-expression inference on noisy untreated scRNA-seq data instead of relying on a single network fit.

## When to use

Use when constructing cell-type-specific gene regulatory networks from untreated scRNA-seq expression profiles and single full-cluster inference would be unstable — i.e., when subsampled PCR fits must be aggregated across clusters before in silico drug-perturbation ranking.

## When NOT to use

- The drug acts as an agonist or has an unknown/multi-target mechanism — scRank primarily focuses on inhibitor drugs, and agonist modeling relies on a simplifying assumption of maximal out-edge weights, while multi-target perturbation requires methodologies not yet developed
- Disease-treatment paired datasets are available and simpler DEG-count or separability methods (e.g., Augur) would suffice without GRN construction
- The input is not single-cell expression data with cell-type annotations but already a feature table or bulk expression matrix

## Inputs

- Seurat object of untreated single-cell expression data (e.g., GSE110894 AML_object.rda demo)
- Cell-type cluster labels (e.g., 'labels' column)
- Drug target gene name (e.g., 'Brd4')
- Feature sets: highly variable genes, transcription factors, drug-target gene

## Outputs

- One normalized gene-gene adjacency matrix (GRN) per cell type stored in the scRank S4 object
- Saved scRank object (e.g., grn.rds) for downstream rank_celltype analysis

## How to apply

Create the S4 scRank object with CreateScRank (e.g., species = 'mouse', cell_type = 'labels', target = 'Brd4' on the GSE110894 AML Seurat object) and run Constr_net, which subsamples cells within each cell-type cluster and selects features from highly variable genes, transcription factors, and the drug-target gene. Fit principal-component regression on each subsampled expression profile to obtain gene-gene adjacency weights, using the keep_ratio parameter to adjust edge weights for differential treatment of node types. Aggregate the subsample-level networks via tensor component analysis with rTensor in R, or the Python module tensorly when use_py is enabled, yielding one normalized GRN per cell type. Save the scRank object holding these normalized cell-type-specific GRNs (e.g., grn.rds) for the downstream rank_celltype step.

## Related tools

- **scRank** (Provides the CreateScRank and Constr_net functions that implement cluster-wise cell subsampling, feature selection, and PCR-based co-expression network construction) — https://github.com/ZJUFanLab/scRank
- **rTensor** (Performs tensor component analysis to aggregate subsample-level networks into one normalized GRN per cell type) — https://github.com/rikenbit/rTensor
- **tensorly** (Alternative Python module for tensor decomposition within Constr_net when use_py is enabled)
- **devtools** (Used to install the scRank package from GitHub)

## Examples

```
obj <- CreateScRank(aml_obj, species='mouse', cell_type='labels', target='Brd4'); obj <- Constr_net(obj); saveRDS(obj, 'grn.rds')
```

## Evaluation signals

- Each cell type yields exactly one normalized GRN adjacency matrix after tensor aggregation of the subsample-level PCR networks
- Feature selection within each subsample includes highly variable genes, transcription factors, and the drug-target gene (e.g., Brd4 in the GSE110894 demo)
- keep_ratio parameter changes are reflected in differential edge weights for node types in the resulting adjacency matrices
- Downstream rank_celltype runs without error on the saved scRank object, confirming the GRNs are complete and well-formed
- Switching between rTensor (R) and tensorly (Python, use_py = TRUE) produces comparable normalized GRNs

## Limitations

- scRank primarily focuses on drugs acting as inhibitors, partly due to the nature of the available data; agonist tests are preliminary only
- Agonist perturbation uses a simplifying assumption — setting out-edge weights of the target gene node to their maximum possible value
- Multi-target perturbation requires specifically tailored methodologies not yet developed, with only preliminary results shown (Figure S17)
- The tensor decomposition backend depends on rTensor or, optionally, the Python tensorly module, adding an installation dependency

## Evidence
