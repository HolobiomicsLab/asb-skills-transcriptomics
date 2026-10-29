---
name: single-cell-subsampling-method-benchmarking
description: Use when comparing sketching/subsampling methods on large scRNA-seq datasets
  (tens of thousands to millions of cells) with metrics like runtime, Gini coefficient,
  Hausdorff distance, or downstream deconvolution/annotation accuracy.
when_to_use_negative:
- The dataset is small enough to analyse in full — sketching trades fidelity for efficiency
  and full-data performance is the reference baseline
- The evaluation target is a purely distributional property other than cell-type balance
  and feature-distribution fidelity (e.g. spatial or trajectory structure), which
  the time/Gini/Hausdorff triad does not capture
- Cell-type labels (cell_type_key) are absent or unreliable, since scValue's DV-weighted
  per-type allocation and the Gini coefficient both require them
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scValue
  role: Primary subsampling method under benchmark; assigns each cell a data value
    (DV) via out-of-bag random forest estimates and selects cells with FB/MTB/TP strategies
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Preprocessing (.h5ad loading, highly_variable_genes with n_top_genes=3000,
    PCA with 50 comps into adata.obsm['X_pca']) and uniform subsampling comparator
- name: Seurat
  role: Reference pipeline for uniform subsampling comparator
- name: GeoSketch
  role: Comparator that partitions gene expression space into equal-sized, non-overlapping
    boxes
- name: Sphetcher
  role: Comparator that covers cells by small-radius spheres
- name: Hopper
  role: Comparator with theoretical guarantees for maintaining dataset structure
- name: TreeHopper
  role: Partition-based variant of Hopper that accelerates sketching
- name: scSampler
  role: Comparator using maximin distance design
- name: KH
  role: Distribution-based sketching comparator
- name: MuSiC
  role: Bulk deconvolution engine run against sketch-based single-cell references
    to compute Pearson/RMSE/MAE on estimated vs true proportions
- name: scikit-learn
  role: Random forest for OOB-based data value computation (v1.5.2)
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:c3ffe2a9abb893bf81c277fe65f32a8916815f0b358b6ffa27e5871b589234b8
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
  - TreeHopper
  - scSampler
  - scValue
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-subsampling-method-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark scRNA-seq subsampling (sketching) methods — scValue, Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler — by generating 10% sketches, ranking them on computation time, Gini coefficient of cell-type proportions, and Hausdorff distance, and optionally validating sketches in downstream tasks such as MuSiC bulk deconvolution.

## When to use

Use when comparing sketching/subsampling methods on large scRNA-seq datasets (tens of thousands to millions of cells) with metrics like runtime, Gini coefficient, Hausdorff distance, or downstream deconvolution/annotation accuracy.

## When NOT to use

- The dataset is small enough to analyse in full — sketching trades fidelity for efficiency and full-data performance is the reference baseline
- The evaluation target is a purely distributional property other than cell-type balance and feature-distribution fidelity (e.g. spatial or trajectory structure), which the time/Gini/Hausdorff triad does not capture
- Cell-type labels (cell_type_key) are absent or unreliable, since scValue's DV-weighted per-type allocation and the Gini coefficient both require them

## Inputs

- Public scRNA-seq datasets in .h5ad format (e.g. mTC mouse T-cell dataset, 89,429 cells, from CZ CELLxGENE)
- Scanpy-preprocessed AnnData with log-normalized expression, top 3,000 HVGs, and 50 PCs in adata.obsm['X_pca']
- Cell-type annotations in adata.obs['cell_type']
- Ground-truth cell-type proportions for simulated bulk samples (deconvolution benchmark)

## Outputs

- Per-dataset wall-clock runtimes for all seven subsampling methods
- Gini coefficient over cell-type proportions per sketch
- Hausdorff distance between sketch and full-data feature distributions per sketch
- Per-dataset metrics plus averaged mean ± SD rank table (e.g. method_rankings_table4.csv, table3_deconvolution_metrics.csv)

## How to apply

Download the benchmark scRNA-seq datasets in .h5ad format (e.g. the mTC mouse T-cell dataset with 89,429 cells from CZ CELLxGENE) and preprocess each with Scanpy: log-normalized data, top 3,000 highly variable genes, and 50 PCs stored in adata.obsm['X_pca']. Generate a 10% sketch with each of the seven methods — scValue (sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42), uniform subsampling (as in Seurat/Scanpy pipelines), GeoSketch box partitioning, Sphetcher sphere covering, Hopper, KH, and scSampler maximin selection — recording each wall-clock runtime. For every sketch compute the Gini coefficient over cell-type proportions to quantify cell-type balance and the Hausdorff distance between sketch and full-data feature distributions to quantify distributional fidelity. Rank the seven methods per dataset on each metric and average ranks across the 16 datasets to obtain mean ± SD summaries (expected scValue ranks: time 2.4±0.8, Gini 2.1±1.5, Hausdorff 5.3±1.1). For downstream validation, build a sketch-based reference for MuSiC and compute Pearson correlation, RMSE, and MAE against known ground-truth proportions across simulated bulk samples (expected scValue: Pearson 0.6851, RMSE 0.0592, MAE 0.0434).

## Related tools

- **scValue** (Primary subsampling method under benchmark; assigns each cell a data value (DV) via out-of-bag random forest estimates and selects cells with FB/MTB/TP strategies) — https://github.com/LHBCB/scvalue
- **Scanpy** (Preprocessing (.h5ad loading, highly_variable_genes with n_top_genes=3000, PCA with 50 comps into adata.obsm['X_pca']) and uniform subsampling comparator)
- **Seurat** (Reference pipeline for uniform subsampling comparator)
- **GeoSketch** (Comparator that partitions gene expression space into equal-sized, non-overlapping boxes)
- **Sphetcher** (Comparator that covers cells by small-radius spheres)
- **Hopper** (Comparator with theoretical guarantees for maintaining dataset structure)
- **TreeHopper** (Partition-based variant of Hopper that accelerates sketching)
- **scSampler** (Comparator using maximin distance design)
- **KH** (Distribution-based sketching comparator)
- **MuSiC** (Bulk deconvolution engine run against sketch-based single-cell references to compute Pearson/RMSE/MAE on estimated vs true proportions)
- **scikit-learn** (Random forest for OOB-based data value computation (v1.5.2))

## Examples

```
scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42); adata_sub = scv.value()
```

## Evaluation signals

- scValue time rank averages 2.4 ± 0.8 across the 16 datasets (second only to Uniform)
- scValue Gini coefficient rank averages 2.1 ± 1.5 (best cell-type balance) and Hausdorff distance rank 5.3 ± 1.1, matching Table 4
- On the T&ILC (A29/A31) MuSiC deconvolution benchmark, scValue reproduces Pearson 0.6851, RMSE 0.0592, MAE 0.0434 across the 10 samples
- scValue runs with default parameters (sketch_size=0.1, use_rep='X_pca', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42) and DV is written to adata.obs['dv']
- Rank table is written with per-dataset metrics plus mean ± SD rank summaries to method_rankings_table4.csv

## Limitations

- scValue ranked worst among comparators on Hausdorff distance (rank 5.3 ± 1.1); it is distributional-property-like uniform sampling, not distance-optimized
- Tool versions were not specified for several benchmarked packages (GeoSketch, Sphetcher, Hopper, KH, scSampler, CellTypist, MuSiC, etc.), complicating exact reproduction
- No accession numbers or repository identifiers are provided for the 16 benchmark datasets (PBMC, mBrain, CxG_min, mACA, Gut, Colon, Spleen, T&ILC)
- Random forest hyperparameters (n_trees, max depth) may need tuning to stabilize OOB estimates in noisy conditions
- Strategy choice matters: FB preserves diversity across the full [0,1] DV range, while MTB considers only bins above the type mean DV and TP selects only highest-DV cells

## Evidence

- [results] In the 16-dataset comparison, scValue achieved a computation time rank of 2.4 ± 0.8 (second only to Uniform), the best Gini coefficient rank of 2.1 ± 1.5, and a Hausdorff distance rank of 5.3 ± 1.1.: [evidence span withheld — anchor status: not_found]
- [methods] Download the 16 public benchmark scRNA-seq datasets (e.g. the mTC mouse T-cell dataset with 89,429 cells from CZ CELLxGENE) in .h5ad format and preprocess each with Scanpy: [evidence span withheld — anchor status: not_found]
- [results] scValue achieved the best deconvolution performance on the T&ILC pseudobulk benchmark: highest average Pearson correlation (0.6851), lowest RMSE (0.0592), and lowest MAE (0.0434): [evidence span withheld — anchor status: not_found]
- [readme] sc.pp.highly_variable_genes(adata, n_top_genes=3000)
sc.pp.pca(adata, n_comps=50) # stores PCs in adata.obsm["X_pca"]: "By default, scValue expects features from `adata.obsm["X_pca"]` (`use_rep="X_pca"`)."
