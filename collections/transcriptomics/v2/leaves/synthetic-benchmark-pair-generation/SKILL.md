---
name: synthetic-benchmark-pair-generation
description: Use when you need reproducible paired trajectory-tree datasets with known
  topology to benchmark branched pseudotime alignment methods, or to stress-test alignment
  robustness against topological and clustering disruptions.
when_to_use_negative:
- The trajectories of interest are not tree-shaped (e.g., contain cycles or disconnected
  graphs) — CAPITAL can deal only with trajectory trees, not complex graphs.
- You need real biological datasets rather than simulator-derived pairs — use the
  packaged paul15, setty19, or velten19 datasets instead.
- You only need linear (unbranched) trajectory alignment — DTW-based methods on single
  lineages suffice and branching-aware tree alignment is unnecessary.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CAPITAL
  role: Provides the packaging format (AnnData datasets), trajectory-tree inference
    (tl.trajectory_tree), and the tree-alignment metric (tl.tree_alignment) that the
    synthetic pairs are built to benchmark
  repo: https://github.com/ykat0/capital
- name: conda
  role: Creates the Python 3.8 environment with graphviz required to install and run
    CAPITAL
- name: Python
  role: Runtime for dyngen simulation, MST-over-clusters computation, and CAPITAL
    benchmark scripts
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:a77eec003825098443c820b10f0be71ef570e47ecb387bde0f755c10ebaab518
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Python
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# synthetic-benchmark-pair-generation

## Summary

Generate synthetic scRNA-seq count datasets with dyngen on a binary-tree backbone (three branches) and filter to trajectory-tree pairs (four-leaf cluster-derived MSTs) for benchmarking CAPITAL's tree-alignment accuracy under controlled topological and clustering perturbations.

## When to use

Use when you need reproducible paired trajectory-tree datasets with known topology to benchmark branched pseudotime alignment methods, or to stress-test alignment robustness against topological and clustering disruptions.

## When NOT to use

- The trajectories of interest are not tree-shaped (e.g., contain cycles or disconnected graphs) — CAPITAL can deal only with trajectory trees, not complex graphs.
- You need real biological datasets rather than simulator-derived pairs — use the packaged paul15, setty19, or velten19 datasets instead.
- You only need linear (unbranched) trajectory alignment — DTW-based methods on single lineages suffice and branching-aware tree alignment is unnecessary.

## Inputs

- dyngen simulation configuration (binary tree backbone with three branches, 1000 cells, 36 TFs, 250 target genes, 250 housekeeping genes)
- Packaged synthetic benchmarks: capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2

## Outputs

- synthetic_datasets.h5ad: 68 filtered count matrices with pseudotime/tree annotations as AnnData objects
- synthetic_alignment_distances.csv: normalized tree-alignment distances per benchmarked pair, grouped by test condition (53,987 topological-similarity pairs, 6,800 clustering-similarity pairs, 40,800 clustering-resolution pairs)

## How to apply

Create a Python 3.8 conda environment with graphviz and install CAPITAL via 'pip install capital', plus dyngen 1.0.3 for single-cell simulation. Run 120 dyngen simulations, each with a binary tree backbone of three branches, 1000 simulated cells, 36 transcription factors, 250 target genes, and 250 housekeeping genes. For each simulation, cluster the simulated cells and compute the minimum spanning tree over the clusters, recording the tree topology. Filter to the 68 simulations whose cluster-derived MST has exactly four leaves, and export the retained count matrices plus pseudotime/tree annotations as a CAPITAL-compatible AnnData object (.h5ad) matching the packaged capital.dataset.synthetic_dataset1 and synthetic_dataset2 reference objects. Downstream, apply capital.tl.preprocessing, infer branched trajectories with capital.tl.trajectory_tree, and score each pair with capital.tl.tree_alignment, recording normalized alignment distances grouped by test condition.

## Related tools

- **CAPITAL** (Provides the packaging format (AnnData datasets), trajectory-tree inference (tl.trajectory_tree), and the tree-alignment metric (tl.tree_alignment) that the synthetic pairs are built to benchmark) — https://github.com/ykat0/capital
- **conda** (Creates the Python 3.8 environment with graphviz required to install and run CAPITAL)
- **Python** (Runtime for dyngen simulation, MST-over-clusters computation, and CAPITAL benchmark scripts)

## Examples

```
conda create -n capital python=3.8 graphviz && conda activate capital && pip install capital
```

## Evaluation signals

- Exactly 68 of 120 dyngen simulations are retained after filtering to four-leaf cluster-derived MSTs.
- Each retained dataset's cluster-derived minimum spanning tree has exactly four leaves.
- The exported AnnData objects are loadable with capital.dataset.synthetic_dataset1/synthetic_dataset2 and align with the paper's reference objects.
- The distance table contains one normalized alignment distance per benchmarked pair across the three test conditions.
- Alignment distances remain robust within matching topological structure but degrade when the root with two or three children is removed (global topological similarity destroyed).

## Limitations

- Alignment accuracy of CAPITAL for datasets with different global topological structure degraded in the benchmarking test, so synthetic pairs must match global topology to demonstrate robust performance.
- Part of the CAPITAL algorithm — computing a trajectory as a minimum spanning tree — may not be the best method, so benchmark conclusions are conditional on the MST-based trajectory representation.
- Benchmark results depend on simulator settings (dyngen 1.0.3, binary tree backbone) and may not transfer to other backbones or cell/genre configurations.

## Evidence

- [other] The benchmarking synthetic datasets were generated with dyngen using a binary tree backbone with three branches, yielding 68 single-cell expression count datasets: [evidence span withheld — anchor status: not_found]
- [discussion] alignment accuracy degradation finding: "alignment accuracy of CAPITAL for datasets with different global topological structure degraded in our benchmarking test"
- [discussion] MST trajectory representation caveat: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
- [readme] environment setup: "We present CAPITAL, a computational method for comparing pseudotime trajectories with tree alignment whereby trajectories including branches can be automatically compared."
- [methods] environment setup command: "conda create -n capital python=3.8 graphviz"
