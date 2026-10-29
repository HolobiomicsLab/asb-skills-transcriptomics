---
name: transcription-factor-regulatory-network-simulation
description: Use when you need synthetic single-cell expression count matrices with
  known branched trajectory trees — e.g. to benchmark trajectory or tree-alignment
  methods where ground-truth topology and pseudotime are required.
when_to_use_negative:
- You need real experimental scRNA-seq data rather than simulations with known ground
  truth — use the packaged paul15, setty19, or velten17 datasets instead.
- Your benchmark requires trajectory graphs with cycles or disconnected components
  — dyngen binary-tree backbones yield trees only, and CAPITAL in its present form
  deals only with trajectory trees.
- Your target trajectory topology differs from a four-leaf tree — the four-leaf MST
  filter would discard your simulations and bias the benchmark.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CAPITAL
  role: Target method being benchmarked; installed via pip and provides the AnnData
    dataset loaders (capital.dataset.synthetic_dataset1/2) that consume the simulated
    datasets
  repo: https://github.com/ykat0/capital
- name: Python 3.8
  role: Runtime environment for CAPITAL and the dyngen simulation/filtering workflow
  repo: https://github.com/ykat0/capital
- name: conda
  role: Environment manager used to create the Python 3.8 + graphviz environment for
    CAPITAL and dyngen
- name: dyngen 1.0.3
  role: Simulator that generates transcription-factor regulatory-network-driven single-cell
    count data on the binary-tree backbone
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:43c55bc3d0803160a1caafad9d34e62aef4532e40d2c5daefa9ac3050d217755
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Python
  - Python 3.8
  - conda
  - dyngen 1.0.3
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# transcription-factor-regulatory-network-simulation

## Summary

Generate synthetic scRNA-seq count datasets with dyngen by simulating transcription-factor-driven regulatory networks on a binary-tree backbone, then filter simulations to those whose cluster-derived minimum spanning tree has exactly four leaves (68 of 120 retained) for benchmarking CAPITAL's tree alignment.

## When to use

Use when you need synthetic single-cell expression count matrices with known branched trajectory trees — e.g. to benchmark trajectory or tree-alignment methods where ground-truth topology and pseudotime are required.

## When NOT to use

- You need real experimental scRNA-seq data rather than simulations with known ground truth — use the packaged paul15, setty19, or velten17 datasets instead.
- Your benchmark requires trajectory graphs with cycles or disconnected components — dyngen binary-tree backbones yield trees only, and CAPITAL in its present form deals only with trajectory trees.
- Your target trajectory topology differs from a four-leaf tree — the four-leaf MST filter would discard your simulations and bias the benchmark.

## Inputs

- dyngen simulation configuration: binary tree backbone with three branches, 1000 cells, 36 transcription factors, 250 target genes, 250 housekeeping genes (120 replicates)
- conda environment with Python 3.8, graphviz, CAPITAL, and dyngen 1.0.3

## Outputs

- 68 filtered single-cell expression count datasets, each with a four-leaf cluster-derived MST
- AnnData .h5ad object (synthetic_datasets.h5ad) with count matrices plus pseudotime/tree annotations, CAPITAL-compatible
- Reference objects equivalent to capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2

## How to apply

Create a conda environment with Python 3.8 and graphviz (`conda create -n capital python=3.8 graphviz`), install CAPITAL via `pip install capital`, and use dyngen 1.0.3 for simulation. Run 120 dyngen simulations, each configured with a binary-tree backbone of three branches, 1000 simulated cells, 36 transcription factors, 250 target genes, and 250 housekeeping genes. For each simulation, cluster the simulated cells and compute a minimum spanning tree over the clusters, recording the tree topology. Filter the 120 simulations to the 68 whose cluster-derived MST has exactly four leaves, discarding any other leaf count. Export the 68 retained count matrices with pseudotime and tree annotations to a CAPITAL-compatible AnnData object (e.g. synthetic_datasets.h5ad), matching the packaged `capital.dataset.synthetic_dataset1` and `capital.dataset.synthetic_dataset2` references.

## Related tools

- **CAPITAL** (Target method being benchmarked; installed via pip and provides the AnnData dataset loaders (capital.dataset.synthetic_dataset1/2) that consume the simulated datasets) — https://github.com/ykat0/capital
- **Python 3.8** (Runtime environment for CAPITAL and the dyngen simulation/filtering workflow) — https://github.com/ykat0/capital
- **conda** (Environment manager used to create the Python 3.8 + graphviz environment for CAPITAL and dyngen)
- **dyngen 1.0.3** (Simulator that generates transcription-factor regulatory-network-driven single-cell count data on the binary-tree backbone)

## Examples

```
conda create -n capital python=3.8 graphviz; conda activate capital; pip install capital  # then run 120 dyngen sims (binary tree, 3 branches, 1000 cells) and keep the 68 with four-leaf cluster MSTs
```

## Evaluation signals

- Exactly 68 datasets are retained after filtering from 120 dyngen simulations
- Every retained dataset's cluster-derived minimum spanning tree has exactly four leaves
- Exported AnnData object loads in CAPITAL and matches the packaged capital.dataset.synthetic_dataset1 and synthetic_dataset2 reference objects
- Each simulation used the specified configuration: binary tree backbone with three branches, 1000 cells, 36 TFs, 250 target genes, 250 housekeeping genes
- Count matrices are accompanied by pseudotime and trajectory-tree annotations in the .h5ad export

## Limitations

- The four-leaf MST filter discards 52 of 120 simulations, so the filtered set is topology-biased toward binary-tree structures with three branches.
- CAPITAL's trajectory is computed as a minimum spanning tree over clusters, which the authors note 'may not be the best method'; alignment accuracy degrades for datasets with different global topological structure.
- The skill yields trajectory trees only — not complex graphs with cycles or disconnected graphs.

## Evidence

- [methods] conda create -n capital python=3.8 graphviz: "conda create -n capital python=3.8 graphviz"
- [methods] synthetic dataset references in CAPITAL package: "capital.dataset.synthetic\_dataset1"
- [discussion] MST-based trajectory computation caveat: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
- [discussion] tree-only scope limitation: "CAPITAL in its present form can deal only with trajectory trees (including single lineages) that are"
