---
name: semi-supervised-cell-type-annotation-via-gating
description: Use when integrating heterogeneous scRNA-seq samples (e.g., TIL collections
  across tumor types) where only partial or noisy cell-type labels exist and you want
  gating-derived priors to drive semi-supervised integration.
when_to_use_negative:
- When complete, high-quality cell type annotations are already available for all
  cells — a fully supervised reference-mapping approach is then more appropriate.
- When the cell types of interest have no marker-based gating models in the scGate
  DB (no signature available to build the gate).
- When only technical replicates are being integrated — cell type-agnostic batch mixing
  metrics and unsupervised correction arguably suffice.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: scGate
  role: Marker-based gating classifier that converts gene signatures into partial
    cell-type annotations used as prior knowledge.
  repo: https://github.com/carmonalab/scGate
- name: scGate_models
  role: Versioned DB of cell-type gating models (e.g., human TME_HiRes collection)
    loaded via get_scGateDB().
  repo: https://github.com/carmonalab/scGate_models
- name: STACAS
  role: Semi-supervised integration method consuming the partial scGate labels (Run.STACAS,
    FindAnchors.STACAS, IntegrateData.STACAS with semisupervised=TRUE); provides annotate.by.neighbors
    for label refinement.
  repo: https://github.com/carmonalab/STACAS
- name: Seurat v4
  role: Underlying framework in which the scGate gating and STACAS integration anchors
    are computed and the Seurat object metadata carries the labels.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:4ab22ddd9f0a6c6fd62923271d4473a6235dec9eb0522de94a87e5851eb0c554
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/scGate
  tools:
  - Harmony
  - STACAS
  - Seurat v4
  - scGate
  - scGate_models
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# semi-supervised-cell-type-annotation-via-gating

## Summary

Uses scGate marker-based gating models to generate partial cell-type annotations on scRNA-seq data, which serve as prior knowledge for semi-supervised STACAS integration, preserving biological variance while correcting batch effects.

## When to use

Use when integrating heterogeneous scRNA-seq samples (e.g., TIL collections across tumor types) where only partial or noisy cell-type labels exist and you want gating-derived priors to drive semi-supervised integration.

## When NOT to use

- When complete, high-quality cell type annotations are already available for all cells — a fully supervised reference-mapping approach is then more appropriate.
- When the cell types of interest have no marker-based gating models in the scGate DB (no signature available to build the gate).
- When only technical replicates are being integrated — cell type-agnostic batch mixing metrics and unsupervised correction arguably suffice.

## Inputs

- Seurat object(s) of scRNA-seq samples (e.g., TIL data from the uTILity collection)
- scGate gating model collection (e.g., models.db$human$TME_HiRes from carmonalab/scGate_models, or CD8_TIL models)
- Gene symbols standardized via StandardizeGeneSymbols (optionally EnsemblGeneTable.Hs)

## Outputs

- Seurat object with partial scGate cell-type labels in metadata (e.g., scGate_multi column, with gate-negative cells left unassigned)
- Refined per-cell annotations after clustering of the integrated embedding and annotate.by.neighbors label transfer
- Semi-supervised STACAS integrated embedding (new assay via IntegrateData.STACAS, semisupervised=TRUE)

## How to apply

Standardize gene symbols (StandardizeGeneSymbols, optionally with EnsemblGeneTable.Hs), load a gating model collection from the scGate DB (get_scGateDB, e.g., models.db$human$TME_HiRes or the CD8_TIL models from carmonalab/scGate_models), and run scGate as a multi-classifier to add partial labels (scGate_multi) to the Seurat object, leaving gate-negative cells unassigned. Feed these partial scGate labels into STACAS as prior knowledge by running Run.STACAS/FindAnchors.STACAS and IntegrateData.STACAS with semisupervised=TRUE. Refine the annotations by clustering the round-1 integrated embedding and transferring labels to unannotated cells with annotate.by.neighbors, then optionally run a second ssSTACAS round with 800 highly variable genes (FindVariableFeatures.STACAS) and dims=50. The rationale is that even partial and imperfect gating-derived annotations are beneficial, because STACAS is robust to incomplete and imprecise input cell type labels and uses them to preserve biological variance during batch correction.

## Related tools

- **scGate** (Marker-based gating classifier that converts gene signatures into partial cell-type annotations used as prior knowledge.) — https://github.com/carmonalab/scGate
- **scGate_models** (Versioned DB of cell-type gating models (e.g., human TME_HiRes collection) loaded via get_scGateDB().) — https://github.com/carmonalab/scGate_models
- **STACAS** (Semi-supervised integration method consuming the partial scGate labels (Run.STACAS, FindAnchors.STACAS, IntegrateData.STACAS with semisupervised=TRUE); provides annotate.by.neighbors for label refinement.) — https://github.com/carmonalab/STACAS
- **Seurat v4** (Underlying framework in which the scGate gating and STACAS integration anchors are computed and the Seurat object metadata carries the labels.)

## Examples

```
models.db <- get_scGateDB(); gated <- scGate(obj, model=models.db$human$generic$Tcell)
```

## Evaluation signals

- After scGate, a metadata column (e.g., scGate_multi) should carry labels for only a subset of cells, with gate-negative cells left unassigned rather than force-labeled.
- The semi-supervised STACAS round should improve celltype_ASW and CiLISI (computed with carmonalab/scIntegrationMetrics) relative to both uncorrected data and unsupervised STACAS.
- Compare against a Harmony integration as a sanity anchor: Harmony is expected to achieve high batch mixing but low celltype ASW.
- annotate.by.neighbors should assign consistent labels to previously unannotated cells that agree with the cluster structure of the round-1 integrated embedding.
- Result should be stable despite partial/imprecise priors — STACAS is documented to be robust to incomplete and imprecise input cell type labels.

## Limitations

- CiLISI and similar metrics require cell type annotations, cannot be calculated for cell types present in only one dataset, and remain suboptimal when datasets have highly unequal numbers of cells.
- Benchmarks that include supervised/semi-supervised integration methods should account for noisy and partial cell-type information; priors are intentionally incomplete (only a cell subset is labeled by the gates).
- Coverage is limited to cell types with marker signatures in the scGate models DB (versions tagged vN.M); cell types without gates remain unannotated until refined by neighbor transfer.

## Evidence

- [intro] Semi-supervised integration via partial gating labels as STACAS prior: "we describe STACAS, a semi-supervised scRNA-seq data integration method that leverages prior knowledge in the form of cell type annotations"
- [abstract] Robustness of STACAS to partial/imprecise labels: "STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels, which are commonly encountered in real-life integration tasks."
- [discussion] Partial annotations beneficial for variance preservation: "we showed that even partial and imperfect annotations can be beneficial towards preserving biological variance while correcting for technical batch effects"
- [readme] scGate gating invocation in the atlas construction workflow: [evidence span withheld — anchor status: not_found]
- [readme] scGate model loading and gating call from scGate_models README: [evidence span withheld — anchor status: not_found]
