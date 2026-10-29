---
name: simulated-scrna-seq-clustering-benchmark
description: Use when a simulated single-cell gene expression matrix must be clustered
  without a pre-specified cluster count, and cluster-merging behavior must be traced
  against the d_bar/2 adaptive stopping criterion.
when_to_use_negative:
- The number of clusters is already known and fixed in advance — the skill's purpose
  is clustering without estimating the number of clusters beforehand
- The input is a feature table or bulk expression matrix rather than a single-cell
  gene expression matrix suitable for AnnData and ZINB-based VAE pre-training
- The analysis only needs raw embeddings from an existing method without the adaptive
  cluster-merging evaluation against d_bar/2
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: scAce
  role: Implements the VAE pre-training, cluster initialization, and adaptive cluster
    merging steps that produce embeddings and cluster labels
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: Provides AnnData container and preprocessing functions (filter_genes, filter_cells,
    normalize_per_cell, log1p, scale) applied before running scAce
- name: Leiden
  role: Used in de novo initialization to obtain initial cluster labels before adaptive
    merging
- name: Python 3.8
  role: Runtime environment for the scAce package and benchmark scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:a3f3b4d945044e3d6bd2336d473f9cf82c984a790f243e89cf22f1b815147865
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
  - MATLAB
  - Python
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- scrna-seq-clustering-benchmarking
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# simulated-scrna-seq-clustering-benchmark

## Summary

Benchmark the scAce adaptive embedding-and-clustering pipeline on simulated scRNA-seq data, producing cell embeddings (adata.obsm['scace_emb']) and cluster labels (adata.obs['scace_cluster']) and recording the adaptive cluster-merging decision sequence.

## When to use

Use when a simulated single-cell gene expression matrix must be clustered without a pre-specified cluster count, and cluster-merging behavior must be traced against the d_bar/2 adaptive stopping criterion.

## When NOT to use

- The number of clusters is already known and fixed in advance — the skill's purpose is clustering without estimating the number of clusters beforehand
- The input is a feature table or bulk expression matrix rather than a single-cell gene expression matrix suitable for AnnData and ZINB-based VAE pre-training
- The analysis only needs raw embeddings from an existing method without the adaptive cluster-merging evaluation against d_bar/2

## Inputs

- Simulated single-cell gene expression matrix loaded as a scanpy AnnData object (adata = sc.AnnData(data))
- Optional pre-existing cluster labels for the clustering-enhancement initialization mode

## Outputs

- adata.obs['scace_cluster'] — final cluster labels per cell
- adata.obsm['scace_emb'] — learned cell embeddings usable for downstream analyses
- cluster_merging_results.csv — per-iteration merge decisions and final positive-integer cluster count

## How to apply

Pre-process the AnnData object with sc.pp.filter_genes(adata, min_cells=3) and sc.pp.filter_cells(adata, min_genes=200), then total-count normalize via sc.pp.normalize_per_cell, store the scale_factor (n_counts / median n_counts), and apply sc.pp.log1p followed by sc.pp.scale. Run the scAce pipeline, which consists of a VAE-based pre-training step, a cluster initialization step (de novo via Leiden, or clustering enhancement from existing labels), and an adaptive cluster merging step that alternates network updates with merging of the pair having the smallest size-weighted inter-cluster centroid distance (wij*dij_inter, Eqs. 7-10). Merging repeats while the minimum size-weighted distance is below d_bar/2, recomputing labels and centroids after each merge; record each merged pair and the cluster count at that step. Validate the run by confirming the final cluster count is a positive integer consistent with the recorded merge sequence and that embeddings and labels are present in adata.obsm['scace_emb'] and adata.obs['scace_cluster'].

## Related tools

- **scAce** (Implements the VAE pre-training, cluster initialization, and adaptive cluster merging steps that produce embeddings and cluster labels) — https://github.com/sldyns/scAce
- **scanpy** (Provides AnnData container and preprocessing functions (filter_genes, filter_cells, normalize_per_cell, log1p, scale) applied before running scAce)
- **Leiden** (Used in de novo initialization to obtain initial cluster labels before adaptive merging)
- **Python 3.8** (Runtime environment for the scAce package and benchmark scripts)

## Examples

```
from scace import run_scace; adata = run_scace(adata)
```

## Evaluation signals

- cluster_merging_results.csv exists with a field for each merge iteration and a final cluster count that is a positive integer
- The recorded merge sequence is consistent with the d_bar/2 stopping criterion: merging stops exactly when the minimum size-weighted inter-cluster distance exceeds d_bar/2
- adata.obs['scace_cluster'] exists with one label per cell and a cardinality equal to the final cluster count
- adata.obsm['scace_emb'] exists with one embedding vector per cell, suitable as input to downstream analyses
- Preprocessing invariants hold: genes filtered at min_cells=3, cells filtered at min_genes=200, scale_factor stored as n_counts / median(n_counts)

## Limitations

- Adaptive merging depends on the size-weighted distance threshold d_bar/2; pairs whose weighted distance straddles the threshold may be merged or left separate in ways sensitive to centroid re-computation after each merge
- In de novo mode initial labels come from Leiden, so initialization quality propagates into the merging outcome; in clustering-enhancement mode results depend on the quality of prior clustering results from other methods
- The paper reports performance on simulated and real scRNA-seq datasets; behavior on other data modalities (e.g., feature tables, bulk data) is not supported

## Evidence

- [abstract] we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance: "we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance"
- [readme] The output adata contains cluster labels in adata.obs['scace_cluster'] and the cell embeddings in adata.obsm['scace_emb'].: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`."
- [readme] Preprocessing filters and normalization steps from README methods block: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [abstract] The scAce package is implemented in python 3.8: "The scAce package is implemented in python 3.8"
