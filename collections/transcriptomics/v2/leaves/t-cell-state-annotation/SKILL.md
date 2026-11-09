---
name: t-cell-state-annotation
description: Use when scRNA-seq counts of tumor-infiltrating or virus-specific T cells
  need interpretable functional state labels (e.g. naive-like, EM, Tex, Tpex, Th1-like,
  Tfh, Treg) rather than de novo unsupervised clustering.
when_to_use_negative:
- Input is not T cell data — the reference atlases are cell-type-specific (TILs, DCs,
  stromal cells); gamma-delta T cells are not represented at all in the TIL references
- The goal is de novo discovery of entirely novel cell types without any reference
  context, and preserving the reference space would be undesirable
- Query data lacks raw counts and no compatible normalization is stored in the Seurat
  'data' layer with skip.normalize=TRUE
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ProjecTILs
  role: 'Core algorithm: embeds query scRNA-seq data into a reference atlas (Run.ProjecTILs)
    or transfers labels only (ProjecTILs.classifier), predicting cell state labels'
  repo: https://github.com/carmonalab/ProjecTILs
- name: STACAS
  role: Multi-study scRNA-seq integration used to build custom reference atlases from
    heterogeneous datasets prior to label transfer
  repo: https://github.com/carmonalab/STACAS
- name: Seurat
  role: Object model and upstream/downstream single-cell analysis framework with which
    ProjecTILs is integrated (Seurat query objects in, Seurat objects out)
- name: TILPRED
  role: Legacy logistic regression-based CD8+ TIL state classifier (Tex, Tpex, etc.);
    no longer maintained, superseded by ProjecTILs
  repo: https://github.com/carmonalab/TILPRED
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:a9c4529f2e814cf8947abed1eae5df1c66090bcfc347deb485bcec7a2ae26edf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - STACAS
  - Seurat
  - TILPRED
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# t-cell-state-annotation

## Summary

Annotate single-cell T lymphocyte transcriptomes into functional T cell states by projecting query scRNA-seq data onto a curated reference atlas (e.g. the mouse TIL atlas) using ProjecTILs, which embeds query cells in the reference PCA/UMAP space and transfers cell state labels without altering the reference structure.

## When to use

Use when scRNA-seq counts of tumor-infiltrating or virus-specific T cells need interpretable functional state labels (e.g. naive-like, EM, Tex, Tpex, Th1-like, Tfh, Treg) rather than de novo unsupervised clustering.

## When NOT to use

- Input is not T cell data — the reference atlases are cell-type-specific (TILs, DCs, stromal cells); gamma-delta T cells are not represented at all in the TIL references
- The goal is de novo discovery of entirely novel cell types without any reference context, and preserving the reference space would be undesirable
- Query data lacks raw counts and no compatible normalization is stored in the Seurat 'data' layer with skip.normalize=TRUE

## Inputs

- Seurat object or expression matrix of raw scRNA-seq counts for query T cells (query_example_seurat as example)
- Reference atlas .rds file (e.g. TIL.atlas.rds) loaded via load.reference.map
- Human-to-mouse ortholog conversion table (Hs2Mm.convert.table) for cross-species projection

## Outputs

- Query Seurat object embedded in the reference PCA and UMAP space with predicted cell state labels per cell
- Cell state composition summaries (e.g. plot.statepred.composition barplots, plot.states.radar gene-expression radar plots)
- Silhouette coefficients for query cells (compute_silhouette) quantifying projection concordance

## How to apply

Load a pre-built reference map with load.reference.map() (by default the mouse tumor-infiltrating T cell atlas) or a custom atlas built via STACAS integration and make.reference. Load query raw counts into a Seurat object (via read.sc.query) — ProjecTILs log-normalizes internally unless skip.normalize=TRUE is set with pre-normalized data in the 'data' layer. Run Run.ProjecTILs(query, ref=ref) to embed the query in the reference PCA/UMAP space and predict cell state labels for each cell, or use the faster ProjecTILs.classifier mode to transfer only labels without changing dimensionality reductions. For cross-species analysis, project human data onto the murine reference via human-mouse ortholog mapping (Hs2Mm.convert.table). Interpret UMAP embeddings only visually, since quantifications are performed in the PCA latent space; characterize 'deviant' query states using find.discriminant.genes and find.discriminant.dimensions against the reference or a control.

## Related tools

- **ProjecTILs** (Core algorithm: embeds query scRNA-seq data into a reference atlas (Run.ProjecTILs) or transfers labels only (ProjecTILs.classifier), predicting cell state labels) — https://github.com/carmonalab/ProjecTILs
- **STACAS** (Multi-study scRNA-seq integration used to build custom reference atlases from heterogeneous datasets prior to label transfer) — https://github.com/carmonalab/STACAS
- **Seurat** (Object model and upstream/downstream single-cell analysis framework with which ProjecTILs is integrated (Seurat query objects in, Seurat objects out))
- **TILPRED** (Legacy logistic regression-based CD8+ TIL state classifier (Tex, Tpex, etc.); no longer maintained, superseded by ProjecTILs) — https://github.com/carmonalab/TILPRED

## Examples

```
library(ProjecTILs); ref <- load.reference.map(); data(query_example_seurat); query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref)
```

## Evaluation signals

- compute_silhouette returns positive silhouette coefficients for most query cells relative to the reference, indicating query cells sit near the assigned reference subtypes
- Predicted labels resolve the expected nine functional subtypes (naive-like CD4/CD8, EM, early-activation, Tex, Tpex, Th1-like, Tfh, Treg) in the reference UMAP/label space
- plot.statepred.composition barplots show biologically plausible state frequencies per sample/condition (e.g. exhausted states enriched in tumor vs. naive-like in dLN)
- Reference atlas embedding is unchanged after projection — query cells occupy but do not distort the reference PCA/UMAP space
- Radar plots (plot.states.radar) of canonical markers (e.g. Pdcd1, Havcr2, Tox, Tcf7, Gzmb) match the expected phenotype of each assigned state

## Limitations

- UMAP embeddings are for visual exploration only; cell-cell distances and densities in UMAP are highly distorted, so quantifications are done in the PCA latent space
- Reference atlases do not yet encompass the full diversity of T cell transcriptional states, especially CD4+ TILs (under-represented vs. CD8+) and gamma-delta T cells (not represented at all)
- TILPRED is no longer maintained and is superseded by ProjecTILs

## Evidence

- [abstract] ProjecTILs allows accurate embedding: "ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure, but also characterizing previously unknown cell states that "deviate" from the"
- [methods] Run.ProjecTILs returns embedded query: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [methods] make.projection description: "`make.projection`   Project a single-cell RNA-seq dataset onto a reference map of cellular states"
- [readme] README default reference and run: [evidence span withheld — anchor status: external_doc]
- [readme] UMAP quantification caution: [evidence span withheld — anchor status: external_doc]
- [readme] TILPRED deprecation: "TILPRED is an R Package for the classification of tumor-infiltrating T lymphocytes (TILs) from single-cell RNA-seq data.

**TILPRED is no longer maintained.**"
