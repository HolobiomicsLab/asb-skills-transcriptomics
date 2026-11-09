---
name: pattern-recovery-purity-evaluation
description: Use when evaluating a decomposition or clustering method (e.g., ACTIONet,
  cNMF, cLDA, cICA, Louvain) on synthetic scRNA-seq benchmarks with planted patterns,
  to measure pattern-recovery purity against generative ground truth.
when_to_use_negative:
- Input is real experimental scRNA-seq data without planted ground-truth patterns
  — purity against generative truth is undefined; use annotation or marker-based validation
  instead.
- The goal is novel pattern discovery rather than method benchmarking — the multiresolution
  decomposition itself (reduce_kernel + run_ACTIONet) is the appropriate skill, not
  the recovery-purity evaluation.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: ACTIONet
  role: Method under test; performs randomized SVD reduction (reduce_kernel) and multiresolution
    ACTION archetypal decomposition (run_ACTIONet) to recover identity and activity
    patterns
  repo: https://github.com/shmohammadi86/ACTIONet
- name: scanpy
  role: 'Preprocessing of each synthetic instance: filter_adata-style filtering, normalize_total
    library-size normalization, and log1p transformation'
- name: Leiden
  role: Graph clustering over the pattern graph used within ACTIONet to prune redundant
    archetypes into equivalent pattern classes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:e01db95fb84a8f804fc56ffe9773272ef63bd1cd033fb392c2e1ae8e414af6a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - Leiden
  - R
  - scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# pattern-recovery-purity-evaluation

## Summary

Benchmark protocol for quantifying how well a single-cell decomposition or clustering method recovers planted identity and activity patterns from synthetic scRNA-seq data, by matching recovered patterns to generative ground truth and computing pattern-recovery purity across instances and noise levels.

## When to use

Use when evaluating a decomposition or clustering method (e.g., ACTIONet, cNMF, cLDA, cICA, Louvain) on synthetic scRNA-seq benchmarks with planted patterns, to measure pattern-recovery purity against generative ground truth.

## When NOT to use

- Input is real experimental scRNA-seq data without planted ground-truth patterns — purity against generative truth is undefined; use annotation or marker-based validation instead.
- The goal is novel pattern discovery rather than method benchmarking — the multiresolution decomposition itself (reduce_kernel + run_ACTIONet) is the appropriate skill, not the recovery-purity evaluation.

## Inputs

- Synthetic scRNA-seq benchmark dataset from ref. 12 (20 instances, 14 planted identity/activity patterns, 3 noise levels)
- Preprocessed count matrices (filtered, normalize_total-scaled, log1p-transformed) per instance
- Planted generative pattern matrices serving as ground truth

## Outputs

- Per-method, per-instance, per-noise-level pattern-recovery purity values
- Aggregated purity comparison table against Fig. 1b-d (pattern_recovery_purity.csv)

## How to apply

Load the synthetic benchmark from ref. 12, comprising 20 generated instances with planted identity and activity patterns evaluated at 3 noise levels. Preprocess each instance using ACTIONet's filter_adata (min_cells_per_feature=0.01, min_features_per_cell=1000), sc.pp.normalize_total library-size normalization, and log1p transformation. Apply an.pp.reduce_kernel (randomized SVD-based low-rank approximation) followed by an.run_ACTIONet to recover multiresolution archetypal patterns, and run the consensus baselines (cNMF, cLDA, cICA, Louvain) on the same preprocessed instances with the component count set to the planted pattern count. For each method, instance, and noise level, match recovered patterns to the planted generative patterns and compute recovery purity; aggregate across instances and noise levels and compare with the reported Fig. 1b-d results, writing the table to pattern_recovery_purity.csv.

## Related tools

- **ACTIONet** (Method under test; performs randomized SVD reduction (reduce_kernel) and multiresolution ACTION archetypal decomposition (run_ACTIONet) to recover identity and activity patterns) — https://github.com/shmohammadi86/ACTIONet
- **scanpy** (Preprocessing of each synthetic instance: filter_adata-style filtering, normalize_total library-size normalization, and log1p transformation)
- **Leiden** (Graph clustering over the pattern graph used within ACTIONet to prune redundant archetypes into equivalent pattern classes)

## Examples

```
an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000); sc.pp.normalize_total(adata); sc.pp.log1p(adata); an.pp.reduce_kernel(adata); an.run_ACTIONet(adata)
```

## Evaluation signals

- ACTIONet recovers all planted identity and activity patterns with near-ceiling purity across all 3 noise levels, matching Fig. 1b-d
- ACTIONet's purity exceeds that of consensus cNMF, cLDA, cICA, and Louvain baselines run with components set to the planted pattern count
- Recovered patterns match planted generative patterns under the pattern-matching step for every one of the 20 instances
- Output table pattern_recovery_purity.csv contains purity values indexed by method, instance, and noise level with no missing combinations

## Limitations

- The synthetic benchmark is drawn from ref. 12 and contains 20 instances with 13-14 activity patterns; results may not generalize to datasets with different planted structure or noise regimes.
- Datasets are cited only by reference numbers without accessions or download links, complicating reproduction.
- No software/tool versions are stated for ACTIONet, R, Python, UMAP, or Leiden implementations, so exact reproducibility of purity values is not guaranteed.

## Evidence

- [discussion] Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns: "Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns"
- [methods] filter_adata parameters: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [discussion] randomized SVD reduction step: "an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size"
- [abstract] superior performance claim: "Using multiple synthetic and real data sets, we demonstrate ACTIONet's superior performance relative to existing alternatives"
- [discussion] Leiden pruning of redundant archetypes: "We then applied the Leiden graph-clustering algorithms over this pattern graph to define equivalent patterns classes"
