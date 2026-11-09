---
name: annotation-accuracy-boxplot-visualization
description: Use when you have annotation accuracies (overall and per-cell-type) from
  multiple subsampling methods across sketch fractions 0.02–0.10 and need side-by-side
  boxplots showing accuracy vs. fraction per dataset-model pair.
when_to_use_negative:
- Downstream analysis evaluates subsampling quality by computational time, Gini coefficient,
  or Hausdorff distance on the 16 public datasets rather than annotation accuracy
  — those metrics need different plots, not accuracy boxplots
- The analysis goal is label transfer, cross-study harmonization, or bulk deconvolution
  case studies (Gut/Colon, Spleen, T&ILC datasets) where the benchmark outputs are
  not cell-type annotation accuracies
- Input is a single-method, single-fraction accuracy value with no baselines or fractions
  to compare — a boxplot comparison across methods is not meaningful
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Used to compute annotation accuracy metrics from predicted vs. original cell-type
    labels feeding the results table and boxplots
- name: scValue
  role: Source of the focal method's accuracy curve (sketches generated with use_rep='X_pca',
    type_weighting='inv_cv', strategy='FB', n_trees=100, seed=42) compared in the
    boxplots
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Provides the uniform random sampling baseline and preprocessing (HVG selection
    + PCA stored in adata.obsm['X_pca']) whose downstream accuracies appear in the
    plots
- name: GeoSketch
  role: Baseline subsampling method whose annotation accuracies are plotted alongside
    scValue for comparison
- name: scSampler
  role: Baseline maximin-distance subsampling method whose annotation accuracies are
    plotted alongside scValue for comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:175aedc6f84766cb51a8613f0022cf69c9f3e6860dce5724dd31a314b64b4957
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/LHBCB/scvalue
  repo_url: https://github.com/LHBCB/scvalue
  tools:
  - GeoSketch
  - Scanpy
  - Sphetcher
  - TreeHopper
  - scSampler
  - scValue
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# annotation-accuracy-boxplot-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates overall and per-cell-type annotation accuracies for every sketching-method-by-fraction combination into a results table and renders per-task boxplots of accuracy vs. sketch fraction, so scValue's performance against six baselines (Uniform, GeoSketch, Sphetcher, TreeHopper, KH, scSampler) can be visually compared on ML/DL cell-type annotation tasks.

## When to use

Use when you have annotation accuracies (overall and per-cell-type) from multiple subsampling methods across sketch fractions 0.02–0.10 and need side-by-side boxplots showing accuracy vs. fraction per dataset-model pair.

## When NOT to use

- Downstream analysis evaluates subsampling quality by computational time, Gini coefficient, or Hausdorff distance on the 16 public datasets rather than annotation accuracy — those metrics need different plots, not accuracy boxplots
- The analysis goal is label transfer, cross-study harmonization, or bulk deconvolution case studies (Gut/Colon, Spleen, T&ILC datasets) where the benchmark outputs are not cell-type annotation accuracies
- Input is a single-method, single-fraction accuracy value with no baselines or fractions to compare — a boxplot comparison across methods is not meaningful

## Inputs

- Annotation accuracy scores per method-by-sketch-fraction combination (predicted labels vs. original cell-type annotations for each of the four dataset-model pairs)
- Sketch fractions in the range 0.02 to 0.10 for each subsampling method (scValue, Uniform, GeoSketch, Sphetcher, TreeHopper, KH, scSampler)
- Per-cell-type accuracy breakdown with rare-cell-type flags

## Outputs

- Aggregated results table of overall and per-cell-type annotation accuracies for every method-by-fraction combination
- Per-task boxplots showing accuracy vs. sketch fraction for all subsampling methods, one per dataset-model pair (PBMC/scANVI, mBrain/scPoli, CxG_min/CellTypist, mACA/ACTINN)

## How to apply

Collect the annotation accuracies produced by scoring each dataset's designated model (scANVI on PBMC, scPoli on mBrain, CellTypist on CxG_min, ACTINN on mACA) against original cell-type labels for every method-by-fraction combination. Compute overall and per-cell-type accuracies, flagging rare cell types so their capture is visible rather than hidden by aggregate means. Aggregate the accuracies into a results table, then render per-task boxplots with accuracy on the y-axis and sketch fraction (0.02–0.10) on the x-axis, one plot per dataset-model pair with all seven methods overlaid for direct comparison. Judge correctness by whether the scValue curve consistently sits above the six baselines and closely tracks full-data performance (e.g., PBMC/scANVI at 10% sketch ≈ 0.8330 vs. 0.8635 full reference) with visibly less variability across replicates.

## Related tools

- **scikit-learn** (Used to compute annotation accuracy metrics from predicted vs. original cell-type labels feeding the results table and boxplots)
- **scValue** (Source of the focal method's accuracy curve (sketches generated with use_rep='X_pca', type_weighting='inv_cv', strategy='FB', n_trees=100, seed=42) compared in the boxplots) — https://github.com/LHBCB/scvalue
- **Scanpy** (Provides the uniform random sampling baseline and preprocessing (HVG selection + PCA stored in adata.obsm['X_pca']) whose downstream accuracies appear in the plots)
- **GeoSketch** (Baseline subsampling method whose annotation accuracies are plotted alongside scValue for comparison)
- **scSampler** (Baseline maximin-distance subsampling method whose annotation accuracies are plotted alongside scValue for comparison)

## Examples

```
for frac in [0.02,0.04,0.06,0.08,0.10]: acc = (adata_sub.obs['pred'] == adata_sub.obs['cell_type']).mean(); results.append({'method': method, 'fraction': frac, 'accuracy': acc})  # then seaborn.boxplot(x='fraction', y='accuracy', hue='method', data=results)
```

## Evaluation signals

- scValue accuracy at each fraction is close to full-data performance (e.g., PBMC/scANVI at 10% sketch ≈ 0.8330 vs. 0.8635 for the full reference)
- scValue consistently exceeds all six baselines across the four dataset-model pairs and the 0.02–0.10 fraction range in the plotted curves
- Per-task boxplots show scValue with less variability across replicates than its counterparts
- Results table contains both overall and per-cell-type accuracies with rare cell types flagged, so rare-type capture is auditable
- Every method-by-fraction combination (7 methods × fractions 0.02–0.10 × 4 dataset-model pairs) is represented in the aggregated table

## Limitations

- Aggregate accuracy can mask rare-cell-type performance; the workflow mitigates this by flagging rare cell types and computing per-cell-type accuracies, but the boxplots alone may not reveal rare-type losses
- No accession numbers or repository identifiers were provided for the benchmark datasets (PBMC, mBrain, CxG_min, mACA, Gut, Colon, Spleen, T&ILC), complicating exact reproduction
- Tool versions are not specified for the benchmarked annotation models (CellTypist, CellHint, MuSiC, scANVI, scPoli, ACTINN), so plotted accuracies may not be exactly reproducible
- Comparison is against only six subsampling baselines referenced by short names (KH, Sphetcher, Hopper, scSampler) without package or implementation details

## Evidence

- [other] Aggregate accuracies into a results table and per-task boxplots showing accuracy vs sketch fraction for all methods.: [evidence span withheld — anchor status: not_found]
- [abstract] scValue consistently outperformed baseline: "Our method consistently outperformed existing subsampling methods, closely matching full-data performance across all annotation tasks"
- [other] PBMC/scANVI accuracy figures: [evidence span withheld — anchor status: not_found]
- [intro] rare cell type flagging: "assessing the ability of different methods to capture rare cell"
- [discussion] dataset-model pairs: "In CTA tasks (PBMC with scANVI, mBrain with scPoli, CxG_min with CellTypist, and mACA with ACTINN)"
- [other] full workflow description: [evidence span withheld — anchor status: not_found]
