---
name: sequential-batch-correction-with-sample-tree
description: Use when integrating many single-cell RNA-seq datasets (e.g., hundreds
  of samples, hundreds of thousands of cells) with partial cell type annotations,
  where batch effects must be corrected while preserving cell-type-specific biology.
when_to_use_negative:
- Data is not single-cell transcriptomics or lacks gene-expression matrices per sample
  (e.g., already-integrated feature tables or bulk RNA-seq).
- The goal is purely unsupervised integration with no cell type labels and no reference
  — the semi-supervised leverage of prior labels is the core of this approach.
- Only two samples need trivial merging where hierarchical tree ordering adds no value.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: STACAS
  role: 'Core semi-supervised batch correction package: FindAnchors.STACAS, FilterAnchors.STACAS,
    SampleTree.STACAS, and IntegrateData.STACAS are all executed with STACAS functions'
  repo: https://github.com/carmonalab/STACAS
- name: Seurat
  role: Underlying single-cell framework into which STACAS anchors and integration
    are computed (anchor-based integration in Seurat)
- name: annotate.by.neighbors
  role: Transfers subtype labels from the annotated seed reference to newly integrated
    cells using k-nearest-neighbor classification (k=20) in the integrated embedding
  repo: https://github.com/carmonalab/STACAS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:9699cf711086435559bfec5e3a17eee0438d4b48b62aca0463cf100fdafd8444
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - Seurat
  - annotate.by.neighbors
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# sequential-batch-correction-with-sample-tree

## Summary

Sequential, semi-supervised scRNA-seq batch correction with STACAS: anchors between datasets are computed and filtered by rPCA distance, a hierarchical sample tree orders the integration, and IntegrateData.STACAS with semisupervised=TRUE uses prior cell type labels to preserve biological variability while removing batch effects.

## When to use

Use when integrating many single-cell RNA-seq datasets (e.g., hundreds of samples, hundreds of thousands of cells) with partial cell type annotations, where batch effects must be corrected while preserving cell-type-specific biology.

## When NOT to use

- Data is not single-cell transcriptomics or lacks gene-expression matrices per sample (e.g., already-integrated feature tables or bulk RNA-seq).
- The goal is purely unsupervised integration with no cell type labels and no reference — the semi-supervised leverage of prior labels is the core of this approach.
- Only two samples need trivial merging where hierarchical tree ordering adds no value.

## Inputs

- Per-sample Seurat objects (e.g., processed .rds gene-expression objects from 10x Cell Ranger runs)
- An annotated seed/reference Seurat object with cell type labels
- Prior cell type annotations (e.g., scGate classes) for semi-supervised integration

## Outputs

- Batch-corrected integrated Seurat object with a new integrated assay (e.g., integrated_atlas.rds)
- Filtered STACAS anchor set and hierarchical sample tree
- Transferred subtype labels for newly integrated cells (via annotate.by.neighbors)

## How to apply

Compute cross-dataset anchors with FindAnchors.STACAS, then filter them by pairwise rPCA distance using FilterAnchors.STACAS to remove spurious anchor pairs. Determine the hierarchical ordering of datasets automatically with SampleTree.STACAS, rooting the tree at the annotated reference/seed so the trusted map anchors the sequential merge. Correct batch effects with IntegrateData.STACAS in sequential mode with semisupervised=TRUE, so prior cell type labels constrain the correction and preserve biological variability (e.g., CD8 T cell subtype signatures). Optionally transfer subtype labels to newly integrated cells with annotate.by.neighbors (k=20 nearest neighbors in the integrated embedding) and validate that the integrated object retains expected per-subtype marker-gene profiles and the expected cell/dataset counts.

## Related tools

- **STACAS** (Core semi-supervised batch correction package: FindAnchors.STACAS, FilterAnchors.STACAS, SampleTree.STACAS, and IntegrateData.STACAS are all executed with STACAS functions) — https://github.com/carmonalab/STACAS
- **Seurat** (Underlying single-cell framework into which STACAS anchors and integration are computed (anchor-based integration in Seurat))
- **annotate.by.neighbors** (Transfers subtype labels from the annotated seed reference to newly integrated cells using k-nearest-neighbor classification (k=20) in the integrated embedding) — https://github.com/carmonalab/STACAS

## Examples

```
anchors <- FindAnchors.STACAS(obj.list); anchors <- FilterAnchors.STACAS(anchors); tree <- SampleTree.STACAS(anchors); integrated <- IntegrateData.STACAS(anchors, sample.tree=tree, semisupervised=TRUE)
```

## Evaluation signals

- The integrated object contains the expected number of cells and datasets (e.g., 571,672 cells across 285 datasets in the uTILity CD8 T cell atlas integration).
- Anchor distance distributions pass FilterAnchors.STACAS filtering based on pairwise rPCA distance (inspected via PlotAnchors.STACAS).
- Per-subtype average marker-gene expression profiles of transferred labels match expected seed-derived CD8 T cell subtype signatures on expert review.
- Batch mixing and cell type separation of the integrated embedding can be quantified with scIntegrationMetrics (LISI, CiLISI, silhouette ASW).
- Semi-supervised mode preserves biological variability better than the unsupervised mode, consistent with the paper's benchmark showing semi-supervised STACAS outperforms unsupervised methods.

## Limitations

- STACAS semi-supervised mode requires at least partial cell type annotations; without labels, the biological-preservation advantage diminishes.
- STACAS is designed for scRNA-seq batch correction, not for cross-modality or non-transcriptomic data integration.
- Integration quality metrics such as CiLISI require cell type annotations, cannot be computed for cell types present in only one dataset, and remain suboptimal when datasets have highly unequal cell numbers.

## Evidence

- [methods] SampleTree.STACAS ordering and semisupervised integration: "`SampleTree.STACAS`   Determine automatically a hierarchical tree for Seurat integration using anchors calculated in STACAS"
- [methods] Anchor computation and filtering: "`FindAnchors.STACAS`  Computes anchors between datasets for integration in Seurat"
- [abstract] Semi-supervised advantage: "semi-supervised STACAS outperforms state-of-the-art unsupervised methods, as well as supervised methods such as scANVI and scGen."
- [abstract] Rationale for semi-supervision: "In this work we present STACAS, a batch correction method for scRNA-seq that leverages prior knowledge on cell types to preserve biological variability upon integration."
- [readme] Applied in practice with prior scGate labels: [evidence span withheld — anchor status: not_found]
