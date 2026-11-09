---
name: leiden-graph-community-detection
description: Use when clustering single-cell protein or RNA data (AnnData object)
  into communities via a neighbors graph, especially when projecting clusters onto
  spatial images (e.g., 46,840-cell lymph node CODEX data).
when_to_use_negative:
- Data is already normalized RNA-seq requiring CPM (1e5) + log1p preprocessing and
  cell/gene filtering (50–3000 genes/cells) — those filter steps in the paper apply
  to the scRNA-seq/scATAC-seq pipelines, not the unnormalized CODEX workflow.
- Goal is supervised cell-type annotation or label transfer (scanpy Ingest / SingleR)
  rather than unsupervised community detection.
- Dataset is too large to fit in memory without the AnnData memory-mapping mode that
  Cellar relies on.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Cellar
  role: Interactive web tool hosting the UMAP -> faiss neighbors -> Leiden clustering
    workflow and spatial projection for CODEX data
  repo: https://github.com/euxhenh/cellar
- name: faiss
  role: Approximate nearest-neighbor search to speed up neighbors graph construction
    for Leiden clustering (15 neighbors)
  repo: https://github.com/facebookresearch/faiss
- name: Leiden (leidenalg/igraph)
  role: Community detection on the neighbors graph at resolution 0.1
- name: UMAP (umap-learn)
  role: Reduces CODEX protein expression to a 10-dimensional embedding used for graph
    construction
- name: AnnData
  role: In-memory annotated data object (memory-mapped) holding the dataset and exported
    cluster labels
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:bc2188176771f99f5414aa37781de4775b2dbcaede9fa7094c59db12e5c42289
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# leiden-graph-community-detection

## Summary

Runs Leiden community detection on an approximate nearest-neighbors graph (faiss, 15 neighbors) built from a low-dimensional embedding (e.g., 10-dimensional UMAP of CODEX protein expression) to cluster single cells, as used in Cellar for the HuBMAP lymph node CODEX spatial proteomics data.

## When to use

Use when clustering single-cell protein or RNA data (AnnData object) into communities via a neighbors graph, especially when projecting clusters onto spatial images (e.g., 46,840-cell lymph node CODEX data).

## When NOT to use

- Data is already normalized RNA-seq requiring CPM (1e5) + log1p preprocessing and cell/gene filtering (50–3000 genes/cells) — those filter steps in the paper apply to the scRNA-seq/scATAC-seq pipelines, not the unnormalized CODEX workflow.
- Goal is supervised cell-type annotation or label transfer (scanpy Ingest / SingleR) rather than unsupervised community detection.
- Dataset is too large to fit in memory without the AnnData memory-mapping mode that Cellar relies on.

## Inputs

- AnnData object of single-cell protein expression (e.g., CODEX_Florida_19-003-lymph-node-R2, 46,840 cells, unnormalized)
- 10-dimensional UMAP embedding of the expression matrix
- Per-cell spatial coordinates for spatial projection

## Outputs

- Per-cell Leiden cluster labels stored in adata.obs['labels'] (.h5ad session export)
- Neighbors graph (faiss, 15 neighbors) underlying the clustering
- Spatial tile image with cells colored by cluster assignment
- Cluster co-localization scores from the spatial tile

## How to apply

Load the single-cell dataset (e.g., CODEX_Florida_19-003-lymph-node-R2, 46,840 cells) into Cellar as an AnnData object without normalization, since the methods explicitly do not normalize CODEX data. Reduce the protein expression matrix to a 10-dimensional UMAP embedding (umap-learn), as the paper reports for the lymph node CODEX data. Construct an approximate nearest-neighbors graph with faiss using 15 neighbors, which Cellar uses to speed up neighbors graph construction for Leiden clustering. Run Leiden community detection (leidenalg/igraph) at resolution 0.1, the reduced resolution chosen for lymph node CODEX data to obtain a reasonable number of clusters. Read each cell's spatial coordinates and color cells by cluster assignment to generate the spatial tile, then compute cluster co-localization scores to verify spatial arrangement (e.g., B-cell clusters surrounded by T cells).

## Related tools

- **Cellar** (Interactive web tool hosting the UMAP -> faiss neighbors -> Leiden clustering workflow and spatial projection for CODEX data) — https://github.com/euxhenh/cellar
- **faiss** (Approximate nearest-neighbor search to speed up neighbors graph construction for Leiden clustering (15 neighbors)) — https://github.com/facebookresearch/faiss
- **Leiden (leidenalg/igraph)** (Community detection on the neighbors graph at resolution 0.1)
- **UMAP (umap-learn)** (Reduces CODEX protein expression to a 10-dimensional embedding used for graph construction)
- **AnnData** (In-memory annotated data object (memory-mapped) holding the dataset and exported cluster labels)

## Examples

```
import leidenalg, igraph; parts = leidenalg.find_partition(g, leidenalg.RBConfigurationVertexPartition, resolution_parameter=0.1, seed=42); adata.obs['labels'] = parts.membership
```

## Evaluation signals

- The rendered spatial tile image exists and cells are colored by cluster assignment
- The exported session (.h5ad) contains per-cell cluster labels under adata.obs['labels'] consistent with the Leiden resolution-0.1 run
- Cluster co-localization scores confirm the reported adjacency of lymphocyte clusters (B-lineage clusters surrounded by T cell clusters)
- Neighbors graph was built with exactly 15 faiss neighbors from the 10-dimensional UMAP embedding
- CODEX data was not normalized before clustering, matching the paper's explicit statement

## Limitations

- Leiden resolution 0.1 was specifically chosen for the lymph node CODEX data to obtain a reasonable number of clusters; other datasets may require different resolution.
- No software versions are stated for Cellar, Python, Dash, or faiss, and reproducibility depends on an external web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact.
- No accession IDs are given for the HuBMAP annotated reference datasets, complicating exact reproduction.

## Evidence

- [methods] UMAP on 10-dim lymph node CODEX: [evidence span withheld — anchor status: not_found]
- [methods] faiss neighbors graph: [evidence span withheld — anchor status: not_found]
- [discussion] faiss speeds up Leiden: "approximate nearest neighbors based on faiss25 to speed up neighbors graph construction for Leiden clustering"
- [intro] spatial projection capability: "Cellar provides methods for semi-supervised clustering and projection of expression clusters in spatial single-cell images"
- [readme] STvEA colocalization context: [evidence span withheld — anchor status: external_doc]
