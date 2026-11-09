---
name: umap-dimensionality-reduction-spatial-proteomics
description: Use when a CODEX or other spatial proteomics cell-by-protein expression
  matrix (e.g., AnnData) must be embedded for downstream Leiden clustering and spatial
  tile visualization, without prior normalization.
when_to_use_negative:
- When analyzing scRNA-seq data that requires CPM normalization, log1p transformation,
  cell/gene filtering (50-3000 expressed genes), and scaling — those are separate
  preprocessing steps, not part of the CODEX spatial-proteomics path
- When the goal is cell-type annotation via reference-based label transfer rather
  than unsupervised spatial clustering of imaging data
- When the input is already a low-dimensional embedding or cluster labels rather than
  a raw expression matrix
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Cellar
  role: Interactive platform that loads the AnnData CODEX dataset, runs the UMAP embedding,
    and projects expression clusters into spatial single-cell images
  repo: https://github.com/euxhenh/cellar
- name: UMAP (umap-learn)
  role: Reduces CODEX protein expression data to a 10-dimensional embedding prior
    to clustering
- name: faiss
  role: Constructs an approximate nearest-neighbors graph from the UMAP embedding
    with 15 neighbors to speed up Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: Leiden (leidenalg/igraph)
  role: Community detection on the neighbors graph at resolution 0.1 to obtain cell
    clusters projected onto the spatial image
- name: AnnData
  role: In-memory annotated data object holding the CODEX expression matrix, embeddings,
    and per-cell cluster labels
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:069b317565b3521980875117943eb2b36794be8e7df90c3a61155c8bd2dc5ec9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/euxhenh/cellar
  tools:
  - AnnData
  - Cellar
  - Leiden (leidenalg/igraph)
  - UMAP (umap-learn)
  - faiss
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# umap-dimensionality-reduction-spatial-proteomics

## Summary

Reduces CODEX spatial proteomics protein expression matrices to a low-dimensional (10-dimension) UMAP embedding as the basis for neighbors-graph construction, Leiden clustering, and projection of clusters onto spatial images in Cellar.

## When to use

Use when a CODEX or other spatial proteomics cell-by-protein expression matrix (e.g., AnnData) must be embedded for downstream Leiden clustering and spatial tile visualization, without prior normalization.

## When NOT to use

- When analyzing scRNA-seq data that requires CPM normalization, log1p transformation, cell/gene filtering (50-3000 expressed genes), and scaling — those are separate preprocessing steps, not part of the CODEX spatial-proteomics path
- When the goal is cell-type annotation via reference-based label transfer rather than unsupervised spatial clustering of imaging data
- When the input is already a low-dimensional embedding or cluster labels rather than a raw expression matrix

## Inputs

- Raw (unnormalized) CODEX cell-by-protein expression matrix as an AnnData object (e.g., 19-003 lymph node R2, 46,840 cells)
- Per-cell spatial coordinates from the CODEX image

## Outputs

- 10-dimensional UMAP embedding of the protein expression data
- Approximate nearest-neighbors graph (faiss, 15 neighbors)
- Per-cell Leiden cluster labels stored in adata.obs['labels'] (resolution 0.1)
- Spatial tile image with cells colored by cluster assignment
- Cluster co-localization scores from the spatial tile

## How to apply

Load the raw CODEX dataset (e.g., CODEX_Florida_19-003-lymph-node-R2, 46,840 cells) into Cellar as an AnnData object without normalization, since the paper explicitly does not normalize CODEX data. Reduce the protein expression data to a 10-dimensional embedding using UMAP (umap-learn), as the paper states 'The lymph node CODEX data was reduced via UMAP with 10 dimensions'. Construct an approximate nearest-neighbors graph from the embedding using faiss with 15 neighbors, then run Leiden community detection at reduced resolution 0.1 to obtain a reasonable number of clusters for lymph node CODEX data. Finally, color each cell by cluster assignment using its spatial coordinates to generate the CODEX spatial tile and compute co-localization scores to verify spatial organization (e.g., B-cell clusters surrounded by T cell clusters).

## Related tools

- **Cellar** (Interactive platform that loads the AnnData CODEX dataset, runs the UMAP embedding, and projects expression clusters into spatial single-cell images) — https://github.com/euxhenh/cellar
- **UMAP (umap-learn)** (Reduces CODEX protein expression data to a 10-dimensional embedding prior to clustering)
- **faiss** (Constructs an approximate nearest-neighbors graph from the UMAP embedding with 15 neighbors to speed up Leiden clustering) — https://github.com/facebookresearch/faiss
- **Leiden (leidenalg/igraph)** (Community detection on the neighbors graph at resolution 0.1 to obtain cell clusters projected onto the spatial image)
- **AnnData** (In-memory annotated data object holding the CODEX expression matrix, embeddings, and per-cell cluster labels)

## Examples

```
import umap; emb = umap.UMAP(n_components=10).fit_transform(adata.X)
```

## Evaluation signals

- The exported session (.h5ad) contains per-cell cluster labels under adata.obs['labels'] consistent with the Leiden resolution-0.1 run
- The rendered spatial tile image exists and shows cells colored by cluster assignment at their spatial coordinates
- Co-localization/adjacency scores computed from the spatial tile confirm the reported arrangement where B-lineage lymphocyte clusters are surrounded by T cell clusters
- The UMAP embedding uses exactly 10 dimensions and the faiss neighbors graph uses 15 neighbors, matching the published parameters
- Cluster count at Leiden resolution 0.1 is a reasonable number of clusters for the lymph node CODEX dataset, as stated in the validation criteria

## Limitations

- Leiden resolution must be reduced (0.1 used here) for lymph node CODEX data to obtain a reasonable number of clusters — default resolutions will over-cluster
- No software versions are stated for Cellar, Python, Dash, or faiss, so exact reproducibility of the embedding and graph is not guaranteed
- Cellar reproducibility depends on an external web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact
- No accession IDs are given for the HuBMAP annotated datasets used as references

## Evidence

- [methods] UMAP 10-dim embedding of CODEX: [evidence span withheld — anchor status: not_found]
- [methods] faiss neighbors graph for clustering: [evidence span withheld — anchor status: not_found]
- [discussion] faiss speeds up Leiden graph construction: "approximate nearest neighbors based on faiss25 to speed up neighbors graph construction for Leiden clustering"
- [intro] Spatial projection capability in Cellar: "Cellar provides methods for semi-supervised clustering and projection of expression clusters in spatial single-cell images"
