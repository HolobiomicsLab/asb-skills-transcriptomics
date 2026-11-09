---
name: simulation-scenario-configuration
description: Use when benchmarking differential state (DS) analysis methods and you
  need synthetic multi-sample scRNA-seq data whose properties are anchored to a real
  reference SingleCellExperiment.
when_to_use_negative:
- The goal is DS testing on real experimental data rather than method benchmarking
  — use muscat's pbDS/mmDS on the actual data instead
- The simulation must accommodate an arbitrary number of groups or hierarchical-type
  genes — the article notes the framework does not yet support these extensions
- You need scenarios such as different percentages of DM mixtures that the paper did
  not test
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: muscat
  role: Provides simData and the example_sce reference dataset used to anchor and
    run the simulation
  repo: github.com/HelenaLC/muscat
- name: R
  role: Implementation environment for the muscat package and simulation workflow
- name: Snakemake
  role: Built to run simulation replicates for evaluating the DS methods
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:220d07f9c5dcdca2e30b571de2ce86c3865fe067a52521a55fc178a274278996
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Snakemake
  - muscat
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# simulation-scenario-configuration

## Summary

Configure and run muscat's reference-anchored simulation framework (simData) to generate synthetic multi-sample, multi-group, multi-subpopulation scRNA-seq count data with embedded differential-state genes, used to benchmark DS analysis methods.

## When to use

Use when benchmarking differential state (DS) analysis methods and you need synthetic multi-sample scRNA-seq data whose properties are anchored to a real reference SingleCellExperiment.

## When NOT to use

- The goal is DS testing on real experimental data rather than method benchmarking — use muscat's pbDS/mmDS on the actual data instead
- The simulation must accommodate an arbitrary number of groups or hierarchical-type genes — the article notes the framework does not yet support these extensions
- You need scenarios such as different percentages of DM mixtures that the paper did not test

## Inputs

- Reference SingleCellExperiment object with counts and colData columns sample_id, group_id, cluster_id (e.g. muscat's example_sce)
- Scenario configuration: samples per group, number of subpopulations (nk), fraction of DS genes, logFC magnitudes

## Outputs

- Simulated multi-sample, multi-subpopulation count matrix (SingleCellExperiment), stored e.g. as simulated_counts.rds
- Ground-truth annotation of embedded DS genes and simulation parameters

## How to apply

Load the example_sce reference SingleCellExperiment shipped with muscat and estimate negative-binomial anchoring parameters from it: subpopulation- and sample-specific means, per-gene dispersions, and sample-specific library sizes. Then configure the simulation scenario — number of samples per group (nsamp), number of cell subpopulations (nk), fraction of genes designated as differential-state (ds), and the logFC magnitude of the induced state changes (fc) — and run muscat::simData to generate simulated counts. Differential state is injected following the Korthauer et al. differential distribution approach via DE, DP, DM, or DB changes. Save the simulated count matrix plus the ground-truth DS gene annotation (e.g. simulated_counts.rds) for downstream evaluation of the 16 DS analysis methods.

## Related tools

- **muscat** (Provides simData and the example_sce reference dataset used to anchor and run the simulation) — github.com/HelenaLC/muscat
- **R** (Implementation environment for the muscat package and simulation workflow)
- **Snakemake** (Built to run simulation replicates for evaluating the DS methods)

## Examples

```
data("example_sce"); sim <- muscat::simData(example_sce, nsamp = 3, nk = 8, ds = 0.1, fc = 2); saveRDS(sim, "simulated_counts.rds")
```

## Evaluation signals

- Simulated data recapitulates the reference dataset's library-size and dispersion structure (sample-specific library sizes, per-gene NB dispersions)
- The set of simulated DS genes matches the specified gene fraction (ds) across subpopulations and samples
- Induced state changes match the specified logFC magnitudes (fc) and the chosen differential distribution class (DE, DP, DM, or DB)
- Simulated object carries correct colData structure: sample_id, group_id, and cluster_id consistent with the configured scenario (nsamp, nk)
- Ground-truth DS annotation is complete and consistent with the count matrix dimensions and cell-to-cluster assignments

## Limitations

- The simulation framework does not yet accommodate an arbitrary number of groups or hierarchical-type genes
- Other simulation scenarios of interest, e.g. different percentages of the DM mixtures, were not tested
- Simulation anchoring depends on the reference dataset (example_sce); scenarios are bounded by the reference's estimated NB parameters

## Evidence

- [intro] Objective: develop a simulation framework anchored to a reference dataset to evaluate 16 DS analysis methods: "we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data and used it to evaluate 16 DS analysis methods"
- [abstract] Flexible simulation mimicking multi-sample scRNA-seq data: "To evaluate method performance, we developed a flexible simulation that mimics multi-sample scRNA-seq data."
- [readme] Reference SingleCellExperiment structure for muscat workflows: "Let `sce` be a [`SingleCellExperiment`](https://www.bioconductor.org/packages/SingleCellExperiment.html) object with cell metadata (`colData`) columns"
- [discussion] Known simulation framework extension limits: "our simulation framework could be extended: (i) to accommodate an arbitrary number of groups"
- [discussion] Simulations used to evaluate DS analysis methods across scenarios: "there may be other scenarios of interest (e.g., different percentages of the DM mixtures)"
