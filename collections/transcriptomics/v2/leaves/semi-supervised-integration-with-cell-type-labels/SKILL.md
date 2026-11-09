---
name: semi-supervised-integration-with-cell-type-labels
description: Use when integrating multiple scRNA-seq datasets where partial or imperfect
  cell type labels exist for at least a reference subset, and preserving cell-type
  biology (e.g., CD8 T cell subtypes) while removing batch effects matters.
when_to_use_negative:
- No cell type annotations of any quality are available for any dataset and the goal
  is purely unsupervised technical-replicate merging, where cell type-agnostic batch
  mixing suffices
- The task is evaluating integration quality rather than performing it — use scIntegrationMetrics
  (LISI/CiLISI/ASW) instead
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STACAS
  role: 'Semi-supervised batch correction: FindAnchors.STACAS, FilterAnchors.STACAS,
    SampleTree.STACAS, IntegrateData.STACAS, and annotate.by.neighbors'
  repo: https://github.com/carmonalab/STACAS
- name: Seurat
  role: Underlying single-cell framework in which STACAS anchors and integration are
    computed (Seurat v4 comparison baseline; FindAnchors.STACAS computes anchors for
    integration in Seurat)
- name: scGate
  role: Provides pre-defined cell type classes used as prior knowledge for the semi-supervised
    STACAS integration in the CD8 TIL atlas construction
  repo: https://github.com/carmonalab/scGate_models
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:aecbf8224d54ee27299ae301c7fe8e5efb068f7814b8083ec59bb956f4b93ca5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - Seurat
  - scGate
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# semi-supervised-integration-with-cell-type-labels

## Summary

Batch-effect correction of scRNA-seq collections with STACAS in semi-supervised mode, supplying prior cell type annotations (e.g., scGate-derived CD8 T cell classes) so biological variability is preserved during integration of many datasets.

## When to use

Use when integrating multiple scRNA-seq datasets where partial or imperfect cell type labels exist for at least a reference subset, and preserving cell-type biology (e.g., CD8 T cell subtypes) while removing batch effects matters.

## When NOT to use

- No cell type annotations of any quality are available for any dataset and the goal is purely unsupervised technical-replicate merging, where cell type-agnostic batch mixing suffices
- The task is evaluating integration quality rather than performing it — use scIntegrationMetrics (LISI/CiLISI/ASW) instead

## Inputs

- Per-dataset Seurat objects (e.g., .rds files from the ncborcherding/utility deposit)
- An annotated seed/reference map with cell type labels (e.g., CD8 T cell subtype annotations from task_005)
- Pre-computed STACAS anchor set (output of FindAnchors.STACAS)

## Outputs

- Integrated Seurat object with a corrected assay (new.assay.name) across all datasets (e.g., integrated_atlas.rds with 571,672 cells from 285 datasets)
- Hierarchical sample tree from SampleTree.STACAS defining sequential integration order
- Label-transferred annotations for query cells via annotate.by.neighbors

## How to apply

Compute anchors between a labeled reference and each query with FindAnchors.STACAS, then filter them by pairwise rPCA distance using FilterAnchors.STACAS. Determine the merge order automatically with SampleTree.STACAS, rooting the tree at the annotated seed reference. Run IntegrateData.STACAS with sample.tree and semisupervised=TRUE so the provided cell type labels constrain the correction and preserve biological variability (e.g., CD8 T cell subtype signatures in the uTILity collection). For large sequential builds, integrate remaining datasets onto the seed map iteratively and transfer labels to new cells with annotate.by.neighbors (k=20 nearest neighbors in the integrated embedding). STACAS is robust to incomplete and imprecise input labels, so partially annotated queries can still be included.

## Related tools

- **STACAS** (Semi-supervised batch correction: FindAnchors.STACAS, FilterAnchors.STACAS, SampleTree.STACAS, IntegrateData.STACAS, and annotate.by.neighbors) — https://github.com/carmonalab/STACAS
- **Seurat** (Underlying single-cell framework in which STACAS anchors and integration are computed (Seurat v4 comparison baseline; FindAnchors.STACAS computes anchors for integration in Seurat))
- **scGate** (Provides pre-defined cell type classes used as prior knowledge for the semi-supervised STACAS integration in the CD8 TIL atlas construction) — https://github.com/carmonalab/scGate_models

## Examples

```
anchors <- FilterAnchors.STACAS(FindAnchors.STACAS(seed, query)); integrated <- IntegrateData.STACAS(anchors, sample.tree = stree, semisupervised = TRUE)
```

## Evaluation signals

- Integrated object contains the expected cell and dataset counts (e.g., 571,672 cells across 285 datasets in the uTILity build)
- Per-subtype average marker-gene expression profiles of transferred labels match the seed-derived CD8 T cell subtype signatures (expert marker concordance review)
- CiLISI (per-cell type LISI) computed with scIntegrationMetrics shows batch mixing without erosion of cell type separation
- Anchor distance distributions inspected with PlotAnchors.STACAS confirm filtered anchors cluster at low rPCA distance
- Semi-supervised benchmark via scib-pipeline shows STACAS compares favorably to Harmony, Seurat v4, scVI, scANVI, and scGen on preservation of biological variance

## Limitations

- CiLISI and similar cell type-aware evaluation metrics require cell type annotations and cannot be calculated for cell types present in only one dataset
- Like LISI, CiLISI remains suboptimal when datasets have highly unequal numbers of cells; batch mixing metrics robust to lopsided cell proportions are still lacking
- Benchmarks including supervised integration methods should account for noisy and partial cell-type information; even partial and imperfect annotations are beneficial but label quality affects the result

## Evidence

- [abstract] Semi-supervised STACAS outperforms state-of-the-art unsupervised methods and supervised methods such as scANVI and scGen: "Through an open-source benchmark, we show that semi-supervised STACAS outperforms state-of-the-art unsupervised methods, as well as supervised methods such as scANVI and scGen."
- [methods] STACAS filters anchors by rPCA distance: "`FilterAnchors.STACAS`  Filters integration anchors based on pairwise rPCA distance, calculated using `FindAnchors.STACAS`"
- [methods] IntegrateData.STACAS supports semisupervised mode: "`IntegrateData.STACAS(anchorset, new.assay.name, features.to.integrate, dims, k.weight, sample.tree, hclust.method, semisupervised, verbose)`"
- [readme] Semi-supervised STACAS used with scGate classes in TIL atlas construction: [evidence span withheld — anchor status: not_found]
- [abstract] STACAS scales to large datasets and tolerates imperfect labels: "STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels, which are commonly encountered in real-life integration tasks."
