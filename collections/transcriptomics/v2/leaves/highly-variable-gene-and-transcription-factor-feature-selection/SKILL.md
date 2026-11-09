---
name: highly-variable-gene-and-transcription-factor-feature-selection
description: Use when constructing a cell-type-specific GRN with scRank and candidate
  genes must be narrowed to highly variable genes, transcription factors, and the
  drug-target gene before network inference.
when_to_use_negative:
- The goal is downstream perturbation scoring (rank_celltype), not network construction
  — this skill only covers the feature-selection/GRN-reconstruction stage.
- The drug has multiple targets, since scRank's multi-target extension is only preliminary
  and requires specifically tailored methodologies not yet developed.
- The input is treated/paired disease-control data or bulk expression rather than
  untreated single-cell transcriptomic data, which is scRank's design scope.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3307
tools:
- name: scRank
  role: Hosts Constr_net, which subsamples cells per cell-type cluster and selects
    features from highly variable genes, transcription factors, and the drug-target
    gene for GRN reconstruction
  repo: https://github.com/ZJUFanLab/scRank
- name: rTensor
  role: R backend for the tensor component analysis that aggregates subsample-level
    networks into one normalized GRN per cell type inside Constr_net
  repo: https://github.com/rikenbit/rTensor
- name: tensorly
  role: Python module optionally integrated into Constr_net (when use_py is enabled)
    as an alternative tensor-decomposition backend
- name: R
  role: Runtime environment (R >4.0) in which the scRank feature-selection and PCR-based
    network construction executes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:ae76fe3a1a95e43732158d73bbc67fedab1d103bcd97bda52a00294c785fc9b3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - rTensor
  - scRank
  - tensorly
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# highly-variable-gene-and-transcription-factor-feature-selection

## Summary

Feature-selection step of scRank's Constr_net stage: cells are subsampled within each cell-type cluster and candidate network nodes are restricted to highly variable genes, transcription factors, and the drug-target gene, so that principal-component regression can infer a tractable cell-type-specific gene-gene adjacency (GRN).

## When to use

Use when constructing a cell-type-specific GRN with scRank and candidate genes must be narrowed to highly variable genes, transcription factors, and the drug-target gene before network inference.

## When NOT to use

- The goal is downstream perturbation scoring (rank_celltype), not network construction — this skill only covers the feature-selection/GRN-reconstruction stage.
- The drug has multiple targets, since scRank's multi-target extension is only preliminary and requires specifically tailored methodologies not yet developed.
- The input is treated/paired disease-control data or bulk expression rather than untreated single-cell transcriptomic data, which is scRank's design scope.

## Inputs

- Seurat object of untreated scRNA-seq data with a cell-type label column (e.g., GSE110894 demo AML_object.rda)
- scRank S4 object created by CreateScRank (species, cell_type, and target gene such as 'Brd4')
- Highly variable gene set and transcription factor list for the chosen species (e.g., mouse)

## Outputs

- Normalized cell-type-specific gene regulatory network as a gene-gene adjacency matrix per cell type
- scRank S4 object saved as grn.rds for use in the downstream rank_celltype step

## How to apply

Inside scRank's Constr_net stage, subsample cells within each cell-type cluster of the input Seurat object (e.g., the GSE110894 AML demo) and assemble the feature set from three node classes: highly variable genes, transcription factors, and the drug-target gene passed via target (e.g., 'Brd4'). Fit principal-component regression on the subsampled expression profiles to derive gene-gene adjacency weights, using the keep_ratio parameter to adjust edge weights for differential treatment of node types. Aggregate the subsample-level networks via tensor component analysis (rTensor in R, or the Python module tensorly when use_py is enabled) to yield one normalized GRN per cell type. The rationale is that restricting the node pool to informative genes keeps co-expression inference tractable while preserving the regulators that define cell-type-intrinsic states, which the paper hypothesizes are reflected by the cell-type-specific GRN. Save the scRank S4 object holding the normalized GRNs as grn.rds for the downstream rank_celltype step.

## Related tools

- **scRank** (Hosts Constr_net, which subsamples cells per cell-type cluster and selects features from highly variable genes, transcription factors, and the drug-target gene for GRN reconstruction) — https://github.com/ZJUFanLab/scRank
- **rTensor** (R backend for the tensor component analysis that aggregates subsample-level networks into one normalized GRN per cell type inside Constr_net) — https://github.com/rikenbit/rTensor
- **tensorly** (Python module optionally integrated into Constr_net (when use_py is enabled) as an alternative tensor-decomposition backend)
- **R** (Runtime environment (R >4.0) in which the scRank feature-selection and PCR-based network construction executes)

## Examples

```
scr <- CreateScRank(seur, species='mouse', cell_type='labels', target='Brd4'); scr <- Constr_net(scr)
```

## Evaluation signals

- Constr_net completes without error and the scRank object contains a normalized gene-gene adjacency matrix for every cell type in the labels column.
- The selected feature set includes the drug-target gene (e.g., Brd4) in addition to highly variable genes and transcription factors.
- keep_ratio was configured so edge weights treat the different node types differentially rather than uniformly.
- The saved grn.rds loads in R and is accepted by the downstream rank_celltype step.
- With use_py enabled, tensorly yields equivalent normalized per-cell-type GRNs to the default rTensor aggregation.

## Limitations

- scRank primarily focuses on drugs acting as inhibitors; agonist modeling rests on a simplifying assumption (setting out-edge weights of the target gene to their maximum value) and tests are preliminary.
- Multi-target perturbation requires specifically tailored methodologies that are not yet developed.
- Adaptability across clustering methodologies is only demonstrated in supplementary benchmark tests (Figure S15).

## Evidence
