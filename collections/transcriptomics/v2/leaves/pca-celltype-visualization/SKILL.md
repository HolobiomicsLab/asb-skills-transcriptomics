---
name: pca-celltype-visualization
description: Use when evaluating whether a denoising method preserved or recovered
  celltype structure in scRNA-seq data with known labels, by comparing PCA embeddings
  of denoised vs. ground-truth matrices.
when_to_use_negative:
- When no ground-truth celltype labels exist (e.g., fully unsampled real data), since
  recovery cannot be visually or quantitatively judged — the article notes evaluating
  denoising is difficult because defining a ground truth for real data is challenging
- When the analysis goal is differential expression or trajectory inference rather
  than assessing celltype structure recovery after denoising
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Python API compatible with DCA, used for PCA computation and visualization
    of denoised AnnData objects
  repo: https://github.com/theislab/dca
- name: anndata
  role: Stores simulated count matrices, ground-truth labels, and denoised outputs
    as h5ad objects feeding the PCA plots
  repo: https://github.com/theislab/anndata
- name: DCA
  role: ZINB-loss denoiser whose output (mean.tsv) is the treatment condition visualized
    against ground truth by PCA
  repo: https://github.com/theislab/dca
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:6f4174d0f9d63b55344743565ea20fbdd4636436ac583706020c419a6ac25d15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Scanpy
  - anndata
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# PCA celltype visualization after denoising

## Summary

Run PCA on ground-truth, DCA-denoised, and baseline-denoised scRNA-seq count matrices and plot cells colored by known celltype labels to visually verify that a denoiser (e.g., DCA with ZINB loss) recovers the original celltype structure that an MSE-based autoencoder fails to recover in dropout data.

## When to use

Use when evaluating whether a denoising method preserved or recovered celltype structure in scRNA-seq data with known labels, by comparing PCA embeddings of denoised vs. ground-truth matrices.

## When NOT to use

- When no ground-truth celltype labels exist (e.g., fully unsampled real data), since recovery cannot be visually or quantitatively judged — the article notes evaluating denoising is difficult because defining a ground truth for real data is challenging
- When the analysis goal is differential expression or trajectory inference rather than assessing celltype structure recovery after denoising

## Inputs

- Denoised count matrices (DCA mean.tsv output, gene x cell TSV) and MSE-autoencoder reconstructions
- Ground-truth no-dropout count matrix from Splatter simulation (200 genes, 2000 cells)
- Ground-truth celltype labels for the simulated cells
- AnnData h5ad objects constructed with the autoencoder.io module

## Outputs

- PCA scatter plots colored by celltype for ground-truth, DCA-denoised, and MSE-denoised conditions (simulation_pca.png)
- Comparison table of silhouette scores and clustering agreement (ARI) per condition (simulation_denoising_metrics.csv)

## How to apply

After denoising the dropout-affected count matrix (e.g., Splatter-simulated two- or six-celltype data with 200 genes and 2000 cells denoised by DCA using the ZINB loss, or by an MSE-reconstruction autoencoder baseline), load the denoised matrix plus the ground-truth matrix and simulated celltype labels into AnnData h5ad objects via the autoencoder.io module. Compute a PCA embedding for each condition (ground truth, DCA-denoised, MSE-autoencoder-denoised) and scatter-plot cells colored by the simulated celltype labels. Support the visual comparison quantitatively with silhouette scores and clustering agreement (e.g., ARI) between cluster assignments and ground-truth labels, writing the table to simulation_denoising_metrics.csv and the PCA figures to simulation_pca.png. Accept the result if DCA's celltype recovery metrics are at least as high as the MSE autoencoder's, demonstrating the count-aware ZINB loss better recovers the no-dropout structure.

## Related tools

- **Scanpy** (Python API compatible with DCA, used for PCA computation and visualization of denoised AnnData objects) — https://github.com/theislab/dca
- **anndata** (Stores simulated count matrices, ground-truth labels, and denoised outputs as h5ad objects feeding the PCA plots) — https://github.com/theislab/anndata
- **DCA** (ZINB-loss denoiser whose output (mean.tsv) is the treatment condition visualized against ground truth by PCA) — https://github.com/theislab/dca

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('dca_denoised.h5ad'); sc.pp.pca(adata); sc.pl.pca(adata, color='celltype', save='_simulation_pca.png')
```

## Evaluation signals

- PCA plots of DCA-denoised data show cells segregating by simulated celltype labels, matching the ground-truth (no-dropout) embedding
- PCA plots of the MSE-autoencoder-denoised data fail to separate the simulated celltypes
- Silhouette scores computed on each condition's PCA/embedding are higher for DCA than the MSE autoencoder and at least as high as ground truth
- Clustering agreement (e.g., ARI) between cluster assignments and ground-truth labels is at least as high for DCA as for the MSE autoencoder
- simulation_denoising_metrics.csv contains silhouette and clustering-agreement values for both two- and six-celltype simulations, both denoisers, and the ground truth

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data, and defining a ground truth for real data is challenging — this visualization is only defensible with simulated data where labels are known
- Overimputation can occur with inadequate hyperparameter choices (e.g., too low-dimensional bottleneck layer), which could distort the PCA celltype structure
- Visual PCA separation is qualitative; it must be backed by quantitative metrics such as silhouette score and ARI

## Evidence

- [other] Run PCA on the ground-truth, DCA-denoised, and MSE-autoencoder-denoised matrices and plot cells colored by simulated celltype.: [evidence span withheld — anchor status: not_found]
- [results] In the simulated dropout data, DCA recovered the original celltypes while the MSE-based autoencoder failed to recover them.: [evidence span withheld — anchor status: not_found]
- [intro] Scanpy users can directly use the "dca" method in the preprocessing package: [evidence span withheld — anchor status: not_found]
- [discussion] the Python API is compatible with the Scanpy33 package: "the Python API is compatible with the Scanpy33 package"
- [discussion] it supports h5ad-formatted HDF5 files: "it supports h5ad-formatted HDF5 files (https://github.com/theislab/anndata)"
- [readme] mean.tsv is the main output of the method which represents the mean parameter of the ZINB distribution: "`mean.tsv` is the main output of the method which represents the mean parameter of the ZINB distribution."
