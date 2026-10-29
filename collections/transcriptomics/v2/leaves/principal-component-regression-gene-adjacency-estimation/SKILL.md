---
name: principal-component-regression-gene-adjacency-estimation
description: Use when constructing cell-type-specific gene regulatory networks from
  untreated scRNA-seq expression profiles as the network-construction stage of scRank,
  prior to in silico drug-perturbation ranking.
when_to_use_negative:
- Data come from drug-treated samples rather than untreated disease-condition scRNA-seq,
  since scRank's tpGRN design targets untreated datasets
- The goal is de novo GRN inference unrelated to drug-target perturbation (no drug
  target gene specified), as the network is built around highly variable genes, TFs,
  and the target gene
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: scRank
  role: Provides the CreateScRank and Constr_net functions implementing random sampling,
    principal-component regression edge fitting, and keep_ratio-weighted GRN construction
  repo: https://github.com/ZJUFanLab/scRank
- name: rTensor
  role: R package used for tensor component analysis aggregating subsample-level adjacency
    networks into one GRN per cell type
  repo: https://github.com/rikenbit/rTensor
- name: tensorly
  role: Optional Python module integrated in Constr_net() to perform the tensor decomposition
    when use_py is enabled
- name: R
  role: Runtime environment for the scRank pipeline (R >= 4.0 required per scRank
    README)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:06f9310f567026636594d928662ef22381db8edc50838d9eab405b231b6a5017
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - rTensor
  - scRank
  - tensorly
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# principal-component-regression-gene-adjacency-estimation

## Summary

Estimates gene-gene co-expression adjacency weights by fitting principal-component regression with a random sampling strategy inside scRank's Constr_net step, then aggregates subsample-level networks via tensor decomposition (rTensor or Python tensorly) into one normalized cell-type-specific GRN.

## When to use

Use when constructing cell-type-specific gene regulatory networks from untreated scRNA-seq expression profiles as the network-construction stage of scRank, prior to in silico drug-perturbation ranking.

## When NOT to use

- Data come from drug-treated samples rather than untreated disease-condition scRNA-seq, since scRank's tpGRN design targets untreated datasets
- The goal is de novo GRN inference unrelated to drug-target perturbation (no drug target gene specified), as the network is built around highly variable genes, TFs, and the target gene

## Inputs

- Seurat object of untreated single-cell expression data (e.g., GSE110894 AML_object.rda)
- Cell-type cluster labels (e.g., 'labels' metadata column)
- Drug-target gene identifier (e.g., Brd4)
- Highly variable genes and transcription factor feature sets

## Outputs

- One normalized gene-gene adjacency matrix (cell-type-specific GRN) per cell type
- scRank S4 object with reconstructed GRNs, stored as grn.rds

## How to apply

Load a Seurat object (e.g., the GSE110894 AML demo, AML_object.rda) and wrap it in an S4 scRank object with CreateScRank, specifying species, the cell-type metadata column, and the drug-target gene (e.g., Brd4). Run Constr_net, which subsamples cells within each cell-type cluster and selects features from highly variable genes, transcription factors, and the drug-target gene. Fit principal-component regression on the subsampled expression profiles to derive gene-gene adjacency weights, using the keep_ratio parameter to adjust edge weights and treat node types differentially. Aggregate the subsample-level networks via tensor component analysis (rTensor, or the Python module tensorly when use_py is enabled) into one normalized GRN per cell type, and save the object as grn.rds for downstream rank_celltype analysis.

## Related tools

- **scRank** (Provides the CreateScRank and Constr_net functions implementing random sampling, principal-component regression edge fitting, and keep_ratio-weighted GRN construction) — https://github.com/ZJUFanLab/scRank
- **rTensor** (R package used for tensor component analysis aggregating subsample-level adjacency networks into one GRN per cell type) — https://github.com/rikenbit/rTensor
- **tensorly** (Optional Python module integrated in Constr_net() to perform the tensor decomposition when use_py is enabled)
- **R** (Runtime environment for the scRank pipeline (R >= 4.0 required per scRank README))

## Examples

```
library(scRank); obj <- CreateScRank(seurat_obj, species='mouse', cell_type='labels', target='Brd4'); obj <- Constr_net(obj); saveRDS(obj, 'grn.rds')
```

## Evaluation signals

- Constr_net completes without error for every cell-type cluster and produces a finite gene-gene adjacency matrix
- The tensor decomposition aggregates all subsample-level networks into a single normalized GRN per cell type (values bounded/normalized)
- The drug-target gene (e.g., Brd4) is retained as a network node alongside highly variable genes and transcription factors
- The saved grn.rds S4 object can be loaded and passed directly to rank_celltype
- Adjusted keep_ratio values visibly change edge weights, reflecting differential treatment of node types

## Limitations

- Results depend on subsample quality; the random sampling strategy means subsampling choices influence the inferred adjacency weights
- Tensor decomposition requires rTensor installed (or Python tensorly when use_py is enabled), adding a dependency step via devtools

## Evidence
