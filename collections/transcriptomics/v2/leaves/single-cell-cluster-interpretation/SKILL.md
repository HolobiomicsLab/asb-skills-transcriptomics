---
name: single-cell-cluster-interpretation
description: Use when a learned SIMLR similarity matrix (or single-cell scRNA-seq
  expression matrix with high noise/dropout) must be converted into 2-D/3-D low-dimensional
  coordinates and scatter plots that reveal subpopulations.
when_to_use_negative:
- Input is already a 2-D/3-D embedding or a feature table — no similarity learning
  or t-SNE stage is needed
- The goal is ranking important features for the learned similarities — use SIMLR_Feature_Ranking
  (SIMLR_Feature_Ranking.R/.m) instead
- Only the number of clusters is needed without visualization — use SIMLR_Estimate_Number_of_Clusters.R/.m
  rather than the t-SNE stage
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3361
tools:
- name: SIMLR
  role: Learns the cell-to-cell similarity matrix via multi-kernel optimization and
    graph diffusion that is fed downstream to t-SNE; also supplies 2-D/3-D visualizations
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: Runtime for the SIMLR.R implementation, t-SNE step, and demo scripts such
    as R_main_demo_SIMLR.R
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: MATLAB
  role: Alternative runtime for the SIMLR.m implementation and Matlab_main_demo_SIMLR.m
    on the 5 example datasets
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: grDevices
  role: Required R package to color the scatter plots of the resulting embedding
- name: igraph
  role: Required R package to compute the normalized mutual information metric when
    comparing clusters
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:5163bff5aceaa9b0a9953a7eaf89295489cc54c578dadb4a56fc6192ad658a45
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/BatzoglouLabSU/SIMLR
  tools:
  - MATLAB
  - Matlab
  - R
  - SIMLR
  - grDevices
  - igraph
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-cluster-interpretation

## Summary

Reconstruct SIMLR's dimension-reduction/visualization stage: feed the cell-to-cell similarity matrix learned by the multi-kernel optimization into a t-distributed stochastic neighbor embedding (t-SNE) step to place single-cell RNA-seq samples in a low-dimensional space, then render colored 2-D/3-D scatter plots for cluster interpretation.

## When to use

Use when a learned SIMLR similarity matrix (or single-cell scRNA-seq expression matrix with high noise/dropout) must be converted into 2-D/3-D low-dimensional coordinates and scatter plots that reveal subpopulations.

## When NOT to use

- Input is already a 2-D/3-D embedding or a feature table — no similarity learning or t-SNE stage is needed
- The goal is ranking important features for the learned similarities — use SIMLR_Feature_Ranking (SIMLR_Feature_Ranking.R/.m) instead
- Only the number of clusters is needed without visualization — use SIMLR_Estimate_Number_of_Clusters.R/.m rather than the t-SNE stage

## Inputs

- Single-cell RNA-seq expression matrix (e.g. Test_1_mECS.RData, Test_2_Kolod.RData, Test_3_Pollen.RData, Test_4_Usoskin.RData, Test_5_Zeisel.RData)
- Cell-to-cell similarity matrix produced by the SIMLR multi-kernel optimization (output of SIMLR.R or SIMLR.m)

## Outputs

- 2-D (and optionally 3-D) low-dimensional coordinates for every cell from the t-SNE embedding
- Colored scatter plot of the embedding to assist interpretation of cell populations
- simlr_tsne_embedding.csv containing the embedding coordinates

## How to apply

Run the SIMLR multi-kernel similarity-learning loop (scripts SIMLR.R / SIMLR.m, with graph diffusion to repair weak similarities from dropout) on the input expression matrix, e.g. the Test_1_mECS.RData example. Feed the resulting cell-to-cell similarity matrix into a t-SNE step: convert similarities into conditional probabilities and minimize the KL divergence to place each cell in 2-D (optionally 3-D) space. Extract the per-cell coordinates and render them as a colored scatter plot (the grDevices R package colors the plots; igraph is used to compute normalized mutual information for cluster agreement). Save the embedding coordinates (e.g. as simlr_tsne_embedding.csv). For large-scale datasets (e.g. Test_5_Zeisel, PBMC 10x Genomics GemCode), use SIMLR_Large_Scale.R/.m which additionally requires Rcpp, pracma, RcppAnnoy and RSpectra.

## Related tools

- **SIMLR** (Learns the cell-to-cell similarity matrix via multi-kernel optimization and graph diffusion that is fed downstream to t-SNE; also supplies 2-D/3-D visualizations) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (Runtime for the SIMLR.R implementation, t-SNE step, and demo scripts such as R_main_demo_SIMLR.R) — https://github.com/BatzoglouLabSU/SIMLR
- **MATLAB** (Alternative runtime for the SIMLR.m implementation and Matlab_main_demo_SIMLR.m on the 5 example datasets) — https://github.com/BatzoglouLabSU/SIMLR
- **grDevices** (Required R package to color the scatter plots of the resulting embedding)
- **igraph** (Required R package to compute the normalized mutual information metric when comparing clusters)

## Examples

```
R CMD SHLIB -c projsplx_R.c && Rscript R_main_demo_SIMLR.R
```

## Evaluation signals

- Known subpopulations in the input dataset (e.g. Test_3_Pollen, Test_4_Usoskin) are separated more accurately than with PCA or plain t-SNE baselines
- Every input cell has exactly one pair of 2-D coordinates in simlr_tsne_embedding.csv (row count matches the input cell count)
- Colored scatter plot renders without error using grDevices, with colors distinguishing the identified clusters
- Normalized mutual information computed with igraph between SIMLR clusters and known labels is at or above the benchmarked improvement reported for SIMLR over existing dimension-reduction methods
- The similarity matrix fed to t-SNE is the one produced by the SIMLR multi-kernel optimization, not a raw distance metric

## Limitations

- Example datasets shipped with the repo (Test_1_mECS.RData through Test_5_Zeisel.RData) are provided purely as examples after pre-processing and should not be used in place of the ones from the respective publications
- External C code must be compiled: if issues arise compiling projsplx_R.c, remove the pre-compiled files projsplx_R.o and projsplx_R.so
- Single-cell data noise, outliers and dropout events weaken conventional similarity measures; SIMLR relies on graph diffusion to improve weak similarities arising from noise or dropout

## Evidence
