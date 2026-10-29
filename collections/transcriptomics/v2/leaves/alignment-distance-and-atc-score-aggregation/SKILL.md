---
name: alignment-distance-and-atc-score-aggregation
description: Use when you need to characterize how a trajectory-tree alignment method's
  accuracy metrics (e.g., normalized alignment distance, ATC score) degrade as a function
  of controlled input perturbation such as Gaussian noise.
when_to_use_negative:
- Input datasets have different global topological structure — the paper notes alignment
  accuracy of CAPITAL degraded in such benchmarking cases
- The trajectories form complex graphs such as cycles or disconnected graphs — CAPITAL
  in its present form deals only with trajectory trees computed from input datasets
- The goal is biological comparison of authentic datasets rather than controlled robustness/benchmarking
  of the alignment method itself
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: CAPITAL
  role: Provides the trajectory tree inference (tl.trajectory_tree), pseudotime (tl.dpt),
    tree alignment (tl.tree_alignment), and dynamic time warping (tl.dtw) functions
    whose noise robustness is measured; installed via pip in a Python 3.8 conda environment
    with graphviz
  repo: https://github.com/ykat0/capital
- name: Python
  role: Implementation and scripting language for running the CAPITAL pipeline and
    aggregating benchmark scores
  repo: https://github.com/ykat0/capital
- name: conda
  role: Environment manager used to create the CAPITAL environment (conda create -n
    capital python=3.8 graphviz)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:e5da9cbcbb32390ee5fcac5c01a67fa614139f64a9bfdadadcd1521a0cd61913
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Python
  - conda
  - graphviz
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# alignment-distance-and-atc-score-aggregation

## Summary

Benchmarking skill from the CAPITAL paper: quantify noise robustness of single-cell trajectory tree alignment by adding Gaussian noise to synthetic trajectory pairs, running the CAPITAL pipeline (preprocessing, trajectory_tree, dpt, tree_alignment, dtw) on each perturbed pair, and aggregating normalized alignment distance and ATC scores by noise level.

## When to use

Use when you need to characterize how a trajectory-tree alignment method's accuracy metrics (e.g., normalized alignment distance, ATC score) degrade as a function of controlled input perturbation such as Gaussian noise.

## When NOT to use

- Input datasets have different global topological structure — the paper notes alignment accuracy of CAPITAL degraded in such benchmarking cases
- The trajectories form complex graphs such as cycles or disconnected graphs — CAPITAL in its present form deals only with trajectory trees computed from input datasets
- The goal is biological comparison of authentic datasets rather than controlled robustness/benchmarking of the alignment method itself

## Inputs

- Synthetic single-cell trajectory pairs from capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2 (6,800 pairs)
- Grid of Gaussian noise standard deviations (e.g., 0 to 5.0+), with replicate count per level (100 in the paper)

## Outputs

- Per-alignment normalized alignment distance values (95,200 in the paper)
- Per-alignment ATC (alignment tree comparison) scores
- Aggregated score distributions by noise level, e.g., noise_robustness_results.csv (reproducing Fig. 3a-b style robustness curves)

## How to apply

Generate synthetic trajectory pairs with the capital.dataset.synthetic_dataset1 and synthetic_dataset2 loaders (6,800 pairs in the paper's setup). For each pair, add independent Gaussian noise at a grid of standard deviations (0 through 5.0 and beyond), repeating each level (100 times per level in the paper, 95,200 perturbed pairs total). For each perturbed pair run capital.tl.preprocessing, infer the trajectory tree with capital.tl.trajectory_tree, compute pseudotime with capital.tl.dpt, then align the two trees with capital.tl.tree_alignment and capital.tl.dtw. Record the normalized alignment distance and the ATC score per alignment, then aggregate scores by noise level (e.g., to noise_robustness_results.csv) and inspect the aggregated distributions for the noise level at which quality breaks down — in the paper both metrics deteriorated at standard deviation >= 3.0, when the cell space began to be shattered.

## Related tools

- **CAPITAL** (Provides the trajectory tree inference (tl.trajectory_tree), pseudotime (tl.dpt), tree alignment (tl.tree_alignment), and dynamic time warping (tl.dtw) functions whose noise robustness is measured; installed via pip in a Python 3.8 conda environment with graphviz) — https://github.com/ykat0/capital
- **Python** (Implementation and scripting language for running the CAPITAL pipeline and aggregating benchmark scores) — https://github.com/ykat0/capital
- **conda** (Environment manager used to create the CAPITAL environment (conda create -n capital python=3.8 graphviz))

## Examples

```
conda create -n capital python=3.8 graphviz && conda activate capital && pip install capital
```

## Evaluation signals

- Both normalized alignment distance and ATC score remain stable for noise SD < 3.0 and deteriorate at SD >= 3.0, matching the paper's reported breakdown point where the cell space begins to be shattered
- Total number of perturbed alignments matches the design (e.g., 6,800 pairs x noise levels x 100 replicates = 95,200 alignments)
- Aggregated output (noise_robustness_results.csv) contains one score distribution per noise level with no missing alignments
- Each individual alignment runs the full CAPITAL chain: tl.preprocessing -> tl.trajectory_tree -> tl.dpt -> tl.tree_alignment -> tl.dtw
- Results are consistent with the paper's conclusion that CAPITAL achieved 'accurate and robust alignments of trajectory trees' on synthetic datasets

## Limitations

- The benchmark does not test datasets with different global topological structure, where CAPITAL's alignment accuracy degraded
- CAPITAL can only handle trajectory trees, not complex graphs such as cycles or disconnected graphs; a predictive model handling graphs beyond trees plus graph alignment would be required
- Computing a trajectory as a minimum spanning tree may not be the best method, which bounds what the benchmark can say about topology recovery

## Evidence

- [other] Noise robustness breakdown at SD >= 3.0: [evidence span withheld — anchor status: not_found]
- [abstract] Synthetic-data robustness claim from abstract: "Computational tests on synthetic datasets and authentic bone marrow cells datasets indicate that CAPITAL has achieved accurate and robust alignments of trajectory trees"
- [methods] Conda environment setup from methods: "conda create -n capital python=3.8 graphviz"
- [discussion] Scope limitation on graphs from discussion: "CAPITAL in its present form can deal only with trajectory trees (including single lineages) that are computed from input datasets"
- [readme] Installation instructions from README: [evidence span withheld — anchor status: external_doc]
