---
name: fitsne-visual-dimensionality-reduction
description: Use when you have high-dimensional per-cell feature vectors (e.g., Miscell
  latent features from scRNA-seq data) and need a fast, reproducible 2D t-SNE embedding
  for visualization and cluster extraction.
when_to_use_negative:
- If the goal is accurate global structure or trajectory inference rather than visualization/density
  clustering — t-SNE distances are not globally meaningful
- If batch effects are present, since Miscell/FIt-SNE do not internally batch-correct;
  correct batch effects before embedding
- If the dataset is very small, since Miscell (which produces the features) demands
  a large sample size for good performance
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: FIt-SNE
  role: FFT-accelerated interpolation-based t-SNE implementation used to compute the
    2D embedding of Miscell latent features
  repo: https://github.com/KlugerLab/FIt-SNE
- name: Miscell
  role: Self-supervised encoder that produces the latent feature matrix passed to
    FIt-SNE
  repo: https://github.com/lixiangchun/Miscell
- name: Scanpy
  role: Builds the neighborhood graph and plots the FIt-SNE embedding colored by Leiden
    clusters (sc.pl.tsne with save='_scanpy.pdf')
- name: scikit-learn
  role: sklearn.cluster.dbscan applied on the FIt-SNE embedding (eps=1, min_samples=5)
    to extract clusters
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:41c5c1b00dce0e6d951cdbca6b5a63655321d4906b793c3f325de8d66221dcc4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  repo_url: https://github.com/KlugerLab/FIt-SNE
  tools:
  - FIt-SNE
  - Miscell
  - Scanpy
  - Seurat
  - anndata
  - scVI
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# fitsne-visual-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Accelerates t-SNE embedding of single-cell latent features (e.g., Miscell-encoded representations) using FFT-accelerated interpolation-based t-SNE (FIt-SNE), producing a 2D embedding suitable for visualization and downstream density clustering.

## When to use

Use when you have high-dimensional per-cell feature vectors (e.g., Miscell latent features from scRNA-seq data) and need a fast, reproducible 2D t-SNE embedding for visualization and cluster extraction.

## When NOT to use

- If the goal is accurate global structure or trajectory inference rather than visualization/density clustering — t-SNE distances are not globally meaningful
- If batch effects are present, since Miscell/FIt-SNE do not internally batch-correct; correct batch effects before embedding
- If the dataset is very small, since Miscell (which produces the features) demands a large sample size for good performance

## Inputs

- Per-cell latent feature matrix extracted by the pretrained Miscell encoder (e.g., numpy array from checkpoint_0199.pth.tar applied to data/x_new_scanpy_HVG.npz)
- FIt-SNE installation with compiled bin/fast_tsne binary and FFTW library

## Outputs

- 2D t-SNE embedding (tsne array, one row per cell) usable for DBSCAN clustering and scanpy visualization
- t-SNE plot colored by Leiden clusters saved as tsne_scanpy.pdf

## How to apply

Extract latent features from the pretrained Miscell model (e.g., via utils.load_pretrained_model and utils.extract_features on the normalized HVG matrix such as x_new_scanpy_HVG.npz), then run FIt-SNE with seed=123, nthreads=12, and a multi-scale perplexity_list=[30, 36, 42, 48]; the multi-scale perplexity combination uncovers both local and global structure. Install FIt-SNE by compiling the C++ code with FFTW, append its path to sys.path, and import fast_tsne. The resulting 2D tsne embedding can be stored in adata.obsm['X_tsne'] for scanpy visualization (sc.pl.tsne colored by Leiden clusters) and fed to DBSCAN (eps=1, min_samples=5) for cluster assignment. Setting seed=123 makes the embedding reproducible across runs.

## Related tools

- **FIt-SNE** (FFT-accelerated interpolation-based t-SNE implementation used to compute the 2D embedding of Miscell latent features) — https://github.com/KlugerLab/FIt-SNE
- **Miscell** (Self-supervised encoder that produces the latent feature matrix passed to FIt-SNE) — https://github.com/lixiangchun/Miscell
- **Scanpy** (Builds the neighborhood graph and plots the FIt-SNE embedding colored by Leiden clusters (sc.pl.tsne with save='_scanpy.pdf'))
- **scikit-learn** (sklearn.cluster.dbscan applied on the FIt-SNE embedding (eps=1, min_samples=5) to extract clusters)

## Examples

```
tsne = fast_tsne(features, seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48]); adata.obsm['X_tsne'] = tsne
```

## Evaluation signals

- fast_tsne is called with seed=123, nthreads=12, and perplexity_list=[30, 36, 42, 48]; verify these parameters match
- The embedding has exactly 2 dimensions and one row per input cell (matching features.shape[0])
- DBSCAN on the embedding (eps=1, min_samples=5) yields cluster labels whose ARI/NMI/V-measure against FACS-sorted CD4+/CD8+ T cell labels are competitive with Scanpy, Seurat, and scVI baselines
- sc.pl.tsne on the AnnData with adata.obsm['X_tsne'] set produces the saved tsne_scanpy.pdf plot without errors
- FIt-SNE binary is compiled and reachable (sys.path.append to the FIt-SNE checkout) so fast_tsne imports cleanly

## Limitations

- FIt-SNE must be recompiled after git pull updates (FFTW dependency; users without sudo must install FFTW in the home directory and pass its path to g++)
- On Windows, only the R wrapper properly calls the executable; Python/Matlab wrappers must be edited to use FItSNE.exe
- FIt-SNE uses Annoy approximate nearest neighbors by default; use vantage-point trees when subtle detail or small clusters matter
- The embedding quality depends on Miscell latent features, which require a large sample size and no uncorrected batch effects

## Evidence

- [readme] We accelerated this implementation as follows: Computation of the N-body Simulation: Instead of approximating the N-body simulation using Barnes-Hut, we interpolate onto an equispaced grid and use FFT: [evidence span withheld — anchor status: external_doc]
- [readme] The perplexity parameter determines the width of the Gaussian kernel... using combination of several perplexity values, resulting in a multi-scale embedding, can be useful: [evidence span withheld — anchor status: external_doc]
