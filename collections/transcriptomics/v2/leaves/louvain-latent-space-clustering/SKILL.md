---
name: louvain-latent-space-clustering
description: Use when you have a MuData object with scConfluence latent embeddings
  in mdata.obsm['latent'] and need to discover subpopulations (e.g. memory/naive/plasma
  B cell subtypes in tri-omics PBMC data) that single-modality annotations miss.
when_to_use_negative:
- When the input lacks a shared latent embedding (no mdata.obsm['latent'] from scConfluence)
  — clustering raw counts instead ignores the diagonal integration arrangement of
  cells
- When cells from only one modality are present, since the skill's value comes from
  cross-modality coherence of the shared latent space
- When the goal is paired (horizontal) integration of the same cells across modalities
  rather than diagonal integration of unpaired cells
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Compute the neighbor graph on the 'latent' representation (sc.pp.neighbors
    with use_rep='latent'), run Louvain clustering at resolutions 0.5 and 0.2, and
    compute/plot the UMAP of the inferred embeddings
- name: scConfluence
  role: Produces the shared low-dimensional cell embeddings (model.get_latent() into
    mdata.obsm['latent']) that the neighbor graph and Louvain clustering operate on
  repo: https://github.com/cantinilab/scconfluence
- name: Muon
  role: Holds the MuData object whose obsm/obs fields store the embeddings and cluster
    labels; also used for QC filtering and normalization upstream of embedding
- name: SciPy
  role: Used for Fisher's exact tests on cross-modality marker overlaps between B
    cell and monocyte clusters, written to marker_overlap_fisher_tests.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:e7307b31574a2b15c9924f2fbda3f820e7b9a6421be8ec3d4a4683c0b37622d1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cantinilab/scconfluence
  repo_url: https://github.com/cantinilab/scconfluence
  tools:
  - Muon
  - Scanpy
  - SciPy
  - scConfluence
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# louvain-latent-space-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters cells in the scConfluence-inferred shared latent embedding by building a neighbor graph on the 'latent' representation (Scanpy) and running Louvain — first genome-wide at resolution 0.5, then re-subclustering B cell and monocyte populations at a finer resolution 0.2 to recover biologically coherent subclusters.

## When to use

Use when you have a MuData object with scConfluence latent embeddings in mdata.obsm['latent'] and need to discover subpopulations (e.g. memory/naive/plasma B cell subtypes in tri-omics PBMC data) that single-modality annotations miss.

## When NOT to use

- When the input lacks a shared latent embedding (no mdata.obsm['latent'] from scConfluence) — clustering raw counts instead ignores the diagonal integration arrangement of cells
- When cells from only one modality are present, since the skill's value comes from cross-modality coherence of the shared latent space
- When the goal is paired (horizontal) integration of the same cells across modalities rather than diagonal integration of unpaired cells

## Inputs

- MuData object with scConfluence latent embeddings in mdata.obsm['latent'] (tri-omics PBMC scRNA-seq, scATAC-seq, CyTOF)
- Coarse cell-type labels (e.g. B cell and monocyte clusters) for subsetting
- Precomputed cross-modality distance matrices referenced in mdata.uns['cross_keys']

## Outputs

- Louvain cluster assignments on all cells (resolution 0.5) stored in mdata.obs
- Fine-grained subcluster assignments for B cell and monocyte subsets (resolution 0.2)
- UMAP embedding of the inferred latent space with cluster and modality overlays
- marker_overlap_fisher_tests.csv: Fisher's exact test table on cross-modality marker overlaps with significance fields

## How to apply

After fitting the ScConfluence model, extract shared embeddings via model.get_latent() into mdata.obsm['latent'] aligned to mdata.obs_names. Compute a neighbor graph with Scanpy (sc.pp.neighbors with use_rep='latent'), then cluster all cells with Louvain at resolution 0.5. Decision point: coarse clusters corresponding to B cells and monocytes should then be subsetted and reclustered with a finer Louvain resolution of 0.2 to split them into subclusters (e.g. memory, naive and plasma B cell subclusters). Visualize cluster and modality labels on the UMAP of the inferred embeddings (sc.tl.umap, sc.pl.umap), then validate subclusters by running Fisher's exact tests on cross-modality marker overlaps (writing marker_overlap_fisher_tests.csv). The rationale is that diagonal integration arranges cells in the latent space by biological similarity independently of modality of origin, so clustering there recovers structure supported across scRNA-seq, scATAC-seq and CyTOF.

## Related tools

- **Scanpy** (Compute the neighbor graph on the 'latent' representation (sc.pp.neighbors with use_rep='latent'), run Louvain clustering at resolutions 0.5 and 0.2, and compute/plot the UMAP of the inferred embeddings)
- **scConfluence** (Produces the shared low-dimensional cell embeddings (model.get_latent() into mdata.obsm['latent']) that the neighbor graph and Louvain clustering operate on) — https://github.com/cantinilab/scconfluence
- **Muon** (Holds the MuData object whose obsm/obs fields store the embeddings and cluster labels; also used for QC filtering and normalization upstream of embedding)
- **SciPy** (Used for Fisher's exact tests on cross-modality marker overlaps between B cell and monocyte clusters, written to marker_overlap_fisher_tests.csv)

## Examples

```
import scanpy as sc; sc.pp.neighbors(mdata, use_rep='latent'); sc.tl.louvain(mdata, resolution=0.5); sc.tl.umap(mdata); sc.pl.umap(mdata, color=['louvain','modality'])
```

## Evaluation signals

- Louvain at resolution 0.5 on the latent representation yields clusters in which B cell and monocyte structure is preserved in the UMAP of the inferred embeddings
- Re-subclustering B cells at resolution 0.2 recovers memory, naive and plasma subclusters, biologically supported by shared markers across modalities
- marker_overlap_fisher_tests.csv exists and contains significance fields for cross-modality marker overlaps (differentially expressed genes of B cell subclusters overlap significantly between scRNA and scATAC: 30 genes for cluster 0, 12 for cluster 1, 232 for cluster 2)
- mdata.obsm['latent'] row indices align to mdata.obs_names before computing the neighbor graph
- Cluster and modality labels overlay coherently on the UMAP computed from the inferred embeddings

## Limitations

- scConfluence and all state-of-the-art methods struggle to properly deal with rare cell populations, which are hard to detect and under-represented in parameter estimation
- The latent dimension is fixed at 16 for all datasets and hidden layer sizes must be adapted to dataset dimensionality (Supplementary Table 5); clustering quality depends on these choices
- The method relies on cross-modality feature connections, so latent spaces suitable for this clustering cannot always be produced (e.g. integrating electrophysiology measurements with gene expression)

## Evidence
