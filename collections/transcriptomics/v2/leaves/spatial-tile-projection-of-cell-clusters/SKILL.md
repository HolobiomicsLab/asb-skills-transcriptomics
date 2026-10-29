---
name: spatial-tile-projection-of-cell-clusters
description: Use when a spatial proteomics (CODEX) dataset has per-cell coordinates
  and you need to render expression-derived clusters on the tissue image and validate
  their spatial adjacency.
when_to_use_negative:
- The dataset lacks per-cell spatial coordinates, so clusters cannot be projected
  onto an image tile
- The input is scRNA-seq or multi-omics data requiring the standard normalization
  pipeline (CPM 1e5, log1p, unit-variance scaling) rather than raw CODEX intensities
- You need cell-type labels rather than unsupervised cluster structure — use reference-based
  label transfer (e.g., scanpy Ingest) instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3444
tools:
- name: Cellar
  role: Interactive platform that loads the CODEX AnnData, runs clustering, and projects
    expression clusters in spatial single-cell images
  repo: https://github.com/euxhenh/cellar
- name: faiss
  role: Approximate nearest-neighbors graph construction (15 neighbors) on the 10-dim
    UMAP embedding for Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: Leiden (leidenalg/igraph)
  role: Community detection on the neighbors graph at resolution 0.1 to assign per-cell
    cluster labels
- name: AnnData
  role: In-memory annotated data object (.h5ad) holding the CODEX matrix and exported
    cluster labels
- name: STvEA
  role: Reference for adjacency-based co-localization assessment of spatial features
    (Adjacency Score)
  repo: https://github.com/CamaraLab/STvEA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:f0df7d48fd19b08190e9b7b29f37957fcc4f750c53ce5aec3d4b670d2ba1365b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/euxhenh/cellar
  tools:
  - AnnData
  - Cellar
  - Leiden (leidenalg/igraph)
  - STvEA
  - faiss
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spatial-tile-projection-of-cell-clusters

## Summary

Project Leiden cluster assignments of a CODEX spatial proteomics dataset back onto the spatial image as a colored spatial tile, then compute cluster co-localization scores to verify spatial organization of cell types (e.g., B cells surrounded by T cells in a lymph node).

## When to use

Use when a spatial proteomics (CODEX) dataset has per-cell coordinates and you need to render expression-derived clusters on the tissue image and validate their spatial adjacency.

## When NOT to use

- The dataset lacks per-cell spatial coordinates, so clusters cannot be projected onto an image tile
- The input is scRNA-seq or multi-omics data requiring the standard normalization pipeline (CPM 1e5, log1p, unit-variance scaling) rather than raw CODEX intensities
- You need cell-type labels rather than unsupervised cluster structure — use reference-based label transfer (e.g., scanpy Ingest) instead

## Inputs

- CODEX spatial proteomics AnnData object (e.g., CODEX_Florida_19-003-lymph-node-R2, 46,840 cells, protein expression matrix)
- Per-cell spatial coordinates from the CODEX image

## Outputs

- Spatial tile image with cells colored by Leiden cluster assignment
- Exported .h5ad session containing per-cell cluster labels in adata.obs['labels']
- Cluster co-localization (adjacency) scores quantifying cluster spatial arrangement

## How to apply

Load the CODEX AnnData object (e.g., CODEX_Florida_19-003-lymph-node-R2, 46,840 cells) into Cellar WITHOUT normalization, since the paper's methods explicitly do not normalize CODEX data. Reduce protein expression to a 10-dimensional UMAP embedding (umap-learn), build an approximate nearest-neighbors graph from the embedding using faiss with 15 neighbors, and run Leiden community detection (leidenalg/igraph) at resolution 0.1 — the reduced resolution used for lymph node CODEX data to obtain a reasonable cluster count. Read each cell's spatial coordinates and color each cell by its cluster assignment to generate the spatial tile. Compute cluster co-localization scores from the tile (adjacency-style scoring, as in the STvEA Adjacency Score) to confirm that B-lineage lymphocyte clusters are surrounded by T cell clusters. Export the session as .h5ad with per-cell labels in adata.obs['labels'].

## Related tools

- **Cellar** (Interactive platform that loads the CODEX AnnData, runs clustering, and projects expression clusters in spatial single-cell images) — https://github.com/euxhenh/cellar
- **faiss** (Approximate nearest-neighbors graph construction (15 neighbors) on the 10-dim UMAP embedding for Leiden clustering) — https://github.com/facebookresearch/faiss
- **Leiden (leidenalg/igraph)** (Community detection on the neighbors graph at resolution 0.1 to assign per-cell cluster labels)
- **AnnData** (In-memory annotated data object (.h5ad) holding the CODEX matrix and exported cluster labels)
- **STvEA** (Reference for adjacency-based co-localization assessment of spatial features (Adjacency Score)) — https://github.com/CamaraLab/STvEA

## Examples

```
import faiss; index = faiss.IndexFlatL2(10); index.add(umap_emb); D, I = index.search(umap_emb, 15)
```

## Evaluation signals

- The rendered spatial tile image exists and shows spatially contiguous B-cell clusters surrounded by T cell and other cell types
- The exported .h5ad contains per-cell cluster labels under adata.obs['labels'] consistent with a Leiden resolution-0.1 run
- Co-localization scores confirm adjacency of lymphocyte clusters as reported in the paper
- No CPM/log1p normalization was applied to the CODEX counts, matching the paper's methods
- The neighbors graph was built from the 10-dimensional UMAP embedding with faiss using 15 neighbors

## Limitations

- No software versions stated for Cellar, Python, Dash, or faiss, complicating exact reproduction
- Reproducibility depends on an external web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact
- Resolution 0.1 was chosen specifically for the lymph node CODEX data to obtain a reasonable cluster count; other tissues may need retuning
- No accession IDs given for the HuBMAP annotated reference datasets

## Evidence

- [intro] semi-supervised clustering and projection of expression clusters in spatial single-cell images: "Cellar provides methods for semi-supervised clustering and projection of expression clusters in spatial single-cell images"
- [methods] 10-dim UMAP on CODEX: [evidence span withheld — anchor status: not_found]
- [methods] faiss 15-neighbor graph: [evidence span withheld — anchor status: not_found]
- [discussion] faiss speeds Leiden graph construction: "approximate nearest neighbors based on faiss25 to speed up neighbors graph construction for Leiden clustering"
- [readme] STvEA adjacency colocalization: [evidence span withheld — anchor status: external_doc]
