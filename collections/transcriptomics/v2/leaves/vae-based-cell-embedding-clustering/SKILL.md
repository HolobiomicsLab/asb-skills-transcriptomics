---
name: vae-based-cell-embedding-clustering
description: Use when an annotated or unannotated single-cell gene-expression matrix
  must be clustered and embedded without pre-specifying the number of clusters, e.g.
  reproducing scRNA-seq ARI benchmarks like Fig. 3A.
when_to_use_negative:
- Input is not single-cell gene expression data (e.g. bulk RNA-seq, protein abundance,
  or already-reduced feature tables) — the VAE expects a raw count matrix
- Cells already have trusted curated labels and the goal is classification rather
  than unsupervised cluster discovery
- The pipeline requires cluster labels fixed before analysis with no merging or splitting
  (adaptive cluster merging may restructure the partition)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scAce
  role: 'Core method: installs via pip as ''scace''; run_scace trains the VAE, initializes
    clusters, and performs adaptive cluster merging'
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: 'Preprocessing: AnnData container, gene/cell filtering, normalize_per_cell,
    log1p, and scale before clustering'
- name: Leiden
  role: 'De novo cluster initialization: obtains initial cluster labels fed into adaptive
    cluster merging'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:1c13db1f31e8bc6f5e0d2ec8fa766569adfc4d8b0051e0c0e3600555b536d20c
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# vae-based-cell-embedding-clustering

## Summary

Applies scAce, a Python 3.8 VAE-based method that jointly learns single-cell embeddings and cluster assignments via ZINB-distribution modeling, Leiden-based cluster initialization, and adaptive cluster merging, to scRNA-seq count matrices.

## When to use

Use when an annotated or unannotated single-cell gene-expression matrix must be clustered and embedded without pre-specifying the number of clusters, e.g. reproducing scRNA-seq ARI benchmarks like Fig. 3A.

## When NOT to use

- Input is not single-cell gene expression data (e.g. bulk RNA-seq, protein abundance, or already-reduced feature tables) — the VAE expects a raw count matrix
- Cells already have trusted curated labels and the goal is classification rather than unsupervised cluster discovery
- The pipeline requires cluster labels fixed before analysis with no merging or splitting (adaptive cluster merging may restructure the partition)

## Inputs

- Single-cell gene expression count matrix loadable as sc.AnnData (e.g. h5ad from GSE84133, GSE65525, PBMC, kidney, hypothalamus, turtle brain datasets)
- Optional cell-type annotations for ARI evaluation
- Optional existing clustering results for the clustering-enhancement initialization mode

## Outputs

- adata.obs['scace_cluster'] — per-cell cluster labels
- adata.obsm['scace_emb'] — per-cell low-dimensional embeddings for downstream analyses
- ARI values per dataset for benchmark comparisons

## How to apply

Load the count matrix into an sc.AnnData object and preprocess with scanpy: sc.pp.filter_genes(adata, min_cells=3), sc.pp.filter_cells(adata, min_genes=200), then preserve adata.raw, compute a scale_factor as n_counts / median(n_counts), and apply normalize_per_cell, log1p, and scale. Run run_scace(adata), which executes three stages: a pre-training step fitting a variational autoencoder that outputs three ZINB parameters (mean, dispersion, proportion of zero) per gene; a cluster initialization step using Leiden (de novo) or a cluster splitting approach over prior clustering results (clustering enhancement); and an adaptive cluster merging step that alternates network updates with cluster merging to iteratively refine labels, embeddings, and centroids. Collect final labels from adata.obs['scace_cluster'] and embeddings from adata.obsm['scace_emb'], then score against cell-type annotations with the Adjusted Rand Index.

## Related tools

- **scAce** (Core method: installs via pip as 'scace'; run_scace trains the VAE, initializes clusters, and performs adaptive cluster merging) — https://github.com/sldyns/scAce
- **scanpy** (Preprocessing: AnnData container, gene/cell filtering, normalize_per_cell, log1p, and scale before clustering)
- **Leiden** (De novo cluster initialization: obtains initial cluster labels fed into adaptive cluster merging)

## Examples

```
from scace import run_scace; adata = run_scace(adata)
```

## Evaluation signals

- Output adata contains cluster labels in adata.obs['scace_cluster'] and embeddings in adata.obsm['scace_emb']
- Preprocessing invariants hold: genes filtered at min_cells=3, cells at min_genes=200, adata.raw preserved, adata.obs['scale_factor'] = n_counts / median(n_counts)
- Adjusted Rand Index computed against provided cell-type annotations is comparable to published rankings (scAce highest mean ARI, followed by DESC, graph-sc, scDeepCluster, ADClust)
- Pipeline reports all three scAce stages executed: VAE pre-training, cluster initialization (Leiden or clustering enhancement), and adaptive cluster merging iterations
- Embeddings are usable as input to downstream analyses per README tutorial guidance

## Limitations

- Clustering-enhancement initialization requires a set of previous clustering results from other methods; only de novo mode (Leiden) works from scratch
- Method is implemented in Python 3.8 (per the article's package statement)
- Adaptive merging iteratively restructures clusters, so the final number of clusters is not fixed in advance and may differ from an expected k

## Evidence

- [methods] three-step pipeline: "a pre-training step based on a variational autoencoder, a cluster initialization step to obtain initial cluster labels, and an adaptive cluster merging step"
- [readme] ZINB VAE output: "the VAE learns and outputs three parameters of a ZINB distribution (mean, dispersion, and proportion of zero)"
- [abstract] adaptive merging: "we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance"
- [methods] scanpy filters: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [readme] outputs: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`."
