---
name: trajectory-inference-method-comparison
description: Use when comparing dimension-reduction/trajectory methods on temporal
  scRNA-seq data where the goal is to quantify subpopulation separation within each
  time point, e.g. deciding whether TCM better preserves subpopulations than t-SNE
  or diffusion map.
when_to_use_negative:
- Datasets lack a complete time index, or the time index is not correlated with the
  underlying dynamic expression pattern (TCM's stated requirement)
- The goal is immediate biological interpretation of clusters on the latent space
  without further pathway analysis, which TCM does not provide
- Input is a single non-temporal scRNA-seq sample with no time.table, since the benchmark
  depends on time-point indicator matrices
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: tcm
  role: TCM method implementation (tcm(X, time.table)) for the trajectory visualization
    being benchmarked
  repo: github.com/gongx030/tcm
- name: scDatasets
  role: Provides the temporal scRNA-seq datasets and time.tables loaded with data(guo),
    data(guo2), data(loh), data(trapnell), etc.
- name: Rtsne
  role: 't-SNE baseline dimension reduction: Rtsne(t(X.log))$Y'
- name: diffusionMap
  role: 'Diffusion map baseline: diffuse(dist(t(X.log)))$X[, c(1, 2)]'
- name: Monocle
  role: Comparison trajectory method applied to the same inputs
- name: TSCAN
  role: Comparison trajectory method applied to the same inputs
- name: Wishbone
  role: Comparison trajectory method applied to the same inputs
- name: diptest
  role: Hartigan's dip test used to quantify subpopulation multimodality within the
    final time point
- name: devtools
  role: Used to install tcm via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:882b3e0093466d39ba8f1d4b43abc695673c65ac14df76cd11ef4b04f5b30541
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle
  - R
  - Rtsne
  - TSCAN
  - Wishbone
  - devtools
  - diffusionMap
  - diptest
  - scDatasets
  - tcm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# trajectory-inference-method-comparison

## Summary

Benchmark dimension-reduction and trajectory-visualization methods (TCM, t-SNE, diffusion map, diffusion pseudotime, Wishbone, Monocle, TSCAN) on temporal single-cell expression datasets by quantifying how well each separates subpopulations of cells from the last time point using Hartigan's dip statistics of the cells' latent-space distribution.

## When to use

Use when comparing dimension-reduction/trajectory methods on temporal scRNA-seq data where the goal is to quantify subpopulation separation within each time point, e.g. deciding whether TCM better preserves subpopulations than t-SNE or diffusion map.

## When NOT to use

- Datasets lack a complete time index, or the time index is not correlated with the underlying dynamic expression pattern (TCM's stated requirement)
- The goal is immediate biological interpretation of clusters on the latent space without further pathway analysis, which TCM does not provide
- Input is a single non-temporal scRNA-seq sample with no time.table, since the benchmark depends on time-point indicator matrices

## Inputs

- Temporal scRNA-seq gene-by-cell raw count matrices (e.g. assays(sim)$count, 2000 genes x 500 cells for the simulation)
- Cell-by-time-point binary indicator matrices (time.table from colData, e.g. 500 cells x 5 time points)
- Eleven datasets from the scDatasets package: guo, guo2, loh, trapnell, iPSC, petropoulos, deng, plus sim.rnaseq.ts simulated temporal data

## Outputs

- 2D latent-space coordinates of cells for each method-dataset pair
- Method-by-dataset matrix of Hartigan's dip statistics for last-time-point subpopulation separation (e.g. dip_scores.csv)
- Visualization plots of each method's latent space colored by time point or lineage

## How to apply

Load the temporal count matrices and binary time.tables (cell x time-point indicator matrices) for the 11 datasets (guo, guo2, loh, trapnell, iPSC, petropoulos, deng, plus sim.rnaseq.ts simulations) via the scDatasets package. Preprocess counts with X <- preprocess(X), and for t-SNE and diffusion map baselines scale and log-transform with X.log <- scale(log(X + 1)). Fit each method: tcm(X, time.table = time.table), Rtsne(t(X.log))$Y, and diffuse(dist(t(X.log)))$X[, c(1, 2)], plus diffusion pseudotime, Wishbone, Monocle and TSCAN on the same inputs. For each method-dataset pair, extract the 2D latent coordinates of cells from the last time point using the time.table indicator matrix, then apply Hartigan's dip test (diptest package) to each coordinate distribution to quantify multimodality within that final time point. Tabulate the dip scores into a method-by-dataset matrix (e.g. dip_scores.csv); TCM should attain consistently higher dip scores across all 11 datasets, as reported in the article.

## Related tools

- **tcm** (TCM method implementation (tcm(X, time.table)) for the trajectory visualization being benchmarked) — github.com/gongx030/tcm
- **scDatasets** (Provides the temporal scRNA-seq datasets and time.tables loaded with data(guo), data(guo2), data(loh), data(trapnell), etc.)
- **Rtsne** (t-SNE baseline dimension reduction: Rtsne(t(X.log))$Y)
- **diffusionMap** (Diffusion map baseline: diffuse(dist(t(X.log)))$X[, c(1, 2)])
- **Monocle** (Comparison trajectory method applied to the same inputs)
- **TSCAN** (Comparison trajectory method applied to the same inputs)
- **Wishbone** (Comparison trajectory method applied to the same inputs)
- **diptest** (Hartigan's dip test used to quantify subpopulation multimodality within the final time point)
- **devtools** (Used to install tcm via install_github('gongx030/tcm')) — https://github.com/hadley/devtools

## Examples

```
library(scDatasets); data(guo); X <- preprocess(assays(guo)$count); mf <- tcm(X, time.table = colData(guo)$time.table); X.log <- scale(log(X + 1)); y <- Rtsne(t(X.log))$Y
```

## Evaluation signals

- TCM attains the highest Hartigan's dip score on all 11 datasets for last-time-point subpopulation separation
- Each dip score is computed only on cells whose time.table row indicates the final time point (verified via the binary indicator matrix)
- Baseline runs use scale(log(X + 1)) transformed inputs for t-SNE and diffusion map, matching the article's preprocessing
- The dip_scores.csv table has one row per method and one column per dataset with numeric dip statistics
- For the easy simulated dataset, both t-SNE and diffusion map separate the lineages, so they serve as a sanity check on the pipeline

## Limitations

- TCM requires scRNA-seq datasets with a complete time index that is correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters; further pathway analysis is needed
- Subpopulations within each time point are usually indistinguishable on diffusion map and t-SNE latent spaces because minor expression differences are dominated by temporal differences, biasing visual comparison
- Large-scale data (e.g. 10,000 cells) requires stochastic variational inference with batching (batch.size = 2000, mc.cores = 4) and takes ~15 mins on an Intel Xeon 2.6GHz computer

## Evidence

- [results] TCM has consistently significantly better performance in separating subpopulations from the final or last time point: [evidence span withheld — anchor status: not_found]
- [readme] Scale and log transform baselines: "We first scale and log-transform the raw read counts data.
r
X.log <- scale(log(X + 1))"
- [methods] tcm call: "mf <- tcm(X, time.table = time.table)"
- [methods] t-SNE baseline: "y <- Rtsne(t(X.log))$Y"
- [methods] diffusion map baseline: "y <- diffuse(dist(t(X.log)))$X[, c(1, 2)]"
- [readme] simulated data generation: "sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5)"
- [discussion] crowding reduction finding: "the average percent of variance explained by non-temporal factors on 11 examined temporal expression datasets increases to 78.6% by using TCM"
- [discussion] TCM limitation on time index: "TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern."
