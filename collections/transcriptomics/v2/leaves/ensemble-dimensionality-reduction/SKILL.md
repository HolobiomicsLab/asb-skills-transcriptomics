---
name: ensemble-dimensionality-reduction
description: Use when a cells-by-genes scRNA-seq count matrix needs simultaneous 2D
  embedding and feature gene extraction, especially to separate rare cell types or
  to benchmark against t-SNE and UMAP with silhouette indices.
when_to_use_negative:
- The input is already a reduced/normalized feature table or embedding rather than
  raw cells-by-genes counts — EDGE expects a count matrix it can median-scale and
  log2(x+1) transform itself.
- The analysis goal is purely embedding quality with no interest in feature gene identification
  — plain UMAP (which preserves local and global structures and is computationally
  efficient) may suffice, though it can miss rare cell types when dominant types exist.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: EDGE
  role: Core R package implementing the ensemble weak-learner similarity hashing,
    spectral embedding, SGD optimization, and entropy-based feature gene ranking via
    endr() and affinity()
  repo: github.com/shawnstat/EDGE
- name: RSpectra
  role: Performs the large-scale eigenvalue decomposition for the spectral embedding
    step
- name: umap
  role: Baseline embedding on the same log-normalized matrix with 15 nearest neighbors
    for silhouette-index comparison
- name: Rtsne
  role: Baseline t-SNE embedding (seeded at 2020) used to benchmark EDGE's structure
    preservation
- name: Rcpp
  role: C++ dependency (>= 1.0.1) required to build the EDGE package
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:d881dac0ef910df75735c6c46251f6198fb94be762570850d320447455905d16
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - R
  - RSpectra
  - Rcpp
  - Rtsne
  - Seurat
  - umap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# ensemble-dimensionality-reduction

## Summary

EDGE (Ensemble Dimensionality reduction and feature Gene Extraction) embeds scRNA-seq data into a 2D subspace via an ensemble of weak-learner similarity (modulo hashing) voters followed by spectral embedding and SGD optimization, while simultaneously ranking feature genes by averaged entropy contribution. It addresses t-SNE's loss of global geometry and t-SNE/UMAP's inability to identify feature genes.

## When to use

Use when a cells-by-genes scRNA-seq count matrix needs simultaneous 2D embedding and feature gene extraction, especially to separate rare cell types or to benchmark against t-SNE and UMAP with silhouette indices.

## When NOT to use

- The input is already a reduced/normalized feature table or embedding rather than raw cells-by-genes counts — EDGE expects a count matrix it can median-scale and log2(x+1) transform itself.
- The analysis goal is purely embedding quality with no interest in feature gene identification — plain UMAP (which preserves local and global structures and is computationally efficient) may suffice, though it can miss rare cell types when dominant types exist.

## Inputs

- Raw scRNA-seq cells-by-genes count matrix (e.g., 3005-cell mouse cortex/hippocampus matrix, Jurkat, PBMC)
- Cell-type label vector for coloring plots and silhouette computation
- endr_defs configuration object (n_wl, n_dm, n_neigs, opt, seed, H)
- Library-size vector (colSums of the count matrix) for median-scaling normalization

## Outputs

- 2D EDGE embedding matrix (simu.endr) of the cells
- Ranked feature gene list with averaged entropy contribution values, selected at μ − 1.5 × σ (e.g., 17/35/43 genes for Jurkat/PBMC/mouse brain)
- Per-method silhouette index averages over 100 replications (e.g., saved to silhouette_indices.csv)
- Side-by-side 2D scatter plots of EDGE, UMAP, and t-SNE embeddings colored by cell-type labels (e.g., fig4a_c_embeddings.png)

## How to apply

Scale each cell's library size to the median total and apply log2(x+1) to obtain a cells-by-genes log-normalized matrix (dat_log); restrict to the 1000 top variable genes for real datasets (500 for simulated data, selected with Seurat's FindVariableFeatures). Configure endr_defs with n_wl=1000 weak learners, n_dm=10 genes per weak learner, n_neigs=15 nearest neighbors, opt=TRUE, seed=2020, and hash table size H=101107 (the reduced, memory-friendly choice vs. the optimal 1,017,881), then run endr(dat_log, custom_defs) for the 2D EDGE embedding, optionally verifying with affinity(dat, n_wl, n_dm, nk, H, seed). Run baselines on the same matrix: umap(dat_log, random_state=2020) with 15 nearest neighbors and Rtsne(dat_log) with set.seed(2020). Internally EDGE constructs sketch vectors by randomly selecting B genes with a random gene-specific threshold, uses modulo hashing to score cell similarity, averages L weak-learner similarity matrices into S, builds a k-NNG, performs spectral embedding on the normalized Laplacian, and optimizes the embedding with a two-stage decreasing-step-size SGD. For feature genes, rank genes by averaged entropy contribution across weak learners and apply the μ − 1.5 × σ cutoff; compare recovered lists against Supplementary Data 1.

## Related tools

- **EDGE** (Core R package implementing the ensemble weak-learner similarity hashing, spectral embedding, SGD optimization, and entropy-based feature gene ranking via endr() and affinity()) — github.com/shawnstat/EDGE
- **RSpectra** (Performs the large-scale eigenvalue decomposition for the spectral embedding step)
- **umap** (Baseline embedding on the same log-normalized matrix with 15 nearest neighbors for silhouette-index comparison)
- **Rtsne** (Baseline t-SNE embedding (seeded at 2020) used to benchmark EDGE's structure preservation)
- **Rcpp** (C++ dependency (>= 1.0.1) required to build the EDGE package)

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$n_dm <- 10; custom_defs$n_neigs <- 15; custom_defs$seed <- 2020; custom_defs$H <- 101107; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- Silhouette index over 100 replications on the mouse brain dataset should reach ~0.621 (SD 0.085) for EDGE vs. 0.597 (0.025) for UMAP and 0.501 (0.017) for t-SNE, within Monte Carlo tolerance.
- Feature gene counts at the μ − 1.5 × σ cutoff must equal 17, 35, and 43 for the Jurkat, PBMC, and mouse brain datasets respectively, matching Supplementary Data 1.
- PBMC feature gene list must recover the marker genes PPBP (platelet), LYZ (CD14+ monocyte), CST3 (dendritic cells), and NKG7 (natural killer cells), with S100A9 having the highest importance score.
- The endr() result can be cross-verified with the affinity(dat, n_wl, n_dm, nk, H, seed) helper on the same inputs.
- Plots should show EDGE separating cell types, including rare populations, more compactly than t-SNE and UMAP (reproducing Figs. 4a-4c).

## Limitations

- Memory: the optimal hash table size is 1,017,881, but the example reduces it to 101,107; the README notes 'Most of the time, this works well and uses less memory,' implying reduced fidelity is possible.
- Values of hyperparameters a and b (nonlinear least squares estimation) and the number of negative samples in the SGD optimization are not reported in the main text; details are deferred to Supplementary Algorithms 1 and 2.
- The exact per-dataset value of B (genes per weak learner) is not stated, only the heuristic B < 0.05*G.
- No version numbers are given for R, Splatter, Seurat, or RSpectra, so exact reproducibility depends on package versions.

## Evidence

- [intro] EDGE proposal and dual purpose: "we propose an Ensemble Dimensionality reduction and feature Gene Extraction (EDGE) method to simultaneously perform dimensionality reduction and feature gene identification."
- [methods] endr invocation: "simu.endr <- endr(dat_log,custom_defs)"
- [readme] Hash table size trade-off: "The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example. Most of the time, this works well and uses less memory."
- [discussion] Feature gene cutoff rule: "We choose μ −  1.5 × σ as the cutoff value to select top feature genes"
- [discussion] Gene pre-filtering rule: "We selected 1000 top variable genes for real datasets and 500 for simulated datasets"
