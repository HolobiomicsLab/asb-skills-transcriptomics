---
name: pseudobulk-sample-aggregation
description: Use when bulk RNA-seq or simulated tissue samples must be collapsed into
  pseudobulk profiles and deconvolved against a sketch-derived single-cell reference,
  with per-sample cell-type proportion estimates compared to known true proportions.
when_to_use_negative:
- Downstream questions require cell-level resolution (e.g. cell-type annotation, label
  transfer) — pseudobulk aggregation collapses cells per sample and only supports
  bulk proportion estimation
- No ground-truth cell-type proportions are available for the bulk samples, since
  Pearson/RMSE/MAE scoring against true proportions is the core of the benchmark
- The single-cell reference lacks cell-type annotations (cell_type_key), making it
  unusable to build the annotated reference matrix MuSiC requires
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scValue
  role: Builds the value-based sketch of the T&ILC reference used as the single-cell
    reference matrix for MuSiC deconvolution
  repo: https://github.com/LHBCB/scvalue
- name: MuSiC
  role: Runs bulk deconvolution on the pseudobulk profiles against each sketch-based
    reference to estimate cell-type proportions
- name: Scanpy
  role: Preprocesses the .h5ad reference (HVG selection, PCA into obsm['X_pca']) and
    provides uniform subsampling for the Uniform baseline
- name: scikit-learn
  role: Computes Pearson correlation, RMSE, and MAE between inferred and true cell-type
    proportions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:e2a92d0d885cf07b9076a5b9a59d0626d1097c00d5fd5a03f37e6a2218707f94
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
  - Hopper
  - KH
  - MuSiC
  - Scanpy
  - Seurat
  - Sphetcher
  - scSampler
  - scValue
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pseudobulk-sample-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates individual single-cell or simulated bulk tissue samples into pseudobulk expression profiles so that a reference single-cell matrix (e.g. a scValue sketch of the T&ILC A29/A31 donors) can be used with MuSiC to infer cell-type proportions, which are then scored against ground truth via Pearson correlation, RMSE, and MAE.

## When to use

Use when bulk RNA-seq or simulated tissue samples must be collapsed into pseudobulk profiles and deconvolved against a sketch-derived single-cell reference, with per-sample cell-type proportion estimates compared to known true proportions.

## When NOT to use

- Downstream questions require cell-level resolution (e.g. cell-type annotation, label transfer) — pseudobulk aggregation collapses cells per sample and only supports bulk proportion estimation
- No ground-truth cell-type proportions are available for the bulk samples, since Pearson/RMSE/MAE scoring against true proportions is the core of the benchmark
- The single-cell reference lacks cell-type annotations (cell_type_key), making it unusable to build the annotated reference matrix MuSiC requires

## Inputs

- Two-donor (A29/A31) T&ILC scRNA-seq reference in .h5ad format with cell_type annotations in adata.obs
- 10 simulated bulk tissue samples with known ground-truth cell-type proportions
- Sketch-based cell-type-annotated single-cell reference matrices (from scValue and six baseline subsamplers)

## Outputs

- Pseudobulk expression matrix (10 samples × genes) aggregated from simulated bulk samples
- Per-sample estimated cell-type proportions from MuSiC deconvolution against each sketch-based reference
- table3_deconvolution_metrics.csv with Pearson correlation, RMSE, and MAE per subsampling method (scValue: 0.6851 / 0.0592 / 0.0434)

## How to apply

Preprocess the two-donor (A29/A31) T&ILC scRNA-seq reference in .h5ad format with Scanpy — select highly variable genes and compute PCA so features sit in adata.obsm['X_pca'] (use_rep='X_pca'), since scValue expects log-normalised input with PCA features. Generate the sketch with scValue (sketch_size=0.1, cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42) alongside the six baselines (Uniform via Scanpy/Seurat, GeoSketch, Sphetcher, Hopper, KH, scSampler), and build a cell-type-annotated single-cell reference matrix from each sketch. Aggregate the 10 simulated bulk tissue samples into pseudobulk profiles by summing (or averaging) per-gene expression over cells within each sample, then run MuSiC deconvolution against each sketch-based reference to estimate cell-type proportions per sample. Compute Pearson correlation, RMSE, and MAE between estimated and ground-truth proportions per subsampling method with scikit-learn, assemble the Table 3 metrics table, and verify the scValue row reproduces Pearson 0.6851, RMSE 0.0592, MAE 0.0434 before saving as table3_deconvolution_metrics.csv.

## Related tools

- **scValue** (Builds the value-based sketch of the T&ILC reference used as the single-cell reference matrix for MuSiC deconvolution) — https://github.com/LHBCB/scvalue
- **MuSiC** (Runs bulk deconvolution on the pseudobulk profiles against each sketch-based reference to estimate cell-type proportions)
- **Scanpy** (Preprocesses the .h5ad reference (HVG selection, PCA into obsm['X_pca']) and provides uniform subsampling for the Uniform baseline)
- **scikit-learn** (Computes Pearson correlation, RMSE, and MAE between inferred and true cell-type proportions)

## Examples

```
pseudobulk = pd.DataFrame(adata.X, index=adata.obs['sample']).groupby('sample').sum()  # 10 simulated bulk samples -> pseudobulk profiles for MuSiC
```

## Evaluation signals

- The scValue row in table3_deconvolution_metrics.csv reproduces Pearson correlation 0.6851, RMSE 0.0592, and MAE 0.0434 across the 10 samples
- Pseudobulk aggregation yields exactly 10 sample-level profiles matching the 10 simulated bulk tissue samples, each with per-sample ground-truth proportions for comparison
- Seven method rows are present in the metrics table (scValue plus Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler), each with Pearson, RMSE, and MAE
- scValue outperforms all six baselines on all three metrics (highest Pearson, lowest RMSE and MAE) on the T&ILC pseudobulk benchmark

## Limitations

- Benchmark is specific to the simulated bulk tissue setting with known ground-truth proportions; performance on real bulk samples without truth is not validated here
- Deconvolution quality depends on the fidelity of the sketch-derived reference; uniform subsampling may not capture the full transcriptional diversity of the dataset
- No accession numbers or repository identifiers are provided for the T&ILC (A29/A31) dataset in the article text, complicating exact reproduction

## Evidence

- [results] scValue achieved the best deconvolution performance on the T&ILC pseudobulk benchmark: highest average Pearson correlation (0.6851), lowest RMSE (0.0592), and lowest MAE (0.0434): [evidence span withheld — anchor status: not_found]
- [abstract] Deconvolution case study on simulated bulk tissue: [evidence span withheld — anchor status: not_found]
- [methods] Aggregate samples into pseudobulk and run MuSiC: [evidence span withheld — anchor status: not_found]
