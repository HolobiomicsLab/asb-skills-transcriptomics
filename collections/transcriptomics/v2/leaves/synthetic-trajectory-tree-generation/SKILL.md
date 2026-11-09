---
name: synthetic-trajectory-tree-generation
description: Use when a branched-trajectory alignment method needs a controlled noise-robustness
  or accuracy benchmark and real paired scRNA-seq datasets are unavailable.
when_to_use_negative:
- The trajectories of interest contain complex graphs such as cycles or disconnected
  graphs — CAPITAL can deal only with trajectory trees computed from input datasets
- The two datasets have very different global topological structures — alignment accuracy
  of CAPITAL degraded in benchmarking for such data
- The goal is alignment of linear trajectories only, where DTW-based single-lineage
  methods already suffice and tree alignment adds unnecessary complexity
tools:
- name: CAPITAL
  role: Provides the synthetic dataset loaders (synthetic_dataset1/synthetic_dataset2)
    and the full alignment pipeline (preprocessing, trajectory_tree, dpt, tree_alignment,
    dtw) used to benchmark noise robustness
  repo: github.com/ykat0/capital
- name: conda
  role: Creates the isolated Python 3.8 environment with graphviz required to install
    and run CAPITAL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:ca7a63ea7ed1ee1bf2730e8f192594044894f97d7cd72692d789a09d8fa053a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAPITAL
  - Python
  - conda
  - graphviz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# synthetic-trajectory-tree-generation

## Summary

Generate synthetic single-cell trajectory pairs via CAPITAL's dataset loaders (capital.dataset.synthetic_dataset1/synthetic_dataset2) and perturb them with Gaussian noise to benchmark trajectory-tree alignment robustness using normalized alignment distance and ATC score.

## When to use

Use when a branched-trajectory alignment method needs a controlled noise-robustness or accuracy benchmark and real paired scRNA-seq datasets are unavailable.

## When NOT to use

- The trajectories of interest contain complex graphs such as cycles or disconnected graphs — CAPITAL can deal only with trajectory trees computed from input datasets
- The two datasets have very different global topological structures — alignment accuracy of CAPITAL degraded in benchmarking for such data
- The goal is alignment of linear trajectories only, where DTW-based single-lineage methods already suffice and tree alignment adds unnecessary complexity

## Inputs

- Synthetic single-cell trajectory pairs from capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2
- Gaussian noise standard deviations (0 through 5.0 and beyond), 100 replicates per level

## Outputs

- 95,200 perturbed trajectory pairs with inferred trajectory trees, pseudotime, and tree alignments
- Per-alignment normalized alignment distance and ATC score
- Aggregated noise-robustness distributions (noise_robustness_results.csv)

## How to apply

Set up a Python 3.8 conda environment with graphviz and install CAPITAL via 'pip install capital'. Load 6,800 synthetic trajectory pairs with capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2, then add independent Gaussian noise at standard deviations of 0 through 5.0 and beyond, repeating each level 100 times (95,200 perturbed pairs total) to probe the noise ceiling. For each perturbed pair, run capital.tl.preprocessing, infer the trajectory tree with capital.tl.trajectory_tree, compute pseudotime with capital.tl.dpt, and align the trees with capital.tl.tree_alignment and capital.tl.dtw. Record normalized alignment distance and ATC score per alignment, aggregate by noise level, and check for the reported deterioration at SD >= 3.0, where the cell space begins to be shattered.

## Related tools

- **CAPITAL** (Provides the synthetic dataset loaders (synthetic_dataset1/synthetic_dataset2) and the full alignment pipeline (preprocessing, trajectory_tree, dpt, tree_alignment, dtw) used to benchmark noise robustness) — github.com/ykat0/capital
- **conda** (Creates the isolated Python 3.8 environment with graphviz required to install and run CAPITAL)

## Examples

```
conda create -n capital python=3.8 graphviz && conda activate capital && pip install capital
```

## Evaluation signals

- Alignment quality is stable for Gaussian noise SD < 3.0 and deteriorates at SD >= 3.0, matching the published curves (Fig. 3a-b)
- 95,200 total perturbed pairs generated (6,800 pairs x ~15 noise levels x 100 repeats)
- Normalized alignment distance increases and ATC score decreases monotonically as noise SD grows past 3.0
- Environment resolves without dependency conflicts (Python 3.8, scanpy==1.9.1, matplotlib==3.5.2, networkx<=2.8.3 pinned per README)

## Limitations

- Noise robustness holds only up to Gaussian SD ~3.0; beyond that the cell space is shattered and both metrics deteriorate
- CAPITAL in its present form can deal only with trajectory trees (including single lineages), not cycles or disconnected graphs
- Part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method
- README installation section recommends Python 3.9 ('conda create -n capital python=3.9 graphviz') while the paper methods cite Python 3.8 — pin versions carefully for reproducibility

## Evidence

- [abstract] Benchmark target metric behavior: "Computational tests on synthetic datasets and authentic bone marrow cells datasets indicate that CAPITAL has achieved accurate and robust alignments of trajectory trees"
- [methods] Environment setup: "conda create -n capital python=3.8 graphviz"
- [readme] Purpose of synthetic benchmarks: "We present CAPITAL, a computational method for comparing pseudotime trajectories with tree alignment whereby trajectories including branches can be automatically compared."
- [readme] Reproducibility capsule: [evidence span withheld — anchor status: external_doc]
