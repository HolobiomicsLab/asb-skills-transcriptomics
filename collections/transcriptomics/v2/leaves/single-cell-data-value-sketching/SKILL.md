---
name: single-cell-data-value-sketching
description: Use when a large scRNA-seq atlas (.h5ad, tens of thousands to millions
  of cells) must be subsampled before downstream ML/DL tasks such as cell-type annotation
  (scANVI, scPoli, CellTypist, ACTINN), label transfer, CellHint harmonization, or
  bulk deconvolution, and uniform or geometric sketching.
when_to_use_negative:
- The dataset is already small enough to analyse directly and the goal is maximum
  fidelity — the full reference is the gold standard (e.g. PBMC/scANVI full data reached
  0.8635 accuracy vs 0.8330 with a 10% sketch)
- The task requires preserving geometric/distance structure of the expression space
  rather than annotation-informative cells — distance-optimized methods (GeoSketch,
  Sphetcher, Hopper, scSampler) target that objective instead
- No cell-type labels are available and the random-forest DV computation against cell_type_key
  cannot be trained
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scValue
  role: 'Core skill implementation: SCValue class computes OOB-based DV, allocates
    sketch size across cell types (type_weighting=''inv_cv''), and selects cells per
    type (strategy=''FB'', ''MTB'', or ''TP''); pip-installable Python package'
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: 'Preprocessing pipeline: reads .h5ad, selects highly variable genes, computes
    PCA into adata.obsm[''X_pca''] consumed by SCValue (experiments used Scanpy 1.10.2,
    AnnData 0.10.8)'
- name: scikit-learn
  role: Random forest backend for out-of-bag DV estimation (paper ran scikit-learn
    1.5.2)
- name: GeoSketch
  role: Baseline sketching method (box-partitioning of expression space) benchmarked
    against scValue
- name: scSampler
  role: Baseline sketching method (maximin distance design) benchmarked against scValue
- name: CellTypist
  role: Downstream tool for training models on sketches and label transfer (e.g. Gut
    428k-cell reference to immune query T-cell populations)
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:84e7ebf7cd91e7aee06372e2c5386d982a144864bbc7b17e2612efb5625d34b3
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
  - CellTypist
  - GeoSketch
  - Hopper
  - Scanpy
  - Seurat
  - Sphetcher
  - TreeHopper
  - scSampler
  - scValue
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# single-cell-data-value-sketching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Creates representative sketches (subsamples) of large scRNA-seq datasets by ranking cells with out-of-bag 'data value' (DV) estimates from a random forest, then value-guided selecting cells per cell type. Used to make ML/DL tasks (cell-type annotation, label transfer, harmonization, deconvolution) tractable while closely matching full-data performance.

## When to use

Use when a large scRNA-seq atlas (.h5ad, tens of thousands to millions of cells) must be subsampled before downstream ML/DL tasks such as cell-type annotation (scANVI, scPoli, CellTypist, ACTINN), label transfer, CellHint harmonization, or bulk deconvolution, and uniform or geometric sketching loses rare cell types.

## When NOT to use

- The dataset is already small enough to analyse directly and the goal is maximum fidelity — the full reference is the gold standard (e.g. PBMC/scANVI full data reached 0.8635 accuracy vs 0.8330 with a 10% sketch)
- The task requires preserving geometric/distance structure of the expression space rather than annotation-informative cells — distance-optimized methods (GeoSketch, Sphetcher, Hopper, scSampler) target that objective instead
- No cell-type labels are available and the random-forest DV computation against cell_type_key cannot be trained

## Inputs

- Annotated scRNA-seq dataset in .h5ad format (e.g. PBMC, mBrain, CxG_min, mACA, Gut, Spleen), log-normalized
- PCA representation stored in adata.obsm['X_pca'] after HVG selection (e.g. sc.pp.highly_variable_genes with n_top_genes=3000, sc.pp.pca n_comps=50)
- Cell-type labels column in adata.obs (cell_type_key, default 'cell_type')
- Sketch size as fraction in (0,1] or integer cell count (paper benchmarks 2%-10%)

## Outputs

- Subsampled AnnData sketch (scv.adata_sub / adata_sub) with per-cell DV stored in adata.obs['dv']
- Per-cell data value table dv_values.csv (cell type + DV) and per-type summary dv_summary.csv when write_dv=True
- Downstream outputs such as annotation accuracy tables, label-transfer agreement dot plots, or CellHint subtype trees computed on the sketch

## How to apply

Load the .h5ad dataset with Scanpy, select highly variable genes (e.g. top 3,000), and compute 50 principal components into adata.obsm['X_pca'] since SCValue by default fits the random forest on X_pca. Instantiate SCValue with sketch_size (float fraction in (0,1] or integer cell count), use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv' (weights proportional to mean(DV)*sqrt(N)/std(DV)), strategy='FB' (DV binned into 0.1 intervals from 0.0 to 1.0 with top-DV cells selected per bin), and seed=42, then call value() to return the sketch as AnnData with per-cell DV in adata.obs['dv']. Optionally set write_dv=True to export dv_values.csv (per-cell DV + type) and dv_summary.csv (per-type DV statistics), and use type_weighting='proportional' (prop_sampling) when balanced cell-type representation by original frequencies is preferred. Benchmarks sketch fractions of 2%-10% against baselines (Uniform, GeoSketch, Sphetcher, Hopper/TreeHopper, KH, scSampler) and compare downstream accuracy or label-transfer agreement against the full-data result.

## Related tools

- **scValue** (Core skill implementation: SCValue class computes OOB-based DV, allocates sketch size across cell types (type_weighting='inv_cv'), and selects cells per type (strategy='FB', 'MTB', or 'TP'); pip-installable Python package) — https://github.com/LHBCB/scvalue
- **Scanpy** (Preprocessing pipeline: reads .h5ad, selects highly variable genes, computes PCA into adata.obsm['X_pca'] consumed by SCValue (experiments used Scanpy 1.10.2, AnnData 0.10.8))
- **scikit-learn** (Random forest backend for out-of-bag DV estimation (paper ran scikit-learn 1.5.2))
- **GeoSketch** (Baseline sketching method (box-partitioning of expression space) benchmarked against scValue)
- **scSampler** (Baseline sketching method (maximin distance design) benchmarked against scValue)
- **CellTypist** (Downstream tool for training models on sketches and label transfer (e.g. Gut 428k-cell reference to immune query T-cell populations))

## Examples

```
from scvalue import SCValue; scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42); adata_sub = scv.value()
```

## Evaluation signals

- Sketch size matches request: sketch_size as float keeps that fraction of cells, as int keeps the exact cell count; DV stored in adata.obs['dv'] and adata_sub.obs['dv']
- Downstream accuracy on the sketch approaches full-data performance (e.g. PBMC/scANVI at 10% sketch reached 0.8330 average accuracy vs 0.8635 for the full reference) and exceeds Uniform, GeoSketch, Sphetcher, TreeHopper, KH, and scSampler
- Gini coefficient indicates balanced cell-type representation and rare cell types are captured; Hausdorff distance and computational time evaluated on 16 public datasets
- With write_dv=True, dv_values.csv contains per-cell DV plus cell type and dv_summary.csv contains per-type DV summary statistics; write_dv=False produces neither file
- Label-transfer agreement with full-reference predictions is highest for scValue sketches (e.g. Gut-to-immune CellTypist transfer matches Figure 4 dot plot)

## Limitations

- scValue prioritizes high-DV cells; the FB strategy (DV binned into 0.1 intervals) is needed to ensure even cells with lower OOB values are included, whereas 'TP' selects only the highest-DV cells per type
- OOB DV estimates can be unstable in noisy conditions; tuning random forest hyperparameters (e.g. increasing n_trees above the default 100, adjusting maximum depth) may be needed
- Using use_rep='X' with sparse matrices may trigger internal densification; the documented default is X_pca
- The scValue-core confidence score threshold for filtering cells scored below it is user-defined with no default given in the paper

## Evidence
