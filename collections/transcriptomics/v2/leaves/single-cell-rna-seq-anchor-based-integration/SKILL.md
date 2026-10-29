---
name: single-cell-rna-seq-anchor-based-integration
description: Use when multiple scRNA-seq samples or datasets need batch correction
  and integration, and prior (even partial or noisy) cell type annotations are available
  to preserve biological variance.
when_to_use_negative:
- Input consists of a single dataset with no batches to correct — anchor-based integration
  is unnecessary
- No cell type information is available and unbiased batch mixing (e.g., of technical
  replicates only) is the sole goal — cell type-agnostic batch mixing metrics/integration
  may suffice, though STACAS remains robust to missing labels
- Cell types of interest are present in only one dataset, making alignment and evaluation
  (e.g., CiLISI) uninformative
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STACAS
  role: 'Semi-supervised anchor-based integration: FindAnchors.STACAS, FilterAnchors.STACAS,
    SampleTree.STACAS, IntegrateData.STACAS, annotate.by.neighbors'
  repo: https://github.com/carmonalab/STACAS
- name: Seurat
  role: Underlying framework in which STACAS computes anchors between datasets and
    performs integration (Seurat v4 comparison baseline)
- name: scIntegrationMetrics
  role: Post-integration quality assessment with LISI, per-cell type CiLISI, and silhouette
    (ASW) metrics on the integrated embedding
  repo: https://github.com/carmonalab/scIntegrationMetrics
- name: scGate
  role: Provides the pre-defined cell type classes used as prior knowledge for semi-supervised
    STACAS integration
  repo: https://github.com/carmonalab/scGate_models
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:70bd77a290fc7119084e7ed8243b832292db1d9602c4eba727d9a71faacd6f62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - Seurat
  - scGate
  - scIntegrationMetrics
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# single-cell-rna-seq-anchor-based-integration

## Summary

Semi-supervised, anchor-based batch-effect correction of scRNA-seq datasets using STACAS, which leverages prior cell type labels to preserve biological variability while integrating multiple samples into a common embedding.

## When to use

Use when multiple scRNA-seq samples or datasets need batch correction and integration, and prior (even partial or noisy) cell type annotations are available to preserve biological variance.

## When NOT to use

- Input consists of a single dataset with no batches to correct — anchor-based integration is unnecessary
- No cell type information is available and unbiased batch mixing (e.g., of technical replicates only) is the sole goal — cell type-agnostic batch mixing metrics/integration may suffice, though STACAS remains robust to missing labels
- Cell types of interest are present in only one dataset, making alignment and evaluation (e.g., CiLISI) uninformative

## Inputs

- Per-sample scRNA-seq Seurat objects (e.g., Seurat .rds objects from 10x filtered gene matrices)
- Prior cell type annotations/labels for at least a subset of cells (e.g., scGate-derived classes)
- A sample.tree / anchorset from STACAS when integrating sequentially
- Ensembl gene table (EnsemblGeneTable.Hs / EnsemblGeneTable.Mm) for gene symbol standardization

## Outputs

- A batch-corrected integrated Seurat object with a new assay (new.assay.name) and integrated embedding (e.g., integrated_atlas.rds)
- A filtered anchor set with pairwise rPCA distance scores
- A hierarchical sample tree determining sequential integration order
- Transferred cell type labels for newly integrated cells (annotate.by.neighbors, k=20)

## How to apply

Compute integration anchors between dataset pairs with FindAnchors.STACAS, then filter anchors by pairwise rPCA distance using FilterAnchors.STACAS (inspect distance distributions first with PlotAnchors.STACAS). Determine the hierarchical merge order automatically with SampleTree.STACAS, and run batch correction with IntegrateData.STACAS in sequential mode, setting semisupervised=TRUE when cell type labels exist so biological variability between cell types is preserved rather than over-mixed. Genes used for anchoring can be selected with FindVariableFeatures.STACAS (with genesBlockList to exclude problematic features) after StandardizeGeneSymbols harmonizes gene IDs via an Ensembl gene table. If a labeled reference is available, transfer labels to newly integrated cells with annotate.by.neighbors using k=20 nearest neighbors in the integrated embedding, as done to label 553,077 cells from 265 Utility datasets onto an annotated CD8 T cell seed map.

## Related tools

- **STACAS** (Semi-supervised anchor-based integration: FindAnchors.STACAS, FilterAnchors.STACAS, SampleTree.STACAS, IntegrateData.STACAS, annotate.by.neighbors) — https://github.com/carmonalab/STACAS
- **Seurat** (Underlying framework in which STACAS computes anchors between datasets and performs integration (Seurat v4 comparison baseline))
- **scIntegrationMetrics** (Post-integration quality assessment with LISI, per-cell type CiLISI, and silhouette (ASW) metrics on the integrated embedding) — https://github.com/carmonalab/scIntegrationMetrics
- **scGate** (Provides the pre-defined cell type classes used as prior knowledge for semi-supervised STACAS integration) — https://github.com/carmonalab/scGate_models

## Examples

```
anchors <- FindAnchors.STACAS(list(seed, util)); anchors <- FilterAnchors.STACAS(anchors); tree <- SampleTree.STACAS(anchors); integrated <- IntegrateData.STACAS(anchors, sample.tree=tree, semisupervised=TRUE)
```

## Evaluation signals

- Anchor distance distributions between datasets are inspectable via PlotAnchors.STACAS and filtered anchors show acceptable pairwise rPCA distances
- Integrated embedding preserves cell type separation: CiLISI/ASW metrics from scIntegrationMetrics show biological variance retained while batches mix
- Semi-supervised STACAS outperforms unsupervised baselines (Harmony, FastMNN, Seurat v4, scVI, Scanorama) and supervised baselines (scANVI, scGen) in the scib-pipeline benchmark
- Label transfer sanity check: per-transferred-label average marker-gene expression profiles match expected CD8 T cell subtype signatures (e.g., 571,672 cells from 285 datasets retaining subtype-specific markers)
- Integrated object contains the expected cell and dataset counts and a new assay named via new.assay.name

## Limitations

- CiLISI and similar cell type-aware integration metrics require cell type annotations and cannot be calculated for cell types present in only one dataset
- CiLISI remains suboptimal when datasets have highly unequal numbers of cells; batch mixing metrics robust to lopsided cell proportions are still lacking
- Benchmarks including supervised integration methods should account for noisy and partial cell-type information; cell type-agnostic batch mixing metrics are arguably very limited and only relevant for technical replicates

## Evidence

- [abstract] Semi-supervised STACAS leverages cell type labels for integration: "we present STACAS, a batch correction method for scRNA-seq that leverages prior knowledge on cell types to preserve biological variability upon integration"
- [abstract] STACAS scales and is robust to imperfect labels: "STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels"
- [methods] Anchor workflow functions: "`FilterAnchors.STACAS`  Filters integration anchors based on pairwise rPCA distance, calculated using `FindAnchors.STACAS`"
- [other] Sequential semi-supervised integration and label transfer: [evidence span withheld — anchor status: not_found]
- [readme] STACAS used for CD8 TIL atlas construction: [evidence span withheld — anchor status: not_found]
- [discussion] Robustness to partial/imperfect annotations: "we showed that even partial and imperfect annotations can be beneficial towards preserving biological variance while correcting for technical batch effects"
