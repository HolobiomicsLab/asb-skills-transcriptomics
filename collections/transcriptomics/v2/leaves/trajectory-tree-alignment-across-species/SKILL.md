---
name: trajectory-tree-alignment-across-species
description: Use when comparing pseudotime trajectories between related scRNA-seq
  datasets from different species (or conditions) whose trajectories contain branches,
  and you need to ask whether cell-type marker gene expression dynamics are conserved
  along aligned pseudotime.
when_to_use_negative:
- Datasets have different global topological structure — CAPITAL's alignment accuracy
  degraded in the authors' benchmarking for such cases
- Trajectories form complex graphs with cycles or disconnected components — CAPITAL
  in its present form deals only with trajectory trees, not general graphs
- The trajectories are simple linear paths with no branches — DTW-based linear alignment
  methods suffice and selecting a single lineage pair manually is straightforward
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: CAPITAL
  role: 'Core method: computes branched pseudotime trajectories, aligns trajectory
    trees globally via DTW extended to trees, and scores gene similarity across species'
  repo: https://github.com/ykat0/capital
- name: Python 3.8
  role: Runtime environment for CAPITAL (conda create -n capital python=3.8 graphviz)
- name: conda
  role: Environment manager used to create the CAPITAL environment with graphviz
- name: JupyterLab
  role: Interactive development environment in which CAPITAL is used
- name: Palantir
  role: Source of the Setty et al. bone marrow replicate data deposited with Palantir
    (capital.dataset.setty19 derives from the Palantir deposit)
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:b0a29474b059ac281907996b393631d33cb3537df000e2a7a1b5840c88d54235
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - JupyterLab
  - Palantir
  - Python
  - Python 3.8
  - conda
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# trajectory-tree-alignment-across-species

## Summary

Globally aligns branched pseudotime trajectory trees from scRNA-seq datasets of different species (e.g., human Velten et al. and mouse Paul et al. bone marrow) using CAPITAL's dynamic-time-warping-extended-to-trees, then compares marker-gene expression kinetics along aligned pseudotime to identify conserved vs. divergent lineage dynamics.

## When to use

Use when comparing pseudotime trajectories between related scRNA-seq datasets from different species (or conditions) whose trajectories contain branches, and you need to ask whether cell-type marker gene expression dynamics are conserved along aligned pseudotime.

## When NOT to use

- Datasets have different global topological structure — CAPITAL's alignment accuracy degraded in the authors' benchmarking for such cases
- Trajectories form complex graphs with cycles or disconnected components — CAPITAL in its present form deals only with trajectory trees, not general graphs
- The trajectories are simple linear paths with no branches — DTW-based linear alignment methods suffice and selecting a single lineage pair manually is straightforward

## Inputs

- Human bone marrow scRNA-seq dataset (capital.dataset.velten17; Velten et al. individual 1, GEO GSE75478)
- Mouse myeloid scRNA-seq dataset (capital.dataset.paul15; Paul et al.)
- Normalized, preprocessed expression (post capital.tl.preprocessing with MAGIC denoising, PCA, k-NN) for each dataset

## Outputs

- Minimum-spanning-tree-based trajectory tree per species (capital.tl.trajectory_tree)
- Global cross-species tree alignment (capital.tl.tree_alignment), rendered as tree_alignment.png
- Aligned marker-gene pseudotime kinetics plots (capital.pl.gene_expression_trend)
- Ranked list of conserved vs. divergent genes via capital.tl.genes_similarity_score

## How to apply

Load the two species' datasets via capital.dataset.velten17 (human, Velten et al., GEO GSE75478) and capital.dataset.paul15 (mouse, Paul et al.) in a Python 3.8/3.9 conda environment with graphviz installed. Run capital.tl.preprocessing on each dataset (MAGIC denoising, PCA, k-NN), then compute a branched pseudotime trajectory per species with capital.tl.dpt followed by capital.tl.trajectory_tree, which builds a minimum-spanning-tree over clusters. Run capital.tl.tree_alignment to compute the optimal global alignment between the two trajectory trees de novo via dynamic time warping extended to trees — no manual selection of linear paths is needed. Render the aligned trees with capital.pl.tree_alignment and inspect conserved lineage marker kinetics (e.g., erythrocyte, monocyte, neutrophil markers) with capital.pl.gene_expression_trend, then rank conservation with capital.tl.genes_similarity_score to quantify divergent vs. conserved genes (e.g., IRF7 and CSF1 human-high; ELANE mouse-high).

## Related tools

- **CAPITAL** (Core method: computes branched pseudotime trajectories, aligns trajectory trees globally via DTW extended to trees, and scores gene similarity across species) — https://github.com/ykat0/capital
- **Python 3.8** (Runtime environment for CAPITAL (conda create -n capital python=3.8 graphviz))
- **conda** (Environment manager used to create the CAPITAL environment with graphviz)
- **JupyterLab** (Interactive development environment in which CAPITAL is used)
- **Palantir** (Source of the Setty et al. bone marrow replicate data deposited with Palantir (capital.dataset.setty19 derives from the Palantir deposit)) — https://github.com/dpeerlab/Palantir

## Examples

```
conda activate capital; pip install capital; import capital as cp; a=cp.dataset.velten17(); b=cp.dataset.paul15(); cp.tl.preprocessing(a); cp.tl.preprocessing(b); cp.tl.dpt(a); cp.tl.dpt(b); cp.tl.trajectory_tree(a); cp.tl.trajectory_tree(b); cp.tl.tree_alignment(a,b)
```

## Evaluation signals

- The rendered aligned tree (tree_alignment.png) matches the topology reported in the paper, with conserved lineage branches aligning between species
- Erythrocyte, monocyte, and neutrophil lineage marker genes show the reported species-biased kinetics: IRF7 and CSF1 higher in human, ELANE higher in mouse
- capital.tl.genes_similarity_score ranks known conserved lineage markers above divergent ones across the aligned pseudotime
- Each species' trajectory tree is a valid minimum spanning tree over its clusters and reproduces expected leaf clusters (e.g., erythrocyte, monocyte, neutrophil branches)

## Limitations

- Part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method
- Alignment accuracy of CAPITAL degraded for datasets with different global topological structure in the benchmarking test
- CAPITAL in its present form can deal only with trajectory trees (including single lineages) and not complex graphs such as cycles or disconnected graphs; a predictive model handling graphs beyond trees plus graph alignment will be required for complex comparative pseudotime analysis
- Denoising and imputing expression counts were likely to generate more predicted clusters than previously reported, including intermediate states

## Evidence

- [methods] Cross-species alignment workflow: "capital.dataset.paul15"
- [methods] Tool installation: "conda create -n capital python=3.8 graphviz"
- [abstract] Core capability claim: "a method for comparing single-cell trajectories with tree alignment whereby branching trajectories can be automatically compared"
- [abstract] Cross-species finding: "revealing various gene expression dynamics including gene–gene correlation conservation between different species"
- [readme] README install instructions: "Create a new environment for CAPITAL (recommended)::"
