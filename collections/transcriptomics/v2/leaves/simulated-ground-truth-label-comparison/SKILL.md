---
name: simulated-ground-truth-label-comparison
description: Use when a clustering or cell-identification method must be benchmarked
  on simulated scRNA-seq data where the true cell-type assignments are known by construction,
  e.g. reproducing scDHA's ARI 0.949 result on six simulated cell types.
when_to_use_negative:
- Input is real experimental data without ground-truth cell-type labels — the ARI
  comparison requires simulated labels known by construction
- The goal is simulating omics layers outside scRNA-seq/scATAC-seq, since sc_mosim
  single-cell simulation is currently restricted to those two techniques
- The method under test needs spatial resolution or proteomics/metabolomics context,
  which MOSim cannot simulate
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: Generates the simulated scRNA-seq count data and ground-truth cell-type labels
    via the sc_mosim function that the clustering method is validated against
  repo: https://github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment; MOSim is a Bioconductor R package (installed via
    BiocManager) used to run simulation and downstream comparison scripts
  repo: https://github.com/ConesaLab/MOSim
- name: scDHA
  role: 'VAE-based deep learning tool under test: trains a variational autoencoder
    for dimensionality reduction and clusters the simulated data for ARI comparison'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:b1ae5d6d2e90c25be2b407ee73ff823875d72d0bc753c414f3261807d9c05c11
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  repo_url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  - scDHA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# simulated-ground-truth-label-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate a single-cell clustering or annotation method by applying it to scMOSim (sc_mosim)-simulated scRNA-seq count data, where the simulated cell-type labels serve as ground truth, and quantify agreement with a metric such as the Adjusted Rand Index (ARI).

## When to use

Use when a clustering or cell-identification method must be benchmarked on simulated scRNA-seq data where the true cell-type assignments are known by construction, e.g. reproducing scDHA's ARI 0.949 result on six simulated cell types.

## When NOT to use

- Input is real experimental data without ground-truth cell-type labels — the ARI comparison requires simulated labels known by construction
- The goal is simulating omics layers outside scRNA-seq/scATAC-seq, since sc_mosim single-cell simulation is currently restricted to those two techniques
- The method under test needs spatial resolution or proteomics/metabolomics context, which MOSim cannot simulate

## Inputs

- scMOSim-simulated scRNA-seq count matrix (one replicate from a single experimental group)
- Simulated ground-truth cell-type labels for each cell, as produced by sc_mosim
- Predicted cluster assignments from the method under test (e.g. scDHA clustering)

## Outputs

- Per-cell table mapping predicted cluster assignments to simulated ground-truth cell types (e.g. scdha_clustering_results.csv)
- Adjusted Rand Index (ARI) quantifying clustering agreement with the simulated labels

## How to apply

Load one replicate from a single experimental group of the scMOSim-simulated scRNA-seq count matrix together with its simulated ground-truth cell-type labels (produced via the sc_mosim function). Preprocess the counts with a log(x+1) transformation (and any tool-specific normalization, e.g. scDHA's nonlinear kernel normalization) before feeding the data to the clustering method, running the method with the cluster count set to the number of simulated cell types (here six). Map predicted clusters onto the simulated ground-truth labels — in the scDHA benchmark five of six cell types were recovered, with cDC and Treg merged into a single cluster — and compute the Adjusted Rand Index between predicted and true assignments. Treat synthetic ground truth as the defining advantage of this setup: it supplies true positive and negative feature sets for rigorous benchmarking that real data cannot provide.

## Related tools

- **MOSim** (Generates the simulated scRNA-seq count data and ground-truth cell-type labels via the sc_mosim function that the clustering method is validated against) — https://github.com/ConesaLab/MOSim
- **R** (Implementation environment; MOSim is a Bioconductor R package (installed via BiocManager) used to run simulation and downstream comparison scripts) — https://github.com/ConesaLab/MOSim
- **scDHA** (VAE-based deep learning tool under test: trains a variational autoencoder for dimensionality reduction and clusters the simulated data for ARI comparison)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("MOSim")
```

## Evaluation signals

- Reproduced ARI is approximately 0.949 against the simulated ground-truth labels
- Exactly five predicted clusters account for the six simulated cell types, with cDC and Treg merged into one cluster
- Per-cell assignment file (e.g. scdha_clustering_results.csv) contains a one-to-one row per simulated cell with both predicted cluster and ground-truth label columns
- Count data received the log(x+1) transformation (plus scDHA's nonlinear kernel normalization) before VAE training, matching the documented preprocessing
- The number of clusters requested of the clustering module equals the number of simulated cell types (six)

## Limitations

- MOSim single-cell simulation is restricted to scRNA-seq and scATAC-seq only, so this ground-truth comparison cannot be extended to other single-cell omics layers
- Closely related cell types (here cDC and Treg) may be merged by the clustering method, so perfect label recovery is not guaranteed even on simulated data
- MOSim is not prepared to simulate GRNs with spatial resolution and does not cover proteomics or metabolomics layers, limiting the scope of benchmarkable hypotheses

## Evidence

- [methods] Log transformation applied to data before downstream analysis: [evidence span withheld — anchor status: not_found]
