---
name: circular-autoencoder-pseudo-time-inference
description: Use when single-cell RNA-seq data contain a latent periodic process such
  as the cell cycle and linear PC-based or regression-based representations cannot
  accurately represent the circular timing of cells.
when_to_use_negative:
- The latent trajectory is linear/non-periodic (e.g., a differentiation branch) —
  the article positions Cyclum specifically for nonlinear periodicity that linear
  pseudo-time methods represent poorly, not for general trajectory inference.
- The goal is marker-gene-only phase scoring — Cyclum's methodology projects data
  to a circular manifold 'instead of using marker genes', so marker-based classification
  is out of scope.
- The matrix has not been log2(x+1)-transformed, centered to mean = 1 and scaled to
  variance = 1; feeding raw counts departs from the preprocessing the method was validated
  on.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cyclum
  role: 'Primary implementation: sinusoidal autoencoder that projects expression data
    onto a circular manifold, infers pseudo-time, and removes cell-cycle effects'
  repo: github.com/KChen-lab/cyclum
- name: Cyclum (cyclum2 notebooks)
  role: Provides the mESC analysis notebook used in the paper's virtual-tumor subclone-separability
    workflow
  repo: github.com/lshh125/cyclum2
- name: reCAT
  role: Model-free GMM baseline used to benchmark Cyclum's pseudo-time quality via
    GSEA NES on melanoma samples
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:2743e44443e07c9ffa074a00d75636b5641b3ef5351cdfc6e7ae9c295b59949c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclum
  - Cyclum (cyclum2 notebooks)
  - Seurat
  - ccRemover
  - reCAT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# circular-autoencoder-pseudo-time-inference

## Summary

Fit Cyclum's sinusoidal autoencoder to scRNA-seq data to project cells onto a circular manifold, unfold it via least-square embedding into per-cell pseudo-time, and optionally deconvolve the inferred cell-cycle effects. It addresses the failure of linear representations (PCA, regression-based regression of cell-cycle scores) to capture periodic, circular trajectories.

## When to use

Use when single-cell RNA-seq data contain a latent periodic process such as the cell cycle and linear PC-based or regression-based representations cannot accurately represent the circular timing of cells.

## When NOT to use

- The latent trajectory is linear/non-periodic (e.g., a differentiation branch) — the article positions Cyclum specifically for nonlinear periodicity that linear pseudo-time methods represent poorly, not for general trajectory inference.
- The goal is marker-gene-only phase scoring — Cyclum's methodology projects data to a circular manifold 'instead of using marker genes', so marker-based classification is out of scope.
- The matrix has not been log2(x+1)-transformed, centered to mean = 1 and scaled to variance = 1; feeding raw counts departs from the preprocessing the method was validated on.

## Inputs

- scRNA-seq gene x cell count matrix (e.g., mESC dataset or 10X Chromium melanoma samples Mel78, Mel79, Mel80, Mel81, Mel88)
- Matrix preprocessed by log2(x+1) transformation with centering to mean = 1 and scaling to variance = 1
- Optional discrete phase labels (G1/S/G2M) from scanpy.tl.score_genes_cell_cycle or Seurat::CellCycleScoring for label refinement

## Outputs

- Per-cell circular pseudo-time values obtained by unfolding the circular manifold onto a linear space
- Cell-cycle-effect-removed scRNA-seq expression matrix
- Refined G1/S/G2M phase labels consistent along pseudotime
- Gene-periodicity match scores identifying genes associated with the periodic process

## How to apply

Preprocess the count matrix with a log2(x+1) transformation, then center to mean = 1 and scale to variance = 1, as recommended for Cyclum. Use `cyclum.tuning.CyclumAutoTune` to select the proper number of linear components so the autoencoder finds the 'most circular' manifold, then fit the sinusoidal (Keras/TensorFlow) autoencoder to project the high-dimensional expression data onto the circular manifold rather than relying on marker genes. Unfold the circular manifold onto a linear space via optimal (least square) embedding to obtain per-cell pseudo-time, and optionally refine external G1/S/G2M labels (e.g., from Seurat::CellCycleScoring or scanpy.tl.score_genes_cell_cycle) along that pseudotime with `cyclum.postproc.refine_labels`. The model-based fit to predefined circular manifolds is what makes the inference robust to random noise and small sample sizes. Downstream, the inferred periodic process can be treated as a confounder and its effects removed, then validated by GSEA NES against the GO cell-cycle gene set or by two-component GMM subclone separability.

## Related tools

- **Cyclum** (Primary implementation: sinusoidal autoencoder that projects expression data onto a circular manifold, infers pseudo-time, and removes cell-cycle effects) — github.com/KChen-lab/cyclum
- **Cyclum (cyclum2 notebooks)** (Provides the mESC analysis notebook used in the paper's virtual-tumor subclone-separability workflow) — github.com/lshh125/cyclum2
- **reCAT** (Model-free GMM baseline used to benchmark Cyclum's pseudo-time quality via GSEA NES on melanoma samples)

## Examples

```
from cyclum.postproc import refine_labels; refine_labels(pseudotime=[4, 5, 1, 2, 3, 6, 7, 8], original_labels=['G1', 'G1', 'G1', 'G1', 'S', 'S', 'G2M', 'G2M'])
```

## Evaluation signals

- GSEA normalized enrichment score (NES) for the GO cell-cycle gene set computed on Cyclum pseudo-time exceeds reCAT, PC1, and PC2 on all five melanoma samples (Mel78–Mel81, Mel88), including low-cycling Mel79 (Fig. 4a / Supplementary Table 1 pattern; gsea_nes_ranking.csv).
- Two-component GMM subclone separability after Cyclum correction exceeds Seurat, ccRemover, and PC1–5, particularly when >400 cell-cycle genes are perturbed in virtual tumor data (Fig. 3c pattern; subclone_separability.csv).
- CyclumAutoTune selects the number of linear components that yields the 'most circular' manifold before fitting the autoencoder.
- Refined labels are locally consistent along pseudotime — e.g., refine_labels replaces an 'S' surrounded by 'G1' cells with 'G1'.
- Preprocessing invariant: after transformation, data are log2(x+1) with mean = 1 and variance = 1.

## Limitations

- Tested with TensorFlow 1.x (1.14.0) and keras 2.2.4; pip installs the latest dependencies, which differ from the tested versions, and TensorFlow < 1.15.2 has a known security issue.
- Runtime scales with data size: 12,280 cells x 33,694 genes takes ~40 min and was demonstrated on a GTX 960M GPU with 2 GB graphic memory.
- Cyclum's advantage over linear baselines is conditional — it is strongest when many cell-cycle genes are perturbed; advantage over model-free reCAT rests on fitting predefined circular manifolds, which presumes the process is genuinely circular.

## Evidence

- [abstract] we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space: "we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space"
- [intro] It effectively unfolds the circular manifold onto a linear space to obtain precise pseudo-time.: "It effectively unfolds the circular manifold onto a linear space to obtain precise pseudo-time."
- [methods] scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well.: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [discussion] Cyclum is a model-based approach that fits the data to predefined circular manifolds. This design makes Cyclum more robust to handle random noise and small sample sizes.: "Cyclum is a model-based approach that fits the data to predefined circular manifolds. This design makes Cyclum more robust to handle random noise and small sample sizes."
- [readme] We also implemented class `cyclum.tuning.CyclumAutoTune`, which automatically select the proper number of linear components to help find the "most circular" manifold.: [evidence span withheld — anchor status: external_doc]
