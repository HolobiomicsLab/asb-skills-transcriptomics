---
name: r-vignette-reproduction
description: Use when a published R package ships a vignette, sample dataset, or benchmark
  result (e.g. an ARI on a named dataset) and the goal is to independently re-run
  that documented analysis and compare the output metrics against the reported values.
when_to_use_negative:
- The goal is to develop or benchmark a novel clustering method against scCAN rather
  than reproduce a documented vignette result
- The dataset has no published reference labels and no reported metric in the vignette,
  so there is nothing to validate the reproduction against
- Input is raw sequencing data (FASTQ/counts) needing the package's full upstream
  preprocessing — the vignette path expects the normalized matrix the authors deposited
  (e.g. readRDS of pollen.rds)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scCAN
  role: 'The R package whose vignette is reproduced: performs autoencoder feature
    selection, VAE dimensionality reduction, network fusion, spectral clustering,
    and provides adjustedRandIndex for evaluation'
  repo: https://cran.r-project.org/package=scCAN
- name: devtools
  role: Used to install the package from GitHub as the README prescribes (devtools::install_github('bangtran365/scCAN'))
  repo: https://cran.r-project.org/package=scCAN
- name: mclust
  role: Adjusted Rand index computation for comparing predicted clusters to published
    cell_type1 labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:f68192ccf5bf529e0106ec362a7ae4e488a15ce759bf6a402f345e8e2c2a5f5e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - devtools
  - mclust
  - scCAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# r-vignette-reproduction

## Summary

Reproduce an R package's documented vignette/benchmark analysis end-to-end — here, running scCAN's clustering pipeline on the Pollen dataset (301 cells) and verifying the published ARI (0.92) and runtime (~1.29 min) — to validate that a paper's reported results are computationally reproducible.

## When to use

Use when a published R package ships a vignette, sample dataset, or benchmark result (e.g. an ARI on a named dataset) and the goal is to independently re-run that documented analysis and compare the output metrics against the reported values.

## When NOT to use

- The goal is to develop or benchmark a novel clustering method against scCAN rather than reproduce a documented vignette result
- The dataset has no published reference labels and no reported metric in the vignette, so there is nothing to validate the reproduction against
- Input is raw sequencing data (FASTQ/counts) needing the package's full upstream preprocessing — the vignette path expects the normalized matrix the authors deposited (e.g. readRDS of pollen.rds)

## Inputs

- Documented R package vignette or README example (install command, sample code, bundled dataset name)
- Bundled or deposited dataset matrix, e.g. pollen.rds normalized expression matrix (301 cells x genes, rows = samples, columns = genes)
- Ground-truth cell labels (e.g. SCE$cell_type1 from the bundled dataset, or published cell_type1 labels)

## Outputs

- Per-cell cluster assignment vector (e.g. result$cluster)
- Summary metrics: adjusted Rand index (ARI), elapsed runtime in minutes, estimated number of clusters
- Results table written to e.g. pollen_sccan_results.csv containing per-cell assignments, ARI, runtime, and cluster count

## How to apply

Install the package as documented (e.g. `devtools::install_github('bangtran365/scCAN')` or from CRAN) and load its bundled data exactly as the vignette prescribes. Follow the vignette's stated preprocessing — for scCAN, log-transform if `max(data) > 100`, min-max rescale to the 0–1 range, keep the top 5000 informative genes via the non-negative-kernel autoencoder, and reduce to d = 15 latent dimensions with the stacked variational autoencoder. Run the clustering with documented defaults (network fusion with k = 30 neighbors, sigma = 0.5, eigengap scan i = 2..15, spectral clustering) and compute the evaluation metric with the package's own utility (e.g. `scCAN::adjustedRandIndex(cluster, label)` against the published `cell_type1` labels). Record per-cell assignments plus summary metrics (ARI, runtime, estimated cluster count) to a results file, and treat the run as validated when the recomputed ARI falls within tolerance (±0.05 of the reported 0.92 for Pollen) and runtime is on the order of the reported value (1.293936 minutes). Use a fixed random seed (e.g. `r.seed = 1` as in the vignette) so the run is reproducible.

## Related tools

- **scCAN** (The R package whose vignette is reproduced: performs autoencoder feature selection, VAE dimensionality reduction, network fusion, spectral clustering, and provides adjustedRandIndex for evaluation) — https://cran.r-project.org/package=scCAN
- **devtools** (Used to install the package from GitHub as the README prescribes (devtools::install_github('bangtran365/scCAN'))) — https://cran.r-project.org/package=scCAN
- **mclust** (Adjusted Rand index computation for comparing predicted clusters to published cell_type1 labels)

## Examples

```
library(scCAN); data('SCE'); data <- t(SCE$data); label <- as.character(SCE$cell_type1); if(max(data)>100) data <- log2(data+1); result <- scCAN(data, r.seed = 1); round(scCAN::adjustedRandIndex(result$cluster,label), 2)
```

## Evaluation signals

- Computed ARI matches the reported Pollen benchmark value of 0.92 within ±0.05
- Elapsed runtime is on the order of the reported 1.293936 minutes
- Estimated number of clusters (~9 for Pollen) is consistent with the published cell_type1 label count
- Script runs end-to-end without error and writes a results CSV with per-cell assignments plus ARI, runtime, and cluster count
- ARI is computed with the package's own utility (scCAN::adjustedRandIndex) against the documented cell_type1 labels, not a hand-rolled metric

## Limitations

- Autoencoder-based pipelines are stochastic; results vary run-to-run unless the documented seed (e.g. r.seed = 1) is fixed
- Runtime and ARI are dataset-size dependent — reported values (1.29 min, ARI 0.92) apply to the 301-cell Pollen dataset, not larger datasets
- The vignette expects the input matrix with rows as samples and columns as genes; transposing the matrix is a common failure mode
- The article's methods/discussion sections state no software version for scCAN or compared tools, so exact version-level reproducibility may not be attainable

## Evidence

- [readme] Install the package using: `devtools::install_github('bangtran365/scCAN')`: "Install the package using: `devtools::install_github('bangtran365/scCAN')`"
- [readme] Add log transformation if data max > 100: "Add log transformation: `if(max(data)>100) data <- log2(data + 1)`"
- [readme] Generate clustering result, rows as samples and columns as genes: "Generate clustering result, the input matrix has rows as samples and columns as genes: `result <- scCAN(data, r.seed = 1)`"
- [readme] Calculating adjusted Rand Index with scCAN utility: "Calculating adjusted Rand Index: `ari <- round(scCAN::adjustedRandIndex(cluster,label), 2)`"
- [methods] Eigengap method to determine number of clusters: "we use the eigengap method48 to determine the number of clusters"
- [methods] Data rescaling to 0 to 1 range: [evidence span withheld — anchor status: not_found]
- [abstract] scCAN package available at CRAN: "The scCAN package is available at https://cran.r-project.org/package=scCAN ."
