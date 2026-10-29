---
name: clustering-accuracy-benchmarking-ari-nmi
description: Use when you need to quantify whether an enhanced or refined clustering
  (e.g. scAce applied to Seurat/CIDR initial labels) improves clustering accuracy
  on scRNA-seq data, scored via ARI and NMI.
when_to_use_negative:
- Goal is de novo clustering rather than enhancing labels from an existing method
  — with de novo initialization scAce uses Leiden for initial labels instead of external
  Seurat/CIDR labels
- Data lacks ground-truth-independent comparability between before/after label sets
  (e.g. initial labels missing), since ARI/NMI require label sets on the same cells
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scAce
  role: Performs clustering enhancement of Seurat/CIDR initial labels via VAE pre-training,
    network update, and adaptive cluster merging; installed with pip install scace
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: Loads data into AnnData and applies the preprocessing chain (filter, normalize_per_cell,
    log1p, scale)
- name: Python 3.8
  role: Runtime environment in which the scAce package is implemented and the benchmark
    is run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:69be196db72aa8e33e1b6e43dab5e6254059414caa54c594b2de17ced19507e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# clustering-accuracy-benchmarking-ari-nmi

## Summary

Benchmarks single-cell clustering accuracy by computing Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) before and after scAce's clustering enhancement, using initial labels from Seurat or CIDR on real scRNA-seq datasets.

## When to use

Use when you need to quantify whether an enhanced or refined clustering (e.g. scAce applied to Seurat/CIDR initial labels) improves clustering accuracy on scRNA-seq data, scored via ARI and NMI.

## When NOT to use

- Goal is de novo clustering rather than enhancing labels from an existing method — with de novo initialization scAce uses Leiden for initial labels instead of external Seurat/CIDR labels
- Data lacks ground-truth-independent comparability between before/after label sets (e.g. initial labels missing), since ARI/NMI require label sets on the same cells

## Inputs

- AnnData object (adata) containing a real scRNA-seq gene expression count matrix
- Initial cluster labels from Seurat for each dataset
- Initial cluster labels from CIDR for each dataset

## Outputs

- adata.obs['scace_cluster'] — scAce-enhanced cluster labels
- adata.obsm['scace_emb'] — scAce cell embeddings
- Tabulated before/after ARI and NMI values per dataset and label source (e.g. scace_enhancement_ari_nmi.csv)

## How to apply

Load each real scRNA-seq dataset into an AnnData object and preprocess with scanpy: filter genes with min_cells=3, filter cells with min_genes=200, normalize per cell, log1p-transform, and scale. Obtain initial cluster labels with Seurat and with CIDR, then run scAce via run_scace with the clustering-enhancement option so the VAE pre-training, network update, and adaptive cluster merging steps iteratively refine the external labels. Collect the enhanced labels from adata.obs['scace_cluster'] and cell embeddings from adata.obsm['scace_emb'] after iteration stops. Compute ARI and NMI of the initial (before) and enhanced (after) labels for every dataset and both label sources, and tabulate all before/after values (e.g. in scace_enhancement_ari_nmi.csv). Validation: after-enhancement ARI and NMI should be higher than before-enhancement on most real datasets for both the Seurat and CIDR initial-label sources (Fig. 4).

## Related tools

- **scAce** (Performs clustering enhancement of Seurat/CIDR initial labels via VAE pre-training, network update, and adaptive cluster merging; installed with pip install scace) — https://github.com/sldyns/scAce
- **scanpy** (Loads data into AnnData and applies the preprocessing chain (filter, normalize_per_cell, log1p, scale))
- **Python 3.8** (Runtime environment in which the scAce package is implemented and the benchmark is run)

## Examples

```
from scace import run_scace; adata = run_scace(adata); print(adata.obs['scace_cluster'], adata.obsm['scace_emb'])
```

## Evaluation signals

- ARI of scAce-enhanced labels exceeds ARI of the initial Seurat or CIDR labels on most real datasets (Fig. 4)
- NMI of scAce-enhanced labels exceeds NMI of the initial labels on most real datasets for both label sources
- Enhanced labels are present at adata.obs['scace_cluster'] and embeddings at adata.obsm['scace_emb'] after iteration stops
- Preprocessing invariants hold: genes filtered at min_cells=3, cells filtered at min_genes=200, scale_factor = n_counts / median n_counts stored in adata.obs
- All before/after ARI and NMI values tabulated for every dataset and both initial-label sources in scace_enhancement_ari_nmi.csv

## Limitations

- Improvement in ARI/NMI after enhancement is reported on 'most' real datasets, not all — some datasets may show no gain
- Requires pre-existing initial cluster labels from another method (Seurat or CIDR); the enhancement path does not estimate the number of clusters in advance
- Benchmark is on real scRNA-seq data; results may not transfer to other data types

## Evidence

- [full_text] Given initial cluster labels from either Seurat or CIDR, scAce's clustering enhancement obviously improved the final clustering accuracy on most real datasets, in terms of both ARI and NMI scores (Fig. 4).: [evidence span withheld — anchor status: not_found]
- [abstract] scAce provides an option to perform clustering enhancement, which can update and enhance cluster assignments based on previous clustering results from other methods: "scAce provides an option to perform clustering enhancement, which can update and enhance cluster assignments based on previous clustering results from other methods"
- [readme] with clustering enhancement, initial cluster labels are obtained by applying a cluster splitting approach to a set of existing clustering results.: "with clustering enhancement, initial cluster labels are obtained by applying a cluster splitting approach to a set of existing clustering results"
- [readme] The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`.: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`"
- [readme] sc.pp.filter_genes(adata, min_cells=3): "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [readme] The final results of cell embeddings and cluster labels are output by scAce after the iteration process stops.: "The final results of cell embeddings and cluster labels are output by scAce after the iteration process stops."
