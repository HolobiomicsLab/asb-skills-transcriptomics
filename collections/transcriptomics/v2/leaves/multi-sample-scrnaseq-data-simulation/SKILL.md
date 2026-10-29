---
name: multi-sample-scrnaseq-data-simulation
description: Use when benchmarking differential state analysis methods on scRNA-seq
  data, or when ground-truth multi-sample multi-subpopulation counts with known DS
  genes/logFCs are needed.
when_to_use_negative:
- The goal is analyzing a real experimental dataset for DS hits — no ground-truth
  simulation is needed; use muscat's pbDS/mmDS analysis workflow instead
- The reference data lack the required colData columns (sample_id, group_id, cluster_id)
  that simData needs to estimate subpopulation- and sample-specific parameters
- You need simulations with an arbitrary number of groups or hierarchical-type genes
  — the article notes the simulation framework does not yet accommodate these
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: muscat
  role: Provides simData to generate reference-anchored multi-sample multi-subpopulation
    simulations and aggregateData for pseudobulk fidelity checks
  repo: github.com/HelenaLC/muscat
- name: R
  role: Runtime environment for running simData, estimating NB parameters, and computing
    fidelity diagnostics
- name: Snakemake
  role: Orchestrates simulation replicates across benchmark scenarios
- name: countsimQC
  role: Generates the comparative fidelity report of mean-dropout and mean-variance
    relationships between simulated and reference data
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:609a4b1a5bb94d285bedcda28e39d178c9f61fa7df443db6be64222e6b8a72b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - MAST
  - R
  - Snakemake
  - countsimQC
  - edgeR
  - limma-voom
  - muscat
  - scDD
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# multi-sample-scrnaseq-data-simulation

## Summary

Simulate multi-sample, multi-group, multi-subpopulation scRNA-seq count data anchored to a real reference dataset using muscat's simData framework, with injected differential state (DE, DP, DM, DB) genes of known truth for benchmarking DS methods.

## When to use

Use when benchmarking differential state analysis methods on scRNA-seq data, or when ground-truth multi-sample multi-subpopulation counts with known DS genes/logFCs are needed.

## When NOT to use

- The goal is analyzing a real experimental dataset for DS hits — no ground-truth simulation is needed; use muscat's pbDS/mmDS analysis workflow instead
- The reference data lack the required colData columns (sample_id, group_id, cluster_id) that simData needs to estimate subpopulation- and sample-specific parameters
- You need simulations with an arbitrary number of groups or hierarchical-type genes — the article notes the simulation framework does not yet accommodate these

## Inputs

- Reference SingleCellExperiment (e.g., muscat's example_sce or Kang PBMC control samples from ExperimentHub EH2259) with counts assay
- Simulation configuration: nsamp (samples per group), ncluster (subpopulations), nCells per subpopulation-sample, fraction of DS genes, logFC magnitude

## Outputs

- Simulated multi-sample, multi-subpopulation count matrix as SingleCellExperiment (e.g., simulated_counts.rds)
- Ground-truth annotation of embedded DS genes, DS categories (DE, DP, DM, DB), and induced logFC values
- Fidelity diagnostics (e.g., countsimqc_report.html comparing simulated vs. reference characteristics)

## How to apply

Anchor the simulation to a reference SingleCellExperiment (e.g., example_sce from muscat, or the Kang PBMC control samples via ExperimentHub EH2259) by estimating subpopulation- and sample-specific negative binomial means, per-gene dispersions, and library sizes. Configure the scenario — number of samples per group (e.g., 3 per group), number of subpopulations (e.g., 3), cells per subpopulation-sample (e.g., 200), fraction of DS genes (e.g., 10%), and logFC magnitudes — then call muscat::simData to generate counts with DS changes injected following the Korthauer et al. differential distribution approach (DE, DP, DM, DB categories). Optionally run replicates of the simulation scenarios with a Snakemake workflow. Validate fidelity by aggregating simulated and reference counts to pseudobulk with aggregateData and comparing mean-dropout and mean-variance relationships (e.g., via countsimQC) and pseudobulk dispersion-mean trends; save simulated counts together with the ground-truth DS gene annotation.

## Related tools

- **muscat** (Provides simData to generate reference-anchored multi-sample multi-subpopulation simulations and aggregateData for pseudobulk fidelity checks) — github.com/HelenaLC/muscat
- **R** (Runtime environment for running simData, estimating NB parameters, and computing fidelity diagnostics)
- **Snakemake** (Orchestrates simulation replicates across benchmark scenarios)
- **countsimQC** (Generates the comparative fidelity report of mean-dropout and mean-variance relationships between simulated and reference data)

## Examples

```
sim <- simData(example_sce, nsamp = 6, ncluster = 3, nCells = 200, fc = 2)
```

## Evaluation signals

- countsimQC report shows simulated data reproduce the reference's mean-dropout and mean-variance relationships (as in Supplementary File 1)
- Pseudobulk dispersion-mean trends of simulated samples match the reference (Supplementary Fig. 1a pattern)
- The fraction of simulated DS genes matches the configured value (e.g., 10% of genes designated differential state)
- Induced logFC magnitudes in the simulated DS genes match the specified logFC scenario
- Simulated library-size and dispersion structure recapitulates the anchor reference dataset

## Limitations

- The simulation framework does not yet accommodate an arbitrary number of groups or hierarchical-type genes
- Other scenarios of interest, e.g., different percentages of DM mixtures, were not tested
- Fidelity is only as good as the single anchor reference; anchoring to a reference with limited sample size may not span realistic sample-to-sample variability

## Evidence

- [intro] Objective: develop a simulation framework anchored to a reference dataset: "we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data and used it to evaluate 16 DS analysis methods"
- [methods] muscat scope for multi-sample multi-group multi-subpopulation data: "`muscat` can be used to perform differrential detection (DD) analyses in multi-sample, multi-group, multi-(cell-)subpopulation scRNA-seq data"
- [readme] muscat provides DS analysis methods and installation route: "provides methods for *Differential State* (DS) analyses in scRNA-seq data"
