---
name: single-cell-temporal-binning-factor-profiling
description: Use when multi-batch single-cell data has a continuous per-cell time
  axis (e.g. scVelo latent time) and you need to test whether learned interpretable
  factors (JOINTLY modules) show stage-specific or monotonic temporal activity.
when_to_use_negative:
- The input cells lack a continuous per-cell time ordering (no pseudotime/latent time),
  so ordered binning along differentiation cannot be defined.
- The analysis goal is batch-effect removal or clustering performance benchmarking
  (ARI/NMI) rather than temporal profiling of interpretable factors.
- The dataset is a static reference atlas task (e.g. WATLAS-style annotation/deconvolution)
  where stage-specific factor dynamics are not the question.
tools:
- name: JOINTLY
  role: Produces the interpretable factor modules and reduction whose activity is
    profiled across temporal bins; run via jointly() with batch.var on the multi-batch
    object
  repo: https://github.com/madsen-lab/rJOINTLY
- name: scVelo
  role: Infers RNA-velocity-based latent time used to order and bin the cells along
    erythroid differentiation
- name: UCell
  role: Computes per-cell module scores (AddModuleScore_UCell) for JOINTLY factor
    gene sets, which are then averaged per temporal bin
  repo: https://github.com/carmonalab/UCell
- name: Seurat
  role: Holds the JOINTLY reduction and factor module scores on the cell-level object
    used for binning and visualization
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:6324d3b7d73fc605d115a6b7401c4ac39af82acac3a302a2e6194984f3c70432
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# single-cell-temporal-binning-factor-profiling

## Summary

Orders single cells along an external continuous axis (e.g. scVelo RNA-velocity latent time), bins cells into ordered intervals, and computes mean activity of interpretable JOINTLY factors per bin to build factor temporal profiles (Fig. 4I-K). Used to show that JOINTLY's factors recover continuous temporal dynamics during mouse gastrulation erythropoiesis.

## When to use

Use when multi-batch single-cell data has a continuous per-cell time axis (e.g. scVelo latent time) and you need to test whether learned interpretable factors (JOINTLY modules) show stage-specific or monotonic temporal activity.

## When NOT to use

- The input cells lack a continuous per-cell time ordering (no pseudotime/latent time), so ordered binning along differentiation cannot be defined.
- The analysis goal is batch-effect removal or clustering performance benchmarking (ARI/NMI) rather than temporal profiling of interpretable factors.
- The dataset is a static reference atlas task (e.g. WATLAS-style annotation/deconvolution) where stage-specific factor dynamics are not the question.

## Inputs

- Seurat or SingleCellExperiment object with multi-batch scRNA-seq counts (e.g. 9815 mouse gastrulation erythropoiesis cells across three batches, loaded via scVelo)
- Per-cell continuous latent/pseudotime ordering (scVelo RNA-velocity latent time)
- Batch annotation column (e.g. batch_label) for jointly(batch.var = ...)
- JOINTLY factor gene modules (list of gene sets per factor)

## Outputs

- Factor temporal profiles: mean JOINTLY factor activity per ordered latent-time bin (Fig. 4I-K style)
- Pathway/gene-set enrichment tables per factor tracked across latent-time bins
- CSV exports of factor temporal profiles and pathway enrichment tables

## How to apply

First run JOINTLY on the multi-batch object with jointly(), passing the batch annotation via batch.var (e.g. three batches of 9815 mouse gastrulation erythropoiesis cells), obtaining the Seurat object with the JOINTLY reduction and factor gene modules. Obtain a per-cell continuous ordering by inferring RNA-velocity-based latent time with scVelo along the erythroid differentiation trajectory. Bin the cells into ordered intervals along the latent time axis, then compute the mean activity of each JOINTLY factor per bin (e.g. via UCell module scores using liver$modules-style factor gene sets) to build the factor temporal profiles. Run pathway/gene-set enrichment on the genes of each factor and track enrichment changes across latent-time bins (early, temporary, early-mid, late-mid, late). If a JOINTLY run is unstable, rerun up to 5 times and keep the best solution, as the authors advise; for complex multi-condition, multi-sample datasets use regular PCA in JOINTLY rather than consensus PCA.

## Related tools

- **JOINTLY** (Produces the interpretable factor modules and reduction whose activity is profiled across temporal bins; run via jointly() with batch.var on the multi-batch object) — https://github.com/madsen-lab/rJOINTLY
- **scVelo** (Infers RNA-velocity-based latent time used to order and bin the cells along erythroid differentiation)
- **UCell** (Computes per-cell module scores (AddModuleScore_UCell) for JOINTLY factor gene sets, which are then averaged per temporal bin) — https://github.com/carmonalab/UCell
- **Seurat** (Holds the JOINTLY reduction and factor module scores on the cell-level object used for binning and visualization) — https://github.com/satijalab/seurat

## Examples

```
liver <- jointly(liver, batch.var = "batch_label"); liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules); bin_means <- tapply(liver$object$factor_1_UCell, cut(liver$object$latent_time, bins), mean)
```

## Evaluation signals

- JOINTLY factors show distinct, non-random temporal profiles across ordered latent-time bins, with factors peaking at early, early-mid, late-mid, or late intervals.
- Pathway enrichment of factor genes changes across latent-time bins and recovers signalling cascades with different timing consistent with erythropoiesis literature.
- The binning is strictly ordered along the scVelo latent time axis, and every cell is assigned to exactly one interval; mean activity per bin is well-defined for all factors.
- If JOINTLY is unstable under a given seed, reruns (up to 5) are performed and the best solution retained, per the authors' advice.
- Factor temporal profiles and pathway enrichment tables are exported as CSV files for downstream verification.

## Limitations

- Setting a seed and re-running JOINTLY produces a different result; up to 5 runs may be needed to pick the best solution, so single-run factor profiles can be unstable.
- For complex multi-condition, multi-sample datasets where samples do not share the same axes of variation, consensus PCA in JOINTLY is not advised; regular PCA should be used.
- Integration results are task-dependent: no method is always best, so the chosen method must be evaluated on each new integration task; integration metrics do not reveal the genes driving clustering.
- All manuscript analyses were performed with the R version of JOINTLY; the Python version does not produce identical results.

## Evidence

- [methods] Bin the cells into ordered intervals along the scVelo latent time and compute the mean activity of each JOINTLY factor per bin to build the factor temporal profiles (Fig. 4I-K).: [evidence span withheld — anchor status: not_found]
- [results] Factor temporal profiles recovered signalling cascades with different timing.: [evidence span withheld — anchor status: not_found]
- [methods] UCell module scores of learned modules.: "## Calculate module scores
liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
- [readme] Rerun advice for unstable JOINTLY solutions.: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
