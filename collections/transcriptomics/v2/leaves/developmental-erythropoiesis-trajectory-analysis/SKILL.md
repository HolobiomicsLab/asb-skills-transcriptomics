---
name: developmental-erythropoiesis-trajectory-analysis
description: Use when multi-batch single-cell RNA-seq of a continuous differentiation
  process (e.g. erythroid development) must be ordered along a trajectory and batch-corrected,
  interpretable factors should be validated against an RNA-velocity latent time.
when_to_use_negative:
- Multi-condition, multi-sample datasets where samples do not share the same axes
  of variation — the paper advises using regular PCA in JOINTLY instead of consensus
  PCA
- Datasets without a batch annotation variable — jointly() requires batch.var to separate
  batch effects from shared cell states
- When reproducibility across R and Python is required — pyJOINTLY and rJOINTLY are
  'very similar [but] does not produce identical results'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: JOINTLY
  role: Batch-aware joint NMF clustering producing interpretable factor modules aligned
    to erythroid latent-time bins
  repo: https://github.com/madsen-lab/rJOINTLY
- name: scVelo
  role: Infers RNA-velocity latent time used to order cells along erythroid differentiation
- name: UCell
  role: Scores JOINTLY factor modules per cell (AddModuleScore_UCell) to quantify
    factor activity in latent-time bins
  repo: https://github.com/carmonalab/UCell
- name: Seurat
  role: Holds the Seurat object, JOINTLY reduction, FindNeighbors/FindClusters (dims
    1:15, res 0.05) and RunUMAP steps
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:8a16a82e8ab03d5aba2c32de88e02e73bc8a905478cd60fd6a8a4b66b040becc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - Seurat
  - UCell
  - scVelo
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# developmental-erythropoiesis-trajectory-analysis

## Summary

Reproduces the mouse gastrulation erythropoiesis pseudotime ordering (9815 cells, three batches) by aligning interpretable JOINTLY factor activities with scVelo RNA-velocity latent time, then tracing gene-set enrichments across latent-time bins to recover temporally staged signalling cascades.

## When to use

Use when multi-batch single-cell RNA-seq of a continuous differentiation process (e.g. erythroid development) must be ordered along a trajectory and batch-corrected, interpretable factors should be validated against an RNA-velocity latent time.

## When NOT to use

- Multi-condition, multi-sample datasets where samples do not share the same axes of variation — the paper advises using regular PCA in JOINTLY instead of consensus PCA
- Datasets without a batch annotation variable — jointly() requires batch.var to separate batch effects from shared cell states
- When reproducibility across R and Python is required — pyJOINTLY and rJOINTLY are 'very similar [but] does not produce identical results'

## Inputs

- Mouse gastrulation erythropoiesis scRNA-seq count matrix (9815 cells, three batches), loadable via scVelo
- Batch annotation variable (e.g. 'batch_label') for jointly(batch.var = ...)
- scVelo-derived latent time per cell (RNA-velocity pseudotime)
- JOINTLY factor gene modules (liver$modules) for UCell scoring

## Outputs

- Seurat/SingleCellExperiment object with the JOINTLY reduction and learned factor modules
- Factor temporal profiles: mean factor activity per latent-time bin (Fig. 4I-K)
- Gene-set/pathway enrichment tables per factor tracked across latent-time bins
- Exported CSV files of factor temporal profiles and enrichment tables

## How to apply

Download the mouse gastrulation erythropoiesis counts (9815 cells, three batches) via scVelo and load them into R as Seurat or SingleCellExperiment objects. Infer scVelo RNA-velocity latent time per cell so cells are ordered along erythroid differentiation, then run JOINTLY's jointly() across the three batches passing batch.var (e.g. 'liver <- jointly(liver, batch.var = "batch_label")'), obtaining a Seurat object with the JOINTLY reduction and learned factor modules. Bin cells into ordered intervals along the scVelo latent time and compute mean activity of each JOINTLY factor per bin to build the factor temporal profiles (Fig. 4I-K). Interpret factors with UCell module scores (AddModuleScore_UCell(features = liver$modules)) and run gene-set enrichment on the genes of each factor across the latent-time bins, checking that early, temporary, early-mid, late-mid and late factors reveal signalling cascades with distinct timing consistent with erythropoiesis literature. Because a seed change produces a different result, rerun JOINTLY up to 5 times and keep the best solution.

## Related tools

- **JOINTLY** (Batch-aware joint NMF clustering producing interpretable factor modules aligned to erythroid latent-time bins) — https://github.com/madsen-lab/rJOINTLY
- **scVelo** (Infers RNA-velocity latent time used to order cells along erythroid differentiation)
- **UCell** (Scores JOINTLY factor modules per cell (AddModuleScore_UCell) to quantify factor activity in latent-time bins) — https://github.com/carmonalab/UCell
- **Seurat** (Holds the Seurat object, JOINTLY reduction, FindNeighbors/FindClusters (dims 1:15, res 0.05) and RunUMAP steps) — https://github.com/satijalab/seurat

## Examples

```
remotes::install_github("madsen-lab/rJOINTLY"); liver <- readRDS("Human_Liver.rds"); liver <- jointly(liver, batch.var = "batch_label")
```

## Evaluation signals

- JOINTLY factors show distinct temporal profiles: mean factor activity differs across ordered scVelo latent-time bins, spanning early, temporary, early-mid, late-mid and late phases
- Pathway enrichment of factor gene sets changes across latent-time bins in a staged pattern consistent with known erythropoiesis signalling cascades
- Cluster agreement with reference labels: README reference run reports ARI 0.7415981 and NMI 0.6325386 via aricode::ARI / aricode::NMI
- Stability across seeds: rerunning JOINTLY up to 5 times and selecting the best solution should not change the qualitative factor-time ordering
- UCell module scores (e.g. factor_1_UCell) visualize as coherent spatial gradients on the JOINTLY-reduction UMAP

## Limitations

- Setting a seed and re-running produces different results; instability can require up to 5 JOINTLY runs keeping the best solution
- R (rJOINTLY) and Python (pyJOINTLY) versions do not produce identical results, so trajectory reproduction must use the R implementation as in the manuscript
- Consensus PCA (the default decomposition path: preprocess -> cpca -> prepareData -> JOINTLYsolve) is not advised for complex multi-condition, multi-sample datasets; regular PCA is recommended instead
- The H/F-matrix batch-aware expression smoothing (MAGIC-like imputation) should be used with caution and only for visualization purposes

## Evidence

- [methods] jointly with batch.var: "liver <- jointly(liver, batch.var = "batch_label")"
- [intro] JOINTLY description: "Here, we introduce JOINTLY, a hybrid linear and non-linear NMF-based joint clustering tool"
- [readme] seed instability / multiple runs: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
- [readme] UCell module scoring for interpretation: "## Calculate module scores
liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
- [discussion] cPCA caveat for multi-condition data: "For more complex datasets, where all samples do not necessarily share the same axes of variation"
