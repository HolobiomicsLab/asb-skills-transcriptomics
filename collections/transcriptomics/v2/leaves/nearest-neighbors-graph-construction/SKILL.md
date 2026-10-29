---
name: nearest-neighbors-graph-construction
description: Use when a single-cell analysis pipeline needs a neighbors graph over
  cell embeddings (PCA/UMAP/cell-by-gene or protein matrices) as input for Leiden
  clustering, and dataset size makes exact kNN too slow.
when_to_use_negative:
- The input is already a neighbors graph or precomputed cluster labels — rebuilding
  a kNN graph would discard the prior representation.
- The embedding is small enough that exact kNN is tractable and approximate search
  overhead offers no benefit.
- The task requires spatial-image-based co-localization scoring directly, which in
  the article is a separate downstream step applied to the spatial CODEX tile, not
  part of graph construction.
tools:
- name: faiss
  role: Builds the approximate nearest-neighbors graph (15 neighbors) over cell embeddings
    to speed up neighbors graph construction for Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: Cellar
  role: Interactive pipeline that wraps faiss-based neighbors graph construction and
    Leiden clustering on AnnData objects
  repo: https://github.com/euxhenh/cellar
- name: AnnData (Annotated Data object)
  role: In-memory / memory-mapped data structure holding the embedding matrix that
    the neighbors graph is built from
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:818b8f0a1b6876e049ba6bc9abe1afa754e89bbe18b60c57150dc3241bd84345
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AnnData
  - AnnData (Annotated Data object)
  - Cellar
  - diffxpy
  - faiss
  - scanpy
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# nearest-neighbors-graph-construction

## Summary

Constructs an approximate nearest-neighbors graph over cell embeddings (e.g., UMAP or PCA reductions of single-cell data) using faiss with 15 neighbors, to feed downstream Leiden community-detection clustering in the Cellar pipeline.

## When to use

Use when a single-cell analysis pipeline needs a neighbors graph over cell embeddings (PCA/UMAP/cell-by-gene or protein matrices) as input for Leiden clustering, and dataset size makes exact kNN too slow.

## When NOT to use

- The input is already a neighbors graph or precomputed cluster labels — rebuilding a kNN graph would discard the prior representation.
- The embedding is small enough that exact kNN is tractable and approximate search overhead offers no benefit.
- The task requires spatial-image-based co-localization scoring directly, which in the article is a separate downstream step applied to the spatial CODEX tile, not part of graph construction.

## Inputs

- Cell embedding matrix (e.g., 10-dim UMAP of CODEX protein expression or 40-dim PCA of cell-by-gene scATAC-seq activity scores)
- AnnData object (memory-mapped) holding the embedding and cell metadata
- Number of neighbors k (15 in the Cellar workflows)

## Outputs

- Approximate nearest-neighbors graph over cells (faiss kNN indices/distances)
- Leiden cluster assignments per cell, stored in adata.obs['labels']

## How to apply

First obtain a reduced cell embedding: e.g., 10-dimensional UMAP for the lymph node CODEX protein data (unnormalized, per the methods), or 40-dimensional randomized-SVD PCA on the CPM-normalized (1e5), log1p-transformed, unit-variance-scaled PBMC scATAC-seq cell-by-gene activity matrix. Build an approximate neighbors graph with faiss using 15 neighbors on these embeddings; Cellar uses faiss specifically to speed up neighbors graph construction for Leiden clustering on large AnnData objects held in memory-mapped mode. The resulting graph is passed directly to Leiden community detection — resolution 1 for the PBMC scATAC-seq case, resolution 0.1 for the lymph node CODEX case to keep the cluster count reasonable. Choose the number of neighbors (15 here) and embedding dimensionality before clustering, since they jointly determine graph connectivity and hence cluster granularity.

## Related tools

- **faiss** (Builds the approximate nearest-neighbors graph (15 neighbors) over cell embeddings to speed up neighbors graph construction for Leiden clustering) — https://github.com/facebookresearch/faiss
- **Cellar** (Interactive pipeline that wraps faiss-based neighbors graph construction and Leiden clustering on AnnData objects) — https://github.com/euxhenh/cellar
- **AnnData (Annotated Data object)** (In-memory / memory-mapped data structure holding the embedding matrix that the neighbors graph is built from)

## Examples

```
import faiss; index = faiss.IndexFlatL2(embedding.shape[1]); index.add(embedding); D, I = index.search(embedding, 15)
```

## Evaluation signals

- The faiss graph is built with exactly 15 neighbors over the reduced embedding (10-dim UMAP for CODEX, 40-dim PCA for scATAC-seq cell-by-gene).
- Downstream Leiden clustering runs on this graph and converges: resolution 1 yields PBMC scATAC-seq clusters whose DE analysis (diffxpy Welch's t-test) recovers KLRD1 as an NK marker for clusters 0 and 4.
- For the lymph node CODEX data, resolution-0.1 Leiden labels in adata.obs['labels'] project onto the spatial tile and co-localization scores confirm B-lineage clusters surrounded by T cell clusters.
- Cluster count is 'reasonable' at the chosen resolution (0.1 for lymph node CODEX was explicitly selected for this reason).

## Limitations

- The article's analyses used fixed parameters (15 neighbors; resolution 1 or 0.1); other datasets may require retuning neighbors and Leiden resolution.
- Approximate (faiss) neighbors are not exact kNN; graph topology, and hence cluster boundaries, can differ from an exact search.
- For CODEX data the methods explicitly do not normalize the protein matrix before UMAP embedding, so this graph construction depends on the upstream normalization decision being correct for the modality.

## Evidence

- [discussion] approximate nearest neighbors based on faiss25 to speed up neighbors graph construction for Leiden clustering: "approximate nearest neighbors based on faiss25 to speed up neighbors graph construction for Leiden clustering"
- [methods] embeddings used to construct faiss graph with 15 neighbors: [evidence span withheld — anchor status: not_found]
- [readme] faiss library description from README: [evidence span withheld — anchor status: external_doc]
- [discussion] memory-mapped AnnData for large datasets: "using the Annotated Data object17 in memory-mapping mode which allows the analysis of large datasets"
