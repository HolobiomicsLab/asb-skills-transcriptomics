---
name: pseudotime-gene-expression-kinetics-comparison
description: Use when you need to ask whether cell-type marker gene expression dynamics
  are conserved or divergent between species/conditions along branched pseudotime
  trajectories, without manually selecting linear lineage paths.
when_to_use_negative:
- 'Your pseudotime trajectories contain cycles or disconnected graphs: CAPITAL in
  its present form can deal only with trajectory trees, not complex graphs'
- 'The two datasets have different global topological structure: alignment accuracy
  of CAPITAL degraded in the authors'' benchmarking test'
- You only need to compare a single known linear lineage and can select paths in advance;
  a simpler DTW-based linear alignment tool may suffice
tools:
- name: CAPITAL
  role: Performs preprocessing, trajectory-tree inference, global tree alignment,
    aligned kinetic plotting, and gene similarity scoring across the two datasets
  repo: https://github.com/ykat0/capital
- name: Python
  role: Implementation language for the CAPITAL workflow, usable in JupyterLab
- name: conda
  role: Creates the CAPITAL environment (conda create -n capital python=3.8 graphviz)
    required to run the pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:d1be5bada32bff57884d30bff5de842d3b1d0d7115a955031c6fc5e0980717b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Python
  - conda
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# pseudotime-gene-expression-kinetics-comparison

## Summary

Compare gene expression dynamics along pseudotime between two scRNA-seq datasets (e.g., human Velten et al. and mouse Paul et al. bone marrow) by globally aligning branched trajectory trees with CAPITAL and scoring cross-species conservation of marker-gene kinetics.

## When to use

Use when you need to ask whether cell-type marker gene expression dynamics are conserved or divergent between species/conditions along branched pseudotime trajectories, without manually selecting linear lineage paths.

## When NOT to use

- Your pseudotime trajectories contain cycles or disconnected graphs: CAPITAL in its present form can deal only with trajectory trees, not complex graphs
- The two datasets have different global topological structure: alignment accuracy of CAPITAL degraded in the authors' benchmarking test
- You only need to compare a single known linear lineage and can select paths in advance; a simpler DTW-based linear alignment tool may suffice

## Inputs

- scRNA-seq count matrices as AnnData objects for each species/condition (human Velten et al. bone marrow via capital.dataset.velten17; mouse Paul et al. myeloid via capital.dataset.paul15)
- Preprocessed, normalized expression from capital.tl.preprocessing
- Cell-type marker gene lists (e.g., erythrocyte, monocyte, neutrophil markers)

## Outputs

- Aligned trajectory trees rendered as tree_alignment.png via capital.pl.tree_alignment
- Aligned pseudotime marker-gene expression trend plots via capital.pl.gene_expression_trend
- Ranked gene similarity/conservation scores from capital.tl.genes_similarity_score identifying conserved and divergent genes

## How to apply

Load each dataset (e.g., capital.dataset.velten17 for human bone marrow and capital.dataset.paul15 for mouse myeloid) in a conda environment with Python 3.8/3.9 and graphviz, then apply capital.tl.preprocessing to each. Infer a branched pseudotime trajectory per dataset with capital.tl.trajectory_tree built on capital.tl.dpt pseudotime, yielding a minimum-spanning-tree-based trajectory tree. Run capital.tl.tree_alignment to globally align the two trees de novo via dynamic time warping extended to trees, and render with capital.pl.tree_alignment (save as tree_alignment.png). Inspect marker kinetics along the aligned pseudotime with capital.pl.gene_expression_trend (e.g., IRF7 and CSF1 higher in human, ELANE higher in mouse), then rank conserved versus divergent genes with capital.tl.genes_similarity_score.

## Related tools

- **CAPITAL** (Performs preprocessing, trajectory-tree inference, global tree alignment, aligned kinetic plotting, and gene similarity scoring across the two datasets) — https://github.com/ykat0/capital
- **Python** (Implementation language for the CAPITAL workflow, usable in JupyterLab)
- **conda** (Creates the CAPITAL environment (conda create -n capital python=3.8 graphviz) required to run the pipeline)

## Examples

```
import capital as cap; adata_h = cap.dataset.velten17(); adata_m = cap.dataset.paul15(); cap.tl.preprocessing(adata_h); cap.tl.preprocessing(adata_m); cap.tl.trajectory_tree(adata_h); cap.tl.trajectory_tree(adata_m); cap.tl.tree_alignment(adata_h, adata_m)
```

## Evaluation signals

- The rendered aligned tree (tree_alignment.png) matches the topology reported in the paper, with conserved lineage branches aligning between species
- Marker genes show the reported species-biased kinetics: IRF7 and CSF1 human-high, ELANE mouse-high
- Subclasses of erythrocyte, monocyte, and neutrophil marker genes show conserved expression dynamics along the CAPITAL-aligned pseudotime
- genes_similarity_score rankings separate conserved from divergent genes consistent with the paper's findings
- Each dataset yields a valid minimum-spanning-tree trajectory from capital.tl.trajectory_tree before alignment

## Limitations

- CAPITAL in its present form can deal only with trajectory trees (including single lineages) and not complex graphs such as cycles or disconnected graphs
- Alignment accuracy degrades for datasets with different global topological structure
- Part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method
- A predictive model handling graphs beyond trees plus graph alignment would be required for complex comparative pseudotime analysis
- Denoising and imputing expression counts were likely to generate more predicted clusters than previously reported, which can affect trajectory structure

## Evidence

- [intro] Comparative analysis of pseudotime trajectories can unveil regulators that determine cell fates: "Comparison of pseudotime trajectories will provide a key to unveiling regulators that determine cell fates."
- [abstract] CAPITAL revealed gene expression dynamics including gene-gene correlation conservation between different species: "revealing various gene expression dynamics including gene–gene correlation conservation between different species"
- [discussion] Evaluate aligned pseudotime kinetics of molecular patterns across experiments: "investigating various molecular patterns between different experiments on the same/different species through aligned pseudotime kinetics"
- [methods] Create the CAPITAL conda environment with graphviz: "conda create -n capital python=3.8 graphviz"
- [readme] CAPITAL automatically compares branching trajectories with tree alignment: [evidence span withheld — anchor status: external_doc]
