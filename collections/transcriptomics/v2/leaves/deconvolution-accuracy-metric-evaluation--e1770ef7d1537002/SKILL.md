---
name: deconvolution-accuracy-metric-evaluation
description: Use when a downsampling/sketching method must be judged by its ability
  to preserve deconvolution accuracy — i.e. cell-type-annotated scRNA-seq reference
  data and bulk (or pseudobulk) samples with known ground-truth proportions are available.
when_to_use_negative:
- The bulk samples have no known ground-truth cell-type proportions — Pearson/RMSE/MAE
  against truth cannot be computed and only a qualitative comparison is possible
- The task is cell-type annotation quality (e.g. scANVI/scPoli/CellTypist benchmarks)
  rather than bulk deconvolution reference quality
- The single-cell reference lacks cell-type labels (adata.obs cell_type_key), since
  both scValue sketching and MuSiC require annotated cell types
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
tools:
- name: scValue
  role: Generates the DV-guided sketch (sketch_size=0.1, type_weighting='inv_cv',
    strategy='FB') from which the single-cell deconvolution reference is built
  repo: https://github.com/LHBCB/scvalue
- name: MuSiC
  role: Runs bulk deconvolution of the 10 pseudobulk samples against each sketch-based
    single-cell reference to estimate cell-type proportions
- name: Scanpy
  role: Preprocesses the .h5ad reference (sc.pp.highly_variable_genes, sc.pp.pca storing
    PCs in adata.obsm['X_pca']) and implements the Uniform baseline subsampling
- name: GeoSketch
  role: Baseline sketching method compared against scValue in the deconvolution benchmark
- name: Sphetcher
  role: Baseline sketching method compared against scValue in the deconvolution benchmark
- name: Hopper
  role: Baseline sketching method compared against scValue in the deconvolution benchmark
- name: scSampler
  role: Baseline maximin-distance sketching method compared against scValue in the
    deconvolution benchmark
- name: scikit-learn
  role: Implements the random-forest OOB data-value estimation behind scValue and
    metric computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:7121c567bc3bc95d2cd1fc3fab3413a2567b1a2e892334b2144506ddd230b3b8
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
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# deconvolution-accuracy-metric-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark of how well a cell-type-annotated single-cell reference (built from a sketch such as scValue) supports bulk RNA-seq deconvolution: run MuSiC against each sketch-based reference on pseudobulk profiles, then score estimated vs. true cell-type proportions with Pearson correlation, RMSE and MAE.

## When to use

Use when a downsampling/sketching method must be judged by its ability to preserve deconvolution accuracy — i.e. cell-type-annotated scRNA-seq reference data and bulk (or pseudobulk) samples with known ground-truth proportions are available.

## When NOT to use

- The bulk samples have no known ground-truth cell-type proportions — Pearson/RMSE/MAE against truth cannot be computed and only a qualitative comparison is possible
- The task is cell-type annotation quality (e.g. scANVI/scPoli/CellTypist benchmarks) rather than bulk deconvolution reference quality
- The single-cell reference lacks cell-type labels (adata.obs cell_type_key), since both scValue sketching and MuSiC require annotated cell types

## Inputs

- Cell-type-annotated scRNA-seq reference in .h5ad format (two-donor A29/A31 T&ILC reference; log-normalised, HVGs and PCA in adata.obsm['X_pca'])
- Simulated bulk tissue samples (10 samples) with known ground-truth cell-type proportions, aggregated into pseudobulk profiles
- Sketches from seven subsamplers: scValue, Uniform (Scanpy/Seurat), GeoSketch, Sphetcher, Hopper, KH, scSampler

## Outputs

- Estimated cell-type proportion matrix per sketching method (from MuSiC deconvolution)
- Table 3 metrics table (table3_deconvolution_metrics.csv) with Pearson correlation, RMSE and MAE per subsampling method (scValue: 0.6851 / 0.0592 / 0.0434)

## How to apply

Load the two-donor (A29/A31) T&ILC scRNA-seq reference as .h5ad and preprocess with Scanpy (select highly variable genes, compute PCA so features sit in adata.obsm['X_pca'], use_rep='X_pca'). Generate sketches with scValue (sketch_size=0.1, cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42) and the six baselines (Uniform via Scanpy/Seurat, GeoSketch, Sphetcher, Hopper, KH, scSampler). Build a cell-type-annotated single-cell reference matrix from each sketch, aggregate the 10 simulated bulk tissue samples into pseudobulk profiles, and run MuSiC deconvolution against each sketch-based reference. Compare estimated proportions to known ground truth and compute Pearson correlation, RMSE and MAE per method — correct application reproduces the scValue row: Pearson 0.6851, RMSE 0.0592, MAE 0.0434.

## Related tools

- **scValue** (Generates the DV-guided sketch (sketch_size=0.1, type_weighting='inv_cv', strategy='FB') from which the single-cell deconvolution reference is built) — https://github.com/LHBCB/scvalue
- **MuSiC** (Runs bulk deconvolution of the 10 pseudobulk samples against each sketch-based single-cell reference to estimate cell-type proportions)
- **Scanpy** (Preprocesses the .h5ad reference (sc.pp.highly_variable_genes, sc.pp.pca storing PCs in adata.obsm['X_pca']) and implements the Uniform baseline subsampling)
- **GeoSketch** (Baseline sketching method compared against scValue in the deconvolution benchmark)
- **Sphetcher** (Baseline sketching method compared against scValue in the deconvolution benchmark)
- **Hopper** (Baseline sketching method compared against scValue in the deconvolution benchmark)
- **scSampler** (Baseline maximin-distance sketching method compared against scValue in the deconvolution benchmark)
- **scikit-learn** (Implements the random-forest OOB data-value estimation behind scValue and metric computation)

## Examples

```
from scvalue import SCValue; scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42); adata_sub = scv.value()
```

## Evaluation signals

- The scValue row of the assembled metrics table matches Pearson 0.6851, RMSE 0.0592 and MAE 0.0434 across the 10 samples
- scValue achieves the highest Pearson and lowest RMSE/MAE among all seven sketching methods on the T&ILC pseudobulk benchmark
- DV values are stored in adata.obs['dv'] (and adata_sub.obs['dv']) and the sketch is returned as AnnData via scv.adata_sub
- Estimated proportions per method are comparable to ground truth only when the sketch retains cell-type-annotated structure, checked by cell-type representation in the reference matrix

## Limitations

- Benchmarked only on simulated bulk tissue samples with known ground truth (10 samples, T&ILC two-donor reference); performance on real bulk data with unknown composition is not established
- Tool versions for MuSiC and the baseline sketchers (KH, Sphetcher, Hopper, scSampler) are not specified, complicating exact reproduction
- Random-forest OOB DV estimates can be unstable in noisy conditions; the README notes tuning n_trees and maximum depth may stabilise them, and the scValue-core filtering threshold is user-defined with no default
- No accession numbers or repository identifiers are provided for the benchmark datasets (PBMC, mBrain, CxG_min, mACA, Gut, Colon, Spleen, T&ILC)

## Evidence

- [results] Deconvolution benchmark metrics: [evidence span withheld — anchor status: not_found]
- [abstract] scValue sketch reference: [evidence span withheld — anchor status: not_found]
