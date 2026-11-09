---
name: wgcna-coexpression-network-inference
description: Use when the input is a preprocessed single-cell expression matrix (e.g.
  .h5ad/HDF5/loom) with cell metadata and the goal is to infer gene co-expression
  modules and hubs for specific cell types or conditions.
when_to_use_negative:
- The goal is directed gene regulatory network reconstruction rather than co-expression
  — GCNs rely on correlation, not causation; use downstream GRN inference (e.g. GRNBoost2
  + RcisTarget) instead
- Data are already bulk RNA-seq or an aggregated feature table — pseudo-bulk sub-clustering
  of single cells is unnecessary
- A full-connection co-expression method specific to single cells is required and
  other dedicated tools (COTAN, scLink, fcoex) may be more appropriate
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCANet
  role: All-in-one Python package orchestrating pseudo-bulk pre-processing, GCN inference,
    module detection, hub identification, and scale-free topology checking
  repo: https://github.com/oubounyt/SCANet
- name: WGCNA
  role: Modified weighted gene co-expression network analysis used by SCANet for network
    inference and module detection
- name: BioNERO
  role: Framework through which WGCNA-based GCN inference and module detection is
    executed within SCANet
- name: Scanpy
  role: Loading the preprocessed scRNA-seq expression matrix (H5ad/HDF5/loom) with
    cell-level metadata
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:3eae3ff5cfebb739dfa93a22035d0952f1738f54c14bd4a019f23a872a0b74d3
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
  - WGCNA
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# wgcna-coexpression-network-inference

## Summary

Infer gene co-expression networks (GCNs) and detect co-expression gene modules from scRNA-seq data using a modified WGCNA implementation (via BioNERO) inside SCANet, after pseudo-bulk aggregation of cells. This converts high-dimensional single-cell expression into trait/cell-type-associated gene modules.

## When to use

Use when the input is a preprocessed single-cell expression matrix (e.g. .h5ad/HDF5/loom) with cell metadata and the goal is to infer gene co-expression modules and hubs for specific cell types or conditions.

## When NOT to use

- The goal is directed gene regulatory network reconstruction rather than co-expression — GCNs rely on correlation, not causation; use downstream GRN inference (e.g. GRNBoost2 + RcisTarget) instead
- Data are already bulk RNA-seq or an aggregated feature table — pseudo-bulk sub-clustering of single cells is unnecessary
- A full-connection co-expression method specific to single cells is required and other dedicated tools (COTAN, scLink, fcoex) may be more appropriate

## Inputs

- Processed scRNA-seq expression matrix (H5ad/HDF5/loom) with cell-level metadata (cell type, condition, batch), e.g. Aliluev et al. 2021 enteroendocrine data (GSE180578 for Cillo et al. COVID-19 processed data)
- Cell-type annotations / trait metadata for module-trait correlation

## Outputs

- Gene co-expression network (GCN) built on a signed adjacency matrix
- Co-expression gene modules (30–195 genes per module in published runs; 25 modules in the ISC analysis, 27 in classical monocytes)
- Network hubs per module (top 10% most connected genes)
- Module–trait correlation results (e.g. M4 vs ISC in HFHSD, M11 vs deceased patients)

## How to apply

Load the processed scRNA-seq expression matrix with cell-level metadata (cell type, condition, batch) using Scanpy through SCANet, then pseudo-bulk the data by sub-clustering each cell type into N sub-clusters and averaging gene expression across cells in each sub-cluster so every cell type is equally represented. Build the pairwise gene-gene correlation matrix (Pearson's r, Spearman's, or biweight midcorrelation) and transform it into a signed adjacency matrix with the modified WGCNA implementation via the BioNERO framework in SCANet. Detect co-expression gene modules from the resulting GCN, identify network hubs as the intersection of the top 10% most connected genes within each module, and correlate modules against cell-level metadata to select trait/cell-type-associated modules (e.g. M4 for ISCs in HFHSD mice, M11 for deceased COVID-19 patients). Assess whether the module graph fits the scale-free topology characteristic of real-world biological networks as a quality check.

## Related tools

- **SCANet** (All-in-one Python package orchestrating pseudo-bulk pre-processing, GCN inference, module detection, hub identification, and scale-free topology checking) — https://github.com/oubounyt/SCANet
- **WGCNA** (Modified weighted gene co-expression network analysis used by SCANet for network inference and module detection)
- **BioNERO** (Framework through which WGCNA-based GCN inference and module detection is executed within SCANet)
- **Scanpy** (Loading the preprocessed scRNA-seq expression matrix (H5ad/HDF5/loom) with cell-level metadata)

## Examples

```
pip install scanet; import scanet as sn
```

## Evaluation signals

- The inferred GCN yields co-expression modules comparable in scale to the 30–195 genes per module reported for classical monocytes (e.g. 25 modules in the ISC run, 27 in monocytes)
- The module graph passes the scale-free topology check, a characteristic of real-world biological networks
- Network hubs are recoverable as the intersection of the top 10% most connected genes within each module
- Selected modules show expected trait correlations (e.g. M4 most strongly correlated with ISCs in HFHSD mice; M11 highest positive correlation with deceased patients)
- Within-module gene correlations reach the expected magnitudes (e.g. 16 of 90 genes in M11 with correlation ≥ 0.9; 11 of 137 genes in M21 with correlation = 1.0)

## Limitations

- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis
- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected
- Downsampling to equal representative cells per cell type discards some single-cell resolution; module counts and sizes depend on the chosen sub-clustering granularity

## Evidence

- [intro] SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA): "SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection"
- [intro] Pseudo-bulk aggregation step: "This is achieved by aggregating the read counts for each gene across a designated number of neighboring cells, reducing the data"
- [methods] Scale-free topology and hub checks: "They can also check whether the module graph fits the scale-free topology, a characteristic of real-world biological networks"
- [methods] Hub identification definition: "users can identify network hubs by looking for the intersection of the top 10% of the most connected genes in the module"
- [results] Module scale benchmark: "We detected 27 co-expression gene modules of classical monocytes among healthy, deceased, and surviving patients (Fig. 2B) with 30–195 genes per module"
- [discussion] Correlation-not-causation caveat: "GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis"
- [supplementary] Python package availability: "SCANet is publicly available as a Python package on Github: https://github.com/oubounyt/SCANet and pypi: https://pypi.org/project/scanet/"
