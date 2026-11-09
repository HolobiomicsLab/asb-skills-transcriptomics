---
name: codex-protein-clustering
description: Use when a CODEX spatial proteomics dataset (per-cell protein expression
  plus spatial coordinates, e.g. AnnData with 46,840 cells) needs unsupervised clustering
  and spatial projection to reveal B/T-cell tissue organization without prior normalization.
when_to_use_negative:
- The input is scRNA-seq or another modality that requires preprocessing (e.g. CPM
  total-count normalization to 1e5, log1p-transform, scaling to unit variance/zero-mean)
  — the CODEX workflow in Cellar explicitly runs without normalization.
- The goal is reference-based cell-type assignment or supervised label transfer (e.g.
  scanpy Ingest between HBMP2-spleen-2 and HBMP3-spleen-CC3) rather than unsupervised
  spatial clustering.
- No spatial coordinates are available, so clusters cannot be projected back onto
  the CODEX image tile.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: Cellar
  role: Interactive platform that loads the CODEX AnnData, runs the UMAP embedding,
    Leiden clustering, and spatial projection of expression clusters in single-cell
    images
  repo: https://github.com/euxhenh/cellar
- name: faiss
  role: Approximate nearest-neighbors search to speed up neighbors-graph construction
    for Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: AnnData
  role: In-memory (memory-mapping mode) data object holding the CODEX protein matrix
    and per-cell cluster labels (adata.obs['labels'])
  repo: https://github.com/euxhenh/cellar
- name: umap-learn
  role: Reduces CODEX protein expression data to a 10-dimensional embedding before
    graph construction
- name: leidenalg/igraph
  role: Community detection on the neighbors graph at resolution 0.1 to obtain cluster
    assignments
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:aa0276ef5b27b352f4ec5581c23dccf36bd0d4fae687db41a960bb6d264a0bd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/euxhenh/cellar
  tools:
  - AnnData
  - Cellar
  - faiss
  - leidenalg/igraph
  - umap-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# codex-protein-clustering

## Summary

Unsupervised Leiden clustering of CODEX spatial-proteomics protein expression (via a 10-dim UMAP embedding and a faiss 15-neighbor graph at resolution 0.1) with the cluster assignments projected back onto the spatial CODEX image to reveal tissue-level organization such as B-cell follicles surrounded by T cells.

## When to use

Use when a CODEX spatial proteomics dataset (per-cell protein expression plus spatial coordinates, e.g. AnnData with 46,840 cells) needs unsupervised clustering and spatial projection to reveal B/T-cell tissue organization without prior normalization.

## When NOT to use

- The input is scRNA-seq or another modality that requires preprocessing (e.g. CPM total-count normalization to 1e5, log1p-transform, scaling to unit variance/zero-mean) — the CODEX workflow in Cellar explicitly runs without normalization.
- The goal is reference-based cell-type assignment or supervised label transfer (e.g. scanpy Ingest between HBMP2-spleen-2 and HBMP3-spleen-CC3) rather than unsupervised spatial clustering.
- No spatial coordinates are available, so clusters cannot be projected back onto the CODEX image tile.

## Inputs

- CODEX spatial proteomics dataset as an AnnData object (e.g. CODEX_Florida_19-003-lymph-node-R2, 46,840 cells, unnormalized protein expression)
- Per-cell spatial coordinates (x, y) from the CODEX image
- Leiden resolution parameter (0.1 for lymph node CODEX)

## Outputs

- Per-cell cluster labels stored under adata.obs['labels'] (Leiden resolution-0.1 assignments)
- Spatial CODEX tile image with each cell colored by cluster assignment
- Cluster co-localization scores computed from the spatial tile

## How to apply

Load the CODEX dataset (e.g. CODEX_Florida_19-003-lymph-node-R2, 46,840 cells) into Cellar as an AnnData object WITHOUT normalization, since the methods explicitly do not normalize CODEX protein data. Reduce the protein expression matrix to a 10-dimensional UMAP embedding (umap-learn), then build an approximate nearest-neighbors graph from that embedding with faiss using 15 neighbors. Run Leiden community detection (leidenalg/igraph) on this graph at resolution 0.1 — the reduced resolution used for lymph node CODEX data to obtain a reasonable number of clusters. Finally, color each cell by its cluster assignment at its spatial coordinates to generate the spatial tile, and compute cluster co-localization scores from that tile. Decision point: the resolution parameter controls cluster granularity; 0.1 was calibrated specifically for the lymph node CODEX data and may need adjustment for other tissues.

## Related tools

- **Cellar** (Interactive platform that loads the CODEX AnnData, runs the UMAP embedding, Leiden clustering, and spatial projection of expression clusters in single-cell images) — https://github.com/euxhenh/cellar
- **faiss** (Approximate nearest-neighbors search to speed up neighbors-graph construction for Leiden clustering) — https://github.com/facebookresearch/faiss
- **AnnData** (In-memory (memory-mapping mode) data object holding the CODEX protein matrix and per-cell cluster labels (adata.obs['labels'])) — https://github.com/euxhenh/cellar
- **umap-learn** (Reduces CODEX protein expression data to a 10-dimensional embedding before graph construction)
- **leidenalg/igraph** (Community detection on the neighbors graph at resolution 0.1 to obtain cluster assignments)

## Evaluation signals

- The rendered spatial tile image exists and shows the expected arrangement (B-lineage lymphocyte clusters tightly together, surrounded by T-cell and other clusters).
- The exported session (.h5ad) contains per-cell cluster labels under adata.obs['labels'] consistent with the Leiden resolution-0.1 run.
- Co-localization scores computed from the spatial tile confirm adjacency of lymphocyte clusters as reported in the paper.
- The neighbors graph was built from the 10-dimensional UMAP embedding with 15 faiss neighbors, matching the paper's parameters.
- The number of clusters is reasonable for lymph node CODEX data at resolution 0.1 (neither one mega-cluster nor over-fragmentation).

## Limitations

- No software versions are stated for Cellar, Python, Dash, faiss, or leidenalg, so exact environment reproduction is not guaranteed.
- Reproducibility depends on an external web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact.
- Resolution 0.1 was chosen empirically for the lymph node CODEX data; other tissues may need retuning.
- CODEX data is used unnormalized by design in this workflow, so applying it to data requiring normalization would distort results.

## Evidence

- [methods] UMAP 10-dim on CODEX lymph node data: [evidence span withheld — anchor status: not_found]
- [methods] faiss neighbors graph for Leiden: [evidence span withheld — anchor status: not_found]
- [intro] Spatial projection of clusters in Cellar: "Cellar provides methods for semi-supervised clustering and projection of expression clusters in spatial single-cell images"
- [discussion] faiss speeds up Leiden graph construction: "approximate nearest neighbors based on faiss25 to speed up neighbors graph construction for Leiden clustering"
- [readme] Co-localization assessment in CODEX/CITE-seq pipelines (README): [evidence span withheld — anchor status: external_doc]
