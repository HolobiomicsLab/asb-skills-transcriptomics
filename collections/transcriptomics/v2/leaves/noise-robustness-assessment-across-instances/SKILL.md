---
name: noise-robustness-assessment-across-instances
description: 'Use when you need to quantify how noise-tolerant a decomposition/clustering
  method is on scRNA-seq data: you have synthetic instances with known planted identity/activity
  patterns (e.g., 20 instances, 14 planted patterns, 3 noise levels) and want pattern-recovery
  purity per noise level.'
when_to_use_negative:
- Input is real experimental data without planted ground-truth patterns — purity matching
  to generative patterns is undefined; use cell-type annotation or pathway validation
  instead.
- You only need a single-run application of ACTIONet to one dataset (e.g., the pbmc_10k_v3.h5
  PBMC workflow) rather than a multi-instance noise sweep.
- The method under test requires a fixed optimal number of clusters and cannot accept
  the reference number of components set to the planted pattern count for fair baseline
  comparison.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ACTIONet
  role: 'Method under test: multiresolution ACTION archetypal decomposition recovering
    identity and activity state patterns via reduce_kernel and an.run_ACTIONet'
  repo: https://github.com/shmohammadi86/ACTIONet
- name: scanpy
  role: 'Preprocessing of each synthetic instance: filter_adata, normalize_total,
    log1p before decomposition'
- name: Leiden
  role: Graph clustering used by ACTIONet to prune redundant archetypes into nonredundant
    equivalent pattern classes for purity scoring
- name: Python
  role: Environment for running the ACTIONet Python package and benchmark harness
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:1e53bad5b503ac354f5c58f92aa77c9e27bc4fb023098ac5e4ce13fabfa85d1c
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# noise-robustness-assessment-across-instances

## Summary

Benchmark a cell-state decomposition or clustering method's robustness to noise by running it over multiple synthetic scRNA-seq instances at several noise levels and scoring pattern-recovery purity against planted generative patterns, as done for ACTIONet versus cNMF, cLDA, cICA, and Louvain baselines.

## When to use

Use when you need to quantify how noise-tolerant a decomposition/clustering method is on scRNA-seq data: you have synthetic instances with known planted identity/activity patterns (e.g., 20 instances, 14 planted patterns, 3 noise levels) and want pattern-recovery purity per noise level.

## When NOT to use

- Input is real experimental data without planted ground-truth patterns — purity matching to generative patterns is undefined; use cell-type annotation or pathway validation instead.
- You only need a single-run application of ACTIONet to one dataset (e.g., the pbmc_10k_v3.h5 PBMC workflow) rather than a multi-instance noise sweep.
- The method under test requires a fixed optimal number of clusters and cannot accept the reference number of components set to the planted pattern count for fair baseline comparison.

## Inputs

- Synthetic scRNA-seq benchmark instances with planted generative patterns (20 instances, multiple noise levels, from ref. 12)
- Planted pattern annotations (identity and activity states) for purity matching
- Preprocessed count matrices (filtered with min_cells_per_feature=0.01, min_features_per_cell=1000, library-size normalized, log1p transformed)

## Outputs

- pattern_recovery_purity.csv with purity values per method, instance, and noise level
- Aggregated purity comparison across 20 instances and 3 noise levels against Fig. 1b-d reference results

## How to apply

Load the synthetic scRNA-seq benchmark dataset (ref. 12 style: 20 generated instances with planted patterns evaluated at 3 noise levels) and preprocess each instance with ACTIONet's filter_adata (min_cells_per_feature=0.01, min_features_per_cell=1000), sc.pp.normalize_total, and log1p. Apply the method under test — e.g., ACTIONet's randomized SVD-based reduce_kernel followed by an.run_ACTIONet for the multiresolution ACTION archetypal decomposition — and run each consensus baseline (cNMF, cLDA, cICA, Louvain) on the same preprocessed instances with the reference number of components set to the planted pattern count. For each method, instance, and noise level, compute purity by matching recovered patterns to the planted generative patterns, then aggregate across the 20 instances and 3 noise levels and compare to the paper's reported Fig. 1b-d results. Write the aggregated table to pattern_recovery_purity.csv.

## Related tools

- **ACTIONet** (Method under test: multiresolution ACTION archetypal decomposition recovering identity and activity state patterns via reduce_kernel and an.run_ACTIONet) — https://github.com/shmohammadi86/ACTIONet
- **scanpy** (Preprocessing of each synthetic instance: filter_adata, normalize_total, log1p before decomposition)
- **Leiden** (Graph clustering used by ACTIONet to prune redundant archetypes into nonredundant equivalent pattern classes for purity scoring)
- **Python** (Environment for running the ACTIONet Python package and benchmark harness)

## Examples

```
an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000); sc.pp.normalize_total(adata); sc.pp.log1p(adata); an.pp.reduce_kernel(adata); an.run_ACTIONet(adata)
```

## Evaluation signals

- ACTIONet recovers all planted identity and activity patterns with high purity across all 3 noise levels, matching near-ceiling purity values reported in Fig. 1b-d
- Purity values are computed for every combination of method × instance (20 instances) × noise level (3 levels) and aggregated before comparison
- Baseline decompositions (cNMF, cLDA, cICA, Louvain) are run with the reference number of components equal to the planted pattern count, ensuring fair comparison
- Output table pattern_recovery_purity.csv contains purity for all method-instance-noise combinations and reproduces the reported superior performance of ACTIONet relative to consensus baselines

## Limitations

- The synthetic dataset is cited only by reference number (ref. 12) without accessions or download links in the article, complicating reproduction
- Software/tool versions are not stated for ACTIONet, R, Python, UMAP, or Leiden implementations (only CMake 3.19 is specified)
- Pattern counts differ between the task card (14 planted patterns) and the article's description of the synthetic data (13 activity patterns), so the planted-pattern count must be confirmed against ref. 12 before matching

## Evidence

- [discussion] Synthetic dataset provenance and structure: "Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns"
- [abstract] Claim of superior performance that the robustness benchmark supports: "Using multiple synthetic and real data sets, we demonstrate ACTIONet's superior performance relative to existing alternatives"
- [methods] Preprocessing thresholds applied to each instance before decomposition: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [discussion] Reduction step used in the decomposition under test: "an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size"
- [discussion] Archetype pruning step whose output feeds purity matching: "We then applied the Leiden graph-clustering algorithms over this pattern graph to define equivalent patterns classes"
