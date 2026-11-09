---
name: latent-space-structural-validation
description: Use when a fitted TCM model (from tcm(X, time.table)) must be verified
  against a simulated or real temporal scRNA-seq dataset before exporting or interpreting
  its 2D latent space and prototype layout.
when_to_use_negative:
- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern (TCM's stated prerequisite)
- The goal is direct biological interpretation of cell clusters or trajectories —
  TCM output requires further pathway analysis and does not provide it immediately
- Input is a single time point or an atemporal population with no time.table, since
  the temporal convolving landscape cannot be constructed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: tcm
  role: Fits the prototype-based Bayesian generative model (Poisson likelihood, Gamma
    priors, RBF-kernel GP prior over prototype positions) by variational EM; produces
    the model object whose latent space is validated
  repo: https://github.com/gongx030/tcm
- name: TCM
  role: The Topographic Cell Map method whose temporal-convolving latent space (early
    stages central, late stages peripheral) is the structure being validated
  repo: https://github.com/gongx030/tcm
- name: R
  role: Runtime environment (R >= 3.4.3) in which the tcm model, landscape grid, CSV
    exports, and structural checks are executed
- name: SummarizedExperiment
  role: Provides assays(sim)$count and colData(sim)$time.table accessors used to assemble
    the count matrix and time.table inputs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:6709904a31e7ba085fd90e36dd0e2ca757508cb191fe918c0102daa1c0fe5ca6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gongx030/tcm
  tools:
  - R
  - SummarizedExperiment
  - TCM
  - tcm
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# latent-space-structural-validation

## Summary

Structurally validates the TCM (Topographic Cell Map) temporal latent space: each time-point-specific subspace must contain the full H = R×S prototype grid, and the convolved global space must place early developmental stages near the center and later stages at the periphery. This checks that the prototype-based generative model reconstructed the expected radial temporal topology before the embedding is trusted.

## When to use

Use when a fitted TCM model (from tcm(X, time.table)) must be verified against a simulated or real temporal scRNA-seq dataset before exporting or interpreting its 2D latent space and prototype layout.

## When NOT to use

- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated with the underlying dynamic expression pattern (TCM's stated prerequisite)
- The goal is direct biological interpretation of cell clusters or trajectories — TCM output requires further pathway analysis and does not provide it immediately
- Input is a single time point or an atemporal population with no time.table, since the temporal convolving landscape cannot be constructed

## Inputs

- Gene-by-cell read count matrix X (e.g. 2000 genes x 500 cells from sim.rnaseq.ts or assays(sim)$count)
- Cell-by-time-point binary matrix time.table (e.g. 500 x 5) timestamping each cell
- Fitted tcm model object (variational EM output), optionally built with an explicit landscape() prototype grid
- Prototype layout parameters R x S (K = 15 x n.prototype = 15, n.circle = 10, n.prev = 2)

## Outputs

- embedding.csv: 2D latent-space coordinates per cell
- prototype_layout.csv: positions of the H = R×S radial prototypes
- Validation report: per-time-point prototype counts equal to H and monotonic mean-time-index vs radial distance check
- Comparative plots of TCM vs t-SNE (Rtsne) and diffusion map (diffuse(dist(t(X.log)))) embeddings colored by lineage

## How to apply

Fit the prototype-based Bayesian model on a temporal scRNA-seq count matrix X and binary cell-by-time-point matrix time.table via variational EM (mf <- tcm(X, time.table = time.table)), optionally supplying an explicit radial grid with landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2). Then run structural checks: (1) export cell coordinates to embedding.csv and prototype positions to prototype_layout.csv and confirm both parse as valid CSV; (2) for each time point, verify the sub-space contains exactly H = R×S prototypes (e.g. 15×15 = 225 with the default landscape); (3) compute the mean time index of cells/prototypes as a function of radial distance from the center and confirm it increases monotonically, reflecting the temporal convolving that places early time points centrally and later ones peripherally. As a scientific criterion, qualitatively compare the TCM embedding against t-SNE and diffusion map on the same data (scale(log(X + 1)) inputs) — TCM should show much improved lineage trajectories even when the alternatives separate lineages.

## Related tools

- **tcm** (Fits the prototype-based Bayesian generative model (Poisson likelihood, Gamma priors, RBF-kernel GP prior over prototype positions) by variational EM; produces the model object whose latent space is validated) — https://github.com/gongx030/tcm
- **TCM** (The Topographic Cell Map method whose temporal-convolving latent space (early stages central, late stages peripheral) is the structure being validated) — https://github.com/gongx030/tcm
- **R** (Runtime environment (R >= 3.4.3) in which the tcm model, landscape grid, CSV exports, and structural checks are executed)
- **SummarizedExperiment** (Provides assays(sim)$count and colData(sim)$time.table accessors used to assemble the count matrix and time.table inputs)

## Examples

```
set.seed(144); sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5); mf <- tcm(X, time.table = time.table); plot(mf, pch = 21, bg = bg.cell)
```

## Evaluation signals

- embedding.csv and prototype_layout.csv both exist and parse as valid CSV
- Each per-time-point latent space contains exactly H = R×S prototypes (e.g. 225 with K = 15, n.prototype = 15)
- Mean time index increases monotonically with radial distance from the center of the convolved latent space (early near center, late at periphery)
- Qualitative agreement with the published simulation result: TCM shows much improved lineage trajectories versus t-SNE and diffusion map on the same data
- On real temporal datasets, variance explained by non-temporal factors should be high — the paper reports TCM raises it to 78.6% on average across 11 temporal expression datasets

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- The radial-ordering check is a structural invariant of the prototype grid, not proof of biological accuracy; scientific acceptance additionally relies on qualitative comparison with t-SNE/diffusion map trajectory visualizations

## Evidence

- [methods] landscape grid definition: "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [methods] TCM fitting interface: "mf <- tcm(X, time.table = time.table)"
- [readme] simulated benchmark dataset used for validation: "sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5)"
- [readme] scientific acceptance criterion vs baseline visualizations: "We find that for a relatively easy dataset, both t-SNE and diffusion map are able to separate three lineages.  However, the visualization produced by TCM show much improved lineage trajectories."
- [abstract] expected global structure of the latent space: "Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point"
