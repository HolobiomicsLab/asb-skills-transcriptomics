---
name: minimum-spanning-tree-trajectory-inference
description: Use when comparing pseudotime trajectories that include branches across
  two or more scRNA-seq datasets, and a tree-structured cell-state transition map
  (rather than a single linear path) is needed before trajectory tree alignment.
when_to_use_negative:
- The input trajectories are single linear lineages where simpler DTW-based linear
  alignment suffices (existing tools mainly focus on linear alignment of single-cell
  trajectories).
- The trajectory structure contains complex graphs such as cycles or disconnected
  graphs — CAPITAL in its present form can deal only with trajectory trees.
- The two datasets have very different global topological structures, since CAPITAL's
  alignment accuracy degraded for such datasets in the benchmarking test.
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CAPITAL
  role: Provides capital.tl.trajectory_tree (MST trajectory inference), capital.tl.dpt
    (pseudotime), capital.tl.tree_alignment, and capital.tl.dtw for branched trajectory
    construction and comparison
  repo: https://github.com/ykat0/capital
- name: Python 3.8
  role: Runtime environment for CAPITAL ('conda create -n capital python=3.8 graphviz')
- name: conda
  role: Environment manager used to create the CAPITAL environment with graphviz before
    'pip install capital'
- name: Palantir
  role: Related trajectory inference reference (dpeerlab/Palantir listed in the article's
    other-section references); not required for the CAPITAL MST workflow itself
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:fbae3657073dae8d3dfe631300d7ba2a51375853b3dba33b874f6a3a49179827
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Palantir
  - Python
  - Python 3.8
  - R
  - Scanorama
  - Seurat
  - SingleCellExperiment
  - Slingshot
  - Totem
  - conda
  - dyndimred
  - scVI
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# minimum-spanning-tree-trajectory-inference

## Summary

Infers branched single-cell pseudotime trajectories by building a minimum-spanning-tree over clusters (capital.tl.trajectory_tree) with pseudotime from diffusion pseudotime (capital.tl.dpt), enabling downstream tree alignment of scRNA-seq datasets in CAPITAL.

## When to use

Use when comparing pseudotime trajectories that include branches across two or more scRNA-seq datasets, and a tree-structured cell-state transition map (rather than a single linear path) is needed before trajectory tree alignment.

## When NOT to use

- The input trajectories are single linear lineages where simpler DTW-based linear alignment suffices (existing tools mainly focus on linear alignment of single-cell trajectories).
- The trajectory structure contains complex graphs such as cycles or disconnected graphs — CAPITAL in its present form can deal only with trajectory trees.
- The two datasets have very different global topological structures, since CAPITAL's alignment accuracy degraded for such datasets in the benchmarking test.

## Inputs

- Preprocessed scRNA-seq dataset(s) as AnnData objects (anndata>=0.7.4), e.g. loaded via capital.dataset.synthetic_dataset1, synthetic_dataset2, paul15, setty19, or velten17
- Cluster assignments on the cells (e.g. Leiden/Louvain via scanpy 1.9.1 within the CAPITAL environment)
- Pseudotime values computed with capital.tl.dpt

## Outputs

- A trajectory tree (minimum spanning tree over clusters) representing the branched cell-state transition structure per dataset
- Tree object suitable for input to capital.tl.tree_alignment and capital.tl.dtw for pairwise trajectory tree alignment

## How to apply

Install CAPITAL in a dedicated environment (e.g. 'conda create -n capital python=3.8 graphviz; pip install capital', README suggests python=3.9 for ver. 1.1.2) and load the dataset via capital.dataset loaders (e.g. capital.dataset.synthetic_dataset1/2, paul15, setty19, velten17). Preprocess with capital.tl.preprocessing, cluster the cells, and compute pseudotime with capital.tl.dpt. Call capital.tl.trajectory_tree to compute the trajectory as a minimum spanning tree over the clusters, which supports branching trajectories without pre-selecting linear paths. The resulting trajectory tree is then the input for capital.tl.tree_alignment (with capital.tl.dtw for alignment scoring) and for scoring with capital.tl.genes_similarity_score. Note the article itself cautions that computing a trajectory as a minimum spanning tree may not be the best method for all topologies.

## Related tools

- **CAPITAL** (Provides capital.tl.trajectory_tree (MST trajectory inference), capital.tl.dpt (pseudotime), capital.tl.tree_alignment, and capital.tl.dtw for branched trajectory construction and comparison) — https://github.com/ykat0/capital
- **Python 3.8** (Runtime environment for CAPITAL ('conda create -n capital python=3.8 graphviz'))
- **conda** (Environment manager used to create the CAPITAL environment with graphviz before 'pip install capital')
- **Palantir** (Related trajectory inference reference (dpeerlab/Palantir listed in the article's other-section references); not required for the CAPITAL MST workflow itself) — https://github.com/dpeerlab/Palantir

## Examples

```
conda create -n capital python=3.8 graphviz && conda activate capital && pip install capital; import capital as cp; adata = cp.dataset.synthetic_dataset1(); cp.tl.preprocessing(adata); cp.tl.dpt(adata); cp.tl.trajectory_tree(adata)
```

## Evaluation signals

- The inferred trajectory is a tree (no cycles, no disconnected components) spanning the dataset's clusters, as expected from the MST construction.
- Branching structure matches known biology of the input dataset (e.g. hematopoiesis branches in paul15 or setty19 bone marrow data).
- Downstream capital.tl.tree_alignment with capital.tl.dtw yields a valid alignment and nonzero capital.tl.genes_similarity_score between related datasets.
- Alignment accuracy should drop for pairs of datasets with different global topological structure, per the article's own limitation — a sanity check that the tree topology is being respected.
- The pipeline runs in the pinned CAPITAL environment (Python>=3.8, scanpy==1.9.1, matplotlib==3.5.2, networkx<=2.8.3) without dependency errors.

## Limitations

- Part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method (authors' own caveat).
- CAPITAL in its present form can deal only with trajectory trees (including single lineages) and not complex graphs such as cycles or disconnected graphs.
- Alignment accuracy of CAPITAL for datasets with different global topological structure degraded in the benchmarking test.
- A predictive model handling graphs beyond trees plus graph alignment will be required for complex comparative pseudotime analysis.

## Evidence

- [methods] pseudotime and alignment steps: "tl.trajectory_tree
   tl.tree_alignment
   tl.dpt
   tl.dtw"
- [discussion] MST limitation: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
- [intro] branched trajectory goal: "CAPITAL seeks to infer a pseudotime trajectory that can include multiple branches for each dataset, and then to compute an optimal alignment between the two trajectories."
- [readme] installation instructions: "Create a new environment for CAPITAL (recommended)::

    conda create -n capital python=3.8 graphviz"
