---
name: single-cell-pseudobulk-aggregation
description: Use when a preprocessed scRNA-seq expression matrix (e.g. H5ad/HDF5/loom)
  must be converted into pseudo-bulk representative cells per cell type/sub-cluster
  before gene co-expression network inference with SCANet/WGCNA.
when_to_use_negative:
- Input is already bulk or pseudo-bulk expression data with no per-cell counts to
  aggregate
- The goal is downstream TF–TG GRN reconstruction (GRNBoost2/RcisTarget) or drug-repurposing
  prediction rather than the upstream pseudo-bulk pre-processing step
- Data are raw unprocessed single-cell counts still requiring QC, normalization, and
  cell-type annotation before aggregation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCANet
  role: Python package implementing the pseudo-bulk aggregation and downstream WGCNA
    co-expression workflow
  repo: https://github.com/oubounyt/SCANet
- name: Scanpy
  role: Loads the preprocessed scRNA-seq expression matrix and cell-level metadata
    for sub-clustering and averaging
- name: BioNERO
  role: Framework through which the aggregated pseudo-bulk matrix is used for WGCNA
    GCN construction
- name: WGCNA
  role: Modified WGCNA algorithm that consumes the pseudo-bulk data for network inference
    and module detection
provenance:
  source_task_ids:
  - task_006
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:f5239006c771a5fe3d1f3f55353a174e70e21caca8af384ee81e38a37b9a8db5
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-pseudobulk-aggregation

## Summary

Aggregates single-cell scRNA-seq read counts into pseudo-bulk representative cells by averaging gene expression within N sub-clusters of each cell type, reducing data dimensionality before co-expression network (GCN) inference with modified WGCNA in SCANet.

## When to use

Use when a preprocessed scRNA-seq expression matrix (e.g. H5ad/HDF5/loom) must be converted into pseudo-bulk representative cells per cell type/sub-cluster before gene co-expression network inference with SCANet/WGCNA.

## When NOT to use

- Input is already bulk or pseudo-bulk expression data with no per-cell counts to aggregate
- The goal is downstream TF–TG GRN reconstruction (GRNBoost2/RcisTarget) or drug-repurposing prediction rather than the upstream pseudo-bulk pre-processing step
- Data are raw unprocessed single-cell counts still requiring QC, normalization, and cell-type annotation before aggregation

## Inputs

- Preprocessed scRNA-seq expression matrix (H5ad/HDF5/loom), rows = cells, columns = genes
- Cell-level metadata (cell type, condition/batch, e.g. diet HFHSD vs control, patient outcome healthy/deceased/surviving)
- Choice of N sub-clusters per cell type

## Outputs

- Pseudo-bulk representative cells (averaged expression profiles per sub-cluster)
- Equalized representative-cell counts per cell type for downstream GCN inference

## How to apply

Load the preprocessed scRNA-seq expression matrix (rows = cells, columns = genes) together with cell-level metadata (cell type, condition, batch) using Scanpy inside SCANet. Sub-cluster the cells of each cell type into N sub-clusters and average the gene expression profiles across all cells in each sub-cluster to create one representative pseudo-bulk cell per sub-cluster — this aggregates read counts for each gene across a designated number of neighboring cells, reducing the data's dimensionality. Downsample so that every cell type is represented by the same number of representative cells, keeping cell-type comparisons balanced. Feed the resulting pseudo-bulk matrix into the WGCNA-based GCN inference (via BioNERO, signed network default) and module detection; success is judged when the downstream GCN yields modules in the scale reported by the paper (30–195 genes per module, e.g. 27 modules for Cillo et al. classical monocytes, 25 modules for Aliluev et al. ISCs) and passes the scale-free topology check.

## Related tools

- **SCANet** (Python package implementing the pseudo-bulk aggregation and downstream WGCNA co-expression workflow) — https://github.com/oubounyt/SCANet
- **Scanpy** (Loads the preprocessed scRNA-seq expression matrix and cell-level metadata for sub-clustering and averaging)
- **BioNERO** (Framework through which the aggregated pseudo-bulk matrix is used for WGCNA GCN construction)
- **WGCNA** (Modified WGCNA algorithm that consumes the pseudo-bulk data for network inference and module detection)

## Examples

```
pip install scanet; import scanet as sn
```

## Evaluation signals

- Every cell type is represented by the same number of representative pseudo-bulk cells after downsampling
- Each representative cell is the averaged gene expression profile of all cells in one sub-cluster of the parent cell type
- Downstream GCN yields co-expression modules comparable in scale to the paper (30–195 genes per module; 27 modules for classical monocytes, 25 for ISCs)
- Detected modules pass the scale-free topology check characteristic of real-world biological networks

## Limitations

- GCNs built from aggregated correlations capture correlation, not causation, potentially leading to false positives in co-expression analysis
- Aggregation depends on prior cell-type annotation and sub-clustering; errors there propagate into the pseudo-bulk profiles
- SCANet officially supports Python 3.6+ only

## Evidence

- [intro] This is achieved by aggregating the read counts for each gene across a designated number of neighboring cells, reducing the data: "This is achieved by aggregating the read counts for each gene across a designated number of neighboring cells, reducing the data"
- [methods] users can identify network hubs by looking for the intersection of the top 10% of the most connected genes in the module: "users can identify network hubs by looking for the intersection of the top 10% of the most connected genes in the module"
- [results] We detected 27 co-expression gene modules of classical monocytes among healthy, deceased, and surviving patients (Fig. 2B) with 30–195 genes per module (M) (Fig. 2C): "We detected 27 co-expression gene modules of classical monocytes among healthy, deceased, and surviving patients (Fig. 2B) with 30–195 genes per module (M) (Fig. 2C)"
- [supplementary] SCANet is publicly available as a Python package on Github: https://github.com/oubounyt/SCANet and pypi: https://pypi.org/project/scanet/: "SCANet is publicly available as a Python package on Github: https://github.com/oubounyt/SCANet and pypi: https://pypi.org/project/scanet/"
- [readme] `SCANet` officially supports Python 3.6+.: "`SCANet` officially supports Python 3.6+."
