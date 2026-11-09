---
name: scrna-seq-multi-study-dataset-integration
description: Use when building a reference T cell atlas (e.g., murine TIL or LCMV
  viral CD8 T cell atlas) from multiple public scRNA-seq studies whose samples, protocols,
  and batches differ and must be aligned before annotation or query projection.
when_to_use_negative:
- The input is a single homogeneous dataset with no batch or study effects — no cross-study
  integration is needed
- The goal is projecting a new query dataset onto an already-built reference atlas
  — use Run.ProjecTILs / make.projection instead of rebuilding the reference
- The data are not scRNA-seq expression matrices (e.g., bulk RNA-seq or flow cytometry)
  — STACAS integration of single-cell transcriptomes does not apply
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STACAS
  role: Semi-supervised scRNA-seq integration/batch-effect correction of heterogeneous
    multi-study datasets, called via ProjecTILs API parameters to build the reference
    atlas
  repo: https://github.com/carmonalab/STACAS
- name: ProjecTILs
  role: R framework whose make.reference function converts the STACAS-integrated,
    annotated Seurat object into a reusable reference atlas for query projection
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Object model and data structures for loading samples and storing the integrated
    atlas; ProjecTILs/STACAS integrate with Seurat for up- and downstream analyses
  repo: https://github.com/carmonalab/ProjecTILs
- name: TILPRED
  role: Supervised classifier used together with MAST marker-gene enrichment to annotate
    integrated cells into T cell functional states before atlas finalization
  repo: https://github.com/carmonalab/TILPRED
merged_aliases:
- scrna-seq-multi-dataset-integration
merged_alias_records:
- alias: single-cell-rna-seq-multi-study-integration
  slug: single-cell-rna-seq-multi-study-integration
  cosine_score: 0.8835
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:ea5ea8211f76c701d8f0bf6b11df320fffe3da8d0408b2252707645dc8d81d1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - FastMNN
  - Harmony
  - ProjecTILs
  - R
  - STACAS
  - Scanorama
  - Seurat
  - Seurat v4
  - TILPRED
  - scANVI
  - scGen
  - scVI
  - scib-pipeline
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# scRNA-seq-multi-study-dataset-integration

## Summary

Integration of heterogeneous single-cell RNA-seq datasets from multiple individuals, tissues, batches, and technologies into a common reference atlas space using STACAS, the semi-supervised integration tool from the Carmona lab, as implemented within the ProjecTILs reference-atlas workflow.

## When to use

Use when building a reference T cell atlas (e.g., murine TIL or LCMV viral CD8 T cell atlas) from multiple public scRNA-seq studies whose samples, protocols, and batches differ and must be aligned before annotation or query projection.

## When NOT to use

- The input is a single homogeneous dataset with no batch or study effects — no cross-study integration is needed
- The goal is projecting a new query dataset onto an already-built reference atlas — use Run.ProjecTILs / make.projection instead of rebuilding the reference
- The data are not scRNA-seq expression matrices (e.g., bulk RNA-seq or flow cytometry) — STACAS integration of single-cell transcriptomes does not apply

## Inputs

- Multiple public scRNA-seq samples/counts matrices from different studies, loaded as Seurat objects (e.g., human melanoma and colon adenocarcinoma TIL samples, murine MC38_dLN data, LCMV P14 datasets)
- Optional prior cell type labels for semi-supervised STACAS integration
- STACAS integration parameter settings (STACAS.anchor.coverage, STACAS.correction.scale, STACAS.k.anchor, STACAS.k.weight)

## Outputs

- An integrated Seurat object of all samples aligned in a common low-dimensional space
- An annotated reference atlas (e.g., TIL.atlas.rds) with nine functional T cell subtypes or seven viral CD8 T cell clusters, embeddable via load.reference.map
- Integrated UMAP/PCA embeddings supporting downstream cell-state annotation and query projection

## How to apply

Collect the public scRNA-seq samples (e.g., 21 human melanoma/colon adenocarcinoma tumor samples plus the murine MC38_dLN dataset, or the three LCMV P14 studies) and load each as a Seurat object. Integrate them with STACAS — a batch-correction method designed specifically for heterogeneous multi-study data — via the STACAS parameters exposed by the ProjecTILs API (STACAS.anchor.coverage, STACAS.correction.scale, STACAS.k.anchor, STACAS.k.weight). Optionally exploit prior cell type labels for semi-supervised integration, which better preserves biological variability. Annotate the integrated space into functional states (e.g., nine TIL subtypes via TILPRED plus MAST marker testing, or seven viral CD8 T cell clusters), then call make.reference with parameters such as ndim, nfeatures, recalculate.umap, umap.method, min_dist, n_neighbors, store.markers, n.markers and seed to finalize the reference atlas (e.g., saved as TIL.atlas.rds). Correct integration is judged by whether the UMAP/label space resolves the expected functional subtypes with distinct phenotypes, functions, metabolic lifestyles, and tissue distributions, and by whether subsequent query projections (Run.ProjecTILs) behave consistently.

## Related tools

- **STACAS** (Semi-supervised scRNA-seq integration/batch-effect correction of heterogeneous multi-study datasets, called via ProjecTILs API parameters to build the reference atlas) — https://github.com/carmonalab/STACAS
- **ProjecTILs** (R framework whose make.reference function converts the STACAS-integrated, annotated Seurat object into a reusable reference atlas for query projection) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Object model and data structures for loading samples and storing the integrated atlas; ProjecTILs/STACAS integrate with Seurat for up- and downstream analyses) — https://github.com/carmonalab/ProjecTILs
- **TILPRED** (Supervised classifier used together with MAST marker-gene enrichment to annotate integrated cells into T cell functional states before atlas finalization) — https://github.com/carmonalab/TILPRED

## Examples

```
remotes::install_github("carmonalab/STACAS"); remotes::install_github("carmonalab/ProjecTILs")
```

## Evaluation signals

- The integrated atlas resolves the expected number of functional subtypes in its UMAP/label space (nine for the murine TIL atlas: naive-like CD4/CD8, EM, early-activation, Tex, Tpex, Th1-like, Tfh, Treg; seven for the viral CD8 atlas)
- The integrated object is saved as a valid reference file (e.g., TIL.atlas.rds) that load.reference.map can load for downstream projection
- Sample-query cells projected onto the integrated reference (Run.ProjecTILs) embed within the reference PCA/UMAP space without distorting the reference structure
- Projected subtype compositions reproduce known biological shifts, e.g., Tex fraction ~20% in WT vs >50% in Ptpn2-KO P14 cells on the LCMV atlas
- STACAS integration parameters (anchor coverage, correction scale, k.anchor, k.weight) are set explicitly and the semi-supervised mode preserves biological variability between conditions

## Limitations

- Integrating datasets from heterogeneous samples from multiple individuals, tissues, batches, protocols and technologies is a major challenge; imprecise or incomplete input cell type labels are common and can degrade integration quality
- The resulting reference atlases do not yet encompass the full diversity of T cell transcriptional states — CD4+ TILs are under-represented and gamma-delta T cells are not represented at all
- Reference-based integration restricts query cells to reference-defined states; truly novel states must be characterized separately as deviations from the reference

## Evidence

- [intro] Integration challenge for multi-study atlases: "A major challenge towards the construction of reference single-cell atlases is the integration of gene expression datasets produced from heterogeneous samples"
- [intro] STACAS designed for heterogeneous data: "We previously developed STACAS14, a bioinformatic tool for scRNA-seq data integration specifically designed for the challenges of integrating heterogeneous datasets"
- [readme] Semi-supervised integration benefit: [evidence span withheld — anchor status: external_doc]
- [readme] STACAS function and benchmark: [evidence span withheld — anchor status: not_found]
- [discussion] ProjecTILs integrated with Seurat: "Because ProjecTILs is integrated with Seurat, it can be easily combined with other tools for up- and down-stream analyses."
- [readme] Atlas subtype resolution: [evidence span withheld — anchor status: external_doc]
