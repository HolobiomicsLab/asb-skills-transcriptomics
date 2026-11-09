---
name: single-cell-embedding-visualization
description: Use when a scRNA-seq count matrix needs a 2D embedding and visualization
  to compare which method (EDGE vs. t-SNE vs. UMAP) preserves cell-type structure
  and separates rare populations.
when_to_use_negative:
- The goal is feature (differentially expressed) gene identification rather than 2D
  visualization — EDGE's feature-gene extraction step, not the embedding plots, is
  the relevant output
- The input is already a normalized/low-dimensional feature table rather than a raw
  count matrix requiring median library-size scaling and log2(x+1) transformation
- The analysis targets global hierarchical structure only and does not require comparison
  against t-SNE/UMAP baselines or rare cell-type detection
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EDGE
  role: 'Primary method: ensemble dimensionality reduction producing the 2D embedding
    via weak-learner similarity, spectral embedding, and SGD optimization'
  repo: github.com/shawnstat/EDGE
- name: UMAP
  role: Baseline embedding with 15 nearest neighbors and random_state=2020 for comparison
    of structure preservation
- name: Rtsne
  role: Baseline t-SNE embedding (Rtsne(dat_log), seed 2020) for comparison; known
    to preserve local but lose global structure
- name: RSpectra
  role: Performs the large-scale eigenvalue decomposition needed for EDGE's spectral
    embedding step
  repo: github.com/shawnstat/EDGE
- name: Rcpp
  role: C++ dependency (>= 1.0.1) required to install and run the EDGE package
  repo: github.com/shawnstat/EDGE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:95fe4aa02b3691ec75cf7682b5f9ee83d91cefc9d0cc73f72db7361259918957
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - RSpectra
  - Rcpp
  - Rtsne
  - UMAP
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# single-cell-embedding-visualization

## Summary

Embed log-normalized scRNA-seq count matrices into 2D subspaces using EDGE, UMAP, and t-SNE in R, with side-by-side scatter plots colored by cell-type labels to compare global/local structure preservation and rare cell-type separation.

## When to use

Use when a scRNA-seq count matrix needs a 2D embedding and visualization to compare which method (EDGE vs. t-SNE vs. UMAP) best preserves cell-type structure and separates rare populations.

## When NOT to use

- The goal is feature (differentially expressed) gene identification rather than 2D visualization — EDGE's feature-gene extraction step, not the embedding plots, is the relevant output
- The input is already a normalized/low-dimensional feature table rather than a raw count matrix requiring median library-size scaling and log2(x+1) transformation
- The analysis targets global hierarchical structure only and does not require comparison against t-SNE/UMAP baselines or rare cell-type detection

## Inputs

- Raw scRNA-seq cell-by-gene count matrix (e.g., 3005-cell mouse cortex/hippocampus matrix)
- Cell-type label vector for coloring points
- EDGE package parameter definitions (endr_defs with n_wl, n_dm, n_neigs, opt, seed, H)

## Outputs

- 2D embeddings from EDGE (simu.endr), UMAP (simu.umap$layout), and t-SNE (simu.tsne$Y)
- Side-by-side scatter plots of embeddings colored by cell-type labels (e.g., fig4a_c_embeddings.png)
- Per-method averaged silhouette indices over replications (e.g., silhouette_indices.csv)

## How to apply

Normalize raw counts by scaling each cell's library size to the median and applying log2(x+1), yielding the log-normalized matrix dat_log. Configure EDGE via endr_defs with n_wl=1000 weak learners, n_dm=10 genes per weak learner, n_neigs=15 nearest neighbors, opt=TRUE, seed=2020, and hash table size H=101107, then run endr(dat_log, custom_defs) to obtain the 2D embedding; EDGE builds a k-NNG from ensemble-averaged similarity scores, performs spectral embedding on the normalized Laplacian, and optimizes the embedding by stochastic gradient descent. Run baselines on the same matrix: umap(dat_log, random_state=2020) with 15 nearest neighbors and Rtsne(dat_log) with set.seed(2020). Plot all three embeddings side by side, colored by cell-type labels. Repeating over ~100 replications and averaging silhouette indices per cell type gives a quantitative comparison of structure preservation.

## Related tools

- **EDGE** (Primary method: ensemble dimensionality reduction producing the 2D embedding via weak-learner similarity, spectral embedding, and SGD optimization) — github.com/shawnstat/EDGE
- **UMAP** (Baseline embedding with 15 nearest neighbors and random_state=2020 for comparison of structure preservation)
- **Rtsne** (Baseline t-SNE embedding (Rtsne(dat_log), seed 2020) for comparison; known to preserve local but lose global structure)
- **RSpectra** (Performs the large-scale eigenvalue decomposition needed for EDGE's spectral embedding step) — github.com/shawnstat/EDGE
- **Rcpp** (C++ dependency (>= 1.0.1) required to install and run the EDGE package) — github.com/shawnstat/EDGE

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$n_neigs <- 15; custom_defs$H <- 101107; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- Averaged silhouette indices over 100 replications should match reported values (EDGE 0.621 SD 0.085, UMAP 0.597 SD 0.025, t-SNE 0.501 SD 0.017) within Monte Carlo tolerance
- Plots should show EDGE separating cell types, including rare populations, more compactly than t-SNE and UMAP
- EDGE mean silhouette index of all cell types should exceed both UMAP and t-SNE on the same log-normalized input
- EDGE run should succeed with the reduced hash table size H=101107 (the optimal 1,017,881 is optional and memory-heavy)
- All three embeddings are computed on the identical log-normalized matrix so the comparison is controlled

## Limitations

- Larger numbers of weak learners and hash table sizes improve results but require large memory; the reduced H=101107 works most of the time
- t-SNE may lose the global geometry/hierarchical structure of the data
- UMAP may be inefficient in identifying rare cell types when dominant/common cell types exist
- Exact per-dataset value of B (genes per weak learner heuristic, typically <0.05*G) is not stated in the main text; algorithm details are deferred to Supplementary Algorithms 1 and 2
- Values of hyperparameters a and b are not reported; only nonlinear least squares estimation via a reference is given

## Evidence

- [readme] EDGE parameter config and normalization: [evidence span withheld — anchor status: not_found]
- [readme] README baseline runs: "simu.umap <- umap(dat_log,random_state=2020)"
- [methods] Article install instructions: "install_github("shawnstat/EDGE")"
- [discussion] Spectral embedding description: "the spectral embedding is performed on the normalized Laplacian D1/2(D − S)D1/2"
- [discussion] Comparative compactness finding: "In all three real datasets, EDGE shows more compactly projected structures compared with t-SNE and UMAP"
- [discussion] Rare cell-type separation: "When there exist rare cell types, EDGE provides a neat separation between rare and common cell types"
