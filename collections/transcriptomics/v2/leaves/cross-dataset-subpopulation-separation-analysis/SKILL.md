---
name: cross-dataset-subpopulation-separation-analysis
description: Use when comparing dimension-reduction/visualization methods for temporal
  scRNA-seq data on their ability to resolve subpopulations within the final time
  point, using dip-test multimodality scores across multiple datasets.
when_to_use_negative:
- The scRNA-seq data lack a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern (TCM's stated requirement)
- The goal is immediate biological interpretation of cell clusters rather than quantitative
  separation benchmarking — the article notes further pathway analysis is needed for
  cluster meaning
- Data are not temporal (no time-course structure); the last-time-point subpopulation
  separation metric is undefined without developmental stages/time points
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: tcm
  role: Method under benchmark; produces the 2D latent space (Topographic Cell Map)
    fitted via tcm(X, time.table = time.table)
  repo: github.com/gongx030/tcm
- name: scDatasets
  role: Provides the 11 temporal single-cell expression datasets and their time tables
    for the cross-dataset benchmark
- name: Rtsne
  role: 't-SNE baseline embedding: Rtsne(t(X.log))$Y on scale(log(X + 1)) transformed
    counts'
- name: diffusionMap
  role: 'Diffusion map baseline embedding: diffuse(dist(t(X.log)))$X[, c(1, 2)]'
- name: diptest
  role: Applies Hartigan's dip test to the last-time-point coordinate distributions
    to quantify multimodality
- name: Monocle
  role: Baseline trajectory/visualization method included in the six-method comparison
- name: TSCAN
  role: Baseline trajectory/visualization method included in the six-method comparison
- name: Wishbone
  role: Baseline trajectory/visualization method included in the six-method comparison
- name: devtools
  role: Installs the tcm package via install_github('gongx030/tcm')
  repo: r-lib/devtools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:88dd1fceb03f4bed3e8d2819a3f0b086f0e38b82d6e29086b1a6085c38d02a95
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cross-dataset-subpopulation-separation-analysis

## Summary

A benchmarking protocol that quantifies how well latent-space embeddings (TCM, t-SNE, diffusion map, diffusion pseudotime, Wishbone, Monocle, TSCAN) separate subpopulations of cells from the last time point across 11 temporal single-cell expression datasets, using Hartigan's dip statistics of the cells' distribution on the latent space.

## When to use

Use when comparing dimension-reduction/visualization methods for temporal scRNA-seq data on their ability to resolve subpopulations within the final time point, using dip-test multimodality scores across multiple datasets.

## When NOT to use

- The scRNA-seq data lack a complete time index, or the time index is not correlated with the underlying dynamic expression pattern (TCM's stated requirement)
- The goal is immediate biological interpretation of cell clusters rather than quantitative separation benchmarking — the article notes further pathway analysis is needed for cluster meaning
- Data are not temporal (no time-course structure); the last-time-point subpopulation separation metric is undefined without developmental stages/time points

## Inputs

- Temporal scRNA-seq gene-by-cell count matrices from 11 datasets (guo, guo2, loh, trapnell, iPSC, petropoulos, deng, plus sim.rnaseq.ts simulated data)
- Cell-by-time-point binary indicator matrices (time.table) marking each cell's timestamp
- Simulated temporal scRNA-seq control: sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5)

## Outputs

- A method-by-dataset matrix of Hartigan's dip statistics for the last-time-point cell distributions (dip_scores.csv)
- Per-dataset, per-method 2D latent coordinates of cells from the final time point
- Comparative ranking showing TCM with consistently significantly better subpopulation separation on all 11 datasets

## How to apply

Load the 11 temporal scRNA-seq count matrices and their time tables via the scDatasets package (guo, guo2, loh, trapnell, iPSC, petropoulos, deng, plus simulated data from sim.rnaseq.ts). Scale and log-transform raw counts as X.log <- scale(log(X + 1)) for the t-SNE and diffusion map baselines, and preprocess counts with preprocess(X) (optionally min.expressed.gene = 0) before TCM fitting. Fit each method to every dataset: tcm(X, time.table = time.table) for TCM, Rtsne(t(X.log))$Y for t-SNE, and diffuse(dist(t(X.log)))$X[, c(1, 2)] for diffusion map, along with diffusion pseudotime, Wishbone, Monocle and TSCAN on the same inputs. For each method-dataset pair, extract the 2D latent coordinates of cells belonging to the last time point using the time.table binary indicator matrix, then apply Hartigan's dip test (diptest package) to quantify subpopulation multimodality within that final time point. Tabulate the dip scores into a method-by-dataset matrix; the benchmark's expected outcome is that TCM attains the consistently, significantly highest dip score on all 11 datasets, which is then written as dip_scores.csv.

## Related tools

- **tcm** (Method under benchmark; produces the 2D latent space (Topographic Cell Map) fitted via tcm(X, time.table = time.table)) — github.com/gongx030/tcm
- **scDatasets** (Provides the 11 temporal single-cell expression datasets and their time tables for the cross-dataset benchmark)
- **Rtsne** (t-SNE baseline embedding: Rtsne(t(X.log))$Y on scale(log(X + 1)) transformed counts)
- **diffusionMap** (Diffusion map baseline embedding: diffuse(dist(t(X.log)))$X[, c(1, 2)])
- **diptest** (Applies Hartigan's dip test to the last-time-point coordinate distributions to quantify multimodality)
- **Monocle** (Baseline trajectory/visualization method included in the six-method comparison)
- **TSCAN** (Baseline trajectory/visualization method included in the six-method comparison)
- **Wishbone** (Baseline trajectory/visualization method included in the six-method comparison)
- **devtools** (Installs the tcm package via install_github('gongx030/tcm')) — r-lib/devtools

## Examples

```
library(tcm); library(scDatasets); data(guo); X <- preprocess(assays(guo)$count); mf <- tcm(X, time.table = colData(guo)$time.table)
```

## Evaluation signals

- The method-by-dataset dip-score matrix contains finite Hartigan's dip statistics for all 7 methods × 11 datasets, computed only on cells from the last time point
- TCM attains the highest dip score for each of the 11 datasets, consistent with the reported finding of consistently significantly better separation of last-time-point subpopulations
- The t-SNE and diffusion map baselines were run on identically preprocessed inputs, X.log <- scale(log(X + 1)), matching the article's comparison setup
- The final table is exported as dip_scores.csv with rows/columns aligning to methods and datasets
- On the simulated dataset (sim.rnaseq.ts, 5 lineages), both t-SNE and diffusion map separate lineages while TCM shows improved lineage trajectories, as a sanity check on embeddings

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- Subpopulations within each time point are usually indistinguishable on diffusion map and t-SNE latent spaces because minor expression differences are dominated by temporal differences — a known failure mode of the baselines, not a data problem
- The average percent of variance explained by non-temporal factors across the 11 datasets increases to 78.6% with TCM; embeddings with lower non-temporal variance indicate the crowding problem

## Evidence

- [methods] dip test on last time point: [evidence span withheld — anchor status: not_found]
- [methods] TCM fitting: "mf <- tcm(X, time.table = time.table)"
- [methods] t-SNE baseline: "y <- Rtsne(t(X.log))$Y"
- [methods] diffusion map baseline: "y <- diffuse(dist(t(X.log)))$X[, c(1, 2)]"
- [discussion] TCM requirement: "TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern."
- [discussion] crowding problem metric: "the average percent of variance explained by non-temporal factors on 11 examined temporal expression datasets increases to 78.6% by using TCM"
- [readme] simulated dataset behavior: "We find that for a relatively easy dataset, both t-SNE and diffusion map are able to separate three lineages.  However, the visualization produced by TCM show much improved lineage trajectories."
- [intro] baseline indistinguishability: "The subpopulations of cells within each time point are usually indistinguishable, due to minor expression differences compared with the more dominant temporal differences."
