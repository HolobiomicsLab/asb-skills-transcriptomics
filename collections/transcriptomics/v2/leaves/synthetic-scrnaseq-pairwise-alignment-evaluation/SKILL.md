---
name: synthetic-scrnaseq-pairwise-alignment-evaluation
description: Use when a tree/trajectory alignment method must be quantitatively compared
  to data-integration baselines over many synthetic dataset pairs, using ATC scores
  and Wilcoxon signed-rank p-values as the benchmark metric.
when_to_use_negative:
- The input datasets have different global topological structure — the article reports
  CAPITAL's alignment accuracy degraded in benchmarking for such cases
- The trajectories of interest are complex graphs (cycles or disconnected graphs)
  rather than trees — CAPITAL in its present form can deal only with trajectory trees
- The goal is linear alignment of single lineages only; existing tools already focus
  on that and pre-selecting linear paths would be required instead of tree alignment
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CAPITAL
  role: 'Method under test: infers trajectory trees via minimum spanning tree, performs
    tree_alignment and dtw, and computes ATC/genes_similarity_score alignment accuracy'
  repo: https://github.com/ykat0/capital
- name: Scanorama
  role: Data-integration baseline used to derive cluster correspondences for the benchmark
    pairs
- name: scVI
  role: Data-integration baseline used to derive cluster correspondences for the benchmark
    pairs
- name: Seurat
  role: Data-integration baseline used to derive cluster correspondences for the benchmark
    pairs
- name: Palantir
  role: Referenced trajectory-inference dependency; capital.tl.dpt is used for pseudotime
    within the CAPITAL workflow
  repo: https://github.com/dpeerlab/Palantir
- name: conda
  role: Environment management for reproducible CAPITAL setup (python=3.8, graphviz)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:781bb66e3c233aabec488490e0ae4c59a1a327c2dd85b11f962b891b676f7eb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Palantir
  - Python
  - Scanorama
  - Seurat
  - conda
  - scVI
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# synthetic-scrnaseq-pairwise-alignment-evaluation

## Summary

Benchmark a trajectory-alignment method (CAPITAL) against data-integration baselines (Scanorama, scVI, Seurat) by aligning all pairs of synthetic scRNA-seq datasets, scoring biological variance conservation with ATC alignment-accuracy scores, and testing score differences with one-sided Wilcoxon signed-rank tests.

## When to use

Use when a tree/trajectory alignment method must be quantitatively compared to data-integration baselines over many synthetic dataset pairs, using ATC scores and Wilcoxon signed-rank p-values as the benchmark metric.

## When NOT to use

- The input datasets have different global topological structure — the article reports CAPITAL's alignment accuracy degraded in benchmarking for such cases
- The trajectories of interest are complex graphs (cycles or disconnected graphs) rather than trees — CAPITAL in its present form can deal only with trajectory trees
- The goal is linear alignment of single lineages only; existing tools already focus on that and pre-selecting linear paths would be required instead of tree alignment

## Inputs

- 68 synthetic scRNA-seq datasets loaded via capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2
- All 2,278 pairwise combinations of the synthetic datasets
- Integration baseline outputs (cluster correspondences) from Scanorama, scVI, and Seurat

## Outputs

- ATC alignment-accuracy score distributions for CAPITAL vs. Scanorama, scVI, and Seurat across 2,278 synthetic dataset pairs
- One-sided Wilcoxon signed-rank p-values for CAPITAL vs. each baseline (reported as < 1.00 × 10−226)
- Benchmark comparison figure (e.g., fig4a_atc_score_benchmark.png)

## How to apply

Create a CAPITAL environment (conda create -n capital python=3.8 graphviz; pip install capital) and load the 68 synthetic datasets via capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2. Enumerate all 2,278 dataset pairs and, for each pair, derive cluster correspondences using Scanorama, scVI, and Seurat as integration baselines. Infer the common minimum-spanning-tree pseudotime trajectory per pair with capital.tl.trajectory_tree (using capital.tl.dpt for pseudotime), compute tree alignments with capital.tl.tree_alignment and capital.tl.dtw, and score them with capital.tl.genes_similarity_score-style ATC alignment-accuracy scoring. Compare CAPITAL's score distribution against each baseline across the 2,278 pairs with one-sided Wilcoxon signed-rank tests and render the comparison as the ATC-score benchmark figure (Fig. 4a).

## Related tools

- **CAPITAL** (Method under test: infers trajectory trees via minimum spanning tree, performs tree_alignment and dtw, and computes ATC/genes_similarity_score alignment accuracy) — https://github.com/ykat0/capital
- **Scanorama** (Data-integration baseline used to derive cluster correspondences for the benchmark pairs)
- **scVI** (Data-integration baseline used to derive cluster correspondences for the benchmark pairs)
- **Seurat** (Data-integration baseline used to derive cluster correspondences for the benchmark pairs)
- **Palantir** (Referenced trajectory-inference dependency; capital.tl.dpt is used for pseudotime within the CAPITAL workflow) — https://github.com/dpeerlab/Palantir
- **conda** (Environment management for reproducible CAPITAL setup (python=3.8, graphviz))

## Examples

```
conda create -n capital python=3.8 graphviz && conda activate capital && pip install capital
```

## Evaluation signals

- CAPITAL's ATC-score distribution is statistically significantly higher than each integration baseline, with one-sided Wilcoxon signed-rank p < 1.00 × 10−226 on 2,278 pairs
- All 68 synthetic datasets load successfully and the enumeration yields exactly 2,278 unique dataset pairs
- Each pair produces a valid trajectory tree from capital.tl.trajectory_tree and a completed tree_alignment/dtw computation without errors
- Integrated clusters from baselines should be inspected for imbalanced cell counts, since anchor-based transformation can wrongly match subpopulations
- The rendered benchmark figure reproduces the qualitative ordering of Fig. 4a (CAPITAL above Scanorama, scVI, and Seurat)

## Limitations

- Part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method
- Alignment accuracy of CAPITAL degraded for datasets with different global topological structure in benchmarking
- CAPITAL handles only trajectory trees, not complex graphs such as cycles or disconnected graphs; graphs beyond trees plus graph alignment would require a predictive model
- Denoising/imputing expression counts can generate more predicted clusters than previously reported, and data integration can produce integrated clusters with imbalanced cell numbers

## Evidence

- [results] ATC benchmark result: [evidence span withheld — anchor status: not_found]
- [results] Wilcoxon significance: [evidence span withheld — anchor status: not_found]
- [methods] Installation: "conda create -n capital python=3.8 graphviz"
- [discussion] MST limitation: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
- [discussion] Tree-only scope: "CAPITAL in its present form can deal only with trajectory trees (including single lineages) that are computed from input datasets"
- [readme] PyPI/version info: [evidence span withheld — anchor status: external_doc]
- [readme] Code Ocean capsule: [evidence span withheld — anchor status: not_found]
