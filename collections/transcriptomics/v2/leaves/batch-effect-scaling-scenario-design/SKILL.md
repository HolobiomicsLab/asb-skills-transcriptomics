---
name: batch-effect-scaling-scenario-design
description: Use when benchmarking scRNA-seq integration methods or metrics and you
  need controlled ground truth where batch effect strength and cell type DE are known
  by construction (e.g., testing whether iLISI rewards overcorrection).
when_to_use_negative:
- Datasets with highly unequal numbers of cells per batch, since LISI and CiLISI remain
  suboptimal with lopsided cell proportions
- Cell types present in only one dataset, where CiLISI cannot be calculated
- Integration of purely technical replicates where no biological variance must be
  preserved and cell type-agnostic batch mixing metrics suffice
tools:
- name: Splatter
  role: Simulates scRNA-seq count matrices with tunable batch effect strength (batch.facLoc/batch.facScale)
    and cell type DE (de.facLoc/de.facScale) for the four scenarios
- name: STACAS
  role: Integrates each simulated dataset via the Run.STACAS wrapper (FindAnchors.STACAS,
    FilterAnchors.STACAS, IntegrateData.STACAS)
  repo: https://github.com/carmonalab/STACAS
- name: scIntegrationMetrics
  role: Computes iLISI, CiLISI, ASW, and normalized cLISI on the integrated embeddings
  repo: https://github.com/carmonalab/scIntegrationMetrics
- name: Seurat v4
  role: Object framework in which simulated data are held and integration embeddings
    are stored for metric computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:6b7f6ffeb4bb196a426796c8354d69834bc40e12886124c7681c97b2cee220b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - Seurat v4
  - Splatter
  - scIntegrationMetrics
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# batch-effect-scaling-scenario-design

## Summary

A simulation-based test design that generates synthetic scRNA-seq datasets with titrated batch effect strengths and biological variance using Splatter, to evaluate how integration metrics (iLISI, CiLISI, ASW) respond to overcorrected vs. variance-preserving integration.

## When to use

Use when benchmarking scRNA-seq integration methods or metrics and you need controlled ground truth where batch effect strength and cell type DE are known by construction (e.g., testing whether iLISI rewards overcorrection).

## When NOT to use

- Datasets with highly unequal numbers of cells per batch, since LISI and CiLISI remain suboptimal with lopsided cell proportions
- Cell types present in only one dataset, where CiLISI cannot be calculated
- Integration of purely technical replicates where no biological variance must be preserved and cell type-agnostic batch mixing metrics suffice

## Inputs

- Splatter simulation parameters (nCells=1000, nBatches=3, nCellTypes=2, batch.facLoc, batch.facScale, de.facLoc, de.facScale)
- Simulated scRNA-seq count matrices for each of the four scenarios (batch0, no-batch-effect variance-preserving, batch-effect, overcorrected)
- Cell type and batch labels for the simulated cells (used by CiLISI and ASW)

## Outputs

- Seurat objects with STACAS-integrated embeddings for each scenario
- Table of integration metric values (iLISI, CiLISI, ASW, normalized cLISI) keyed by scenario and metric
- Validation comparison showing iLISI trajectory across scenarios (~1.75 in batch0 to ~2.75 in overcorrected)

## How to apply

Simulate count matrices with Splatter under four scenarios (1000 cells, 3 batches, 2 cell types), tuning batch.facLoc/batch.facScale to set batch effect strength and de.facLoc/de.facScale to set cell type differential expression; construct the 'overcorrected' scenario so that batch mixing destroys cell type structure. Integrate each simulated dataset with STACAS via the Run.STACAS wrapper (anchor finding with FindAnchors.STACAS, anchor filtering with FilterAnchors.STACAS based on pairwise rPCA distance, then IntegrateData.STACAS). Compute integration metrics on each integrated embedding using the scIntegrationMetrics package: iLISI (batch mixing), CiLISI (cell type label mixing), cell type average silhouette width (ASW), and normalized cLISI. Assemble metric values into a table keyed by scenario and metric. The rationale is that cell type-agnostic iLISI approaches the number of batches (3) under overcorrection, while the cell type-aware CiLISI discriminates no-batch-effect scenarios regardless of preserved biological variance.

## Related tools

- **Splatter** (Simulates scRNA-seq count matrices with tunable batch effect strength (batch.facLoc/batch.facScale) and cell type DE (de.facLoc/de.facScale) for the four scenarios)
- **STACAS** (Integrates each simulated dataset via the Run.STACAS wrapper (FindAnchors.STACAS, FilterAnchors.STACAS, IntegrateData.STACAS)) — https://github.com/carmonalab/STACAS
- **scIntegrationMetrics** (Computes iLISI, CiLISI, ASW, and normalized cLISI on the integrated embeddings) — https://github.com/carmonalab/scIntegrationMetrics
- **Seurat v4** (Object framework in which simulated data are held and integration embeddings are stored for metric computation)

## Examples

```
library(splatter); sim <- splatSimulate(groupCells=500, nGroups=2, batchCells=c(333,333,334), batch.facLoc=0.5, batch.facScale=0.5, de.facLoc=1, de.facScale=1)
```

## Evaluation signals

- iLISI rises from ~1.75 in the batch0 scenario to ~2.75 in the overcorrected scenario, approaching the total of 3 batches
- CiLISI remains comparable across the no-batch-effect scenarios regardless of whether biological variance is preserved
- ASW computed per cell type confirms cell type structure is destroyed in the overcorrected scenario but preserved in variance-preserving ones
- Each scenario yields metrics for all four quantities (iLISI, CiLISI, ASW, normalized cLISI), i.e., the scenario-by-metric table is complete
- STACAS integration completes for every scenario with anchors passing FilterAnchors.STACAS rPCA-distance filtering

## Limitations

- CiLISI and similar cell type-aware metrics require cell type annotations
- CiLISI cannot be calculated for cell types that are only present in one dataset
- Batch mixing metrics that are more robust to lopsided cell proportions are still lacking, so simulated batch sizes should be balanced
- Cell type-agnostic batch mixing metrics like iLISI have utility mainly for integration of technical replicates

## Evidence

- [methods] Wrapper function to run the complete anchor-finding algorithm in STACAS.: "Wrapper function to run the complete anchor-finding algorithm in STACAS."
- [readme] readme scIntegrationMetrics scope: [evidence span withheld — anchor status: external_doc]
- [discussion] CiLISI annotation requirement: "One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations."
- [discussion] lopsided proportions limitation: "Batch mixing metrics that are more robust to lopsided cell proportions are still lacking"
