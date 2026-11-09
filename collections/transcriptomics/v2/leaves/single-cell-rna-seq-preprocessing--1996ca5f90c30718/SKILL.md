---
name: single-cell-rna-seq-preprocessing
description: Use when preparing single-cell count matrices (e.g., capital.dataset.setty19,
  velten17, paul15) for branched pseudotime trajectory inference or trajectory-tree
  alignment, where denoised, dimension-reduced expression is required.
when_to_use_negative:
- If the goal is batch integration of multiple datasets — the article notes anchor/integration
  approaches (Scanorama, Seurat) use mutual nearest neighbors whereas CAPITAL computes
  Spearman's correlation for alignment, and integration can wrongly match cell subpopulations
- If trajectories are expected to contain cycles or disconnected graphs — CAPITAL
  handles only trajectory trees computed from the preprocessed data
- If the input is already denoised/embedded and compatible with capital.tl.trajectory_tree,
  re-running preprocessing may generate an undesirably different cluster structure
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CAPITAL
  role: Provides capital.tl.preprocessing (MAGIC denoising, PCA, k-NN) that prepares
    scRNA-seq counts for trajectory-tree inference
  repo: https://github.com/ykat0/capital
- name: Scanpy
  role: Underlying single-cell toolkit CAPITAL is built on, providing preprocessing
    and plotting functions usable in the CAPITAL environment
  repo: https://scanpy.readthedocs.io/en/latest/index.html
- name: conda
  role: Creates the Python 3.8/3.9 environment with graphviz required to install and
    run CAPITAL's preprocessing
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:d813110f4fd254d174b886a411aafe66c61a84dabf7236394ee3a0046c7b051e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - CHETAH
  - Garnett
  - JupyterLab
  - Python
  - R
  - SCINA
  - SELINA
  - Scanpy
  - SingleCellNet
  - SingleR
  - conda
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# single-cell-rna-seq-preprocessing

## Summary

Standardizes raw scRNA-seq count data for trajectory-tree alignment by running CAPITAL's MAGIC denoising, PCA, and k-NN preprocessing on each dataset (e.g., Setty19 human CD34+ bone marrow, Velten17, Paul15) before pseudotime and tree inference.

## When to use

Use when preparing single-cell count matrices (e.g., capital.dataset.setty19, velten17, paul15) for branched pseudotime trajectory inference or trajectory-tree alignment, where denoised, dimension-reduced expression is required.

## When NOT to use

- If the goal is batch integration of multiple datasets — the article notes anchor/integration approaches (Scanorama, Seurat) use mutual nearest neighbors whereas CAPITAL computes Spearman's correlation for alignment, and integration can wrongly match cell subpopulations
- If trajectories are expected to contain cycles or disconnected graphs — CAPITAL handles only trajectory trees computed from the preprocessed data
- If the input is already denoised/embedded and compatible with capital.tl.trajectory_tree, re-running preprocessing may generate an undesirably different cluster structure

## Inputs

- Raw single-cell expression count matrix as an AnnData object (e.g., from capital.dataset.setty19, capital.dataset.velten17, capital.dataset.paul15, or packaged synthetic_dataset1/2)
- Preprocessing configuration (MAGIC denoising, PCA dimensions, k-NN graph settings per CAPITAL documentation)

## Outputs

- Preprocessed AnnData object with MAGIC-denoised/imputed expression, PCA embedding, and k-NN graph, ready for capital.tl.dpt and capital.tl.trajectory_tree

## How to apply

Create a conda environment with Python 3.8/3.9 and graphviz (conda create -n capital python=3.8 graphviz) and install CAPITAL from PyPI (pip install capital). Load the packaged dataset (e.g., capital.dataset.setty19 for the Setty et al. Palantir replicate 1 bone marrow data, or capital.dataset.velten17 for Velten individual 1 from GEO GSE75478) and apply capital.tl.preprocessing with the documented MAGIC denoising, PCA, and k-NN settings. Rationale: the article reports that denoising and imputing expression counts generates more predicted clusters than previously reported, including intermediate states, which the downstream MST-based trajectory tree (capital.tl.trajectory_tree over clusters) relies on. Preprocessing must be run on each dataset independently before computing pseudotime with capital.tl.dpt; CAPITAL builds on Scanpy so Scanpy preprocessing functions are also usable. Preprocessed AnnData objects feed directly into tree alignment (capital.tl.tree_alignment) and DTW gene-kinetics analysis (capital.tl.dtw).

## Related tools

- **CAPITAL** (Provides capital.tl.preprocessing (MAGIC denoising, PCA, k-NN) that prepares scRNA-seq counts for trajectory-tree inference) — https://github.com/ykat0/capital
- **Scanpy** (Underlying single-cell toolkit CAPITAL is built on, providing preprocessing and plotting functions usable in the CAPITAL environment) — https://scanpy.readthedocs.io/en/latest/index.html
- **conda** (Creates the Python 3.8/3.9 environment with graphviz required to install and run CAPITAL's preprocessing)

## Examples

```
conda create -n capital python=3.8 graphviz; conda activate capital; pip install capital; import capital; adata = capital.dataset.setty19(); capital.tl.preprocessing(adata)
```

## Evaluation signals

- Preprocessing runs without errors in a Python 3.8 conda environment with anndata>=0.7.4, magic-impute, graphviz, scanpy==1.9.1, and other pinned CAPITAL dependencies
- Denoised/imputed expression yields more predicted clusters than raw counts (including intermediate states), consistent with the article's discussion of denoising effects
- Downstream capital.tl.trajectory_tree over the preprocessed clusters produces a minimum-spanning-tree trajectory with the expected number of leaves (e.g., four-leaf MSTs for dyngen synthetic datasets; erythrocyte, megakaryocyte, pre-B-cell, and dendritic-cell leaves for bone marrow data)
- The preprocessed AnnData feeds capital.tl.dpt pseudotime computation without missing PCA or k-NN fields

## Limitations

- Denoising and imputing expression counts likely generates more predicted clusters than previously reported, including intermediate states, which can alter downstream tree topology
- The downstream trajectory computation as a minimum spanning tree may not be the best method, so preprocessing choices affect tree quality
- CAPITAL is implemented against specific pinned versions (e.g., scanpy==1.9.1, matplotlib==3.5.2, networkx<=2.8.3), so environment drift can break preprocessing

## Evidence

- [readme] Preprocessing with MAGIC denoising, PCA and k-NN precedes trajectory-tree inference: [evidence span withheld — anchor status: external_doc]
- [discussion] Denoising changes cluster structure: "We confirmed that denoising and imputing expression counts of single cells were likely to generate more number of predicted clusters than previously reported"
- [methods] Environment setup for CAPITAL: "conda create -n capital python=3.8 graphviz"
- [discussion] MS trajectory may be suboptimal: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
- [discussion] Comparison to integration approaches: "Scanorama and Seurat calculate mutual nearest neighbors in a low-dimensional space, whereas CAPITAL computes Spearman’s correlation"
