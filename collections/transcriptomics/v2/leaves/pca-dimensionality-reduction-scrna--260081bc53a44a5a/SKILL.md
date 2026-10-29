---
name: pca-dimensionality-reduction-scrna
description: Use when a scRNA-seq count matrix (e.g., AnnData .h5ad) has been QC-filtered
  and normalized and you need a compact latent representation for clustering or UMAP
  visualization, such as reproducing HuBMAP spleen cluster counts.
when_to_use_negative:
- Data has not yet been QC-filtered and normalized (CPM 1e5 + log1p + scaling); applying
  PCA directly on raw counts will be dominated by library size
- Data is scATAC-seq peak/bin matrices where a gene activity score conversion (e.g.,
  BinToGene) or topic modeling (cisTopic) is the article's stated alternative
- The goal is differential expression or functional enrichment rather than building
  a latent representation for clustering
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Provides the PCA implementation with randomized SVD solver used to reduce
    the scaled expression matrix to 40 dimensions
- name: scanpy
  role: Preprocessing of the data (filtering, CPM normalization, log1p, scaling) before
    PCA
- name: faiss
  role: Constructs an approximate nearest-neighbors graph (15 neighbors) from the
    PCA embeddings for Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: Cellar
  role: Interactive tool whose pipeline orchestrates PCA + UMAP dimensionality reduction
    prior to clustering
  repo: https://github.com/euxhenh/cellar
- name: AnnData (Annotated Data object)
  role: In-memory representation for the count matrix and reduced embeddings, supporting
    memory-mapped analysis of large datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:e4bf45d7fd9bdc418c7c5e610b3997c9d8d12fd8f9a58901c456e7e8918e8203
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AnnData
  - AnnData (Annotated Data object)
  - Cellar
  - SCANPY
  - Scikit-learn
  - faiss
  - scSemiProfiler
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- pca-dimensionality-reduction
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# PCA dimensionality reduction for scRNA-seq (Cellar pipeline)

## Summary

Reduces a QC-filtered, CPM-normalized single-cell RNA-seq count matrix to a low-dimensional embedding using scikit-learn PCA (randomized SVD solver) with 40 dimensions, followed by UMAP to 2D for visualization, as implemented in Cellar. This embedding feeds approximate nearest-neighbor graph construction (faiss) for Leiden clustering.

## When to use

Use when a scRNA-seq count matrix (e.g., AnnData .h5ad) has been QC-filtered and normalized and you need a compact latent representation for clustering or UMAP visualization, such as reproducing HuBMAP spleen cluster counts.

## When NOT to use

- Data has not yet been QC-filtered and normalized (CPM 1e5 + log1p + scaling); applying PCA directly on raw counts will be dominated by library size
- Data is scATAC-seq peak/bin matrices where a gene activity score conversion (e.g., BinToGene) or topic modeling (cisTopic) is the article's stated alternative
- The goal is differential expression or functional enrichment rather than building a latent representation for clustering

## Inputs

- AnnData (.h5ad) scRNA-seq count matrix, e.g., the 5273-cell HBMP3-spleen-CC2 HuBMAP spleen dataset
- Preprocessed matrix: filtered (50–3000 genes/cell and cells/gene), CPM-normalized (total count = 1e5), log1p-transformed, scaled to zero mean/unit variance

## Outputs

- 40-dimensional PCA embedding of the cells (scikit-learn randomized SVD)
- 2D UMAP embedding for visualization
- Cell embedding matrix suitable for faiss neighbors graph construction and Leiden clustering

## How to apply

Start from the AnnData matrix after cell/gene filtering (cells with 50–3000 expressed genes, genes expressed in 50–3000 cells) and after CPM normalization to total count = 1e5, log1p transformation, and scaling to zero mean and unit variance (scanpy preprocessing). Reduce the scaled data to 40 dimensions using the scikit-learn PCA implementation with the randomized SVD solver, as done in Cellar for the 5273-cell HBMP3-spleen-CC2 HuBMAP dataset. Use these embeddings to construct an approximate neighbors graph with faiss using 15 neighbors for Leiden clustering, and additionally reduce the embedding to 2D with UMAP for visualization. A key decision point is the number of PCA dimensions (40 here) and neighbor count (15), which together determine the resulting cluster granularity; the reference run on HBMP3-spleen-CC2 yields 16 Leiden clusters at resolution 1.

## Related tools

- **scikit-learn** (Provides the PCA implementation with randomized SVD solver used to reduce the scaled expression matrix to 40 dimensions)
- **scanpy** (Preprocessing of the data (filtering, CPM normalization, log1p, scaling) before PCA)
- **faiss** (Constructs an approximate nearest-neighbors graph (15 neighbors) from the PCA embeddings for Leiden clustering) — https://github.com/facebookresearch/faiss
- **Cellar** (Interactive tool whose pipeline orchestrates PCA + UMAP dimensionality reduction prior to clustering) — https://github.com/euxhenh/cellar
- **AnnData (Annotated Data object)** (In-memory representation for the count matrix and reduced embeddings, supporting memory-mapped analysis of large datasets)

## Examples

```
from sklearn.decomposition import PCA; emb = PCA(n_components=40, svd_solver='randomized').fit_transform(adata.X)
```

## Evaluation signals

- Number of cells in the embedding must equal the input cell count (5273 for HBMP3-spleen-CC2)
- Downstream Leiden clustering at resolution 1 on the 15-neighbor faiss graph must reproduce 16 clusters, matching Supplementary Fig. 1a
- PCA output dimensionality must be 40 (randomized SVD solver)
- Input must already be scaled to zero mean and unit variance; verify preprocessing preceded PCA
- UMAP output must be a 2D embedding aligned cell-for-cell with the PCA embedding

## Limitations

- No software versions stated for Cellar, Python, or the underlying libraries, so exact numerical reproducibility is not guaranteed
- Reproducibility partly depends on the Cellar web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact
- Cluster count (16) depends on resolution and the 40-dimension/15-neighbor parameter choices; changing these alters granularity

## Evidence

- [methods] PCA implementation with randomized SVD solver: "We used the PCA implementation of the scikit-learn package with a randomized SVD solver."
- [methods] faiss 15-neighbor graph for Leiden: [evidence span withheld — anchor status: not_found]
- [methods] preprocessing via scanpy: "Preprocessing of the data was done via scanpy"
- [methods] cell/gene filtering thresholds: "we filtered cells with less than 50 or more than 3000 expressed genes"
- [methods] normalization and scaling: "The data matrix was then CPM total count normalized (total count = 1e5) and log1p-transformed."
- [results] 16 clusters reference outcome: [evidence span withheld — anchor status: not_found]
