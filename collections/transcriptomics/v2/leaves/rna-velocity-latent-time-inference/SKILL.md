---
name: rna-velocity-latent-time-inference
description: Use when multi-batch single-cell transcriptomic data captures a continuous
  biological process (e.g., erythroid differentiation) and cells must be ordered in
  pseudo-time to validate that learned latent factors show distinct temporal profiles.
when_to_use_negative:
- When the goal is discrete cell-type clustering rather than continuous temporal dynamics
  — use JOINTLY's FindNeighbors/FindClusters (dims 1:15, res = 0.05) workflow instead
  of latent-time binning
- When data are complex multi-condition, multi-sample datasets where samples do not
  share the same axes of variation — the article advises regular PCA in JOINTLY over
  consensus PCA, and latent-time interpretation presumes a continuous axis
- When no spliced/unspliced layers exist, since RNA-velocity latent time cannot be
  computed by scVelo
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scVelo
  role: Infers RNA-velocity-based latent time for each cell so cells can be ordered
    along erythroid differentiation; also source of the erythropoiesis test dataset
- name: JOINTLY
  role: Joint NMF-based clustering across the three batches with interpretable factors
    whose activities are profiled over latent-time bins
  repo: https://github.com/madsen-lab/rJOINTLY
- name: Seurat
  role: Handles the single-cell objects; stores the JOINTLY reduction and hosts downstream
    clustering/UMAP and plotting
  repo: https://github.com/satijalab/seurat
- name: UCell
  role: Computes module scores for JOINTLY factor gene sets used as factor activity
    when building temporal profiles
  repo: https://github.com/carmonalab/UCell
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:f26d68d0d12eb0066e5a828a094122ce2a8f96cc2757d7429aded8e1e747c806
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - Seurat
  - UCell
  - scVelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rna-velocity-latent-time-inference

## Summary

Infers RNA-velocity-based latent time with scVelo on multi-batch scRNA-seq data (e.g., 9815 mouse gastrulation erythropoiesis cells across three batches) to order cells along continuous differentiation, providing a continuous temporal axis against which interpretable JOINTLY factor activities can be profiled.

## When to use

Use when multi-batch single-cell transcriptomic data captures a continuous biological process (e.g., erythroid differentiation) and cells must be ordered in pseudo-time to validate that learned latent factors show distinct temporal profiles.

## When NOT to use

- When the goal is discrete cell-type clustering rather than continuous temporal dynamics — use JOINTLY's FindNeighbors/FindClusters (dims 1:15, res = 0.05) workflow instead of latent-time binning
- When data are complex multi-condition, multi-sample datasets where samples do not share the same axes of variation — the article advises regular PCA in JOINTLY over consensus PCA, and latent-time interpretation presumes a continuous axis
- When no spliced/unspliced layers exist, since RNA-velocity latent time cannot be computed by scVelo

## Inputs

- Multi-batch scRNA-seq count matrix (e.g., mouse gastrulation erythropoiesis: 9815 cells, three batches, obtainable via scVelo) as Seurat/SingleCellExperiment objects
- Batch annotation variable (e.g., batch_label) passed to JOINTLY
- Spliced/unspliced count layers required for scVelo velocity estimation

## Outputs

- Per-cell RNA-velocity-based latent time (pseudo-time ordering along erythroid differentiation)
- Factor temporal profiles: mean activity of each JOINTLY factor per latent-time bin (Fig. 4I-K)
- Pathway/gene-set enrichment tables per JOINTLY factor tracked across latent-time bins (exportable as CSV)

## How to apply

Download the scRNA-seq counts (9815 cells, three batches) through scVelo and load them into R as Seurat/SingleCellExperiment objects. Run scVelo on each cell to obtain RNA-velocity-based latent time so cells can be ordered along erythroid differentiation. Independently run JOINTLY joint clustering across the three batches with jointly(), passing the batch annotation via batch.var (e.g., 'liver <- jointly(liver, batch.var = "batch_label")'), yielding the Seurat object with the JOINTLY reduction and learned factor modules. Bin cells into ordered intervals along the scVelo latent time and compute mean activity of each JOINTLY factor per bin to build factor temporal profiles (Fig. 4I-K). Score factors with UCell (AddModuleScore_UCell with liver$modules) and run gene-set enrichment per factor, tracking enrichment changes across latent-time bins. Decision point: if the JOINTLY run is unstable (setting a seed and re-running produces a different result), rerun up to 5 times and keep the best solution.

## Related tools

- **scVelo** (Infers RNA-velocity-based latent time for each cell so cells can be ordered along erythroid differentiation; also source of the erythropoiesis test dataset)
- **JOINTLY** (Joint NMF-based clustering across the three batches with interpretable factors whose activities are profiled over latent-time bins) — https://github.com/madsen-lab/rJOINTLY
- **Seurat** (Handles the single-cell objects; stores the JOINTLY reduction and hosts downstream clustering/UMAP and plotting) — https://github.com/satijalab/seurat
- **UCell** (Computes module scores for JOINTLY factor gene sets used as factor activity when building temporal profiles) — https://github.com/carmonalab/UCell

## Examples

```
liver <- jointly(liver, batch.var = "batch_label"); liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)
```

## Evaluation signals

- JOINTLY factors should show distinct, non-degenerate temporal profiles across ordered scVelo latent-time bins (Fig. 4I-K)
- Pathway enrichment of factor gene sets should reveal signalling cascades with different timing consistent with erythropoiesis literature (early, temporary, early-mid, late-mid, late latent time)
- Binning sanity check: each latent-time interval should contain a comparable, non-empty set of cells from all three batches
- JOINTLY run stability: if a rerun with a changed seed diverges, keep the best of up to 5 runs; the run retained should yield a coherent UMAP/reduction
- Factor scores from AddModuleScore_UCell should vary smoothly (not bimodally artifactually) across adjacent latent-time bins

## Limitations

- Setting a seed and re-running JOINTLY produces a different result; the workflow can be unstable and may require up to 5 runs choosing the best solution
- For complex multi-condition, multi-sample datasets where samples do not share the same axes of variation, consensus PCA in JOINTLY is not advised — regular PCA is recommended
- Software version numbers for scVelo/Seurat/UCell are not stated in the article methods, so exact reproducibility of latent-time values is not guaranteed
- Performance of the integration underlying factor interpretation varies task-to-task; the chosen method should be evaluated on each new integration task

## Evidence

- [other] Infer RNA-velocity-based latent time for each cell with scVelo so that cells can be ordered along erythroid differentiation.: [evidence span withheld — anchor status: not_found]
- [results] JOINTLY factors show distinct temporal profiles, and pathway analysis of genes associated with early, temporary, early-mid, late-mid, or late latent time reveals signalling cascades with different timing, consistent with erythropoiesis literature.: [evidence span withheld — anchor status: not_found]
- [methods] Bin the cells into ordered intervals along the scVelo latent time and compute the mean activity of each JOINTLY factor per bin to build the factor temporal profiles (Fig. 4I-K).: [evidence span withheld — anchor status: not_found]
- [readme] Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution.: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
- [methods] Calculate module scores using the learned modules: "liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
