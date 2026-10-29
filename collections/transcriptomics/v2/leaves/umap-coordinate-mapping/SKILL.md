---
name: umap-coordinate-mapping
description: Use when reconciling multiple single-cell clustering/annotation sets
  stored outside an AnnData object, requiring prior cluster labels and UMAP embeddings
  be registered as obs/obsm fields before decision-level integration.
when_to_use_negative:
- When only a single clustering result exists and no conflicting annotations need
  to be reconciled — run standard Leiden clustering via the scanpy_recipe instead
- When the AnnData already contains all annotation columns and adata.obsm['X_umap']
  — re-adding annotations is unnecessary
- When modalities need joint latent-space integration (e.g., Seurat-WNN, TotalVI)
  rather than decision-level mix-and-match of labels
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTriangulate
  role: Provides scTriangulate.preprocessing.add_annotation and add_umap functions
    to attach external cluster labels and UMAP coordinates to an AnnData object prior
    to integration
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Framework in which scTriangulate is seamlessly integrated; used to read/write
    AnnData and generate UMAPs via sctriangulate.preprocessing.scanpy_recipe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:c9828399389cdcca83e811581bcdafe86dddb17b51493d627e1f11f9c957ed27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# umap-coordinate-mapping

## Summary

Maps alternative clustering annotations and their UMAP coordinates onto an existing AnnData object using scTriangulate.preprocessing.add_annotation and add_umap, so that conflicting clustering solutions (e.g., ICGS2, Seurat3, Monocle3) can be integrated by scTriangulate and visualized on a shared embedding.

## When to use

Use when reconciling multiple single-cell clustering/annotation sets stored outside an AnnData object, requiring prior cluster labels and UMAP embeddings be registered as obs/obsm fields before decision-level integration.

## When NOT to use

- When only a single clustering result exists and no conflicting annotations need to be reconciled — run standard Leiden clustering via the scanpy_recipe instead
- When the AnnData already contains all annotation columns and adata.obsm['X_umap'] — re-adding annotations is unnecessary
- When modalities need joint latent-space integration (e.g., Seurat-WNN, TotalVI) rather than decision-level mix-and-match of labels

## Inputs

- AnnData (.h5ad) object with raw UMI counts and existing clustering annotations
- Alternative clustering annotation tables (e.g., ICGS2, Seurat3, Monocle3, curated labels) from a prior benchmarking study
- UMAP coordinate matrices corresponding to the alternative clusterings
- Synapse-deposited inputs (syn26320732)

## Outputs

- AnnData object with alternative annotations registered in adata.obs and UMAP coordinates in adata.obsm['X_umap']
- Normalized expression matrix (log CPTT) suitable for scTriangulate input

## How to apply

Retrieve alternative clustering results and their UMAP coordinates (in the bone marrow atlas case, from the Synapse deposit syn26320732). Use scTriangulate.preprocessing.add_annotation to attach each annotation column to adata.obs and scTriangulate.preprocessing.add_umap to place coordinates in adata.obsm['X_umap'], enabling automatic UMAP visualization by ScTriangulate. The AnnData must contain properly normalized data in adata.X (log CPTT for RNA, CLR for ADT), must not have an adata.raw attribute (delete it if present, as it interferes with scanpy marker-gene calculation), and needs at least two annotation columns to pass to the query argument. After mapping, normalize raw UMI counts to logarithmic counts per ten thousand (CPTT) and run ScTriangulate (e.g., win_fraction_cutoff=0.3, add_metrics='tfidf5') on the merged object.

## Related tools

- **scTriangulate** (Provides scTriangulate.preprocessing.add_annotation and add_umap functions to attach external cluster labels and UMAP coordinates to an AnnData object prior to integration) — https://github.com/frankligy/scTriangulate
- **scanpy** (Framework in which scTriangulate is seamlessly integrated; used to read/write AnnData and generate UMAPs via sctriangulate.preprocessing.scanpy_recipe)

## Examples

```
from sctriangulate.preprocessing import add_annotation, add_umap; add_annotation(adata, annotation_df, name='ICGS2'); add_umap(adata, umap_df, name='ICGS2')
```

## Evaluation signals

- After add_umap, adata.obsm['X_umap'] exists with coordinates matching the number of cells in adata, so ScTriangulate can auto-generate visualizations
- Each added annotation appears as a distinct column in adata.obs and can be passed to the ScTriangulate query argument
- The scTriangulate run completes on all input annotation sets (ICGS2, Monocle3, Seurat3, curated)
- a data.raw is deleted if present, otherwise scanpy marker-gene calculation is confounded
- Resulting agreement table against Hay et al. reference clusters reproduces the Supplementary Fig. 4d pattern

## Limitations

- UMAP coordinates and annotations must correspond cell-for-cell with the existing AnnData object; misaligned indices will break the mapping
- adata.raw must be removed before running scTriangulate, as it interferes with how scanpy calculates marker genes
- Population naming of integrated clusters remains largely non-standardized
- Programmatic default analytical settings may not be optimal for all study designs

## Evidence

- [methods] The obtained annotations and UMAP coordinates24 were added to the existing AnnData by the scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions: "The obtained annotations and UMAP coordinates24 were added to the existing AnnData by the scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions"
- [readme] at least two columns representing conflicting annotations in adata.obs, passed to query argument: "(b) at least two columns representing conflicting annotations in adata.obs, passed to query argument
(c) adata.obsm['X_umap'] for automatically generate visualization, check"
- [readme] don't have adata.raw attribute: [evidence span withheld — anchor status: external_doc]
- [methods] using results deposited in the Synapse database (syn26320732): "using results deposited in the Synapse database (syn26320732)"
