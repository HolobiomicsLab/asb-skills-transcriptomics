---
name: umap-embedding-visualization
description: Use when you have a learned low-dimensional joint embedding (e.g. scConfluence
  latent space in MuData obsm) and need a 2D visualization to inspect cell-type organization,
  subclusters, or modality alignment.
when_to_use_negative:
- When the embedding is derived from paired data with ground-truth correspondences
  where quantitative alignment metrics like FOSCTTM are more appropriate than qualitative
  UMAP inspection
- When the input is raw high-dimensional counts rather than a reduced latent representation
  — dimensionality reduction (e.g. scConfluence's uncoupled autoencoders with latent
  dimension 16) must precede the neighbor graph
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scConfluence
  role: Provides the shared latent embeddings (model.get_latent() into mdata.obsm['latent'])
    that the UMAP visualizes
  repo: https://github.com/cantinilab/scconfluence
- name: Scanpy
  role: Builds the neighbor graph on the latent representation (sc.pp.neighbors with
    use_rep='latent') and renders the UMAP (sc.tl.umap, sc.pl.umap)
- name: Muon
  role: Holds the MuData object whose obsm field is populated with latent embeddings
    and whose obs carries labels used for UMAP coloring
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:00a7da2833e4e0bd9a22f7fb8dff35d792f8d5b5db64d5099052e03d381db6fa
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
  - Liger
  - MultiMAP
  - Muon
  - Scanpy
  - SciPy
  - Seurat
  - Uniport
  - scConfluence
  - scGLUE
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# umap-embedding-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute a neighbor graph on inferred shared latent embeddings (e.g. scConfluence's mdata.obsm['latent']) and embed with UMAP to visualize joint cell structure across modalities, coloring by cell type, cluster, modality, or gene expression.

## When to use

Use when you have a learned low-dimensional joint embedding (e.g. scConfluence latent space in MuData obsm) and need a 2D visualization to inspect cell-type organization, subclusters, or modality alignment.

## When NOT to use

- When the embedding is derived from paired data with ground-truth correspondences where quantitative alignment metrics like FOSCTTM are more appropriate than qualitative UMAP inspection
- When the input is raw high-dimensional counts rather than a reduced latent representation — dimensionality reduction (e.g. scConfluence's uncoupled autoencoders with latent dimension 16) must precede the neighbor graph

## Inputs

- MuData object with latent embeddings in mdata.obsm['latent'] aligned to mdata.obs_names
- Cell-level annotations in mdata.obs (cell type labels, Louvain clusters, modality of origin)
- Gene expression values (e.g. Fezf2 counts) for feature coloring

## Outputs

- UMAP embedding of the joint latent representation
- Figures coloring the UMAP by cell type, cluster, modality, or gene expression (e.g. reproducing Fig. 3b-e or Fig. 6b-d layouts)

## How to apply

After extracting the shared latent embeddings with model.get_latent() into mdata.obsm['latent'] (aligned to mdata.obs_names), build the neighbor graph with sc.pp.neighbors(mdata, use_rep='latent') and embed with sc.tl.umap(mdata). Color the UMAP by cell-type or cluster labels (e.g. Louvain at resolution 0.5 for global structure, resolution 0.2 for finer B cell/monocyte reclustering in tri-omics PBMC data), by modality of origin, and by genes of interest such as Fezf2 to reproduce Fig. 6b-d layouts, or by method/dataset to reproduce benchmark layouts like Fig. 3b-e. The rationale is that the UMAP of the shared latent space directly visualizes whether cells are arranged by biological similarity independent of modality, which is the stated aim of diagonal integration.

## Related tools

- **scConfluence** (Provides the shared latent embeddings (model.get_latent() into mdata.obsm['latent']) that the UMAP visualizes) — https://github.com/cantinilab/scconfluence
- **Scanpy** (Builds the neighbor graph on the latent representation (sc.pp.neighbors with use_rep='latent') and renders the UMAP (sc.tl.umap, sc.pl.umap))
- **Muon** (Holds the MuData object whose obsm field is populated with latent embeddings and whose obs carries labels used for UMAP coloring)

## Examples

```
import scanpy as sc; sc.pp.neighbors(mdata, use_rep='latent'); sc.tl.umap(mdata); sc.pl.umap(mdata, color=['celltype','Fezf2'])
```

## Evaluation signals

- Tufted and untufted IT neurons should appear organized along the apical-dendrite manifold with a graded Fezf2 expression pattern in the UMAP (expert-review validation)
- B cell and monocyte cluster structure should be preserved in the UMAP of the inferred embeddings (tri-omics PBMC validation)
- mdata.obsm['latent'] must exist and be row-aligned to mdata.obs_names before neighbors/UMAP are computed
- UMAP layouts per method and dataset should visually match the benchmark figures (Fig. 3b-e), with scConfluence showing better mixing and cell-type separation than Seurat, Liger, MultiMAP, Uniport, and scGLUE
- The neighbor graph must be computed with use_rep='latent', not on raw counts or PCA of the original modality features

## Limitations

- UMAP is qualitative; it should be paired with quantitative integration metrics (purity, transfer accuracy, connectivity, FOSCTTM) rather than relied on alone
- Rare cell populations are hard to detect and may be visually under-represented, since even the underlying scConfluence method struggles with rare populations in mini-batch optimization
- UMAP appearance is not guaranteed to be stable across runs/parameters; comparisons against published figure layouts are approximate

## Evidence

- [methods] Compute the neighbor graph on the latent representation with sc.pp.neighbors(mdata, use_rep='latent') and embed with sc.tl.umap(mdata).: [evidence span withheld — anchor status: not_found]
- [results] Color the UMAP by the tufted/untufted IT neuron labels and by Fezf2 expression to reproduce Fig. 6b-d, and verify via expert review: [evidence span withheld — anchor status: not_found]
- [discussion] Render UMAP visualizations of the joint embeddings per method and dataset (Scanpy neighbors + UMAP on the latent representations), reproducing the layout of Fig. 3b-e.: [evidence span withheld — anchor status: not_found]
