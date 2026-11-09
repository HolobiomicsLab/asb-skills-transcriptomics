---
name: rbf-gaussian-process-prior-specification
description: Use when fitting TCM to temporal single-cell RNA-seq data and you must
  configure the radial prototype landscape and the RBF-kernel Gaussian-process prior
  over prototype positions before variational EM fitting.
when_to_use_negative:
- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern (a stated requirement of TCM)
- No temporal/staged sampling exists — for non-temporal single-cell data a generic
  dimension-reduction method (t-SNE, diffusion map) is more appropriate
- You need immediate biological interpretation of cell clusters; the article notes
  pathway analysis must be conducted separately on TCM's latent space
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tcm
  role: R package implementing the TCM generative model (Poisson likelihood, Gamma
    priors, RBF-kernel GP prior) with variational EM fitting via tcm() and landscape
    configuration
  repo: https://github.com/gongx030/tcm
- name: R
  role: Runtime environment; TCM is provided as an R implementation installable from
    GitHub
  repo: https://github.com/gongx030/tcm
- name: devtools
  role: Used to install the tcm package via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:187506f96366f2e588de8a79de35ef1362423eb0fc4377fe02e39a1dbbf27a85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gongx030/tcm
  tools:
  - R
  - TCM
  - devtools
  - tcm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# rbf-gaussian-process-prior-specification

## Summary

Specifies the generative-model prior structure of the TCM (Topographic Cell Map) algorithm: prototype metagenes laid out on an R×S radial landscape grid whose latent positions are governed by an RBF-kernel Gaussian-process prior, coupled with Poisson read-count likelihoods and Gamma priors on rates, so temporal scRNA-seq cells are convolved onto one 2D latent space with early time points at the center and later ones at the periphery.

## When to use

Use when fitting TCM to temporal single-cell RNA-seq data and you must configure the radial prototype landscape and the RBF-kernel Gaussian-process prior over prototype positions before variational EM fitting.

## When NOT to use

- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated with the underlying dynamic expression pattern (a stated requirement of TCM)
- No temporal/staged sampling exists — for non-temporal single-cell data a generic dimension-reduction method (t-SNE, diffusion map) is more appropriate
- You need immediate biological interpretation of cell clusters; the article notes pathway analysis must be conducted separately on TCM's latent space

## Inputs

- Gene-by-cell read count matrix (e.g. 2,000 genes × 500 cells) from sim.rnaseq.ts or preprocessed real data
- Cell-by-time-point binary time.table matrix (e.g. 500 × 5) indicating each cell's timestamp
- Landscape specification object from landscape(type = 'temporal.convolving', ...) defining the R×S prototype grid

## Outputs

- Fitted TCM model object (mf) with 2D latent coordinates for all cells
- Prototype positions on the radial R×S grid per time-point latent space (exportable as prototype_layout.csv)
- Cell embedding in the single convolved 2D latent space (exportable as embedding.csv)

## How to apply

First construct the R×S radial prototype grid with the landscape() function, e.g. landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2), which places early developmental time points at the center and later ones at the periphery of each time-point-specific latent space. In the TCM generative model, read counts follow a Poisson likelihood driven by prototype metagenes, rates receive Gamma priors, and prototype positions receive an RBF-kernel Gaussian-process prior that smooths prototype locations so adjacent prototypes have correlated expression profiles. Fit the model with variational EM (variational Bayes / GTM-style responsibility updates) via the tcm(X, time.table = time.table, ls = ls) interface; for large datasets (>2,000 cells), use control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4). The rationale is that the GP prior over prototype positions combined with temporal convolving yields a single latent space preserving global developmental trajectories while retaining subpopulations within each time point.

## Related tools

- **tcm** (R package implementing the TCM generative model (Poisson likelihood, Gamma priors, RBF-kernel GP prior) with variational EM fitting via tcm() and landscape configuration) — https://github.com/gongx030/tcm
- **R** (Runtime environment; TCM is provided as an R implementation installable from GitHub) — https://github.com/gongx030/tcm
- **devtools** (Used to install the tcm package via install_github('gongx030/tcm')) — https://github.com/hadley/devtools

## Examples

```
ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2); mf <- tcm(X, time.table = time.table, ls = ls)
```

## Evaluation signals

- Each per-time-point latent space contains exactly H = R × S prototypes (e.g. 15 × 15 = 225 with the documented landscape settings)
- Mean time index increases monotonically with radial distance from the center: early time points sit near the center, later ones at the periphery
- Exported embedding.csv and prototype_layout.csv exist and parse as valid CSV files
- Qualitative check: on simulated data the TCM visualization shows much improved lineage trajectories compared with t-SNE and diffusion map
- Variational EM converges without error; for large data (10,000 cells) the stochastic optimization with batch.size = 2000 and mc.cores = 4 completes in roughly ~15 minutes on an Intel Xeon 2.6GHz machine

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- Fitting the full model on large datasets is computationally heavier and requires stochastic variational inference (~15 mins for 2,000 genes × 10,000 cells on 4 cores)
- Extension to spatial expression patterns and other-omics data is proposed but not implemented

## Evidence

- [methods] landscape call specifying temporal convolving grid: "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [methods] TCM model fitting interface: "mf <- tcm(X, time.table = time.table)"
- [readme] stochastic variational inference settings for large data: "mf <- tcm(X, time.table = time.table, control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4))"
- [readme] TCM purpose on temporal scRNA-seq: "TCM is an R package for visualizing temporal single cell RNA-seq data."
- [abstract] goal of the convolved latent space: "Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point"
- [methods] simulated-data benchmark vs t-SNE/diffusion map: "We find that for a relatively easy dataset, both t-SNE and diffusion map are able to separate three lineages. However, the visualization produced by TCM show much improved lineage trajectories."
