---
name: cell-type-annotated-reference-construction
description: Use when a large cell-type-annotated scRNA-seq reference (.h5ad) must
  be reduced to a representative sketch to serve as the reference matrix for deconvolution
  of pseudobulk/bulk RNA-seq samples, and sketch quality must be judged by proportion-estimate
  accuracy.
when_to_use_negative:
- The goal is annotation-quality model training (e.g. scANVI/CellTypist label transfer)
  rather than serving as a deconvolution reference matrix
- No cell-type labels are available in the scRNA-seq reference, since MuSiC needs
  cell-type-stratified reference profiles
- The input is already a compact reference matrix with no need for subsampling
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scValue
  role: Generates the value-based sketch (10% of cells, inv_cv weighting, FB strategy)
    used to build the cell-type-annotated deconvolution reference
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Preprocesses the .h5ad reference (highly variable gene selection, PCA into
    adata.obsm['X_pca']) and provides uniform subsampling baseline
- name: MuSiC
  role: Performs bulk deconvolution against each sketch-based reference to estimate
    cell-type proportions
- name: GeoSketch
  role: Baseline subsampling method compared for deconvolution accuracy
- name: Sphetcher
  role: Baseline subsampling method compared for deconvolution accuracy
- name: Hopper
  role: Baseline subsampling method compared for deconvolution accuracy
- name: scSampler
  role: Baseline subsampling method compared for deconvolution accuracy
- name: scikit-learn
  role: Implementation dependency for scValue's random-forest data value estimation
    (scikit-learn 1.5.2)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:028642f98f033e74fdb0c18703e792d533a0ea8191c85f5778da2f2127b1ee82
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cell-type-annotated-reference-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a cell-type-annotated single-cell reference matrix from a sketched scRNA-seq dataset (e.g. a scValue sketch of the two-donor A29/A31 T&ILC reference) and uses it as the reference profile for MuSiC bulk RNA-seq deconvolution, benchmarking subsampling methods by accuracy of inferred cell-type proportions.

## When to use

Use when a large cell-type-annotated scRNA-seq reference (.h5ad) must be reduced to a representative sketch to serve as the reference matrix for deconvolution of pseudobulk/bulk RNA-seq samples, and sketch quality must be judged by proportion-estimate accuracy.

## When NOT to use

- The goal is annotation-quality model training (e.g. scANVI/CellTypist label transfer) rather than serving as a deconvolution reference matrix
- No cell-type labels are available in the scRNA-seq reference, since MuSiC needs cell-type-stratified reference profiles
- The input is already a compact reference matrix with no need for subsampling

## Inputs

- Cell-type-annotated scRNA-seq reference in .h5ad format (e.g. two-donor A29/A31 T&ILC reference)
- Bulk or pseudobulk expression profiles for the target samples (10 simulated bulk tissue samples)
- Ground-truth cell-type proportions per bulk sample for evaluation
- Cell type label column in adata.obs (cell_type_key='cell_type')

## Outputs

- Cell-type-annotated single-cell reference matrix derived from each sketch
- Per-sample estimated cell-type proportions from MuSiC deconvolution
- Metrics table (Pearson correlation, RMSE, MAE) per subsampling method, e.g. table3_deconvolution_metrics.csv

## How to apply

Load the cell-type-annotated scRNA-seq reference in .h5ad format (here the two-donor A29/A31 T&ILC scRNA-seq reference) and preprocess with Scanpy: select highly variable genes and compute PCA so features are available in adata.obsm['X_pca'] (use_rep='X_pca'). Generate a sketch — scValue with sketch_size=0.1, cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42, plus baselines (Uniform via Scanpy/Seurat, GeoSketch, Sphetcher, Hopper, KH, scSampler) when benchmarking. From each sketch, assemble a cell-type-annotated single-cell reference matrix for MuSiC deconvolution. Aggregate the bulk samples (10 simulated bulk tissue samples) into pseudobulk profiles, run MuSiC against each sketch-based reference to estimate cell-type proportions, and compare with ground-truth proportions using Pearson correlation, RMSE, and MAE — scValue achieved Pearson 0.6851, RMSE 0.0592, MAE 0.0434, outperforming all six baselines.

## Related tools

- **scValue** (Generates the value-based sketch (10% of cells, inv_cv weighting, FB strategy) used to build the cell-type-annotated deconvolution reference) — https://github.com/LHBCB/scvalue
- **Scanpy** (Preprocesses the .h5ad reference (highly variable gene selection, PCA into adata.obsm['X_pca']) and provides uniform subsampling baseline)
- **MuSiC** (Performs bulk deconvolution against each sketch-based reference to estimate cell-type proportions)
- **GeoSketch** (Baseline subsampling method compared for deconvolution accuracy)
- **Sphetcher** (Baseline subsampling method compared for deconvolution accuracy)
- **Hopper** (Baseline subsampling method compared for deconvolution accuracy)
- **scSampler** (Baseline subsampling method compared for deconvolution accuracy)
- **scikit-learn** (Implementation dependency for scValue's random-forest data value estimation (scikit-learn 1.5.2))

## Examples

```
scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42); adata_sub = scv.value()
```

## Evaluation signals

- scValue row of the metrics table reproduces Pearson 0.6851, RMSE 0.0592, and MAE 0.0434 on the T&ILC pseudobulk benchmark
- Estimated proportions from MuSiC correlate with known ground-truth proportions across all 10 samples
- Sketch reference preserves cell-type labels: adata_sub.obs contains cell_type and dv columns
- PCA representation exists in adata.obsm['X_pca'] before sketching (use_rep='X_pca')
- Table3 saved as table3_deconvolution_metrics.csv with one row per subsampling method

## Limitations

- Benchmark datasets (e.g. T&ILC A29/A31) lack accession numbers/repository identifiers in the article, complicating exact reproduction
- Tool versions for MuSiC and some baseline subsamplers (KH, Sphetcher, Hopper, scSampler) are not specified
- scValue's use_rep='X' with sparse expression matrices may trigger internal densification (memory cost)

## Evidence

- [results] scValue achieved the best deconvolution performance on the T&ILC pseudobulk benchmark: [evidence span withheld — anchor status: not_found]
- [abstract] constructed a more reliable single-cell immune reference for deconvolution: [evidence span withheld — anchor status: not_found]
- [readme] README: expects features from adata.obsm X_pca: "By default, scValue expects features from `adata.obsm["X_pca"]` (`use_rep="X_pca"`)"
- [readme] README: parameter settings for sketch: "sketch_size=0.1,          # float in (0, 1] or an integer cell count"
- [readme] README: strategy definition: ""FB" (full binning, default): DV is binned into 0.1 intervals from 0.0 to 1.0; selects top-DV cells within each bin."
