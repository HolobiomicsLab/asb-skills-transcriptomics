---
name: bone-marrow-hematopoiesis-lineage-annotation
description: Use when comparing pseudotime trajectories of two related bone-marrow/hematopoiesis
  scRNA-seq datasets that branch, and you must match lineage leaf clusters (erythrocytes,
  megakaryocytes, pre-B-cells, dendritic cells) without manually selecting linear
  paths.
when_to_use_negative:
- Datasets whose trajectories form cycles or disconnected graphs, or with different
  global topological structure — CAPITAL in its present form can deal only with trajectory
  trees, and alignment accuracy degraded in benchmarking for differing topology.
- The goal is expression-matrix batch integration (e.g. mutual-nearest-neighbor correction
  with Scanorama or Seurat) rather than comparing cell-state transition trajectory
  trees.
- Each dataset contains only a single unbranched lineage — plain DTW-based linear
  alignment of that lineage suffices without tree construction.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: CAPITAL
  role: 'Core method: MAGIC-based preprocessing, DPT pseudotime, minimum-spanning-tree
    trajectory trees, global tree alignment, and DTW gene-kinetics comparison (pip
    install capital)'
  repo: github.com/ykat0/capital
- name: Palantir
  role: Source of the processed Setty et al. replicate-1 human CD34+ bone marrow dataset
    packaged as capital.dataset.setty19
  repo: dpeerlab/Palantir
- name: Scanpy
  role: Single-cell toolkit CAPITAL is implemented on, providing preprocessing, plotting,
    and dataset utilities
  repo: https://scanpy.readthedocs.io/en/latest/index.html
- name: JupyterLab
  role: Interactive development environment in which CAPITAL's Python workflow is
    run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:a0ec35403abaa07281aa286b7d55803c36c89968019239039ea1744dc7fed7a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAPITAL
  - JupyterLab
  - Palantir
  - Python
  - Scanpy
  - conda
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# bone-marrow-hematopoiesis-lineage-annotation

## Summary

Annotates hematopoietic lineages in human bone marrow scRNA-seq by globally aligning branched pseudotime trajectory trees from two related datasets (Setty et al. 2019; Velten et al. 2017) with CAPITAL, matching leaf clusters (e.g. erythrocytes, megakaryocytes, pre-B-cells, dendritic cells) and comparing marker-gene dynamics via DTW.

## When to use

Use when comparing pseudotime trajectories of two related bone-marrow/hematopoiesis scRNA-seq datasets that branch, and you must match lineage leaf clusters (erythrocytes, megakaryocytes, pre-B-cells, dendritic cells) without manually selecting linear paths.

## When NOT to use

- Datasets whose trajectories form cycles or disconnected graphs, or with different global topological structure — CAPITAL in its present form can deal only with trajectory trees, and alignment accuracy degraded in benchmarking for differing topology.
- The goal is expression-matrix batch integration (e.g. mutual-nearest-neighbor correction with Scanorama or Seurat) rather than comparing cell-state transition trajectory trees.
- Each dataset contains only a single unbranched lineage — plain DTW-based linear alignment of that lineage suffices without tree construction.

## Inputs

- Setty et al. human CD34+ bone marrow scRNA-seq replicate 1 (capital.dataset.setty19; h5ad from the Palantir deposit human_cd34_bm_rep1.h5ad)
- Velten et al. human bone marrow scRNA-seq individual 1 (capital.dataset.velten17; GEO GSE75478)
- MAGIC-denoised AnnData objects with PCA embedding and k-NN graph produced by capital.tl.preprocessing
- Curated marker-gene lists for hematopoietic lineages to align pseudotime kinetics

## Outputs

- Per-dataset branched trajectory tree (minimum spanning tree over clusters), rendered via capital.pl.trajectory_tree
- Global leaf-cluster alignment mapping between the two trajectory trees covering erythrocyte, megakaryocyte, pre-B-cell, and dendritic-cell lineages
- DTW-aligned marker-gene pseudotime kinetics and expression trend plots (capital.pl.gene_expression_trend, capital.pl.dtw)

## How to apply

Create a conda environment for CAPITAL (conda create -n capital python=3.8 graphviz, activate it, pip install capital; version 1.1.2 pins scanpy==1.9.1, matplotlib==3.5.2, networkx<=2.8.3). Load the two packaged bone marrow datasets — capital.dataset.setty19 (Setty et al. replicate 1 from the Palantir deposit, human_cd34_bm_rep1.h5ad) and capital.dataset.velten17 (Velten et al. individual 1, GEO GSE75478) — and run capital.tl.preprocessing on each with the documented MAGIC denoising, PCA, and k-NN settings. Infer pseudotime per dataset with capital.tl.dpt, then compute each branched trajectory as a minimum spanning tree over clusters with capital.tl.trajectory_tree. Run capital.tl.tree_alignment to obtain the global, de novo alignment between the two trees — the key decision point is that no linear path is selected manually; CAPITAL computes Spearman's correlation-based matches rather than mutual nearest neighbors as in Scanorama/Seurat integration. Align marker-gene pseudotime kinetics between datasets with capital.tl.dtw and render with capital.pl.trajectory_tree, capital.pl.gene_expression_trend, and capital.pl.dtw (interactive use in JupyterLab is supported). Validate that the aligned tree reproduces the expected lineage leaves — erythrocytes, megakaryocytes, pre-B-cells, dendritic cells — and that DTW-aligned kinetics are visually consistent with reported marker dynamics.

## Related tools

- **CAPITAL** (Core method: MAGIC-based preprocessing, DPT pseudotime, minimum-spanning-tree trajectory trees, global tree alignment, and DTW gene-kinetics comparison (pip install capital)) — github.com/ykat0/capital
- **Palantir** (Source of the processed Setty et al. replicate-1 human CD34+ bone marrow dataset packaged as capital.dataset.setty19) — dpeerlab/Palantir
- **Scanpy** (Single-cell toolkit CAPITAL is implemented on, providing preprocessing, plotting, and dataset utilities) — https://scanpy.readthedocs.io/en/latest/index.html
- **JupyterLab** (Interactive development environment in which CAPITAL's Python workflow is run)

## Examples

```
conda create -n capital python=3.8 graphviz && conda activate capital && pip install capital
```

## Evaluation signals

- The global tree alignment matches corresponding leaf clusters across the Setty and Velten trees for the expected lineages: erythrocytes, megakaryocytes, pre-B-cells, and dendritic cells.
- capital.pl.trajectory_tree renders each dataset as a minimum spanning tree over clusters without errors after capital.tl.trajectory_tree.
- capital.tl.dtw produces aligned marker-gene kinetics plots that are visually consistent with reported expression dynamics along pseudotime (capital.pl.gene_expression_trend / capital.pl.dtw).
- Pseudotime values exist per dataset after capital.tl.dpt run on the MAGIC-denoised, PCA/k-NN-processed AnnData from capital.tl.preprocessing.
- Environment checks: CAPITAL 1.1.2 installs in a python=3.8/3.9 conda env with graphviz, honoring the pinned dependencies (scanpy==1.9.1, matplotlib==3.5.2, networkx<=2.8.3, anndata>=0.7.4).

## Limitations

- CAPITAL in its present form can deal only with trajectory trees (including single lineages) and not complex graphs such as cycles or disconnected graphs; a predictive model handling graphs beyond trees plus graph alignment would be required.
- Alignment accuracy of CAPITAL for datasets with different global topological structure degraded in the benchmarking test.
- Part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method.
- Denoising and imputing (MAGIC) expression counts of single cells generated more predicted clusters than previously reported, including intermediate states, which changes the tree structure being aligned.
- Installation depends on version-pinned libraries (e.g. scanpy==1.9.1, matplotlib==3.5.2, networkx<=2.8.3); newer dependency resolutions may break.

## Evidence

- [abstract] Here we present CAPITAL (comparative analysis of pseudotime trajectory inference with tree alignment), a method for comparing single-cell trajectories with tree alignment: "Here we present CAPITAL (comparative analysis of pseudotime trajectory inference with tree alignment), a method for comparing single-cell trajectories with tree alignment"
- [discussion] CAPITAL is a computational tool for comparing cell-state transition trajectory trees of different but related scRNA-seq datasets: "CAPITAL is a computational tool for comparing cell-state transition trajectory trees of different but related scRNA-seq datasets"
- [discussion] alignment accuracy of CAPITAL for datasets with different global topological structure degraded in our benchmarking test: "alignment accuracy of CAPITAL for datasets with different global topological structure degraded in our benchmarking test"
- [discussion] CAPITAL in its present form can deal only with trajectory trees (including single lineages) that are computed from input datasets: "CAPITAL in its present form can deal only with trajectory trees (including single lineages) that are computed from input datasets"
- [methods] We implemented CAPITAL in Python, which can also be used in the interactive development environment JupyterLab: "We implemented CAPITAL in Python, which can also be used in the interactive development environment JupyterLab"
- [readme] We present CAPITAL, a computational method for comparing pseudotime trajectories with tree alignment whereby trajectories including branches can be automatically compared.: "We present CAPITAL, a computational method for comparing pseudotime trajectories with tree alignment whereby trajectories including branches can be automatically compared."
