---
name: single-cell-pca-latent-space-projection
description: Use when a cell-by-gene or cell-by-peak AnnData matrix must be embedded
  before clustering or label transfer, e.g., scATAC-seq gene activity scores or a
  query dataset being projected onto a reference PCA via scanpy Ingest.
when_to_use_negative:
- Input is already an embedding or dimensionality-reduced representation (e.g., cisTopic
  topics already selected, such as the 40 cisTopic topics obtained via log-likelihood
  model selection)
- The data have not been normalized/transformed — applying PCA directly to raw counts
  violates the CPM (1e5) + log1p + scaling preprocessing chain the paper prescribes
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scikit-learn
  role: Provides the PCA implementation with a randomized SVD solver used to reduce
    the normalized matrix to 40 dimensions
- name: scanpy Ingest
  role: Projects a query dataset into the 40-component PCA latent space fit on a reference
    dataset and transfers labels
- name: faiss
  role: Builds an approximate nearest-neighbors graph on the PCA embeddings with 15
    neighbors to speed up Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: AnnData
  role: Holds the matrix and embedding in memory-mapping mode, allowing analysis of
    large datasets
- name: Cellar
  role: Interactive pipeline in which PCA-based representation feeds clustering, label
    transfer, and differential expression steps
  repo: https://github.com/euxhenh/cellar
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:9fcc9e93d8d90d1023ef0200a9f374114876b73472b5ebf5246b4aea15d14000
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AnnData
  - Cellar
  - diffxpy
  - faiss
  - scanpy
  - scanpy Ingest
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# single-cell-pca-latent-space-projection

## Summary

Projects single-cell matrices into a low-dimensional PCA latent space (40 components, randomized SVD) after CPM normalization, log1p transformation, and scaling, serving as the embedding for downstream neighbors-graph construction, clustering, and reference-based label transfer in Cellar workflows.

## When to use

Use when a cell-by-gene or cell-by-peak AnnData matrix must be embedded before clustering or label transfer, e.g., scATAC-seq gene activity scores or a query dataset being projected onto a reference PCA via scanpy Ingest.

## When NOT to use

- Input is already an embedding or dimensionality-reduced representation (e.g., cisTopic topics already selected, such as the 40 cisTopic topics obtained via log-likelihood model selection)
- The data have not been normalized/transformed — applying PCA directly to raw counts violates the CPM (1e5) + log1p + scaling preprocessing chain the paper prescribes

## Inputs

- AnnData object holding a cell-by-gene activity score matrix (e.g., PBMC 10k scATAC-seq cell-by-gene matrix)
- Paired reference and query AnnData objects sharing overlapping genes (e.g., HBMP3-spleen-CC3 reference, HBMP2-spleen-2 query)

## Outputs

- 40-dimensional PCA embedding (cells x 40) stored in the AnnData object
- Latent representation suitable for faiss approximate-neighbor graph construction and Leiden clustering, or for scanpy Ingest label transfer

## How to apply

First preprocess the count matrix: CPM-normalize to a total count of 1e5, log1p-transform, then scale to unit variance and zero mean. Reduce the matrix to 40 dimensions using scikit-learn's PCA implementation with a randomized SVD solver. The resulting embedding is then used to construct an approximate neighbors graph with faiss using 15 neighbors for Leiden clustering, or, in the label-transfer workflow, the query dataset (e.g., HBMP2-spleen-2) is projected into a 40-component PCA latent space fit on the reference (e.g., HBMP3-spleen-CC3) via scanpy Ingest. The 40-component dimension is the paper's working setting for both the scATAC-seq cell-by-gene and HuBMAP spleen datasets.

## Related tools

- **scikit-learn** (Provides the PCA implementation with a randomized SVD solver used to reduce the normalized matrix to 40 dimensions)
- **scanpy Ingest** (Projects a query dataset into the 40-component PCA latent space fit on a reference dataset and transfers labels)
- **faiss** (Builds an approximate nearest-neighbors graph on the PCA embeddings with 15 neighbors to speed up Leiden clustering) — https://github.com/facebookresearch/faiss
- **AnnData** (Holds the matrix and embedding in memory-mapping mode, allowing analysis of large datasets)
- **Cellar** (Interactive pipeline in which PCA-based representation feeds clustering, label transfer, and differential expression steps) — https://github.com/euxhenh/cellar

## Examples

```
from sklearn.decomposition import PCA; import scanpy as sc; sc.pp.normalize_total(adata, target_sum=1e5); sc.pp.log1p(adata); sc.pp.scale(adata); X_pca = PCA(n_components=40, svd_solver='randomized').fit_transform(adata.X)
```

## Evaluation signals

- Preprocessing chain verified: matrix is CPM-normalized to total count 1e5, log1p-transformed, and scaled to zero mean and unit variance before PCA
- Reduced representation has exactly 40 dimensions (components) per cell
- Downstream faiss neighbors graph built with 15 neighbors on the embedding supports Leiden clustering at resolution 1
- For label transfer, ARI of Ingest-transferred labels against human-expert annotations is ~0.39 on HBMP2-spleen-2, falling between unconstrained Leiden (0.27) and semi-supervised frozen-cluster Leiden (0.66) as in Supplementary Fig. 2
- For the scATAC-seq workflow, differential expression (Welch's t-test, one vs rest via diffxpy) of clusters on the PCA-derived clustering recovers KLRD1 as a top marker for clusters 0 and 4

## Limitations

- No software versions stated for Cellar, Python, Dash, scikit-learn, or faiss, complicating exact reproduction
- Reproducibility depends on the external Cellar web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact
- No accession IDs given for the HuBMAP annotated reference datasets used to fit the PCA latent space

## Evidence

- [methods] PCA with randomized SVD on preprocessed matrix: "We used the PCA implementation of the scikit-learn package with a randomized SVD solver."
- [methods] faiss neighbors graph on embeddings: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing chain for PCA input: "The data matrix was then CPM total count normalized (total count = 1e5) and log1p-transformed."
- [methods] Ingest label transfer into reference PCA space: "Label transfer between HBMP2-spleen-2 and HBMP3-spleen-CC3 was performed using scanpy’s Ingest"
- [discussion] AnnData memory-mapping for large datasets: "using the Annotated Data object17 in memory-mapping mode which allows the analysis of large datasets"
