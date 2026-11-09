---
name: variational-autoencoder-clustering
description: Use when clustering a raw single-cell gene expression matrix (AnnData)
  and the number of cell types is unknown, or when benchmarking against methods that
  require K.
when_to_use_negative:
- When the exact number of clusters is known and a method requiring a pre-specified
  K is mandated — scAce's strength is operating without estimating cluster number
  in advance
- When the data are not single-cell gene expression (e.g., bulk RNA-seq or a feature
  table), since the ZINB-based VAE assumes scRNA-seq count structure
- When the task is only dimensionality reduction without clustering — use a dedicated
  embedding method instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scAce
  role: Main VAE-based clustering method; provides run_scace which outputs cluster
    labels and cell embeddings
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: AnnData container and preprocessing (filter_genes/filter_cells min_cells=3
    / min_genes=200, normalize_per_cell, log1p, scale)
- name: Leiden
  role: Obtains initial cluster labels during de novo cluster initialization
- name: Python 3.8
  role: Implementation language of the scAce package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:441958aee8e0c850afd0c2b0b4a6ee5421199e48aca98ca766c31ad72bf45a18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# variational-autoencoder-clustering

## Summary

Cluster single-cell RNA-seq gene expression data with scAce, which trains a variational autoencoder to jointly learn cell embeddings and cluster assignments, using adaptive cluster merging so no pre-specified cluster number is needed.

## When to use

Use when clustering a raw single-cell gene expression matrix (AnnData) and the number of cell types is unknown, or when benchmarking against methods that require K.

## When NOT to use

- When the exact number of clusters is known and a method requiring a pre-specified K is mandated — scAce's strength is operating without estimating cluster number in advance
- When the data are not single-cell gene expression (e.g., bulk RNA-seq or a feature table), since the ZINB-based VAE assumes scRNA-seq count structure
- When the task is only dimensionality reduction without clustering — use a dedicated embedding method instead

## Inputs

- Single-cell gene expression matrix wrapped in a scanpy AnnData object (raw counts)
- Optional: prior clustering results from other methods for clustering-enhancement mode

## Outputs

- adata.obs['scace_cluster'] — per-cell cluster labels
- adata.obsm['scace_emb'] — low-dimensional cell embeddings for downstream analyses
- Benchmark table of ARI/NMI and predicted cluster number (e.g., ari_nmi_table.csv)

## How to apply

Load the expression matrix into a scanpy AnnData object and preprocess exactly as prescribed: sc.pp.filter_genes(adata, min_cells=3), sc.pp.filter_cells(adata, min_genes=200), store adata.raw, normalize per cell, record adata.obs['scale_factor'] = n_counts / median(n_counts), then log1p and scale. Run run_scace(adata) with no cluster-number argument; internally scAce (Python 3.8) pre-trains a VAE whose decoder outputs three ZINB parameters per gene (mean, dispersion, zero proportion), obtains initial labels via Leiden ('de novo' initialization) or a cluster-splitting approach over existing results ('clustering enhancement'), then alternates network-update and adaptive cluster-merging steps to iteratively refine labels, embeddings, and centroids. Read cluster labels from adata.obs['scace_cluster'] and embeddings from adata.obsm['scace_emb']; the embeddings can feed downstream analyses. Optionally supply prior cluster assignments from other methods to use the clustering-enhancement mode. Validate by computing ARI/NMI against ground-truth labels (e.g., on the simulated five-cell-type benchmark scAce was the only method of ten reaching ARI = 1 with the correct cluster count).

## Related tools

- **scAce** (Main VAE-based clustering method; provides run_scace which outputs cluster labels and cell embeddings) — https://github.com/sldyns/scAce
- **scanpy** (AnnData container and preprocessing (filter_genes/filter_cells min_cells=3 / min_genes=200, normalize_per_cell, log1p, scale))
- **Leiden** (Obtains initial cluster labels during de novo cluster initialization)
- **Python 3.8** (Implementation language of the scAce package)

## Examples

```
from scace import run_scace; adata = run_scace(adata)  # cluster labels in adata.obs['scace_cluster'], embeddings in adata.obsm['scace_emb']
```

## Evaluation signals

- On the simulated five-cell-type dataset, scAce should achieve ARI = 1 and recover exactly 5 clusters without K being provided
- adata.obs['scace_cluster'] exists with one label per cell and adata.obsm['scace_emb'] has one embedding per cell
- Preprocessing invariants: adata.raw stored before normalization, adata.obs['scale_factor'] equals n_counts / median(n_counts)
- NMI computed alongside ARI against ground-truth labels should exceed values from comparison methods (Seurat, CIDR, scDeepCluster, DESC, scVI, scGMAAE, SCCAF, ADClust, graph-sc)
- Predicted cluster number from the adaptive merging should match ground truth without manual K specification

## Limitations

- scAce iterates network update and cluster merging until the process stops, so results depend on convergence of the iterative procedure
- Clustering-enhancement mode requires existing clustering results from other methods as input; de novo mode relies on Leiden initialization
- Package is implemented for Python 3.8; older scanpy APIs (e.g., normalize_per_cell) are assumed

## Evidence

- [abstract] VAE jointly learns embeddings and clusters: "constructs a variational autoencoder to simultaneously learn cell embeddings and cluster assignments"
- [abstract] Adaptive merging avoids K estimation: "we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance"
- [methods] Filtering thresholds: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [methods] Normalization and scale factor: "sc.pp.normalize_per_cell(adata)
adata.obs['scale_factor'] = adata.obs.n_counts / adata.obs.n_counts.median()"
- [readme] Three major steps of scAce: "a pre-training step based on a variational autoencoder, a cluster initialization step to obtain initial cluster labels, and an adaptive cluster merging step"
- [readme] ZINB output parameters: "the VAE learns and outputs three parameters of a ZINB distribution (mean, dispersion, and proportion of zero)"
- [readme] Output locations: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`"
- [results] Simulated benchmark result: [evidence span withheld — anchor status: not_found]
