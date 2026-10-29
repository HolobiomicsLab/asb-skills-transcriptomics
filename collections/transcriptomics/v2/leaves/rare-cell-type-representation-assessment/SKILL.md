---
name: rare-cell-type-representation-assessment
description: Use when a subsampled single-cell dataset (e.g. a scValue, GeoSketch,
  Sphetcher, TreeHopper, KH, or Uniform sketch) must be checked for loss of rare cell
  types before downstream ML/DL tasks such as cell-type annotation.
when_to_use_negative:
- The goal is computational speed or global distributional fidelity alone, with no
  annotated cell-type labels available to compute the Gini coefficient or per-type
  accuracies
- The dataset is small enough that full-data analysis is feasible — the article motivates
  sketching by the computational and memory challenges of large atlases (tens of thousands
  to millions of cells)
- Evaluating supervised feature selection or model training on the full data, where
  subsampling and its representation biases are not involved
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scValue
  role: Value-based subsampling method under evaluation; produces sketches with DV-guided
    per-type allocation ('inv_cv' weighting, 'FB' strategy) and stores per-cell DV
    in adata.obs['dv']
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Preprocessing (highly variable gene selection, PCA) and uniform random subsampling
    baseline; also used to visualize the sketch via neighbours/UMAP
- name: GeoSketch
  role: Baseline sketching method compared on rare-type capture (Gini coefficient)
    and downstream accuracy
- name: scSampler
  role: Maximin-distance baseline sketching method included in the rare-type representation
    comparison
- name: scikit-learn
  role: Powers the random forest OOB data-value estimates underlying per-type sketch
    allocation (scikit-learn 1.5.2)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:fbb6a315cb1aaf753a04fe936aa7c9cd1745dbcbb4d5331a747ce353b81792ed
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# rare-cell-type-representation-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses whether a subsampling ('sketching') method for scRNA-seq data preserves rare cell types and balanced cell-type representation, using the Gini coefficient of per-type proportions, Hausdorff distance, and per-cell-type downstream annotation accuracy.

## When to use

Use when a subsampled single-cell dataset (e.g. a scValue, GeoSketch, Sphetcher, TreeHopper, KH, or Uniform sketch) must be checked for loss of rare cell types before downstream ML/DL tasks such as cell-type annotation.

## When NOT to use

- The goal is computational speed or global distributional fidelity alone, with no annotated cell-type labels available to compute the Gini coefficient or per-type accuracies
- The dataset is small enough that full-data analysis is feasible — the article motivates sketching by the computational and memory challenges of large atlases (tens of thousands to millions of cells)
- Evaluating supervised feature selection or model training on the full data, where subsampling and its representation biases are not involved

## Inputs

- Annotated scRNA-seq dataset in .h5ad format (log-normalized, with HVG-selected PCA in adata.obsm['X_pca'] and cell-type labels in adata.obs)
- Sketch/subsample AnnData objects produced by each method at matched sketch fractions (e.g. 0.02–0.10)
- Per-cell predicted vs. original cell-type labels from the downstream annotation model

## Outputs

- Gini coefficient of cell-type proportions per sketch (rare-type capture metric)
- Hausdorff distance between sketch and full-data distributions
- Per-cell-type annotation accuracy table for every method-by-fraction combination
- Boxplots of accuracy vs. sketch fraction for all methods, with rare cell types flagged

## How to apply

For each method-by-fraction combination (sketch fractions 0.02 to 0.10 in the scValue benchmark), compute the Gini coefficient of cell-type proportions in the sketch to quantify over- or under-representation of rare cell types, and Hausdorff distance to measure how far the sketch lies from the full data distribution. Flag rare cell types and compute per-cell-type annotation accuracy separately (e.g. scANVI on PBMC, scPoli on mBrain, CellTypist on CxG_min, ACTINN on mACA) rather than relying on overall accuracy, since overall scores can hide minority-class dropouts. Use scValue's per-type DV weighting ('inv_cv' weights ∝ mean(DV) * sqrt(N) / std(DV)) and the FB strategy, which bins DV into 0.1 intervals from 0.0 to 1.0 so that even low-value cells are included, as a comparison point. Aggregate per-type accuracies into results tables and accuracy-vs-fraction boxplots; a method applied correctly should show low Gini, low Hausdorff distance, and stable per-type accuracy across fractions.

## Related tools

- **scValue** (Value-based subsampling method under evaluation; produces sketches with DV-guided per-type allocation ('inv_cv' weighting, 'FB' strategy) and stores per-cell DV in adata.obs['dv']) — https://github.com/LHBCB/scvalue
- **Scanpy** (Preprocessing (highly variable gene selection, PCA) and uniform random subsampling baseline; also used to visualize the sketch via neighbours/UMAP)
- **GeoSketch** (Baseline sketching method compared on rare-type capture (Gini coefficient) and downstream accuracy)
- **scSampler** (Maximin-distance baseline sketching method included in the rare-type representation comparison)
- **scikit-learn** (Powers the random forest OOB data-value estimates underlying per-type sketch allocation (scikit-learn 1.5.2))

## Examples

```
scv = SCValue(adata=adata, sketch_size=0.1, use_rep="X_pca", cell_type_key="cell_type", n_trees=100, type_weighting="inv_cv", strategy="FB", write_dv=True, seed=42); adata_sub = scv.value()
```

## Evaluation signals

- Sketch cell-type proportions yield a low Gini coefficient, indicating balanced representation including rare types (article evaluates subsampling quality via Gini on 16 public datasets)
- Hausdorff distance between the sketch and the full data is comparable to or lower than baselines at matched sketch fractions
- Per-cell-type accuracy for rare cell types does not collapse relative to full-data performance (e.g. PBMC/scANVI at 10% sketch reaches 0.8330 vs. 0.8635 full-data accuracy)
- scValue sketch accuracy is less variable across seeds/fractions than the six baselines (Uniform, GeoSketch, Sphetcher, TreeHopper, KH, scSampler)
- DV values are written to adata.obs['dv'] (when write_dv=True, dv_values.csv and dv_summary.csv are produced), enabling per-type DV distribution checks

## Limitations

- Assessing rare cell-type capture is critical but relatively few empirical studies have evaluated how sketching methods perform in downstream learning tasks; results are task- and dataset-dependent (PBMC/scANVI, mBrain/scPoli, CxG_min/CellTypist, mACA/ACTINN)
- No accession numbers or repository identifiers are provided for the benchmark datasets, complicating direct reproduction
- Tool versions for benchmarked annotation software (CellTypist, scANVI, scPoli, ACTINN, GeoSketch, etc.) are not specified, and subsampling baselines (KH, Sphetcher, Hopper, scSampler) are referenced only by short names without package details
- Random forest OOB estimates can be noisy; the authors note that tuning hyperparameters (increasing n_trees, adjusting maximum depth) may be needed to stabilize DV estimates

## Evidence
