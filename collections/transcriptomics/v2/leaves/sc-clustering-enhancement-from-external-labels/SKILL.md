---
name: sc-clustering-enhancement-from-external-labels
description: Use when you already have cluster labels for a scRNA-seq dataset from
  another method (Seurat, CIDR, etc.) and want to improve their accuracy rather than
  cluster de novo.
when_to_use_negative:
- You have no prior cluster labels and want clustering from scratch — use scAce's
  de novo initialization, where Leiden obtains initial cluster labels
- You need to estimate the number of clusters in advance — scAce's adaptive cluster
  merging achieves improved results without estimating the number of clusters beforehand
- Input is not single-cell gene expression data (e.g. bulk RNA-seq or already-embedded
  features)
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scAce
  role: 'Implements the clustering-enhancement option: VAE pre-training, network update,
    and adaptive cluster merging that refine external initial labels; installed via
    ''pip install scace'''
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: Provides AnnData container and preprocessing (filter_genes min_cells=3, filter_cells
    min_genes=200, normalize_per_cell, log1p, scale) before run_scace
- name: Python 3.8
  role: Runtime for the scAce package
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:2b657d705ce707328f2a4ec8f22054e4a4e6dc62132962e77f59df6677c43225
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
yes_rate: 0.8
n_claims: 10
---

# sc clustering-enhancement-from-external-labels

## Summary

Run scAce's clustering-enhancement option to refine externally supplied initial cluster labels (e.g. from Seurat or CIDR) using its VAE pre-training, network update, and adaptive cluster merging steps, typically improving ARI/NMI on real scRNA-seq datasets (Fig. 4).

## When to use

Use when you already have cluster labels for a scRNA-seq dataset from another method (Seurat, CIDR, etc.) and want to improve their accuracy rather than cluster de novo.

## When NOT to use

- You have no prior cluster labels and want clustering from scratch — use scAce's de novo initialization, where Leiden obtains initial cluster labels
- You need to estimate the number of clusters in advance — scAce's adaptive cluster merging achieves improved results without estimating the number of clusters beforehand
- Input is not single-cell gene expression data (e.g. bulk RNA-seq or already-embedded features)

## Inputs

- scRNA-seq gene expression matrix loaded into a scanpy AnnData object
- Initial cluster labels from an external method (e.g. Seurat or CIDR) for the same cells

## Outputs

- Refined cluster labels in adata.obs['scace_cluster']
- Cell embeddings in adata.obsm['scace_emb'] usable for downstream analyses
- Before/after ARI and NMI comparison table (e.g. scace_enhancement_ari_nmi.csv)

## How to apply

Load each dataset into an AnnData object and preprocess with scanpy: filter genes with min_cells=3, filter cells with min_genes=200, normalize_per_cell (storing scale_factor = n_counts / median n_counts), then log1p and scale. Obtain initial cluster labels from the external method (Seurat or CIDR) and pass them to run_scace with the clustering-enhancement option, so the pre-trained VAE, network update, and adaptive cluster merging iteratively refine the external labels. Collect the refined labels from adata.obs['scace_cluster'] and cell embeddings from adata.obsm['scace_emb'] once the iteration stops. Compute ARI and NMI for the labels before and after enhancement to quantify the gain; the paper reports improvement on most real datasets for both Seurat and CIDR initial labels (Fig. 4). Note that clustering enhancement differs from de novo initialization, where Leiden provides the initial labels.

## Related tools

- **scAce** (Implements the clustering-enhancement option: VAE pre-training, network update, and adaptive cluster merging that refine external initial labels; installed via 'pip install scace') — https://github.com/sldyns/scAce
- **scanpy** (Provides AnnData container and preprocessing (filter_genes min_cells=3, filter_cells min_genes=200, normalize_per_cell, log1p, scale) before run_scace)
- **Python 3.8** (Runtime for the scAce package)

## Examples

```
from scace import run_scace; adata = run_scace(adata, initial_label=initial_labels)  # preprocessing: sc.pp.filter_genes(adata, min_cells=3); sc.pp.filter_cells(adata, min_genes=200); sc.pp.normalize_per_cell(adata); sc.pp.log1p(adata); sc.pp.scale(adata)
```

## Evaluation signals

- ARI and NMI computed on the enhanced labels are higher than on the initial external labels for most real datasets, for both Seurat- and CIDR-derived initial labels (Fig. 4)
- adata.obs['scace_cluster'] exists and contains one label per cell in the AnnData object
- adata.obsm['scace_emb'] exists and its row count equals the number of cells
- Preprocessing invariants: genes filtered at min_cells=3, cells at min_genes=200, and adata.obs['scale_factor'] = n_counts / median(n_counts) stored before log1p and scale

## Limitations

- Improvement over the external labels is reported on 'most' real datasets, not universally — some datasets may show no ARI/NMI gain
- The enhancement depends on the quality of the external initial labels supplied to the cluster splitting approach
- Requires prior preprocessing (filtering, total-count normalization, log1p, scale) per the README; skipping it may affect results

## Evidence

- [abstract] clustering enhancement option refines external labels: "scAce provides an option to perform clustering enhancement, which can update and enhance cluster assignments based on previous clustering results from other methods"
- [abstract] adaptive cluster merging without k estimation: "we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance"
- [readme] README preprocessing steps: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [readme] README outputs: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`."
- [readme] README enhancement vs de novo initialization: "with clustering enhancement, initial cluster labels are obtained by applying a cluster splitting approach to a set of existing clustering results."
- [results] workflow finding on ARI/NMI improvement: [evidence span withheld — anchor status: not_found]
