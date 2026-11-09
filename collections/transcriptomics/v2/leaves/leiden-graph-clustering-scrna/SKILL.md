---
name: leiden-graph-clustering-scrna
description: Use when a single-cell dataset (scRNA-seq or scATAC-seq cell-by-gene
  matrix in AnnData format) must be partitioned into cell clusters after dimensionality
  reduction, e.g. to reproduce Cellar's reported cluster counts.
when_to_use_negative:
- The input is already a 2D embedding or a precomputed neighbors graph rather than
  a raw/filterable count matrix requiring scanpy preprocessing
- Cell-type labels exist and the goal is label transfer (use Cellar's supervised label
  transfer / scanpy Ingest instead of de novo Leiden clustering)
- The data are spatial proteomics/CODEX images where dimensionality reduction parameters
  differ (e.g. UMAP with 10 dimensions for lymph node CODEX data)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Cellar
  role: Interactive tool wrapping the full QC → normalization → PCA → faiss neighbors
    → Leiden clustering workflow; writes assignments to AnnData session files
  repo: https://github.com/euxhenh/cellar
- name: scanpy
  role: Performs data preprocessing (cell/gene filtering, CPM normalization, log1p,
    scaling) prior to clustering
- name: scikit-learn
  role: Provides the PCA implementation with randomized SVD solver that generates
    the 40-dim embedding fed into the neighbors graph
- name: faiss
  role: Builds the approximate nearest-neighbors graph (15 neighbors) that speeds
    up neighbors graph construction for Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: AnnData
  role: Stores the matrix and cluster labels (adata.obs['labels']) in memory-mapping
    mode for large datasets
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:ac570aca2ad01030766253c630ef13199ca049c350760ac4886de1f15e64e273
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/euxhenh/cellar
  tools:
  - AnnData
  - Cellar
  - diffxpy
  - faiss
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Leiden graph-based clustering of single-cell data (Cellar pipeline)

## Summary

Clusters single-cell data by building a 15-nearest-neighbor approximate neighbors graph on PCA embeddings (or cell-by-gene representations) with faiss and running Leiden community detection at resolution 1, as implemented in Cellar for HuBMAP scRNA-seq and scATAC-seq datasets.

## When to use

Use when a single-cell dataset (scRNA-seq or scATAC-seq cell-by-gene matrix in AnnData format) must be partitioned into cell clusters after dimensionality reduction, e.g. to reproduce Cellar's reported cluster counts.

## When NOT to use

- The input is already a 2D embedding or a precomputed neighbors graph rather than a raw/filterable count matrix requiring scanpy preprocessing
- Cell-type labels exist and the goal is label transfer (use Cellar's supervised label transfer / scanpy Ingest instead of de novo Leiden clustering)
- The data are spatial proteomics/CODEX images where dimensionality reduction parameters differ (e.g. UMAP with 10 dimensions for lymph node CODEX data)

## Inputs

- Cell-by-gene count matrix in AnnData (.h5ad) format, e.g. the 5273-cell HBMP3-spleen-CC2 HuBMAP spleen dataset
- Gene activity score matrix converted from scATAC-seq (e.g. PBMC 10k cell-by-gene activity scores)
- QC-filtered matrix (cells with 50–3000 expressed genes; genes expressed in 50–3000 cells)

## Outputs

- Cluster labels per cell in adata.obs['labels'] (Leiden communities at resolution 1)
- UMAP 2D embedding of the 40-dimensional PCA representation for visualization
- Per-cluster cell-count table exported from the AnnData session file

## How to apply

First normalize the count matrix to CPM (total count = 1e5), log1p-transform, and scale to zero mean and unit variance via scanpy. Reduce to 40 dimensions with scikit-learn PCA using the randomized SVD solver, then construct an approximate nearest-neighbors graph with faiss using 15 neighbors. Run Leiden community detection at resolution 1 on the neighbors graph and write cluster assignments to adata.obs['labels']. The same recipe applies to scATAC-seq cell-by-gene activity matrices before downstream differential expression with diffxpy; adjust resolution if a reference cluster count (e.g. 16 clusters for HBMP3-spleen-CC2) is not matched.

## Related tools

- **Cellar** (Interactive tool wrapping the full QC → normalization → PCA → faiss neighbors → Leiden clustering workflow; writes assignments to AnnData session files) — https://github.com/euxhenh/cellar
- **scanpy** (Performs data preprocessing (cell/gene filtering, CPM normalization, log1p, scaling) prior to clustering)
- **scikit-learn** (Provides the PCA implementation with randomized SVD solver that generates the 40-dim embedding fed into the neighbors graph)
- **faiss** (Builds the approximate nearest-neighbors graph (15 neighbors) that speeds up neighbors graph construction for Leiden clustering) — https://github.com/facebookresearch/faiss
- **AnnData** (Stores the matrix and cluster labels (adata.obs['labels']) in memory-mapping mode for large datasets)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('HBMP3-spleen-CC2.h5ad'); sc.pp.pca(adata, n_comps=40, svd_solver='randomized'); sc.pp.neighbors(adata, n_neighbors=15); sc.tl.leiden(adata, resolution=1, key_added='labels')
```

## Evaluation signals

- Output AnnData contains assignments for all input cells (e.g. exactly 5273 for HBMP3-spleen-CC2)
- Leiden at resolution 1 on PCA+faiss graph reproduces 16 clusters for HBMP3-spleen-CC2, matching Supplementary Fig. 1a
- Cluster structure on the PBMC 10k scATAC-seq cell-by-gene representation yields clusters (0 and 4) whose top DE markers include KLRD1 (NK cells), consistent with Supplementary Fig. 3
- Preceding QC and normalization were applied with the specified thresholds (cells 50–3000 genes; genes in 50–3000 cells; CPM total count 1e5; unit variance, zero mean)
- Neighbors graph was built with 15 neighbors via faiss and PCA retained 40 dimensions with the randomized SVD solver

## Limitations

- Cluster number at resolution 1 is data-dependent; if a reference cluster count (e.g. 16) is not reproduced, the resolution or QC thresholds may need adjustment
- Cellar relies on an external web server (cellar.cmu.hubmapconsortium.org) rather than a locally installable archived artifact, complicating version-locked reproduction; no software versions stated for Cellar, Python, Dash, or faiss
- Approximate (faiss) nearest neighbors trades exactness for speed, so the neighbors graph — and hence cluster boundaries — may differ slightly from exact kNN results

## Evidence

- [discussion] approximate nearest neighbors based on faiss to speed up neighbors graph construction for Leiden clustering: "approximate nearest neighbors based on faiss25 to speed up neighbors graph construction for Leiden clustering"
- [methods] The embeddings were then used to construct an approximate neighbors graph using faiss with 15 neighbors: [evidence span withheld — anchor status: not_found]
- [methods] we filtered cells with less than 50 or more than 3000 expressed genes: "we filtered cells with less than 50 or more than 3000 expressed genes"
- [discussion] Cellar is written in Python using the Dash framework: "Cellar is written in Python using the Dash framework"
- [readme] Faiss is a library for efficient similarity search and clustering of dense vectors.: [evidence span withheld — anchor status: external_doc]
