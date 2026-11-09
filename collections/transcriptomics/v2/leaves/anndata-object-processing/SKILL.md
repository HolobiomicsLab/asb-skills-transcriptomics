---
name: anndata-object-processing
description: Use when the analysis input is or should be a Scanpy AnnData object (e.g.
  scanpyobj.h5ad) that must be loaded, sub-clustered, or transformed (log2 fold change
  vs. average expression) to feed CaDRReS-Sc or PRISM drug response prediction.
when_to_use_negative:
- Input is raw 10x mtx/CSV data needing full preprocessing from scratch rather than
  an existing AnnData object
- Data is bulk RNA-seq expression — CaDRReS-Sc's per-cell log2 fold-change features
  require single-cell AnnData input
- Clusters are not annotated in adata.obs (e.g. no 'louvain' column), making --clusters
  sub-clustering impossible without prior clustering
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDrug
  role: Provides single_cell_analysis.py and drug_response_prediction.py which consume
    and produce AnnData (.h5ad) objects and compute log2 fold-change features
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Underlying library performing data preprocessing, clustering, and AnnData
    object handling in the scDrug pipeline
- name: CaDRReS-Sc
  role: Consumes the AnnData-derived log2 fold-change expression matrix to predict
    IC50 / sensitivity per cell and per cluster
  repo: https://github.com/CSB5/CaDRReS-SC
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:11991e77257095ff9d85dd4798bd56f2124f59e29adb0c89cd9fdc6536fd4e2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CaDRReS-Sc
  - Python
  - Scanpy
  - scDrug
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# anndata-object-processing

## Summary

Handles Scanpy AnnData (.h5ad) objects within the scDrug workflow: loading scanpyobj.h5ad, computing log2 fold-change expression features against average gene expression, and optionally sub-clustering specified Louvain clusters before downstream drug response prediction.

## When to use

Use when the analysis input is or should be a Scanpy AnnData object (e.g. scanpyobj.h5ad) that must be loaded, sub-clustered, or transformed (log2 fold change vs. average expression) to feed CaDRReS-Sc or PRISM drug response prediction.

## When NOT to use

- Input is raw 10x mtx/CSV data needing full preprocessing from scratch rather than an existing AnnData object
- Data is bulk RNA-seq expression — CaDRReS-Sc's per-cell log2 fold-change features require single-cell AnnData input
- Clusters are not annotated in adata.obs (e.g. no 'louvain' column), making --clusters sub-clustering impossible without prior clustering

## Inputs

- Scanpy AnnData object file (scanpyobj.h5ad)
- 10x-Genomics-formatted mtx directory or CSV file convertible into AnnData by Single-Cell Data Analysis
- Cluster specification string (e.g. '1,3,8,9') and obs column name (default 'louvain') for sub-clustering

## Outputs

- scanpyobj.h5ad AnnData object with preprocessing, clustering and metadata
- Sub-clustered AnnData when --clusters is supplied
- cluster_DEGs.csv (differentially expressed genes) and umap_cluster.png from Single-Cell Data Analysis
- Expression matrices ready for IC50_prediction.csv / drug_kill_prediction.csv (GDSC) or AUC_prediction.csv (PRISM) prediction

## How to apply

Run scDrug's single_cell_analysis.py with -f h5ad --input scanpyobj.h5ad and, when targeting tumor subpopulations, pass --clusters (e.g. '1,3,8,9') with --cname 'louvain' to sub-cluster only those groups, optionally with --auto-resolution to pick a resolution in [0.4, 1.4]. The produced AnnData object is the required input format for drug_response_prediction.py, which computes each cell's log2 fold change with respect to the average gene expression values of the AnnData object before applying the CaDRReS-Sc pre-trained GDSC model (10-dimensional latent space) or the PRISM model (140-dimensional latent space). Quality-filtered, normalized AnnData (cells >200 genes, mitochondrial fraction <30%, normalized to 10,000 counts and log-transformed) is assumed. Validation: the h5ad loads in Scanpy, contains cluster annotations in adata.obs under the louvain key, and downstream prediction writes the expected CSVs (IC50_prediction.csv, drug_kill_prediction.csv, or AUC_prediction.csv) covering all clusters.

## Related tools

- **scDrug** (Provides single_cell_analysis.py and drug_response_prediction.py which consume and produce AnnData (.h5ad) objects and compute log2 fold-change features) — https://github.com/ailabstw/scDrug
- **Scanpy** (Underlying library performing data preprocessing, clustering, and AnnData object handling in the scDrug pipeline)
- **CaDRReS-Sc** (Consumes the AnnData-derived log2 fold-change expression matrix to predict IC50 / sensitivity per cell and per cluster) — https://github.com/CSB5/CaDRReS-SC

## Examples

```
python3 single_cell_analysis.py -f h5ad --input scanpyobj.h5ad --clusters 1,3,8,9 --auto-resolution --cpus 4
```

## Evaluation signals

- scanpyobj.h5ad exists, loads with Scanpy, and contains a louvain cluster annotation in adata.obs
- When sub-clustering, output AnnData contains only cells from the specified clusters (e.g. '1,3,8,9')
- Per-cell log2 fold-change features are computed relative to average gene expression of the AnnData object before model prediction
- Downstream drug_response_prediction.py runs without format errors and writes prediction CSVs (IC50_prediction.csv, drug_kill_prediction.csv, or AUC_prediction.csv) covering all clusters
- Preprocessing invariants hold: cells with <200 genes filtered, genes in <3 cells filtered, mitochondrial fraction <30%, normalized to 10,000 counts

## Limitations

- Sub-clustering requires previously computed cluster labels; a --cname column must exist and be named explicitly if not 'louvain'
- scDrug's Single-Cell Data Analysis must have produced the AnnData; arbitrary h5ad files may lack expected keys or preprocessing
- Prediction limited to experimentally screened compounds (GDSC/PRISM) from the AnnData-derived features

## Evidence

- [readme] h5ad (Anndata object for subclustering with --clusters CLUSTERS): "Single-Cell Data Analysis could take previously produced Anndata as input and applied sub-clustering on specified clusters."
- [methods] Compute the log2 fold change of each cell's gene expression with respect to the average gene expression values: "we first constructed the workflow, the scRNA-seq analysis pipeline, for a comprehensive analysis of the scRNA-seq data"
- [intro] drug response prediction via CaDRReS-Sc on AnnData input: "recent studies have leveraged scRNA-seq and bulk transcriptomic profiles to predict drug response for heterogeneous tumor cells, such as Beyondcell and CaDRReS-Sc"
- [methods] filtered out cells with less than 200 genes expressed and genes expressed in less than three cells: "we filtered out cells with less than 200 genes expressed and genes expressed in less than three cells and kept the cells with a proportion of mitochondrial genes below 30 %"
