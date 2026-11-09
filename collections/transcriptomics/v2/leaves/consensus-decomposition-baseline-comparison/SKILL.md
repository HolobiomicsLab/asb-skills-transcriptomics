---
name: consensus-decomposition-baseline-comparison
description: Use when a synthetic scRNA-seq benchmark with planted generative patterns
  (identity and activity states) is available and you need to quantify whether a decomposition
  method recovers them better than consensus NMF, LDA, ICA, or Louvain baselines.
when_to_use_negative:
- Data are real (non-synthetic) cells with no planted generative patterns, since recovery
  purity cannot be computed by matching to planted patterns.
- The analysis goal is cell-type discovery on a new dataset rather than benchmarking
  decomposition methods against each other.
- Baseline methods require a priori component selection and the planted pattern count
  is unknown, making the fixed-component comparison invalid.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ACTIONet
  role: Runs the multiresolution ACTION archetypal decomposition (reduce_kernel, an.run_ACTIONet)
    whose pattern recovery is being benchmarked
  repo: https://github.com/shmohammadi86/ACTIONet
- name: Leiden
  role: Graph-clustering baseline used in the consensus benchmark over pattern/cell
    graphs
- name: scanpy
  role: Python preprocessing (normalize_total, log1p, read_10x_h5) of instances before
    decomposition
- name: Python
  role: Implementation environment for ACTIONet Python package and baseline pipelines
- name: R
  role: Implementation environment for ACTIONet R-release package
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:efaf17f97b918e37fb3234fa033068e9289a4dde324cd056e52d9eeefb61e0f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - Leiden
  - Python
  - R
  - scanpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# consensus-decomposition-baseline-comparison

## Summary

Benchmarks ACTIONet's multiresolution archetypal decomposition against consensus baselines (consensus NMF, LDA, ICA, and Louvain clustering) on synthetic scRNA-seq data by measuring pattern-recovery purity for planted identity and activity patterns.

## When to use

Use when a synthetic scRNA-seq benchmark with planted generative patterns (identity and activity states) is available and you need to quantify whether a decomposition method recovers them better than consensus NMF, LDA, ICA, or Louvain baselines.

## When NOT to use

- Data are real (non-synthetic) cells with no planted generative patterns, since recovery purity cannot be computed by matching to planted patterns.
- The analysis goal is cell-type discovery on a new dataset rather than benchmarking decomposition methods against each other.
- Baseline methods require a priori component selection and the planted pattern count is unknown, making the fixed-component comparison invalid.

## Inputs

- Synthetic scRNA-seq count matrices from ref. 12 (20 generated instances, 14 planted identity/activity patterns, 3 noise levels)
- Reference number of components (planted pattern count) for baseline decomposition methods

## Outputs

- pattern_recovery_purity.csv with purity values per method, instance, and noise level
- Aggregated purity comparison versus reported Fig. 1b-d results

## How to apply

Load the synthetic benchmark from ref. 12 (20 instances, 14 planted patterns, 3 noise levels), preprocess each instance with ACTIONet's filter_adata (min_cells_per_feature=0.01, min_features_per_cell=1000), library-size normalization (normalize_total), and log1p transformation. Reduce the processed matrix with ACTIONet's randomized SVD-based reduce_kernel, then run an.run_ACTIONet for the multiresolution ACTION archetypal decomposition; run the four consensus baselines on the identical preprocessed instances with the component count set to the planted pattern count. Match each recovered pattern to the planted generative patterns and compute recovery purity per method, instance, and noise level, aggregating across the 20 instances and 3 noise levels. The decision point is whether ACTIONet achieves near-ceiling purity on both identity and activity states while baselines recover only one class of pattern or degrade with noise.

## Related tools

- **ACTIONet** (Runs the multiresolution ACTION archetypal decomposition (reduce_kernel, an.run_ACTIONet) whose pattern recovery is being benchmarked) — https://github.com/shmohammadi86/ACTIONet
- **Leiden** (Graph-clustering baseline used in the consensus benchmark over pattern/cell graphs)
- **scanpy** (Python preprocessing (normalize_total, log1p, read_10x_h5) of instances before decomposition)
- **Python** (Implementation environment for ACTIONet Python package and baseline pipelines)
- **R** (Implementation environment for ACTIONet R-release package)

## Examples

```
an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000); sc.pp.normalize_total(adata); sc.pp.log1p(adata); an.pp.reduce_kernel(adata); an.run_ACTIONet(adata)
```

## Evaluation signals

- ACTIONet recovers all planted identity AND activity patterns with high (near-ceiling) purity across all 3 noise levels, matching Fig. 1b-d
- Consensus NMF, LDA, and ICA baselines do not simultaneously recover both identity and activity states at comparable purity
- Purity is computed by matching each recovered pattern to a planted generative pattern and is aggregated over all 20 instances and 3 noise levels
- Results are written to pattern_recovery_purity.csv with one row per method/instance/noise-level combination
- Baselines are run on the identical preprocessed instances (same filter_adata, normalize_total, log1p steps) so the comparison is fair

## Limitations

- Matrix decomposition methods require a priori selection of the number of underlying dominant patterns, which affects baseline behavior and is not trivial to determine.
- The synthetic dataset from ref. 12 is cited only by reference without accessions or download links, and no tool versions are stated for ACTIONet, R, Python, or Leiden.
- Findings are limited to the tested synthetic cases; real-data validation used separate PBMC and cortex datasets rather than purity metrics.

## Evidence

- [discussion] Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns: "Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns"
- [methods] an.pp.filter_adata: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [methods] an.pp.reduce_kernel: "an.pp.reduce_kernel(adata)"
- [discussion] Leiden graph-clustering: "We then applied the Leiden graph-clustering algorithms over this pattern graph to define equivalent patterns classes"
- [discussion] randomized SVD-based low-rank approximation: "an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size"
- [abstract] superior performance relative to existing alternatives: "Using multiple synthetic and real data sets, we demonstrate ACTIONet's superior performance relative to existing alternatives"
