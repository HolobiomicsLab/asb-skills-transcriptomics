---
name: scdha-vae-dimensionality-reduction
description: Use when benchmarking a single-cell clustering or dimensionality reduction
  method on synthetic scRNA-seq count data for which simulated ground-truth cell-type
  labels exist (e.g. scMOSim output), so clustering accuracy can be scored by ARI.
when_to_use_negative:
- The data are not single-cell transcriptomics counts — MOSim's single-cell simulation
  covers only scRNA-seq and scATAC-seq, and the paper notes limited applicability
  to proteomics/metabolomics or spatial GRN data
- There are no ground-truth cell-type labels, so ARI cannot be computed and the benchmark
  cannot be scored
- The counts have already been dimensionality-reduced or are a normalized feature/embedding
  table rather than the raw simulated count matrix
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDHA
  role: VAE-based deep learning tool performing nonlinear kernel normalization, dimensionality
    reduction, and clustering of the scRNA-seq count matrix
- name: MOSim
  role: 'Upstream simulator: its sc_mosim function generates the simulated scRNA-seq
    count matrix and ground-truth cell-type labels used as benchmark input'
  repo: github.com/ConesaLab/MOSim
- name: R
  role: 'Execution environment: MOSim is a Bioconductor R package (installable via
    BiocManager) and the benchmark is run in R'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:01c99d53cf807e6a68713bed9552486fca8d460dfc5a2d7ab233f79d4461634a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  - scDHA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# scdha-vae-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply scDHA — a variational autoencoder with nonlinear kernel normalization and a clustering module — to scRNA-seq count matrices with simulated ground-truth cell-type labels (e.g. produced by scMOSim) to evaluate deep-learning dimensionality reduction and clustering. Its performance is quantified by Adjusted Rand Index against the known labels (≈0.949 in the reference benchmark, recovering 5 of 6 simulated cell types).

## When to use

Use when benchmarking a single-cell clustering or dimensionality reduction method on synthetic scRNA-seq count data for which simulated ground-truth cell-type labels exist (e.g. scMOSim output), so clustering accuracy can be scored by ARI.

## When NOT to use

- The data are not single-cell transcriptomics counts — MOSim's single-cell simulation covers only scRNA-seq and scATAC-seq, and the paper notes limited applicability to proteomics/metabolomics or spatial GRN data
- There are no ground-truth cell-type labels, so ARI cannot be computed and the benchmark cannot be scored
- The counts have already been dimensionality-reduced or are a normalized feature/embedding table rather than the raw simulated count matrix

## Inputs

- scRNA-seq simulated count matrix (one replicate, single experimental group, e.g. from sc_mosim / MOSim simulation)
- Simulated ground-truth cell-type labels per cell (six cell types in the reference benchmark, including cDC and Treg)
- Number of simulated cell types (used as scDHA's cluster count parameter)

## Outputs

- Per-cell cluster assignments (predicted cluster for every cell)
- Adjusted Rand Index (ARI) between predicted clusters and ground-truth labels, target ≈ 0.949
- CSV file scdha_clustering_results.csv containing assignments and the ARI metric

## How to apply

Load one replicate from a single experimental group of the simulated scRNA-seq count matrix together with its simulated ground-truth cell-type labels (produced upstream by sc_mosim). Apply the log(x+1) transformation to the counts, then scDHA's nonlinear kernel normalization, to prepare input for the variational autoencoder — the same log transformation logic MOSim's own methods use to preprocess data. Train the scDHA VAE for dimensionality reduction and run its clustering module with the number of clusters set to the number of simulated cell types (six in the reference benchmark). Map predicted clusters onto the ground-truth labels and compute the Adjusted Rand Index to quantify agreement; expect closely related simulated populations (cDC and Treg) to merge into a single cluster. Write per-cell cluster assignments plus the ARI to scdha_clustering_results.csv.

## Related tools

- **scDHA** (VAE-based deep learning tool performing nonlinear kernel normalization, dimensionality reduction, and clustering of the scRNA-seq count matrix)
- **MOSim** (Upstream simulator: its sc_mosim function generates the simulated scRNA-seq count matrix and ground-truth cell-type labels used as benchmark input) — github.com/ConesaLab/MOSim
- **R** (Execution environment: MOSim is a Bioconductor R package (installable via BiocManager) and the benchmark is run in R)

## Examples

```
if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager"); BiocManager::install("MOSim")
```

## Evaluation signals

- Reproduced Adjusted Rand Index should be approximately 0.949 against the simulated ground-truth labels
- Clustering should recover five distinct clusters representing the six simulated cell types, with cDC and Treg merged into one cluster
- The number of requested clusters must equal the number of simulated cell types (six in the reference run)
- Every cell in the input count matrix must receive exactly one cluster assignment, and assignments must be written with the ARI to scdha_clustering_results.csv
- Preprocessing must follow the specified order: log(x+1) transformation of counts before scDHA's nonlinear kernel normalization

## Limitations

- Reference benchmark merged the cDC and Treg populations into a single cluster, so the VAE cannot fully separate closely related simulated cell types (5 of 6 recovered)
- sc_mosim-based simulation is restricted to scRNA-seq and scATAC-seq, so this benchmark does not test the method on other omics layers such as proteomics or metabolomics
- Simulation does not cover spatially resolved GRN data, so the skill cannot validate spatial resolution of clustering
- A single replicate from one experimental group is used, so results are not averaged over replicate variability

## Evidence

- [methods] a log transformation (log(x+1)) [42] was applied to the data: [evidence span withheld — anchor status: not_found]
