---
name: integration-benchmark-metric-visualization
description: Use when evaluating or visualizing how well integrated scRNA-seq embeddings
  mix batches while preserving cell types, especially to detect overcorrection where
  batch mixing destroys biological structure.
when_to_use_negative:
- Cell type annotations are unavailable — CiLISI and similar cell type-aware metrics
  require cell type labels and cannot be computed without them
- Cell types are present in only one dataset (e.g., a dataset with a unique cell type),
  where CiLISI cannot be calculated
- Datasets have highly unequal numbers of cells per batch — LISI/CiLISI are suboptimal
  and batch-mixing scores can be misleading with lopsided cell proportions
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_3474
tools:
- name: scIntegrationMetrics
  role: Computes iLISI, per-cell-type CiLISI, normalized cLISI, and cell type ASW
    on integrated Seurat embeddings for benchmarking
  repo: https://github.com/carmonalab/scIntegrationMetrics
- name: STACAS
  role: Semi-supervised anchor-based integration producing the embeddings on which
    metrics are evaluated; Run.STACAS wraps anchor finding, filtering, and integration
  repo: https://github.com/carmonalab/STACAS
- name: Splatter
  role: Simulates synthetic scRNA-seq count matrices with controlled batch effect
    and cell type DE strength to create benchmark scenarios
- name: Seurat v4
  role: Object model and integration framework within which STACAS anchors and integrated
    embeddings are stored and metrics are computed
- name: scib-pipeline
  role: Snakemake benchmarking pipeline that computes CiLISI via scIntegrationMetrics
    to compare (semi-)supervised integration tools
  repo: https://github.com/carmonalab/scib-pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:cbb2733ff2206f5d4d01461b37eb574e45a79dd64d9ceaa9b43551f2bd9cb7ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/scIntegrationMetrics
  tools:
  - STACAS
  - Seurat v4
  - Splatter
  - scIntegrationMetrics
  - scib-pipeline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# integration-benchmark-metric-visualization

## Summary

Benchmark single-cell RNA-seq data integration by computing batch-mixing and cell-type-preservation metrics (iLISI, CiLISI, ASW, normalized cLISI) on integrated embeddings using the scIntegrationMetrics R package, with STACAS as the integration method.

## When to use

Use when evaluating or visualizing how well integrated scRNA-seq embeddings mix batches while preserving cell types, especially to detect overcorrection where batch mixing destroys biological structure.

## When NOT to use

- Cell type annotations are unavailable — CiLISI and similar cell type-aware metrics require cell type labels and cannot be computed without them
- Cell types are present in only one dataset (e.g., a dataset with a unique cell type), where CiLISI cannot be calculated
- Datasets have highly unequal numbers of cells per batch — LISI/CiLISI are suboptimal and batch-mixing scores can be misleading with lopsided cell proportions

## Inputs

- Seurat object with an integrated embedding (e.g., from STACAS IntegrateData.STACAS) plus batch and cell type metadata columns
- Splatter-simulated scRNA-seq count matrices parameterized by batch effect strength (batch.facLoc, batch.facScale) and cell type DE strength (de.facLoc, de.facScale)

## Outputs

- Table of integration metric values keyed by scenario and metric (iLISI, CiLISI, ASW, normalized cLISI)
- Per-cell-type LISI and silhouette scores from scIntegrationMetrics for inspecting batch mixing vs. cell type separation

## How to apply

Generate or load an integrated Seurat object (e.g., simulate scRNA-seq count matrices with Splatter across scenarios such as no-batch-effect, batch effect, and overcorrected — built by tuning batch.facLoc/batch.facScale and de.facLoc/de.facScale — then integrate with STACAS via Run.STACAS, FindAnchors.STACAS, FilterAnchors.STACAS, and IntegrateData.STACAS). Compute iLISI (batch mixing) and CiLISI (cell type label mixing) plus cell type average silhouette width (ASW) on the integrated embedding using scIntegrationMetrics, which extends immunogenomics/LISI with per-cell-type metrics and Seurat-object compatibility. Assemble metric values into a table keyed by scenario and metric. Judge correctness by checking that iLISI approaches the number of batches (e.g., rising from ~1.75 in a batch0 scenario to ~2.75 in an overcorrected scenario with 3 batches) while CiLISI remains comparable across no-batch-effect scenarios regardless of biological variance — this contrast demonstrates that plain iLISI favors overcorrected embeddings, motivating cell type-aware metrics.

## Related tools

- **scIntegrationMetrics** (Computes iLISI, per-cell-type CiLISI, normalized cLISI, and cell type ASW on integrated Seurat embeddings for benchmarking) — https://github.com/carmonalab/scIntegrationMetrics
- **STACAS** (Semi-supervised anchor-based integration producing the embeddings on which metrics are evaluated; Run.STACAS wraps anchor finding, filtering, and integration) — https://github.com/carmonalab/STACAS
- **Splatter** (Simulates synthetic scRNA-seq count matrices with controlled batch effect and cell type DE strength to create benchmark scenarios)
- **Seurat v4** (Object model and integration framework within which STACAS anchors and integrated embeddings are stored and metrics are computed)
- **scib-pipeline** (Snakemake benchmarking pipeline that computes CiLISI via scIntegrationMetrics to compare (semi-)supervised integration tools) — https://github.com/carmonalab/scib-pipeline

## Examples

```
integrated <- Run.STACAS(seurat_obj)
```

## Evaluation signals

- On Splatter-simulated data (1000 cells, 3 batches, 2 cell types), iLISI rises from ~1.75 in the batch0 scenario to ~2.75 in the overcorrected scenario, approaching the 3-batch maximum
- CiLISI scores the no-batch-effect scenarios similarly regardless of biological variance, unlike iLISI which favors the overcorrected embedding
- All metric values assemble into a single complete table keyed by scenario and metric with no missing values
- iLISI for a fully batch-mixed embedding approaches the number of batches (N) rather than 1, while a non-integrated embedding approaches ~1

## Limitations

- CiLISI and similar cell type-aware metrics require cell type annotations
- CiLISI cannot be calculated for cell types that are only present in one dataset
- Like LISI, CiLISI remains suboptimal when datasets have highly unequal numbers of cells, and batch mixing metrics robust to lopsided cell proportions are still lacking
- Cell type-agnostic batch mixing metrics have utility arguably limited to integration of technical replicates

## Evidence

- [readme] A collection of metrics to evaluate scRNA-seq data integration quality, including LISI, per-cell type LISI (CiLISI), and silhouette average width (ASW).: [evidence span withheld — anchor status: external_doc]
- [methods] Run.STACAS wrapper: "`Run.STACAS`   Wrapper function to run the complete anchor-finding algorithm in STACAS."
- [methods] IntegrateData.STACAS signature: "`IntegrateData.STACAS(anchorset, new.assay.name, features.to.integrate, dims, k.weight, sample.tree, hclust.method, semisupervised, verbose)`"
- [discussion] CiLISI requires annotations: "One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations."
- [discussion] CiLISI single-dataset cell types: "Another limitation is that CiLISI cannot be calculated for cell types that are only present in one dataset."
- [readme] CiLISI in benchmark pipeline: [evidence span withheld — anchor status: external_doc]
- [abstract] STACAS vs unsupervised and supervised methods: "we show that semi-supervised STACAS outperforms state-of-the-art unsupervised methods, as well as supervised methods such as scANVI and scGen."
