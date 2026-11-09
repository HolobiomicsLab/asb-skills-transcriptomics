---
name: cross-species-ortholog-gene-mapping
description: Use when a human single-cell query must be interpreted against a mouse
  reference atlas — e.g., projecting human tumor biopsy T cells onto the murine TIL
  map to test human-mouse T cell subtype conservation.
when_to_use_negative:
- When both query and reference are from the same species and share gene identifiers
  — no ortholog table is needed
- When a high-quality human T cell reference atlas covering the desired subtypes exists
  — direct human-to-human projection avoids ortholog mapping artifacts
- For gamma-delta T cells or under-represented CD4+ TIL states, which are not (or
  poorly) represented in the mouse TIL reference
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3174
tools:
- name: ProjecTILs
  role: Provides Hs2Mm.convert.table, make.projection and cellstate.predict to map
    human queries into the mouse reference atlas via ortholog mapping
  repo: https://github.com/carmonalab/ProjecTILs
- name: R
  role: Implementation language of the ProjecTILs package used for ortholog conversion
    and reference projection
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Underlying Seurat object structure for query and reference; make.projection
    and celltype.heatmap operate on Seurat objects
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:491135482011f4225b6b85b374889e1c8d4c026c788089af6348f3e5662bdb2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-species-ortholog-gene-mapping

## Summary

Enables projection of human scRNA-seq data onto a murine reference atlas by converting human gene identifiers to mouse orthologs (Hs2Mm.convert.table) and embedding the query in the mouse reference space with make.projection, supporting cross-species meta-analysis of TIL subtypes.

## When to use

Use when a human single-cell query must be interpreted against a mouse reference atlas — e.g., projecting human tumor biopsy T cells onto the murine TIL map to test human-mouse T cell subtype conservation.

## When NOT to use

- When both query and reference are from the same species and share gene identifiers — no ortholog table is needed
- When a high-quality human T cell reference atlas covering the desired subtypes exists — direct human-to-human projection avoids ortholog mapping artifacts
- For gamma-delta T cells or under-represented CD4+ TIL states, which are not (or poorly) represented in the mouse TIL reference

## Inputs

- Human single-cell RNA-seq expression matrices (raw counts) from tumor biopsies
- Mouse TIL reference atlas (loaded via load.reference.map)
- Human gene identifiers/symbols requiring conversion to mouse orthologs

## Outputs

- Query Seurat object embedded in the mouse reference PCA/UMAP space with predicted cell state labels
- Projected subtype-study average expression profiles comparable across human and mouse
- Human-mouse co-clustering statistics for reference subtypes

## How to apply

Load the default mouse tumour-infiltrating lymphocyte (TIL) reference map with load.reference.map, then load each human query expression matrix with read.sc.query (raw counts expected; ProjecTILs log-normalizes internally). Build a human-to-mouse ortholog table with Hs2Mm.convert.table and pass it as the ortholog_table parameter to make.projection, which embeds the query into the PCA/UMAP space of the mouse reference without altering the reference. Assign cell states to the projected cells with cellstate.predict using the default nearest-neighbor algorithm (default k and min.confidence). This ortholog-mapping step is what allows 132 human biopsies across 10 studies and 7 cancer types to be compared directly against the mouse TIL atlas, producing subtype-study average profiles that cluster by TIL reference subtype rather than by study, cancer type, or species (human-mouse co-clustering p < 3e-6). Judge correctness by confirming the query embeds within the reference space and that projected human cells co-cluster with murine cells of the same reference subtype.

## Related tools

- **ProjecTILs** (Provides Hs2Mm.convert.table, make.projection and cellstate.predict to map human queries into the mouse reference atlas via ortholog mapping) — https://github.com/carmonalab/ProjecTILs
- **R** (Implementation language of the ProjecTILs package used for ortholog conversion and reference projection) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Underlying Seurat object structure for query and reference; make.projection and celltype.heatmap operate on Seurat objects)

## Examples

```
ortholog_table <- Hs2Mm.convert.table(genes.human = rownames(query)); query.projected <- make.projection(ref, query, ortholog.table = ortholog_table); query.projected <- cellstate.predict(ref, query.projected)
```

## Evaluation signals

- Projected human query cells embed within the PCA/UMAP space of the mouse reference without altering the reference structure
- cellstate.predict assigns cell state labels to query cells with the default nearest-neighbor confidence settings
- Averaged subtype-study profiles cluster by TIL reference subtype, with human and murine samples co-clustering within each of the nine reference subtypes (p < 3e-6)
- Human gene symbols are fully converted to mouse orthologs before projection (ortholog_table supplied to make.projection)
- Query input retains raw counts in the counts layer (or normalized data in the 'data' layer with skip.normalize=TRUE)

## Limitations

- Reference atlases do not encompass the full diversity of T cell transcriptional states, especially CD4+ TILs (under-represented relative to CD8+) and gamma-delta T cells (not represented at all)
- No high-quality human T cell reference atlas yet exists, motivating mouse-atlas projection via ortholog mapping as an interim solution
- UMAP embeddings are for visual exploration only; cell state quantification is performed in the PCA latent space because UMAP distances and densities are highly distorted

## Evidence

- [discussion] ortholog mapping offers a viable solution to interpret human T cell responses in the context of a robust mouse atlas: "ortholog mapping offers a viable solution to interpret human T cell responses in the context of a robust mouse atlas"
- [discussion] by meta-analysis of 132 cancer patient biopsies, ProjecTILs accurately projects human T cell transcriptomes onto the mouse atlas: "by meta-analysis of 132 cancer patient biopsies, we have shown that ProjecTILs can accurately project human T cell transcriptomes onto a reference mouse atlas"
- [abstract] A meta-analysis of tumor-infiltrating T cells from several cohorts reveals a strong conservation of T cell subtypes between human and mouse: "A meta-analysis of tumor-infiltrating T cells from several cohorts reveals a strong conservation of T cell subtypes between human and mouse"
- [abstract] ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure, but also characterizing previously unknown cell states: "ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure, but also characterizing previously unknown cell states that "deviate" from the"
- [readme] ProjecTILs is a computational method to project scRNA-seq data into reference single-cell atlases in a stable, annotated system of coordinates: "ProjecTILs` is a computational method to project scRNA-seq data into reference single-cell atlases, enabling their direct comparison in a stable, annotated system of coordinates."
