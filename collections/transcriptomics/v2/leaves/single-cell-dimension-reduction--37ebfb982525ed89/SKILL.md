---
name: single-cell-dimension-reduction
description: Use when a learned cell-to-cell similarity matrix from SIMLR's multi-kernel
  optimization is available and a low-dimensional embedding is needed to visualize
  and interpret cell populations in scRNA-seq data.
when_to_use_negative:
- Input data has not yet been processed by SIMLR's multi-kernel similarity learning
  — the t-SNE stage consumes the learned similarity matrix, not raw expression data.
- The task is feature/gene ranking rather than visualization — use SIMLR_Feature_Ranking
  instead.
- The dataset is too large for standard SIMLR — use SIMLR_Large_Scale for large scale
  datasets.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SIMLR
  role: learns the multi-kernel cell-to-cell similarity measure and provides the dimension
    reduction / visualization stage consumed by this skill
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: implementation language for SIMLR.R, the similarity learning and t-SNE embedding
    workflow
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: Matlab
  role: alternative implementation (SIMLR.m) of the same similarity learning and embedding
    stage
- name: grDevices
  role: R package used to color the plots of the resulting embedding
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:b2f794d54ed21a9ba625d8767038fcc5bb557fed144744a2ce0e587fc0005c80
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/BatzoglouLabSU/SIMLR
  tools:
  - Matlab
  - R
  - SIMLR
  - SIMLR_Large_Scale
  - grDevices
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-dimension-reduction

## Summary

Applies SIMLR's stochastic neighbor embedding stage to transform the learned multi-kernel cell-to-cell similarity matrix into low-dimensional (2-D/3-D) coordinates for visualizing single-cell RNA-seq populations.

## When to use

Use when a learned cell-to-cell similarity matrix from SIMLR's multi-kernel optimization is available and a low-dimensional embedding is needed to visualize and interpret cell populations in scRNA-seq data.

## When NOT to use

- Input data has not yet been processed by SIMLR's multi-kernel similarity learning — the t-SNE stage consumes the learned similarity matrix, not raw expression data.
- The task is feature/gene ranking rather than visualization — use SIMLR_Feature_Ranking instead.
- The dataset is too large for standard SIMLR — use SIMLR_Large_Scale for large scale datasets.

## Inputs

- single-cell expression dataset (e.g., Test_1_mECS.RData from the package data directory)
- learned SIMLR cell-to-cell similarity matrix

## Outputs

- 2-D (and optionally 3-D) embedding coordinates for every cell
- colored scatter plot of the embedding
- simlr_tsne_embedding.csv (saved embedding coordinates)

## How to apply

Run the SIMLR multi-kernel similarity-learning loop (e.g., on Test_1_mECS.RData) to obtain the cell-to-cell similarity matrix, then feed that matrix into a t-distributed stochastic neighbor embedding (t-SNE) step that converts similarities into conditional probabilities and minimizes KL divergence to place cells in a low-dimensional space. Extract the resulting 2-D (and optionally 3-D) coordinates for every cell. SIMLR provides 2-D and 3-D visualizations specifically to assist interpretation of single-cell data, and the tool is unsupervised, allowing de novo discovery rather than pre-selecting gene subsets. Render the embedding as a colored scatter plot (e.g., with the grDevices package) and save the coordinates (e.g., simlr_tsne_embedding.csv).

## Related tools

- **SIMLR** (learns the multi-kernel cell-to-cell similarity measure and provides the dimension reduction / visualization stage consumed by this skill) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (implementation language for SIMLR.R, the similarity learning and t-SNE embedding workflow) — https://github.com/BatzoglouLabSU/SIMLR
- **Matlab** (alternative implementation (SIMLR.m) of the same similarity learning and embedding stage)
- **grDevices** (R package used to color the plots of the resulting embedding)

## Examples

```
source("https://bioconductor.org/biocLite.R"); biocLite("SIMLR"); results = SIMLR(X = Test_1_mECS$in_X, c = 3); plot(results$ydata, col = Test_1_mECS$true_labs[,1])
```

## Evaluation signals

- The embedding separates known subpopulations of the example dataset more accurately than existing dimension reduction methods such as PCA and t-SNE applied directly to the data.
- Every cell in the input similarity matrix receives a 2-D (and optionally 3-D) coordinate pair in the output.
- The KL divergence between the input similarities (as conditional probabilities) and the embedding's neighbor distribution is minimized by the t-SNE step.
- The colored scatter plot renders distinct cell populations interpretable in 2-D space.
- Coordinates are saved in the expected output file (simlr_tsne_embedding.csv) with one row per cell.

## Limitations

- Example datasets in the repository are provided purely as examples and after some pre-processing; they should not be used in place of the ones provided in the respective publications.
- SIMLR's similarity learning addresses high levels of dropout events and noise, but the t-SNE visualization stage depends on the quality of the learned similarity measure.
- For large scale datasets, the standard implementation may be impractical; use SIMLR_Large_Scale with its additional Rcpp, pracma, RcppAnnoy and RSpectra dependencies.

## Evidence
