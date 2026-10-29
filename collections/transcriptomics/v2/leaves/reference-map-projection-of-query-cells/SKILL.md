---
name: reference-map-projection-of-query-cells
description: Use when scRNA-seq query data (raw counts, 10x/h5 or Seurat objects)
  must be mapped onto a pre-built reference atlas — e.g. the LCMV viral CD8+ T cell
  atlas — to quantify how a perturbation (KO, infection condition) shifts cell subtype
  frequencies.
when_to_use_negative:
- Only cell state labels are needed and query dimensionality reductions must be preserved
  — use the classifier mode (ProjecTILs.classifier) rather than full reference embedding
- The cell types of interest (e.g. CD4+ TIL subtypes, gamma-delta T cells) are under-represented
  or absent from the available reference atlases
- The goal is de novo discovery of transcriptomic states without an annotated reference
  — unsupervised clustering may be preferable, though ProjecTILs can flag cells that
  deviate from reference subtypes
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: ProjecTILs
  role: 'Core algorithm: Run.ProjecTILs / make.projection embed query data in the
    reference map and cellstate.predict assigns cell state labels by nearest-neighbor
    prediction; plot.statepred.composition quantifies subtype frequencies'
  repo: https://github.com/carmonalab/ProjecTILs
- name: STACAS
  role: Semi-supervised scRNA-seq integration used to build the reference atlas by
    aligning heterogeneous multi-study datasets
  repo: https://github.com/carmonalab/STACAS
- name: Seurat
  role: Data structure and downstream/upstream analysis framework; ProjecTILs is integrated
    with Seurat, so query objects are standard Seurat objects
- name: R
  role: Implementation language; ProjecTILs is an R package installed via remotes::install_github
  repo: https://github.com/carmonalab/ProjecTILs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:efd0409f22149782f5ae35a5cb8636cb3aedb7bba5279ff11e6d7c833e1bb728
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - STACAS
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# reference-map-projection-of-query-cells

## Summary

Project new single-cell RNA-seq query data (e.g. Ptpn2-KO/WT or Tox-KO P14 T cells) onto a curated reference T cell atlas with ProjecTILs, embedding queries in the reference PCA/UMAP space and assigning cell state labels by nearest-neighbor prediction, without altering the reference structure. This enables direct comparison of cell subtype composition across perturbation conditions in a stable system of coordinates.

## When to use

Use when scRNA-seq query data (raw counts, 10x/h5 or Seurat objects) must be mapped onto a pre-built reference atlas — e.g. the LCMV viral CD8+ T cell atlas — to quantify how a perturbation (KO, infection condition) shifts cell subtype frequencies.

## When NOT to use

- Only cell state labels are needed and query dimensionality reductions must be preserved — use the classifier mode (ProjecTILs.classifier) rather than full reference embedding
- The cell types of interest (e.g. CD4+ TIL subtypes, gamma-delta T cells) are under-represented or absent from the available reference atlases
- The goal is de novo discovery of transcriptomic states without an annotated reference — unsupervised clustering may be preferable, though ProjecTILs can flag cells that deviate from reference subtypes

## Inputs

- Query scRNA-seq expression matrix (10x hdf5, raw or log-normalized counts) or Seurat object, e.g. Lafleur et al. Ptpn2-KO/WT co-transfer or Yao et al. Tox-KO P14 data
- Reference Seurat atlas built with make.reference (e.g. three LCMV P14 studies integrated via STACAS) or downloaded with load.reference.map()
- Optional human-to-mouse ortholog conversion table (Hs2Mm.convert.table) for projecting human data onto a mouse reference

## Outputs

- Projected Seurat object with query cells embedded in the reference PCA/UMAP space and predicted functional.cluster labels
- Cell state composition frequencies per condition, visualized with plot.statepred.composition barplots
- Radar plots of marker gene expression per state (plot.states.radar), e.g. Klrg1 levels in SLEC

## How to apply

Build or load a reference map first: use ProjecTILs' make.reference to integrate heterogeneous LCMV studies via STACAS alignment into seven functional CD8 T cell clusters, or call load.reference.map() to download a pre-defined map such as the mouse acute/chronic viral infection CD8 T cell atlas (~7,000 virus-specific cells from LCMV Arm/cl13 spleen samples). Load the query expression matrix with read.sc.query (10x/hdf5/raw or log counts supported; raw counts are internally log-normalized, otherwise pass skip.normalize=TRUE if data are already normalized). Then run Run.ProjecTILs(query, ref), which wraps make.projection and cellstate.predict to embed queries in the reference PCA/UMAP space and assign cell state labels by nearest-neighbor prediction; use ProjecTILs.classifier instead if only label transfer is needed and dimensionality reductions should be left intact. Finally quantify subtype shifts with plot.statepred.composition barplots (e.g. verifying the Tex fraction ~20% WT vs >50% in Ptpn2-KO, or SLEC expansion with high Klrg1 in Tox-KO). Quantifications are performed in the PCA latent space; UMAP is for visual exploration only.

## Related tools

- **ProjecTILs** (Core algorithm: Run.ProjecTILs / make.projection embed query data in the reference map and cellstate.predict assigns cell state labels by nearest-neighbor prediction; plot.statepred.composition quantifies subtype frequencies) — https://github.com/carmonalab/ProjecTILs
- **STACAS** (Semi-supervised scRNA-seq integration used to build the reference atlas by aligning heterogeneous multi-study datasets) — https://github.com/carmonalab/STACAS
- **Seurat** (Data structure and downstream/upstream analysis framework; ProjecTILs is integrated with Seurat, so query objects are standard Seurat objects)
- **R** (Implementation language; ProjecTILs is an R package installed via remotes::install_github) — https://github.com/carmonalab/ProjecTILs

## Examples

```
query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref); plot.statepred.composition(list(query.projected, ref=ref))
```

## Evaluation signals

- Projected subtype compositions of query data match reported shifts: Tex compartment ~20% of WT P14 cells but >50% of Ptpn2-KO cells (Lafleur), and SLEC expansion at the expense of Tpex with high Klrg1 (Yao Tox-KO)
- The reference atlas structure is unchanged after projection — ProjecTILs embeds new data without altering the reference space; silhouette coefficients of query cells can be computed with compute_silhouette to check embedding quality
- All projected objects and composition barplots are generated without error for every condition (WT, KO, and repeat queries)
- Query cells are assigned to one of the seven functional CD8 T cell clusters defined in the LCMV reference (e.g. Tex, Tpex, SLEC), and the label distribution is consistent with expected marker expression (e.g. Pdcd1, Havcr2, Tox in Tex; Klrg1 in SLEC)

## Limitations

- UMAP embeddings are provided for visual exploration only and should be interpreted with caution as cell-cell distances and densities in UMAP are highly distorted; quantifications must be done in the PCA latent space
- Cell state quantification is performed in the PCA latent space, not UMAP
- Reference atlases do not yet encompass the full diversity of T cell transcriptional states, especially CD4+ TILs (under-represented) and gamma-delta T cells (not represented at all)
- Query data are expected as raw counts for internal log-normalization; using other normalization strategies requires storing them in the 'data' layer and setting skip.normalize=TRUE

## Evidence

- [abstract] ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure, but also characterizing previously unknown cell states: "ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure, but also characterizing previously unknown cell states"
- [methods] Run.ProjecTILs returns query embedded in reference PCA/UMAP space with predicted cell state labels: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [methods] make.projection projects a dataset onto a reference map of cellular states: "`make.projection`   Project a single-cell RNA-seq dataset onto a reference map of cellular states"
- [readme] README states quantification must be in PCA space, not UMAP: [evidence span withheld — anchor status: external_doc]
- [readme] README expects raw counts internally log-normalized: [evidence span withheld — anchor status: external_doc]
