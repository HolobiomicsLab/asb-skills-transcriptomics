---
name: cell-state-composition-frequency-analysis
description: Use when projecting scRNA-seq query data (e.g., Regnase-1-null vs WT
  CD8+ TILs) onto a reference atlas and you need cell-state frequency compositions
  per condition to detect enrichment of subtypes like Tpex or Tex.
when_to_use_negative:
- The question is whether gene programs within a fixed cell state deviate between
  conditions rather than whether subtype frequencies change — use find.discriminant.dimensions
  (PCA/ICA) instead of composition analysis
- Input data are normalized beyond ProjecTILs' expected raw counts without setting
  skip.normalize=TRUE, which would double-normalize and distort state label assignment
- The cell type of interest (e.g., γδ T cells) is not represented in the reference
  atlas, so predicted state frequencies would be meaningless
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ProjecTILs
  role: Reference atlas projection and cell-state label prediction (load.reference.map,
    read.sc.query, Run.ProjecTILs, make.projection, cellstate.predict, plot.statepred.composition)
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Object model for query and reference data; ProjecTILs is integrated with Seurat
    so composition and downstream analyses combine easily
  repo: https://github.com/carmonalab/ProjecTILs
- name: R
  role: Implementation language for the ProjecTILs package and the composition/frequency
    workflow
  repo: https://github.com/carmonalab/ProjecTILs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:64d22dd7d1c056c1efac55200d2517454e983e9ff3b413a11ad4dca5e07bf67f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cell-state-composition-frequency-analysis

## Summary

Quantifies and compares the frequency of predicted reference cell states (e.g., Tpex, Tex in the mouse TIL atlas) between conditions after ProjecTILs projection, revealing whether a perturbation (e.g., Regnase-1 deletion) shifts subtype composition or alters programs within a state.

## When to use

Use when projecting scRNA-seq query data (e.g., Regnase-1-null vs WT CD8+ TILs) onto a reference atlas and you need cell-state frequency compositions per condition to detect enrichment of subtypes like Tpex or Tex.

## When NOT to use

- The question is whether gene programs within a fixed cell state deviate between conditions rather than whether subtype frequencies change — use find.discriminant.dimensions (PCA/ICA) instead of composition analysis
- Input data are normalized beyond ProjecTILs' expected raw counts without setting skip.normalize=TRUE, which would double-normalize and distort state label assignment
- The cell type of interest (e.g., γδ T cells) is not represented in the reference atlas, so predicted state frequencies would be meaningless

## Inputs

- ProjecTILs mouse TIL reference atlas loaded via load.reference.map
- Query scRNA-seq raw count expression matrices for each condition (e.g., Regnase-1-null and WT CD8+ TILs from Wei et al.), formatted as Seurat objects via read.sc.query
- Projected Seurat objects with cell-state labels from Run.ProjecTILs

## Outputs

- Barplot of cell-state frequency composition per condition (plot.statepred.composition)
- Fold-change table of cell-state abundances between conditions (e.g., Tpex ~3-fold, Tex ~1.8-fold in Regnase-1-null)
- Projected query objects embedded in reference PCA/UMAP space with predicted cell-state labels

## How to apply

Load the tumour-infiltrating lymphocyte reference atlas with load.reference.map, format the query expression matrices (Regnase-1-null and WT CD8+ TIL counts) with read.sc.query, and embed plus label each condition with Run.ProjecTILs, which wraps make.projection and cellstate.predict. Then call plot.statepred.composition to produce a barplot of cell-state frequencies for each condition and compute fold-changes between conditions (in the Wei et al. Regnase-1 case: ~3-fold enrichment of Tpex and ~1.8-fold of Tex in Regnase-1-null cells). The rationale is that reference-based labelling places query cells in a stable, annotated coordinate system so composition differences are directly comparable across conditions. If fold-changes alone do not explain the phenotype, follow up with find.discriminant.dimensions (e.g., ICA reduction) to detect gene programs that deviate within a state, as done for ICA 25 (KS statistic = 0.612, driven by Lag3/Klrc1).

## Related tools

- **ProjecTILs** (Reference atlas projection and cell-state label prediction (load.reference.map, read.sc.query, Run.ProjecTILs, make.projection, cellstate.predict, plot.statepred.composition)) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Object model for query and reference data; ProjecTILs is integrated with Seurat so composition and downstream analyses combine easily) — https://github.com/carmonalab/ProjecTILs
- **R** (Implementation language for the ProjecTILs package and the composition/frequency workflow) — https://github.com/carmonalab/ProjecTILs

## Examples

```
ref <- load.reference.map(); query.projected <- Run.ProjecTILs(query, ref=ref); plot.statepred.composition(query.projected)
```

## Evaluation signals

- Regnase-1-null condition shows ~3-fold enrichment of Tpex and ~1.8-fold enrichment of Tex relative to WT after projection
- Every query cell receives a predicted cell-state label from the reference atlas via cellstate.predict
- Composition barplot sums to 100% per condition and includes the same set of reference states across conditions
- Query cells embed in the reference PCA/UMAP space without altering the reference structure
- Projection reproduces with load.reference.map's default mouse TIL reference and Run.ProjecTILs on raw-count inputs

## Limitations

- UMAP embeddings are for visual exploration only; cell-state quantifications are performed in the PCA latent space because UMAP distances and densities are highly distorted
- Reference atlases do not encompass the full diversity of T cell transcriptional states; CD4+ TILs are under-represented and γδ T cells are not represented at all, limiting composition calls for those lineages
- Frequency composition alone may not capture within-state program changes — the Regnase-1-null Tex cells also showed ICA 25 (Lag3/Klrc1-driven) down-regulation not visible from frequencies

## Evidence

- [methods] plot.statepred.composition barplot: "`plot.statepred.composition`   Makes a barplot of the frequency of cell states in a query object"
- [methods] Run.ProjecTILs embeds and labels query: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [discussion] Regnase-1 KO composition finding: "analysis of the Regnase-1 KO data of Wei and colleagues not only explained the increased T cell persistence and tumor control in terms of changes in T cell subtype frequencies"
- [readme] Default mouse TIL reference: [evidence span withheld — anchor status: external_doc]
- [readme] Query input expectation: [evidence span withheld — anchor status: external_doc]
