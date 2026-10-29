---
name: bulk-rnaseq-deconvolution-with-single-cell-references
description: Use when estimating cell-type proportions in bulk or simulated bulk RNA-seq
  samples with a single-cell reference, and when choosing which subsampling/sketching
  method should build that reference from a large scRNA-seq atlas.
when_to_use_negative:
- The task is cell-type annotation or label transfer of single-cell data rather than
  estimating proportions in bulk samples — the scValue benchmark's core annotation
  tasks (PBMC/scANVI, mBrain/scPoli) are a different skill
- The bulk samples have no ground-truth or reference proportions, so Pearson/RMSE/MAE
  against truth cannot be computed for method comparison
- The single-cell reference lacks cell-type labels (cell_type_key column), which MuSiC's
  reference-based deconvolution requires
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: MuSiC
  role: Runs the bulk deconvolution against each sketch-based single-cell reference
    to estimate cell-type proportions
- name: scValue
  role: Generates the value-based sketch of the single-cell reference (sketch_size=0.1,
    strategy='FB', type_weighting='inv_cv') that achieved the best deconvolution metrics
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Preprocesses the .h5ad reference (highly variable gene selection, PCA into
    X_pca) and implements uniform random subsampling baseline
- name: GeoSketch
  role: 'Baseline subsampler: partitions gene expression space into equal-sized boxes
    and selects representative cells'
- name: Sphetcher
  role: 'Baseline subsampler: covers cells with small-radius spheres to preserve the
    transcriptomic landscape'
- name: Hopper
  role: Baseline subsampler with theoretical guarantees for maintaining dataset structure
- name: scSampler
  role: Baseline subsampler using a maximin distance design
- name: scikit-learn
  role: Computes Pearson correlation, RMSE and MAE between inferred and true cell-type
    proportions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:6b5a6f4362a0f51159aa871cb6202e8a7581b642e0bb34563432eb0775e8547d
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
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# bulk-rnaseq-deconvolution-with-single-cell-references

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark cell-type deconvolution of bulk/pseudobulk RNA-seq samples against sketch-based single-cell references using MuSiC, comparing subsampling methods (scValue, Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler) by Pearson correlation, RMSE and MAE against known true proportions.

## When to use

Use when estimating cell-type proportions in bulk or simulated bulk RNA-seq samples with a single-cell reference, and when choosing which subsampling/sketching method should build that reference from a large scRNA-seq atlas.

## When NOT to use

- The task is cell-type annotation or label transfer of single-cell data rather than estimating proportions in bulk samples — the scValue benchmark's core annotation tasks (PBMC/scANVI, mBrain/scPoli) are a different skill
- The bulk samples have no ground-truth or reference proportions, so Pearson/RMSE/MAE against truth cannot be computed for method comparison
- The single-cell reference lacks cell-type labels (cell_type_key column), which MuSiC's reference-based deconvolution requires

## Inputs

- Single-cell reference in .h5ad format with cell-type annotations (e.g. T&ILC two-donor A29/A31 dataset), log-normalised with HVG-selected PCA in adata.obsm['X_pca']
- Bulk or pseudobulk RNA-seq samples with known ground-truth cell-type proportions (e.g. 10 simulated bulk tissue samples)
- Sketches produced by each subsampler (scValue, Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler) at a fixed sketch size

## Outputs

- Per-method estimated cell-type proportions for each bulk sample from MuSiC
- Metrics table (e.g. table3_deconvolution_metrics.csv) with Pearson correlation, RMSE and MAE per subsampling method
- Ranked comparison identifying the best-performing sketching method for deconvolution

## How to apply

Load the single-cell reference (e.g. the two-donor A29/A31 T&ILC scRNA-seq dataset in .h5ad), preprocess with Scanpy by selecting highly variable genes and computing PCA stored in adata.obsm['X_pca'], then generate sketches with scValue (sketch_size=0.1, cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42) and the six baselines (Uniform via Scanpy/Seurat, GeoSketch, Sphetcher, Hopper, KH, scSampler). Build a cell-type-annotated reference matrix from each sketch, aggregate the 10 bulk samples into pseudobulk profiles, and run MuSiC deconvolution against each sketch-based reference. Compare estimated proportions to ground truth per method using Pearson correlation, RMSE and MAE; the scValue sketch should yield the best metrics (Pearson 0.6851, RMSE 0.0592, MAE 0.0434 on this benchmark), reflecting its DV-weighted, inv_cv type weighting that favours cell types with higher variability in data value.

## Related tools

- **MuSiC** (Runs the bulk deconvolution against each sketch-based single-cell reference to estimate cell-type proportions)
- **scValue** (Generates the value-based sketch of the single-cell reference (sketch_size=0.1, strategy='FB', type_weighting='inv_cv') that achieved the best deconvolution metrics) — https://github.com/LHBCB/scvalue
- **Scanpy** (Preprocesses the .h5ad reference (highly variable gene selection, PCA into X_pca) and implements uniform random subsampling baseline)
- **GeoSketch** (Baseline subsampler: partitions gene expression space into equal-sized boxes and selects representative cells)
- **Sphetcher** (Baseline subsampler: covers cells with small-radius spheres to preserve the transcriptomic landscape)
- **Hopper** (Baseline subsampler with theoretical guarantees for maintaining dataset structure)
- **scSampler** (Baseline subsampler using a maximin distance design)
- **scikit-learn** (Computes Pearson correlation, RMSE and MAE between inferred and true cell-type proportions)

## Examples

```
scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42); adata_sub = scv.value()
```

## Evaluation signals

- scValue's reference reproduces Pearson correlation 0.6851, RMSE 0.0592 and MAE 0.0434 across the 10 T&ILC pseudobulk samples
- scValue row ranks first on all three metrics (highest Pearson, lowest RMSE and MAE) versus all six baseline sketching methods
- Sketches are built with identical sketch_size (0.1) and seed so method differences reflect sampling strategy, not subset size
- Estimated proportions are non-negative and sum to ~1 per sample for every MuSiC run
- Metrics table saved as table3_deconvolution_metrics.csv contains one row per subsampling method with all three metrics populated

## Limitations

- Deconvolution was evaluated on simulated bulk tissue samples with known ground-truth proportions; real bulk samples lack ground truth, so method ranking cannot be directly verified
- The T&ILC reference is a two-donor (A29/A31) dataset, so results may not generalise to references with more donors or broader tissue diversity
- Tool versions for baseline methods (KH, Sphetcher, Hopper, scSampler, MuSiC) are not specified in the article, complicating exact reproduction
- No accession numbers are given for the T&ILC dataset, requiring retrieval from the paper's data availability section

## Evidence

- [results] scValue achieved the best deconvolution performance on the T&ILC pseudobulk benchmark: highest average Pearson correlation (0.6851), lowest RMSE (0.0592), and lowest MAE (0.0434) between inferred and true cell-type proportions across the 10 samples, outperforming all six baseline sketching methods.: [evidence span withheld — anchor status: not_found]
- [abstract] scValue more effectively preserved T-cell annotations across human gut-colon datasets, more accurately reproduced T-cell subtype relationships in a human spleen dataset, and constructed a more: "scValue more effectively preserved T-cell annotations across human gut-colon datasets, more accurately reproduced T-cell subtype relationships in a human spleen dataset, and constructed a more"
- [methods] Load the two-donor (A29/A31) T&ILC scRNA-seq reference in .h5ad format and preprocess with Scanpy: select highly variable genes and compute PCA so features are available in adata.obsm['X_pca'] (use_rep='X_pca').: [evidence span withheld — anchor status: not_found]
- [methods] Aggregate the 10 simulated bulk tissue samples into pseudobulk profiles and run MuSiC deconvolution against each sketch-based reference to estimate cell-type proportions.: [evidence span withheld — anchor status: not_found]
- [readme] By default, scValue expects features from `adata.obsm["X_pca"]` (`use_rep="X_pca"`).: "By default, scValue expects features from `adata.obsm["X_pca"]` (`use_rep="X_pca"`)."
