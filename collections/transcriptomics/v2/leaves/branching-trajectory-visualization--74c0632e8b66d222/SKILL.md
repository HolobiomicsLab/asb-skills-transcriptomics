---
name: branching-trajectory-visualization
description: Use when pseudotime trajectories contain branches (e.g. human vs mouse
  bone marrow scRNA-seq) and you need to visualize trajectory trees or a globally
  aligned tree to inspect conserved/divergent lineage marker kinetics.
when_to_use_negative:
- The trajectories of interest are graphs with cycles or disconnected components —
  'CAPITAL in its present form can deal only with trajectory trees (including single
  lineages)'
- You have linear, unbranched lineages that existing linear DTW-based alignment tools
  already handle, and selecting single paths is not a problem
- The two datasets have very different global topological structures, where CAPITAL's
  alignment accuracy degraded in benchmarking
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CAPITAL
  role: Computes branched pseudotime trajectory trees, aligns them via tree DTW, and
    renders alignments and gene expression trends (tl.trajectory_tree, tl.tree_alignment,
    pl.tree_alignment, pl.gene_expression_trend)
  repo: https://github.com/ykat0/capital
- name: Scanpy
  role: Underlying toolkit CAPITAL uses so users can access preprocessing, plotting,
    and datasets inside the CAPITAL environment
  repo: https://scanpy.readthedocs.io/en/latest/index.html
- name: graphviz
  role: Graph rendering dependency installed via conda for drawing trajectory trees
    and alignments
- name: JupyterLab
  role: Interactive development environment in which CAPITAL's Python implementation
    can be used
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:c58447a016fed2451a877307f1bc8be7474ae4ed556389247d354e6b1ecceb43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - JupyterLab
  - Python
  - Scanpy
  - conda
  - graphviz
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# branching-trajectory-visualization

## Summary

Render and inspect branched single-cell pseudotime trajectories as minimum-spanning-tree trajectory trees, including cross-dataset tree alignments, using CAPITAL's plotting functions (capital.pl.tree_alignment, capital.pl.gene_expression_trend) on scRNA-seq datasets such as human Velten17 and mouse Paul15 bone marrow.

## When to use

Use when pseudotime trajectories contain branches (e.g. human vs mouse bone marrow scRNA-seq) and you need to visualize trajectory trees or a globally aligned tree to inspect conserved/divergent lineage marker kinetics.

## When NOT to use

- The trajectories of interest are graphs with cycles or disconnected components — 'CAPITAL in its present form can deal only with trajectory trees (including single lineages)'
- You have linear, unbranched lineages that existing linear DTW-based alignment tools already handle, and selecting single paths is not a problem
- The two datasets have very different global topological structures, where CAPITAL's alignment accuracy degraded in benchmarking

## Inputs

- scRNA-seq count data for branched trajectories as AnnData objects (e.g. capital.dataset.velten17 human bone marrow, capital.dataset.paul15 mouse myeloid)
- pseudotime values computed with capital.tl.dpt for each dataset
- lineage marker gene lists (e.g. erythrocyte, monocyte, neutrophil markers)

## Outputs

- Rendered aligned trajectory tree image (tree_alignment.png) from capital.pl.tree_alignment
- Per-species trajectory trees (minimum spanning trees over clusters)
- Aligned pseudotime mapping between the two trees
- Gene expression trend plots across aligned pseudotime (capital.pl.gene_expression_trend)
- Ranked conserved/divergent gene similarity scores (capital.tl.genes_similarity_score)

## How to apply

Set up a CAPITAL environment (Python 3.8+, graphviz installed via conda, e.g. 'conda create -n capital python=3.8 graphviz') and install with 'pip install capital' (CAPITAL ver. 1.1.2; scanpy==1.9.1, matplotlib==3.5.2, networkx<=2.8.3 pinned). Load each branched dataset (e.g. capital.dataset.velten17 for human, capital.dataset.paul15 for mouse), run capital.tl.preprocessing, then compute each minimum-spanning-tree trajectory with capital.tl.trajectory_tree using capital.tl.dpt pseudotime. For cross-species comparison, run capital.tl.tree_alignment (dynamic time warping extended to trees) and render the result with capital.pl.tree_alignment, saving it as tree_alignment.png; validate that the rendered topology matches the published alignment (e.g. Fig. 6, erythrocyte/monocyte/neutrophil branches aligning between species). Follow up with capital.pl.gene_expression_trend to plot marker kinetics across the aligned pseudotime (e.g. IRF7 and CSF1 higher in human, ELANE higher in mouse) and capital.tl.genes_similarity_score to rank conserved vs divergent genes. Rationale: CAPITAL globally aligns trajectory trees de novo, so no linear paths need to be selected beforehand.

## Related tools

- **CAPITAL** (Computes branched pseudotime trajectory trees, aligns them via tree DTW, and renders alignments and gene expression trends (tl.trajectory_tree, tl.tree_alignment, pl.tree_alignment, pl.gene_expression_trend)) — https://github.com/ykat0/capital
- **Scanpy** (Underlying toolkit CAPITAL uses so users can access preprocessing, plotting, and datasets inside the CAPITAL environment) — https://scanpy.readthedocs.io/en/latest/index.html
- **graphviz** (Graph rendering dependency installed via conda for drawing trajectory trees and alignments)
- **JupyterLab** (Interactive development environment in which CAPITAL's Python implementation can be used)

## Examples

```
import capital; a1=capital.dataset.velten17(); a2=capital.dataset.paul15(); capital.tl.preprocessing(a1); capital.tl.trajectory_tree(a1); capital.tl.tree_alignment(a1,a2); capital.pl.tree_alignment(a1,a2)
```

## Evaluation signals

- The rendered tree_alignment.png matches the topology reported in the paper, with conserved lineage branches (erythrocyte, monocyte, neutrophil) aligning between human and mouse
- Marker gene kinetics on the aligned pseudotime reproduce the reported species bias: IRF7 and CSF1 higher in human, ELANE higher in mouse
- Each dataset yields a connected trajectory tree computed from clusters as a minimum spanning tree using capital.tl.dpt pseudotime
- capital.tl.genes_similarity_score ranks marker genes consistently with the observed conservation/divergence in the trend plots
- Environment reproducibility: CAPITAL ver. 1.1.2 with the pinned dependencies (scanpy==1.9.1, matplotlib==3.5.2, networkx<=2.8.3) runs without import errors

## Limitations

- Part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method
- Alignment accuracy of CAPITAL degrades for datasets with different global topological structure
- CAPITAL can deal only with trajectory trees (including single lineages), not complex graphs such as cycles or disconnected graphs; a graph-handling model plus graph alignment would be required for such cases
- Denoising and imputing expression counts was observed to generate more predicted clusters (including intermediate states) than previously reported, which can affect tree topology

## Evidence

- [abstract] a method for comparing single-cell trajectories with tree alignment whereby branching trajectories can be automatically compared: "a method for comparing single-cell trajectories with tree alignment whereby branching trajectories can be automatically compared"
- [discussion] CAPITAL is a computational tool for comparing cell-state transition trajectory trees of different but related scRNA-seq datasets: "CAPITAL is a computational tool for comparing cell-state transition trajectory trees of different but related scRNA-seq datasets"
- [methods] conda create -n capital python=3.8 graphviz: "conda create -n capital python=3.8 graphviz"
- [readme] We present CAPITAL, a computational method for comparing pseudotime trajectories with tree alignment whereby trajectories including branches can be automatically compared.: "We present CAPITAL, a computational method for comparing pseudotime trajectories with tree alignment whereby trajectories including branches can be automatically compared."
- [discussion] part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
