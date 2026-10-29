---
name: single-cell-scrnaseq-synthetic-benchmarking
description: Use when evaluating whether a cell-state decomposition or clustering
  method recovers known planted identity and activity patterns from synthetic scRNA-seq
  data with planted ground truth.
when_to_use_negative:
- Input is real scRNA-seq data with no planted ground-truth patterns — purity against
  planted patterns cannot be computed
- The goal is a single-analysis cell annotation or integration run rather than method
  benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ACTIONet
  role: 'Candidate method under benchmark: randomized SVD-based reduce_kernel reduction
    plus an.run_ACTIONet multiresolution archetypal decomposition to recover identity
    and activity state patterns'
  repo: https://github.com/shmohammadi86/ACTIONet
- name: scanpy
  role: 'Preprocessing of each synthetic instance: read 10x h5 input, normalize_total,
    log1p'
- name: Leiden
  role: Graph-clustering over the pattern graph to define equivalent pattern classes
    and prune redundant archetypes before purity scoring
- name: R
  role: Environment for the stable ACTIONet R package used to run the decomposition
  repo: https://github.com/shmohammadi86/ACTIONet/tree/R-release
- name: Python
  role: Alternative environment for the ACTIONet Python package and scanpy preprocessing
  repo: https://github.com/shmohammadi86/ACTIONet@python-devel
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:60f3fcb7cd22da9c564abf8cfed3cf0d6995782705f58d56e7cbfc059d8c6ce3
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-scrnaseq-synthetic-benchmarking

## Summary

Benchmark pattern-recovery fidelity of single-cell decomposition methods on synthetic scRNA-seq data by matching recovered identity and activity patterns to planted generative patterns, as done for ACTIONet against cNMF, cLDA, cICA, and Louvain baselines.

## When to use

Use when evaluating whether a cell-state decomposition or clustering method recovers known planted identity and activity patterns from synthetic scRNA-seq data with planted ground truth.

## When NOT to use

- Input is real scRNA-seq data with no planted ground-truth patterns — purity against planted patterns cannot be computed
- The goal is a single-analysis cell annotation or integration run rather than method benchmarking

## Inputs

- Synthetic scRNA-seq benchmark dataset from ref. 12 (20 instances, 14 planted patterns, 3 noise levels)
- Preprocessed cell-by-gene count matrices (filter_adata min_cells_per_feature=0.01, min_features_per_cell=1000; normalize_total; log1p)
- Planted generative pattern definitions (ground-truth identity and activity states)
- Baseline decomposition methods: consensus NMF, consensus LDA, consensus ICA, Louvain clustering

## Outputs

- pattern_recovery_purity.csv: purity values per method, instance, and noise level
- Aggregated purity comparison against reported Fig. 1b-d results

## How to apply

Load the synthetic scRNA-seq benchmark dataset from ref. 12, which contains 20 generated instances with 14 planted identity/activity patterns evaluated at 3 noise levels. Preprocess each instance following the ACTIONet pipeline: filter_adata with min_cells_per_feature=0.01 and min_features_per_cell=1000, sc.pp.normalize_total library-size normalization, and log1p transformation. Run the candidate method (e.g., ACTIONet's reduce_kernel followed by an.run_ACTIONet multiresolution archetypal decomposition) and the four consensus baseline decompositions on the same preprocessed instances, setting the reference number of components to the planted pattern count. For each method, instance, and noise level, compute pattern-recovery purity by matching recovered patterns to the planted generative patterns, then aggregate across the 20 instances and 3 noise levels and compare against the paper's Fig. 1b-d near-ceiling purity values.

## Related tools

- **ACTIONet** (Candidate method under benchmark: randomized SVD-based reduce_kernel reduction plus an.run_ACTIONet multiresolution archetypal decomposition to recover identity and activity state patterns) — https://github.com/shmohammadi86/ACTIONet
- **scanpy** (Preprocessing of each synthetic instance: read 10x h5 input, normalize_total, log1p)
- **Leiden** (Graph-clustering over the pattern graph to define equivalent pattern classes and prune redundant archetypes before purity scoring)
- **R** (Environment for the stable ACTIONet R package used to run the decomposition) — https://github.com/shmohammadi86/ACTIONet/tree/R-release
- **Python** (Alternative environment for the ACTIONet Python package and scanpy preprocessing) — https://github.com/shmohammadi86/ACTIONet@python-devel

## Examples

```
an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000); sc.pp.normalize_total(adata); sc.pp.log1p(adata); an.pp.reduce_kernel(adata); an.run_ACTIONet(adata)
```

## Evaluation signals

- ACTIONet recovers all 14 planted identity and activity patterns with high purity across all 3 noise levels
- Purity values approach ceiling and match the near-ceiling values reported in Fig. 1b-d
- ACTIONet purity is superior to consensus NMF, cLDA, cICA, and Louvain baselines across the 20 instances
- pattern_recovery_purity.csv contains one purity entry per method-instance-noise-level combination with no missing values
- Redundant archetypes are consolidated: Leiden clustering over the pattern graph yields one lowest-resolution pattern per equivalent class

## Limitations

- Datasets are cited only by reference numbers without accessions or download links, so obtaining the synthetic benchmark requires tracing ref. 12
- No software/tool versions stated for ACTIONet, R, Python, UMAP, or Leiden implementations (CMake 3.19 is the only version given)
- Number of planted patterns is reported inconsistently in the discussion (13 activity patterns) versus the workflow (14 planted patterns)
- Matrix decomposition baselines require a priori selection of the number of underlying dominant patterns, which is itself nontrivial to determine

## Evidence

- [discussion] Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns: "Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns"
- [methods] filter_adata preprocessing parameters: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [discussion] We then applied the Leiden graph-clustering algorithms over this pattern graph to define equivalent patterns classes: "We then applied the Leiden graph-clustering algorithms over this pattern graph to define equivalent patterns classes (patterns clusters). Finally, for each equivalent class, we keep the pattern"
- [abstract] Using multiple synthetic and real data sets, we demonstrate ACTIONet's superior performance relative to existing alternatives: "Using multiple synthetic and real data sets, we demonstrate ACTIONet's superior performance relative to existing alternatives"
- [discussion] an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size: "an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size"
