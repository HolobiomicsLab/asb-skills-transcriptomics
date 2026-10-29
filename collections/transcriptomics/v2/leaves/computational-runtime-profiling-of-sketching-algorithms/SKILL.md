---
name: computational-runtime-profiling-of-sketching-algorithms
description: Use when comparing subsampling/sketching methods for large scRNA-seq
  datasets on computational time, cell-type balance (Gini coefficient), and distributional
  fidelity (Hausdorff distance), or when reproducing the scValue Table 4 rank comparison
  across 16 datasets.
when_to_use_negative:
- The analysis goal is downstream ML/DL task performance (e.g. cell-type annotation
  accuracy with scANVI or CellTypist) rather than subsampling efficiency and distributional
  fidelity metrics
- The input AnnData lacks cell type annotations usable via cell_type_key, since scValue's
  DV computation and per-type sketch allocation depend on labeled cell types
- Only a single dataset is available, making the rank-averaging across 16 datasets
  (mean ± SD ranks) undefined or uninformative
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scValue
  role: Primary sketching method being profiled; generates 10% sketches and records
    runtime via SCValue(...).value() with use_rep='X_pca'
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Preprocesses each .h5ad dataset (log-normalization, top 3,000 HVGs, 50 PCs
    in adata.obsm['X_pca']) before sketching
- name: Seurat
  role: Reference for uniform subsampling baseline (as in Seurat/Scanpy pipelines)
- name: GeoSketch
  role: 'Comparator method: box partitioning of gene expression space with representative
    cell selection'
- name: Sphetcher
  role: 'Comparator method: covering cells by small-radius spheres'
- name: Hopper
  role: Comparator method with theoretical guarantees for maintaining dataset structure
- name: scSampler
  role: 'Comparator method: maximin distance (inverse distance measure) selection'
- name: KH
  role: Comparator method included in the six-method benchmark alongside Uniform,
    GeoSketch, Sphetcher, Hopper, scSampler
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:b9dda1637839e0ec96c834f371f3e0e1f4e59502ee7a4d887bfe284e5c49ab8f
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
  - Scanpy
  - Seurat
  - Sphetcher
  - scSampler
  - scValue
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# computational-runtime-profiling-of-sketching-algorithms

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the wall-clock runtime (and complementary quality metrics) of scRNA-seq subsampling ('sketching') algorithms — scValue versus Uniform, GeoSketch, Sphetcher, Hopper, KH, and scSampler — at a fixed sketch fraction across multiple public .h5ad datasets, ranking methods per dataset to compare with published rank summaries.

## When to use

Use when comparing subsampling/sketching methods for large scRNA-seq datasets on computational time, cell-type balance (Gini coefficient), and distributional fidelity (Hausdorff distance), or when reproducing the scValue Table 4 rank comparison across 16 datasets.

## When NOT to use

- The analysis goal is downstream ML/DL task performance (e.g. cell-type annotation accuracy with scANVI or CellTypist) rather than subsampling efficiency and distributional fidelity metrics
- The input AnnData lacks cell type annotations usable via cell_type_key, since scValue's DV computation and per-type sketch allocation depend on labeled cell types
- Only a single dataset is available, making the rank-averaging across 16 datasets (mean ± SD ranks) undefined or uninformative

## Inputs

- 16 public scRNA-seq datasets in .h5ad format (e.g. mTC mouse T-cell dataset, 89,429 cells, from CZ CELLxGENE)
- Scanpy-preprocessed AnnData objects: log-normalized, 3,000 HVGs, 50 PCs in adata.obsm['X_pca']
- Cell type annotations in adata.obs (column matching cell_type_key)

## Outputs

- Wall-clock runtimes for each of the seven sketching methods per dataset at sketch_size=0.1
- Gini coefficient over cell-type proportions and Hausdorff distance between sketch and full-data distributions per method per dataset
- Mean ± SD rank summaries per metric across datasets; per-dataset metrics plus rank table written to method_rankings_table4.csv

## How to apply

Preprocess each input .h5ad with Scanpy (log-normalized data, top 3,000 highly variable genes, 50 PCs stored in adata.obsm['X_pca']), then generate a 10% sketch per dataset with each method, recording wall-clock runtime for each. Run scValue via SCValue(adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42); run uniform subsampling as in Seurat/Scanpy pipelines plus GeoSketch box partitioning, Sphetcher sphere covering, Hopper, KH, and scSampler maximin selection. For each sketch compute the Gini coefficient over cell-type proportions and the Hausdorff distance between sketch and full-data feature distributions. Rank the seven methods per dataset on each of the three metrics and average ranks across the 16 datasets to obtain mean ± SD rank summaries; the paper's reported targets are scValue time rank 2.4 ± 0.8, Gini rank 2.1 ± 1.5, and Hausdorff rank 5.3 ± 1.1.

## Related tools

- **scValue** (Primary sketching method being profiled; generates 10% sketches and records runtime via SCValue(...).value() with use_rep='X_pca') — https://github.com/LHBCB/scvalue
- **Scanpy** (Preprocesses each .h5ad dataset (log-normalization, top 3,000 HVGs, 50 PCs in adata.obsm['X_pca']) before sketching)
- **Seurat** (Reference for uniform subsampling baseline (as in Seurat/Scanpy pipelines))
- **GeoSketch** (Comparator method: box partitioning of gene expression space with representative cell selection)
- **Sphetcher** (Comparator method: covering cells by small-radius spheres)
- **Hopper** (Comparator method with theoretical guarantees for maintaining dataset structure)
- **scSampler** (Comparator method: maximin distance (inverse distance measure) selection)
- **KH** (Comparator method included in the six-method benchmark alongside Uniform, GeoSketch, Sphetcher, Hopper, scSampler)

## Examples

```
from scvalue import SCValue; scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42); adata_sub = scv.value()
```

## Evaluation signals

- Reproduced mean ranks should match the reported Table 4 values: scValue time rank 2.4 ± 0.8, Gini rank 2.1 ± 1.5, Hausdorff rank 5.3 ± 1.1 (second fastest behind Uniform)
- Each method's sketch should contain ~10% of cells (sketch_size=0.1) per dataset
- Per-dataset metrics and ranks written to method_rankings_table4.csv with all seven methods and 16 datasets represented
- scValue DV values should be stored in adata.obs['dv'] and propagated to adata_sub.obs['dv'] after each run
- Runtimes must be recorded as wall-clock time under identical preprocessing (3,000 HVGs, 50 PCs) for all seven methods

## Limitations

- Tool versions for comparator software (GeoSketch, Sphetcher, Hopper, KH, scSampler) are not specified; only scValue's environment is documented (Python 3.10, scikit-learn 1.5.2, SciPy 1.14.0, NumPy 1.26.4, pandas 1.5.3, joblib 1.4.2; experiments used Scanpy 1.10.2 and AnnData 0.10.8)
- No accession numbers or repository identifiers are provided for the 16 benchmark datasets; the README documents only the mTC dataset's direct download URL
- Uniform subsampling is expected to be fastest, so runtime comparisons alone do not capture transcriptional diversity or cell-type balance

## Evidence
