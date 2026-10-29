---
name: trajectory-tree-topology-filtering
description: Use when building benchmark synthetic scRNA-seq data for branched trajectory
  alignment and only simulations whose cluster-derived MST has a specified topology
  (e.g., exactly four leaves) should enter the benchmark.
when_to_use_negative:
- When aligning already-generated real scRNA-seq trajectory trees (e.g., bone marrow,
  human vs. mouse) — use CAPITAL's tl.trajectory_tree and tl.tree_alignment directly
  instead of simulating and filtering.
- When the trajectory structures of interest are cycles or disconnected graphs — CAPITAL
  'can deal only with trajectory trees (including single lineages)' and not complex
  graphs.
- When the target analysis requires topologies other than four-leaf trees — the filter
  discards all other leaf counts and alignment accuracy degrades for datasets with
  different global topological structure.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: dyngen
  role: Simulates single-cell expression count datasets from a binary tree backbone
    with three branches; version 1.0.3 used to generate the 120 candidate simulations
- name: CAPITAL
  role: Consumes the filtered AnnData synthetic datasets (capital.dataset.synthetic_dataset1/2)
    for benchmarking trajectory tree alignment
  repo: https://github.com/ykat0/capital
- name: conda
  role: Creates the Python 3.8 + graphviz environment used to run dyngen simulation
    and CAPITAL dataset export
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:b21a106c7ed4407c7e5e012a3de9d61992234ece041c3769337c8ea87250eb92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Python
  - conda
  - dyngen
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# trajectory-tree-topology-filtering

## Summary

Generates and then filters synthetic scRNA-seq datasets by trajectory-tree topology: run dyngen simulations on a binary-tree backbone, compute the minimum spanning tree (MST) over cell clusters for each simulation, and retain only those whose cluster-derived MST has the required leaf count (four leaves, yielding 68 of 120 datasets for CAPITAL benchmarking).

## When to use

Use when building benchmark synthetic scRNA-seq data for branched trajectory alignment and only simulations whose cluster-derived MST has a specified topology (e.g., exactly four leaves) should enter the benchmark.

## When NOT to use

- When aligning already-generated real scRNA-seq trajectory trees (e.g., bone marrow, human vs. mouse) — use CAPITAL's tl.trajectory_tree and tl.tree_alignment directly instead of simulating and filtering.
- When the trajectory structures of interest are cycles or disconnected graphs — CAPITAL 'can deal only with trajectory trees (including single lineages)' and not complex graphs.
- When the target analysis requires topologies other than four-leaf trees — the filter discards all other leaf counts and alignment accuracy degrades for datasets with different global topological structure.

## Inputs

- dyngen 1.0.3 simulation configuration (binary tree backbone with three branches, 1000 cells, 36 transcription factors, 250 target genes, 250 housekeeping genes)
- Simulated single-cell expression count matrices from 120 dyngen runs
- Cluster assignments of simulated cells (for per-simulation cluster-derived MSTs)
- Cluster-derived minimum spanning trees with recorded leaf counts

## Outputs

- 68 filtered synthetic scRNA-seq count datasets whose cluster-derived MST has exactly four leaves
- CAPITAL-compatible AnnData object (synthetic_datasets.h5ad) containing count matrices plus pseudotime and tree annotations
- Reference dataset objects matching capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2

## How to apply

Set up a conda environment with Python 3.8 and graphviz, install CAPITAL (pip install capital) plus dyngen 1.0.3, and run 120 dyngen simulations, each with a binary tree backbone of three branches, 1000 simulated cells, 36 transcription factors, 250 target genes, and 250 housekeeping genes. For each simulation, cluster the simulated cells and compute the minimum spanning tree over the clusters, recording the resulting tree topology and its leaf count. Apply the hard filter: keep only simulations whose cluster-derived MST has exactly four leaves — this retained 68 of 120 datasets in the CAPITAL benchmark. Export the retained count matrices together with pseudotime and tree annotations to a CAPITAL-compatible AnnData object (synthetic_datasets.h5ad). The rationale is that CAPITAL aligns trajectory trees inferred as MSTs over clusters, so benchmark datasets must share a comparable global topology; the authors themselves note that 'part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method', making topological homogenization of test data essential for fair benchmarking.

## Related tools

- **dyngen** (Simulates single-cell expression count datasets from a binary tree backbone with three branches; version 1.0.3 used to generate the 120 candidate simulations)
- **CAPITAL** (Consumes the filtered AnnData synthetic datasets (capital.dataset.synthetic_dataset1/2) for benchmarking trajectory tree alignment) — https://github.com/ykat0/capital
- **conda** (Creates the Python 3.8 + graphviz environment used to run dyngen simulation and CAPITAL dataset export)

## Examples

```
conda create -n capital python=3.8 graphviz; conda activate capital; pip install capital
```

## Evaluation signals

- Exactly 68 datasets are retained out of 120 dyngen simulations (retention check, not a count approximation).
- Every retained dataset's cluster-derived MST has exactly four leaves; any simulation with a different leaf count must have been discarded.
- Each simulation contains 1000 simulated cells with 36 transcription factors, 250 target genes, and 250 housekeeping genes on a binary tree backbone of three branches.
- The exported .h5ad loads as a CAPITAL-compatible AnnData object and matches the packaged capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2 reference objects.

## Limitations

- The four-leaf filter is topology-specific: datasets with other global topological structures are discarded, and CAPITAL's alignment accuracy degrades for datasets with different global topological structure.
- The MST-over-clusters route to trajectory trees is acknowledged by the authors as possibly suboptimal: 'part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method'.
- Only tree-shaped trajectories are supported — cycles or disconnected graphs cannot be represented, so this skill's filtered data cannot benchmark graph-alignment extensions.

## Evidence

- [methods] dyngen simulation parameters: [evidence span withheld — anchor status: not_found]
- [methods] four-leaf filter: [evidence span withheld — anchor status: not_found]
- [discussion] MST limitation acknowledged in discussion: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
- [readme] Scanpy-based implementation: [evidence span withheld — anchor status: external_doc]
