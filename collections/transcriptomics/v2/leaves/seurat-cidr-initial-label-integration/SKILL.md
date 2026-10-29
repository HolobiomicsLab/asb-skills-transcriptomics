---
name: seurat-cidr-initial-label-integration
description: Use when you already have initial cluster labels from another method
  (e.g., Seurat or CIDR) on an scRNA-seq dataset and want to refine/enhance them rather
  than run de novo clustering, evaluating gains in ARI/NMI.
when_to_use_negative:
- When no prior clustering results exist and de novo clustering is desired — scAce's
  de novo initialization with Leiden is the appropriate path instead
- When the number of clusters must be estimated in advance or constrained — scAce's
  adaptive cluster merging improves clustering without estimating cluster number
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scAce
  role: 'Runs the clustering enhancement: VAE pre-training, network update, and adaptive
    cluster merging refine the external Seurat/CIDR labels; invoked via run_scace'
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: Constructs the AnnData object and performs the preprocessing (filtering, per-cell
    normalization, log1p, scale) required before run_scace
- name: Python 3.8
  role: Runtime environment in which the scAce package is implemented and executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:68a21a94aace58a6b2c4774abd9439020cbd11148bf10c3f4d85858871a859db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# seurat-cidr-initial-label-integration

## Summary

Applies scAce's clustering enhancement option, feeding external initial cluster labels from Seurat or CIDR into scAce's VAE pre-training, network update, and adaptive cluster merging loop to improve final clustering accuracy over the source labels.

## When to use

Use when you already have initial cluster labels from another method (e.g., Seurat or CIDR) on an scRNA-seq dataset and want to refine/enhance them rather than run de novo clustering, evaluating gains in ARI/NMI.

## When NOT to use

- When no prior clustering results exist and de novo clustering is desired — scAce's de novo initialization with Leiden is the appropriate path instead
- When the number of clusters must be estimated in advance or constrained — scAce's adaptive cluster merging improves clustering without estimating cluster number

## Inputs

- scRNA-seq gene expression matrix loaded into a scanpy AnnData object
- Initial cluster labels per cell from Seurat
- Initial cluster labels per cell from CIDR

## Outputs

- Enhanced cluster labels in adata.obs['scace_cluster']
- Cell embeddings in adata.obsm['scace_emb'] for downstream analyses
- Before/after ARI and NMI comparison table (e.g., scace_enhancement_ari_nmi.csv)

## How to apply

Preprocess each dataset as an AnnData object in scanpy: filter_genes(min_cells=3), filter_cells(min_genes=200), normalize_per_cell, store adata.obs['scale_factor'] = n_counts / median(n_counts), then log1p and scale. Obtain initial labels separately from Seurat and from CIDR, then run scAce via run_scace with the clustering-enhancement option so the pre-trained VAE, network update, and adaptive cluster merging steps iteratively refine the external labels. Collect enhanced labels from adata.obs['scace_cluster'] and cell embeddings from adata.obsm['scace_emb'] after iteration stops. Assess enhancement by computing ARI and NMI of the labels before and after scAce refinement for each dataset and each initial-label source.

## Related tools

- **scAce** (Runs the clustering enhancement: VAE pre-training, network update, and adaptive cluster merging refine the external Seurat/CIDR labels; invoked via run_scace) — https://github.com/sldyns/scAce
- **scanpy** (Constructs the AnnData object and performs the preprocessing (filtering, per-cell normalization, log1p, scale) required before run_scace)
- **Python 3.8** (Runtime environment in which the scAce package is implemented and executed)

## Examples

```
from scace import run_scace; adata = run_scace(adata, init_labels=seurat_labels)  # or cidr_labels; results in adata.obs['scace_cluster'], adata.obsm['scace_emb']
```

## Evaluation signals

- ARI and NMI computed on the scAce-enhanced labels exceed those of the original Seurat/CIDR initial labels on most real datasets (Fig. 4 validation criterion)
- Output AnnData contains both adata.obs['scace_cluster'] and adata.obsm['scace_emb'] after iteration stops
- Preprocessing invariants: genes filtered with min_cells=3, cells with min_genes=200, and adata.obs['scale_factor'] equals n_counts / median n_counts
- Improvement holds for both initial-label sources (Seurat and CIDR), not only one

## Limitations

- Improvement in ARI and NMI is reported on 'most' real datasets, not universally — some datasets may show no gain from enhancement
- Requires valid external initial labels; quality of the enhancement depends on the Seurat or CIDR starting partitions
- scAce results are produced after an iterative process stops; the number of iterations/stopping behavior is not fixed in advance

## Evidence

- [results] enhancement improves external labels: [evidence span withheld — anchor status: not_found]
- [abstract] clustering enhancement option: "scAce provides an option to perform clustering enhancement, which can update and enhance cluster assignments based on previous clustering results from other methods"
- [readme] adaptive merging iteratively updates labels and embeddings: "the network parameters, cell embeddings, cluster labels and centroids are iteratively updated by alternately performing network update and cluster merging steps"
- [readme] output locations: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`"
- [methods] preprocessing steps: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
