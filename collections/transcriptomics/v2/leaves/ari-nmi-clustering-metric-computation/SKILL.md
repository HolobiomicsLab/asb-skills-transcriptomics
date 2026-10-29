---
name: ari-nmi-clustering-metric-computation
description: Use when evaluating clustering quality of scRNA-seq cluster assignments
  against known ground-truth cell-type labels, or when comparing multiple clustering
  methods' accuracy (e.g. verifying whether a method like scAce achieves ARI = 1).
when_to_use_negative:
- Ground-truth cell-type labels are unavailable — external clustering metrics like
  ARI/NMI cannot be computed
- Evaluating continuous embeddings or trajectory quality rather than discrete cluster
  assignments
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scAce
  role: Produces cluster labels (adata.obs['scace_cluster']) and embeddings (adata.obsm['scace_emb'])
    whose ARI/NMI are computed in the benchmark
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: Loads the gene expression matrix into an AnnData object and preprocesses (filtering,
    normalize_per_cell, log1p, scale) so cluster labels align across methods
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:fc7bb7fbc05596e75b8a541c5b0e50b3976c9cdf0ee4ffa59459b1bb19874fdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Harmony
  - LIGER
  - Leiden
  - Python 3.8
  - Scanorama
  - Seurat
  - scAce
  - scETM
  - scVAE-GM
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ari-nmi-clustering-metric-computation

## Summary

Compute Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) to benchmark single-cell clustering accuracy against ground-truth cell-type labels, as used to compare scAce against nine state-of-the-art scRNA-seq clustering methods on simulated five-cell-type data.

## When to use

Use when evaluating clustering quality of scRNA-seq cluster assignments against known ground-truth cell-type labels, or when comparing multiple clustering methods' accuracy (e.g. verifying whether a method like scAce achieves ARI = 1).

## When NOT to use

- Ground-truth cell-type labels are unavailable — external clustering metrics like ARI/NMI cannot be computed
- Evaluating continuous embeddings or trajectory quality rather than discrete cluster assignments

## Inputs

- AnnData object with predicted cluster labels in adata.obs (e.g. adata.obs['scace_cluster'])
- Ground-truth cell-type label vector for each cell
- Predicted cluster-number per method (for methods where cluster count is specified or reported)

## Outputs

- ARI and NMI values per clustering method against ground-truth labels
- Results table ari_nmi_table.csv containing ARI, NMI, and predicted cluster number for each of the ten methods

## How to apply

After running each clustering method (scAce via run_scace, plus comparisons such as Seurat, CIDR, scDeepCluster, DESC, scVI, scGMAAE, SCCAF, ADClust, graph-sc) on the same preprocessed AnnData, extract predicted cluster labels (e.g. adata.obs['scace_cluster']) and record each method's predicted cluster number. Compute ARI and NMI between predicted labels and ground-truth cell-type labels, and assemble all metrics into a single results table saved as ari_nmi_table.csv. ARI=1 with the correct cluster number indicates perfect recovery of the simulated five cell types; pass the true cell-type count as the cluster number only for methods that require it, since scAce infers the number adaptively via cluster merging.

## Related tools

- **scAce** (Produces cluster labels (adata.obs['scace_cluster']) and embeddings (adata.obsm['scace_emb']) whose ARI/NMI are computed in the benchmark) — https://github.com/sldyns/scAce
- **scanpy** (Loads the gene expression matrix into an AnnData object and preprocesses (filtering, normalize_per_cell, log1p, scale) so cluster labels align across methods)

## Examples

```
from scace import run_scace; adata = run_scace(adata); ari = adjusted_rand_score(adata.obs['scace_cluster'], adata.obs['celltype'])
```

## Evaluation signals

- scAce is the only method among the ten achieving ARI = 1 on the simulated five-cell-type dataset
- ARI = 1 coincides with the correct (five) predicted cluster number for scAce, without pre-specifying it
- ari_nmi_table.csv contains ARI, NMI, and predicted cluster number for all ten methods (scAce, Seurat, CIDR, scDeepCluster, DESC, scVI, scGMAAE, SCCAF, ADClust, graph-sc)
- All methods are evaluated on the identical preprocessed dataset (min_cells=3 gene filter, min_genes=200 cell filter, per-cell normalization with recorded scale factor n_counts/median(n_counts), log1p, scale)

## Limitations

- Comparison methods other than scAce require the true cluster number to be passed in, which favors them; scAce's adaptive cluster merging determines the number of clusters without prior estimation
- ARI/NMI require ground-truth labels, so this evaluation scheme applies only to simulated data or manually annotated real datasets
- Metrics depend on the shared preprocessing pipeline; changes to filtering or normalization alter comparability

## Evidence

- [results] On the simulated data, scAce was the only method among the ten evaluated that achieved an ARI of 1, clearly separating all five cell types in the low-dimensional embedding.: [evidence span withheld — anchor status: not_found]
- [abstract] adaptive cluster merging without estimating cluster number: "we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance"
- [readme] preprocessing filters and normalization: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [readme] scale factor recorded during preprocessing: "adata.obs['scale_factor'] = adata.obs.n_counts / adata.obs.n_counts.median()"
