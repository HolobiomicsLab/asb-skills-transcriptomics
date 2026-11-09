---
name: cell-cycle-scoring-single-cell
description: Use when a projected single-cell query contains predicted cycling cells
  (e.g. Mki67+ exhausted T cells) that need to be quantified and visualized against
  the projected cell-state labels.
when_to_use_negative:
- The query has not yet been projected onto a reference map and cell-state labels
  are unavailable — score cycling only after Run.ProjecTILs so scores can be attributed
  to subtypes
- The biological question concerns non-T-cell data or gamma-delta T cells, which are
  not represented in the ProjecTILs reference atlases
- Cell-cycle regression rather than visualization/interpretation of cycling states
  is the goal — the skill provides marker-set scoring, not batch/regression correction
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ProjecTILs
  role: Provides the cell.cycle.obj marker sets and the projected query object on
    which cycling scores are computed and visualized
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Underlying framework for the Seurat query object and module-scoring of gene
    sets in single-cell data
- name: R
  role: Runtime environment in which the ProjecTILs package and cell-cycle scoring
    functions are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:00d8a0dfc8b856ef4dfe3621acccdc93726608d1b4991feb6060d57ac668e27d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# cell-cycle-scoring-single-cell

## Summary

Score single-cell transcriptomes for cell-cycle activity using the cell.cycle.obj marker sets shipped with the ProjecTILs R package, and visualize cycling scores alongside a reference-map projection. This separates proliferating cells (e.g. Mki67+ Tex in tumor-specific CD8 TILs) from non-cycling states within the same projected query.

## When to use

Use when a projected single-cell query contains predicted cycling cells (e.g. Mki67+ exhausted T cells) that need to be quantified and visualized against the projected cell-state labels.

## When NOT to use

- The query has not yet been projected onto a reference map and cell-state labels are unavailable — score cycling only after Run.ProjecTILs so scores can be attributed to subtypes
- The biological question concerns non-T-cell data or gamma-delta T cells, which are not represented in the ProjecTILs reference atlases
- Cell-cycle regression rather than visualization/interpretation of cycling states is the goal — the skill provides marker-set scoring, not batch/regression correction

## Inputs

- Seurat query object with projected cell-state labels returned by Run.ProjecTILs
- cell.cycle.obj cell-cycle marker gene sets shipped with the ProjecTILs package
- Raw scRNA-seq counts in the 'counts' layer of the query Seurat object (or log-normalized data in 'data' with skip.normalize=TRUE)

## Outputs

- Per-cell cell-cycle module scores for the projected query cells
- Cycling-score visualization rendered alongside the projection/state-prediction plot
- Assignment of cycling cells to predicted reference subtypes (e.g. Mki67+ Tex)

## How to apply

After projecting a query (e.g. Miller et al. tetramer+ CD8+ TIL from untreated B16 melanoma) onto the murine TIL reference atlas with Run.ProjecTILs, score each query cell for cell-cycle activity using the cell.cycle.obj marker objects shipped with the ProjecTILs package, which follow Seurat's AddModuleScore-style scoring of S-phase and G2/M gene sets. Render the cycling-score visualization overlaid on the reference-embedded UMAP, next to the projection plot colored by predicted cell state, so that cycling cells can be attributed to specific subtypes (in the B16-OVA case, cycling cells fall largely within the ~90.2% Tex compartment). Interpret UMAP-based visuals cautiously since UMAP densities are distorted; cell-state quantification is done in the PCA latent space.

## Related tools

- **ProjecTILs** (Provides the cell.cycle.obj marker sets and the projected query object on which cycling scores are computed and visualized) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Underlying framework for the Seurat query object and module-scoring of gene sets in single-cell data)
- **R** (Runtime environment in which the ProjecTILs package and cell-cycle scoring functions are executed)

## Examples

```
library(ProjecTILs); query.projected <- Run.ProjecTILs(query, ref=load.reference.map()); query.projected <- AddModuleScore(query.projected, features=cell.cycle.obj$genes, name='cc')
```

## Evaluation signals

- Cycling-score visualization and projection plot are both produced non-empty
- Cycling cells, identified by high cell-cycle module score (Mki67+), fall predominantly within predicted Tex rather than naive or memory subtypes for the B16-OVA tetramer+ query
- Query counts are raw before projection; if pre-normalized data were used, skip.normalize=TRUE was set so scores are not computed on double-normalized values
- The reference UMAP embedding is unchanged by the projection, confirming scores are mapped into the stable reference coordinate system

## Limitations

- UMAP embeddings are for visual exploration only; cell-cell distances and densities are highly distorted, so cycling-cell localization on UMAP should not be over-interpreted
- Reference atlases do not encompass all T cell transcriptional states (e.g. gamma-delta T cells absent, CD4+ TILs under-represented), which can affect subtype attribution of cycling cells
- Cycling scores depend on the marker sets shipped with the package; custom cell-cycle gene sets require a different scoring approach

## Evidence

- [other] Score cycling cells using the cell.cycle.obj marker sets shipped with the package and render the cycling-score visualization alongside the projection plot.: [evidence span withheld — anchor status: not_found]
- [readme] ProjecTILs performs quantifications in PCA space; UMAP is for visualization only: [evidence span withheld — anchor status: external_doc]
- [discussion] ProjecTILs is integrated with Seurat for downstream analyses: "Because ProjecTILs is integrated with Seurat, it can be easily combined with other tools for up- and down-stream analyses."
- [readme] Query object should contain raw counts that are log-normalized internally: [evidence span withheld — anchor status: external_doc]
