---
name: variational-autoencoder-cell-embedding
description: Use when scRNA-seq count data has been preprocessed into an AnnData object
  and you need low-dimensional cell embeddings plus cluster labels without knowing
  the number of clusters in advance.
when_to_use_negative:
- Input is a normalized feature/summary table rather than a raw single-cell gene expression
  count matrix (scAce expects the expression matrix as VAE input)
- The number of clusters is already known and fixed by the experimental design and
  only a static clustering is needed — scAce's adaptive merging is designed for the
  case where the number of clusters should not be estimated in advance
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scAce
  role: Implements the VAE pre-training, cluster initialization, and adaptive cluster
    merging that produce the embeddings and labels
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: Builds the AnnData input and performs the required preprocessing (filtering,
    normalize_per_cell, log1p, scale)
- name: Leiden
  role: Used for de novo cluster initialization on the pre-trained VAE output
- name: Python 3.8
  role: Runtime environment in which the scAce package is implemented
provenance:
  source_task_ids:
  - task_007
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:aacedf8f7b5338e5b8eb4fab3e371c80279050efa59df7f985c3bc2888eb4e86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# variational-autoencoder-cell-embedding

## Summary

Train scAce's variational autoencoder on a single-cell gene expression matrix to simultaneously learn cell embeddings and cluster assignments, with the VAE outputting ZINB distribution parameters (mean, dispersion, zero proportion) per gene that support downstream clustering.

## When to use

Use when scRNA-seq count data has been preprocessed into an AnnData object and you need low-dimensional cell embeddings plus cluster labels without knowing the number of clusters in advance.

## When NOT to use

- Input is a normalized feature/summary table rather than a raw single-cell gene expression count matrix (scAce expects the expression matrix as VAE input)
- The number of clusters is already known and fixed by the experimental design and only a static clustering is needed — scAce's adaptive merging is designed for the case where the number of clusters should not be estimated in advance

## Inputs

- AnnData object with a raw single-cell gene expression count matrix
- Optional: initial cluster labels from other methods (e.g. Seurat or CIDR) for clustering-enhancement mode

## Outputs

- Cell embeddings stored in adata.obsm['scace_emb']
- Cluster labels stored in adata.obs['scace_cluster']

## How to apply

Load the count matrix into a scanpy AnnData object and preprocess: filter_genes(min_cells=3), filter_cells(min_genes=200), normalize_per_cell while storing scale_factor = n_counts / median(n_counts), then log1p and scale. Feed the gene expression matrix into scAce's pre-training step, where the VAE learns three ZINB parameters (mean, dispersion, and proportion of zero) for each gene. After pre-training, scAce performs cluster initialization (Leiden for de novo mode) and an adaptive cluster merging step that alternately performs network update and cluster merging so network parameters, cell embeddings, cluster labels and centroids are iteratively updated until the iteration stops. Run via run_scace(adata); enhanced mode accepts initial labels from other methods such as Seurat or CIDR.

## Related tools

- **scAce** (Implements the VAE pre-training, cluster initialization, and adaptive cluster merging that produce the embeddings and labels) — https://github.com/sldyns/scAce
- **scanpy** (Builds the AnnData input and performs the required preprocessing (filtering, normalize_per_cell, log1p, scale))
- **Leiden** (Used for de novo cluster initialization on the pre-trained VAE output)
- **Python 3.8** (Runtime environment in which the scAce package is implemented)

## Examples

```
from scace import run_scace; adata = run_scace(adata)
```

## Evaluation signals

- Output AnnData contains adata.obs['scace_cluster'] with cluster labels and adata.obsm['scace_emb'] with cell embeddings
- Downstream ARI/NMI of scAce labels against ground-truth cell types should exceed competing methods (e.g. average ARI 0.82 vs scDeepCluster 0.72 and ADClust 0.61 across seven datasets)
- Adaptive cluster merging should monotonically refine cluster counts (e.g. Mouse kidney: 25 -> 17 -> 10 -> 9 -> 8 clusters), not over-merge as ADClust did (5 clusters)
- Clustering-enhancement mode should raise ARI and NMI relative to the initial Seurat/CIDR labels on most real datasets
- Preprocessing invariants: genes filtered at min_cells=3, cells at min_genes=200, and adata.obs['scale_factor'] stores n_counts / median n_counts before log1p and scale

## Limitations

- The adaptive cluster merging process runs iteratively and its stopping point is data-dependent; competing tools on the same data may stop early (SCCAF at 17 clusters on Mouse kidney) or over-merge (ADClust to 5 clusters), so iteration behavior should be logged per dataset
- Preprocessing choices are prescribed by the README (filtering thresholds, normalize_per_cell, log1p, scale); deviating from them may invalidate the scale_factor used by the ZINB loss
- Embeddings are intended as input for downstream analyses but the README does not guarantee they are suitable for every downstream task without further checking

## Evidence

- [readme] VAE learns and outputs three parameters of a ZINB distribution: "For each gene, the VAE learns and outputs three parameters of a ZINB distribution (mean, dispersion, and proportion of zero)."
- [abstract] constructs a variational autoencoder to simultaneously learn cell embeddings and cluster assignments: "constructs a variational autoencoder to simultaneously learn cell embeddings and cluster assignments"
- [readme] network parameters, cell embeddings, cluster labels and centroids iteratively updated: "the network parameters, cell embeddings, cluster labels and centroids are iteratively updated by alternately performing network update and cluster merging steps"
- [readme] cluster labels in adata.obs['scace_cluster'] and embeddings in adata.obsm['scace_emb']: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`."
- [abstract] adaptive cluster merging without estimating the number of clusters in advance: "we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance"
- [methods] preprocessing: filter genes min_cells=3, filter cells min_genes=200: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
