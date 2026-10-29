---
name: semi-supervised-scrna-seq-integration
description: Use when integrating multiple scRNA-seq batches where partial or imperfect
  cell type labels are available and preserving cell-type biological variance (e.g.,
  celltype ASW, CiLISI) is a priority alongside batch mixing.
when_to_use_negative:
- Cell type labels are entirely absent and cannot be partially inferred — the method's
  advantage depends on prior cell type knowledge
- A cell type is present in only one dataset — CiLISI cannot be calculated for such
  cell types, so evaluation of cell-type-aware batch mixing is suboptimal
- Datasets have highly unequal numbers of cells — LISI/CiLISI batch-mixing metrics
  remain suboptimal under lopsided cell proportions
edam_operation: http://edamontology.org/operation_0566
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STACAS
  role: Semi-supervised anchor-based batch correction method; FindAnchors.STACAS,
    FilterAnchors.STACAS, SampleTree.STACAS, IntegrateData.STACAS (semisupervised
    mode) implement the integration
  repo: https://github.com/carmonalab/STACAS
- name: scib-pipeline
  role: Snakemake benchmarking pipeline for (semi)supervised integration tools, based
    on scib, used to run and score integrations
  repo: https://github.com/carmonalab/scib-pipeline
- name: scIntegrationMetrics
  role: R package computing LISI, per-cell-type CiLISI, and ASW metrics to assess
    batch mixing and cell type separation of the integrated embedding
  repo: https://github.com/carmonalab/scIntegrationMetrics
- name: scANVI
  role: Supervised deep-learning integration baseline; robust to label shuffling,
    benchmarked against semi-supervised STACAS
- name: scGen
  role: Supervised integration baseline; drops sharply in performance at 10-20% shuffled
    input labels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:8872d3a3e6ea2c8dce8bbf4f95a33b3df051954c06e185d80e36aed5620382ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - scANVI
  - scGen
  - scIntegrationMetrics
  - scib-pipeline
  edam_operation: http://edamontology.org/operation_0566
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# semi-supervised-scrna-seq-integration

## Summary

Semi-supervised integration of scRNA-seq datasets using STACAS, which leverages prior cell type annotations during anchor-based batch correction to preserve biological variability while removing technical batch effects.

## When to use

Use when integrating multiple scRNA-seq batches where partial or imperfect cell type labels are available and preserving cell-type biological variance (e.g., celltype ASW, CiLISI) is a priority alongside batch mixing.

## When NOT to use

- Cell type labels are entirely absent and cannot be partially inferred — the method's advantage depends on prior cell type knowledge
- A cell type is present in only one dataset — CiLISI cannot be calculated for such cell types, so evaluation of cell-type-aware batch mixing is suboptimal
- Datasets have highly unequal numbers of cells — LISI/CiLISI batch-mixing metrics remain suboptimal under lopsided cell proportions

## Inputs

- Multiple scRNA-seq batches/datasets as Seurat objects with cell type annotations in metadata
- Prior cell type labels (partial or noisy, can include 'unknown' or shuffled labels)
- EnsemblGeneTable.Hs or EnsemblGeneTable.Mm for gene symbol standardization
- genes.blocklist for FindVariableFeatures.STACAS feature selection

## Outputs

- Integrated Seurat object with a new assay (new.assay.name) and integrated dimensional reduction
- Integration anchor set from FindAnchors.STACAS / FilterAnchors.STACAS
- Benchmark scores (celltype_ASW, CiLISI) per method and corruption level, e.g. label_corruption_benchmark_scores.csv

## How to apply

Run the STACAS anchor-finding pipeline: compute anchors between datasets with FindAnchors.STACAS, filter them by pairwise rPCA distance (FilterAnchors.STACAS), optionally determine an integration tree with SampleTree.STACAS, and integrate via IntegrateData.STACAS with semisupervised mode enabled so prior cell type labels inform the integrated embedding. Prior to integration, select features with FindVariableFeatures.STACAS (using genesBlockList to exclude problematic genes) and standardize gene symbols with StandardizeGeneSymbols using EnsemblGeneTable.Hs/Mm. Cell type labels may be incomplete or noisy: STACAS was shown to be robust to missing ('unknown') and shuffled labels, unlike supervised methods (scGen drops sharply at 10-20% shuffled labels). Evaluate the integration against true labels with scib metrics celltype_ASW and CiLISI computed via carmonalab/scIntegrationMetrics, benchmarking within the modified carmonalab/scib-pipeline Snakemake framework.

## Related tools

- **STACAS** (Semi-supervised anchor-based batch correction method; FindAnchors.STACAS, FilterAnchors.STACAS, SampleTree.STACAS, IntegrateData.STACAS (semisupervised mode) implement the integration) — https://github.com/carmonalab/STACAS
- **scib-pipeline** (Snakemake benchmarking pipeline for (semi)supervised integration tools, based on scib, used to run and score integrations) — https://github.com/carmonalab/scib-pipeline
- **scIntegrationMetrics** (R package computing LISI, per-cell-type CiLISI, and ASW metrics to assess batch mixing and cell type separation of the integrated embedding) — https://github.com/carmonalab/scIntegrationMetrics
- **scANVI** (Supervised deep-learning integration baseline; robust to label shuffling, benchmarked against semi-supervised STACAS)
- **scGen** (Supervised integration baseline; drops sharply in performance at 10-20% shuffled input labels)

## Examples

```
anchors <- FindAnchors.STACAS(obj.list, reduction="pca"); anchors <- FilterAnchors.STACAS(anchors); integrated <- IntegrateData.STACAS(anchors, new.assay.name="STACAS", semisupervised=TRUE)
```

## Evaluation signals

- celltype_ASW on the integrated embedding, evaluated against TRUE (uncorrupted) labels, should be preserved and higher than scANVI across benchmark tasks (Pancreas, Immune, Lung, T cells)
- CiLISI indicates cell-type-aware batch mixing; verify per-cell-type LISI values are computed for cell types present in at least two datasets
- Robustness check: performance curves over label-shuffling (0-100%) and unknown-label (0-100%) fractions should show ssSTACAS and scANVI remain robust while scGen drops sharply at 10-20% shuffling
- Anchor quality: PlotAnchors.STACAS distributions of anchor distances between datasets should show a plausible low-distance anchor population before/after FilterAnchors.STACAS

## Limitations

- CiLISI and similar cell type-aware metrics require cell type annotations
- CiLISI cannot be calculated for cell types that are only present in one dataset
- As LISI, CiLISI remains suboptimal when datasets have highly unequal numbers of cells
- Batch mixing metrics that are more robust to lopsided cell proportions are still lacking
- Benchmarks including supervised integration methods should account for noisy and partial cell-type information

## Evidence

- [abstract] STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels: "STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels, which are commonly encountered in real-life integration tasks."
- [abstract] semi-supervised STACAS outperforms scANVI and scGen: "Through an open-source benchmark, we show that semi-supervised STACAS outperforms state-of-the-art unsupervised methods, as well as supervised methods such as scANVI and scGen."
- [methods] IntegrateData.STACAS with semisupervised parameter: "`IntegrateData.STACAS(anchorset, new.assay.name, features.to.integrate, dims, k.weight, sample.tree, hclust.method, semisupervised, verbose)`"
- [methods] FilterAnchors.STACAS filters anchors by rPCA distance: "`FilterAnchors.STACAS`  Filters integration anchors based on pairwise rPCA distance, calculated using `FindAnchors.STACAS`"
- [discussion] CiLISI limitations require annotations: "One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations."
- [readme] STACAS README robustness claim: "STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels, which are commonly encountered in real-life integration tasks."
