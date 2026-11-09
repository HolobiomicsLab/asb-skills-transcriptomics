---
name: single-cell-expression-matrix-preprocessing
description: Use when raw single-cell read counts (e.g. an .h5ad/HDF5/loom matrix
  from scRNA-seq) must be reduced into pseudo-bulk representative cells before GCN
  inference, because single-cell expression is too sparse for WGCNA-style correlation.
when_to_use_negative:
- The input is already bulk RNA-seq or an aggregated expression matrix — aggregation
  across neighbors would be redundant
- The goal is cell clustering, annotation, or trajectory analysis rather than co-expression
  network inference on pseudo-bulk profiles
- The expression matrix is already a gene-gene adjacency or module assignment table,
  i.e. downstream of this preprocessing stage
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCANet
  role: Python package that performs the pseudo-bulk pre-processing by aggregating
    read counts across neighboring cells before modified-WGCNA GCN inference
  repo: https://github.com/oubounyt/SCANet
- name: Scanpy
  role: Loads the preprocessed scRNA-seq expression matrix (H5ad/HDF5/loom) with cell-level
    metadata into the workflow
- name: Python
  role: Runtime environment; SCANet officially supports Python 3.6+ and is installed
    via pip
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:1f2a996fcd2c303e5a5b6536605459a4208866d6a445c27506a32aa75877227a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - BioNERO
  - GRNBoost2
  - Python
  - RcisTarget
  - SCANet
  - Scanpy
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-expression-matrix-preprocessing

## Summary

Pre-process single-cell scRNA-seq read counts for co-expression network inference by sub-clustering cells and aggregating read counts into pseudo-bulk representative cells, the preprocessing stage SCANet performs before its modified-WGCNA GCN inference.

## When to use

Use when raw single-cell read counts (e.g. an .h5ad/HDF5/loom matrix from scRNA-seq) must be reduced into pseudo-bulk representative cells before GCN inference, because single-cell expression is too sparse for WGCNA-style correlation.

## When NOT to use

- The input is already bulk RNA-seq or an aggregated expression matrix — aggregation across neighbors would be redundant
- The goal is cell clustering, annotation, or trajectory analysis rather than co-expression network inference on pseudo-bulk profiles
- The expression matrix is already a gene-gene adjacency or module assignment table, i.e. downstream of this preprocessing stage

## Inputs

- Processed scRNA-seq expression matrix (H5ad/HDF5/loom) with rows = cells, columns = genes
- Cell-level metadata (cell type, diet/condition, batch)

## Outputs

- Pseudo-bulk representative-cell expression matrix with equal representation per cell type
- Pseudo-bulk matrix with retained cell-type/trait metadata for module–trait correlation

## How to apply

Load the preprocessed scRNA-seq expression matrix (rows = cells, columns = genes) with cell-level metadata (cell type, condition, batch) using Scanpy inside SCANet. Group the cells of each cell type into N sub-clusters and average gene expression profiles across all cells within each sub-cluster, aggregating read counts for each gene across a designated number of neighboring cells. Down-sample so that every cell type is represented by the same number of representative pseudo-bulk cells, which balances the input for downstream correlation-based network construction (Pearson, Spearman, or biweight midcorrelation within BioNERO/WGCNA). Keep the metadata attached so the pseudo-bulk cells can later be correlated against traits such as HFHSD versus control diet or deceased versus surviving patients.

## Related tools

- **SCANet** (Python package that performs the pseudo-bulk pre-processing by aggregating read counts across neighboring cells before modified-WGCNA GCN inference) — https://github.com/oubounyt/SCANet
- **Scanpy** (Loads the preprocessed scRNA-seq expression matrix (H5ad/HDF5/loom) with cell-level metadata into the workflow)
- **Python** (Runtime environment; SCANet officially supports Python 3.6+ and is installed via pip)

## Examples

```
pip install scanet; import scanet as sn
```

## Evaluation signals

- The pseudo-bulk matrix contains the same number of representative cells per cell type after down-sampling
- Downstream GCN yields gene co-expression modules of comparable scale to the 30–195 genes per module reported for classical monocytes
- The inferred module graphs pass the scale-free topology check characteristic of real-world biological networks
- Cell-level metadata (e.g. diet condition, patient outcome) is preserved so modules can be correlated against traits

## Limitations

- Aggregating read counts across neighboring cells reduces the data, discarding single-cell resolution
- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis
- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected

## Evidence

- [intro] Pre-processing single-cell data by creating pseudo-bulk RNA-seq via aggregating read counts across neighboring cells: "This is achieved by aggregating the read counts for each gene across a designated number of neighboring cells, reducing the data"
- [readme] Workflow: load expression matrix with Scanpy through SCANet keeping cell-level metadata: "we explored all aspects of gene coexpression networks (GCNs) using `SCANet` through a full analysis of the 3k PBMCs from 10x Genomics"
- [results] Downstream GCN inference on processed scRNA-seq data: "using the processed scRNA-seq data from Cillo et al. (2021) (Fig. 2A)"
- [supplementary] SCANet availability and installation: "SCANet is publicly available as a Python package on Github: https://github.com/oubounyt/SCANet"
