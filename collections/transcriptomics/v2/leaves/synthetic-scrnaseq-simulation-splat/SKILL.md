---
name: synthetic-scrnaseq-simulation-splat
description: Use when you need ground-truth-controlled scRNA-seq datasets (known batches,
  known cell types) to reproduce or validate integration-metric behavior, e.g., checking
  whether iLISI favors overcorrected embeddings.
when_to_use_negative:
- The dataset is real experimental data without known ground-truth cell type/batch
  labels for all cells — CiLISI cannot be calculated for cell types present in only
  one dataset and requires annotations
- Datasets have highly unequal numbers of cells — LISI/CiLISI remain suboptimal in
  this regime, so metric conclusions would be unreliable
- The goal is to evaluate batch mixing of technical replicates only, where cell type-agnostic
  metrics suffice and simulation of DE structure adds unnecessary complexity
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Splatter
  role: Simulates scRNA-seq count matrices for the four scenarios with tuned batch
    effect (batch.facLoc/batch.facScale) and cell type DE (de.facLoc/de.facScale)
    parameters
- name: STACAS
  role: Integrates each simulated dataset via its Run.STACAS wrapper (anchor finding,
    anchor filtering, data integration)
  repo: https://github.com/carmonalab/STACAS
- name: scIntegrationMetrics
  role: Computes iLISI, CiLISI, ASW, and normalized cLISI on the integrated embeddings
  repo: https://github.com/carmonalab/scIntegrationMetrics
- name: Seurat v4
  role: Object framework in which simulated data are integrated and metrics computed
    (STACAS anchors operate on Seurat objects)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:255f249da7acbc0a323502b75f592284a70a5fc08e3620df3ced7465c613abc3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - Seurat v4
  - Splatter
  - scIntegrationMetrics
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Synthetic scRNA-seq simulation with Splatter for integration-metric validation

## Summary

Generates simulated scRNA-seq count matrices with Splatter under controlled batch-effect and cell-type-DE scenarios, then integrates them with STACAS and scores them with scIntegrationMetrics (iLISI, CiLISI, ASW) to test whether batch-mixing metrics penalize variance-preserving integration.

## When to use

Use when you need ground-truth-controlled scRNA-seq datasets (known batches, known cell types) to reproduce or validate integration-metric behavior, e.g., checking whether iLISI favors overcorrected embeddings.

## When NOT to use

- The dataset is real experimental data without known ground-truth cell type/batch labels for all cells — CiLISI cannot be calculated for cell types present in only one dataset and requires annotations
- Datasets have highly unequal numbers of cells — LISI/CiLISI remain suboptimal in this regime, so metric conclusions would be unreliable
- The goal is to evaluate batch mixing of technical replicates only, where cell type-agnostic metrics suffice and simulation of DE structure adds unnecessary complexity

## Inputs

- Simulation design parameters (n cells, n batches, n cell types, batch.facLoc/batch.facScale, de.facLoc/de.facScale)
- Cell type labels for the simulated cells (for CiLISI and per-type ASW)
- Batch labels for the simulated cells (for iLISI)

## Outputs

- Splatter-simulated scRNA-seq count matrices for each scenario (e.g., batch0, overcorrected)
- STACAS-integrated embeddings per scenario
- Table of integration metric values (iLISI, CiLISI, ASW, normalized cLISI) keyed by scenario and metric

## How to apply

Simulate count matrices with Splatter's groups mode for four scenarios (1000 cells, 3 batches, 2 cell types), tuning batch.facLoc/batch.facScale to set batch effect strength and de.facLoc/de.facScale to set cell type differential expression; construct the overcorrected scenario so that batch mixing destroys cell type structure. Integrate each simulated dataset with STACAS via the Run.STACAS wrapper (which runs the complete anchor-finding, anchor filtering by pairwise rPCA distance, and data integration). Compute integration metrics on each integrated embedding using the scIntegrationMetrics package: iLISI (batch mixing), CiLISI (cell type-aware label mixing), cell type average silhouette width (ASW), and normalized cLISI, assembling values into a table keyed by scenario and metric. Rationale: with known ground truth, metric behavior can be judged directly — e.g., iLISI should rise from ~1.75 (batch0 scenario) toward ~2.75 (overcorrected, approaching the 3-batch maximum) while CiLISI stays comparable across no-batch-effect scenarios regardless of biological variance.

## Related tools

- **Splatter** (Simulates scRNA-seq count matrices for the four scenarios with tuned batch effect (batch.facLoc/batch.facScale) and cell type DE (de.facLoc/de.facScale) parameters)
- **STACAS** (Integrates each simulated dataset via its Run.STACAS wrapper (anchor finding, anchor filtering, data integration)) — https://github.com/carmonalab/STACAS
- **scIntegrationMetrics** (Computes iLISI, CiLISI, ASW, and normalized cLISI on the integrated embeddings) — https://github.com/carmonalab/scIntegrationMetrics
- **Seurat v4** (Object framework in which simulated data are integrated and metrics computed (STACAS anchors operate on Seurat objects))

## Examples

```
library(Splatter); sim <- splatSimulate(nGenes=1000, group.prob=c(0.5,0.5), method="groups", nBatches=3, batch.facLoc=0.5, batch.facScale=0.5, de.facLoc=1, de.facScale=1)
```

## Evaluation signals

- iLISI rises from ~1.75 in the batch0 scenario to ~2.75 in the overcorrected scenario (approaching the 3-batch maximum)
- CiLISI remains comparable across the no-batch-effect scenarios regardless of the level of biological (cell type DE) variance
- Each scenario uses the designed dimensions: 1000 cells, 3 batches, 2 cell types
- Metric table contains values for all four scenarios × all four metrics (iLISI, CiLISI, ASW, normalized cLISI) with no missing cells
- Integrated embeddings are produced by the full STACAS chain (FindAnchors.STACAS → FilterAnchors.STACAS → IntegrateData.STACAS) rather than raw uncorrected matrices

## Limitations

- CiLISI and similar cell type-aware metrics require cell type annotations
- CiLISI cannot be calculated for cell types that are only present in one dataset
- As LISI, CiLISI remains suboptimal when datasets have highly unequal numbers of cells; batch mixing metrics robust to lopsided cell proportions are still lacking

## Evidence

- [methods] Wrapper function to run the complete anchor-finding algorithm in STACAS.: "Wrapper function to run the complete anchor-finding algorithm in STACAS."
- [discussion] CiLISI requires annotations: "One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations."
- [readme] scIntegrationMetrics description: [evidence span withheld — anchor status: external_doc]
- [abstract] STACAS motivation: "In this work we present STACAS, a batch correction method for scRNA-seq that leverages prior knowledge on cell types to preserve biological variability upon integration."
- [methods] Anchor filtering step: "Filters integration anchors based on pairwise rPCA distance, calculated using `FindAnchors.STACAS`"
