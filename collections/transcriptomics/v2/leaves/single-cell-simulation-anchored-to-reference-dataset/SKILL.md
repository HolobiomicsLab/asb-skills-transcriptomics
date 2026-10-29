---
name: single-cell-simulation-anchored-to-reference-dataset
description: Use when benchmarking differential state (DS) methods requires simulated
  multi-sample scRNA-seq data that faithfully mimic a reference dataset's characteristics
  (mean-dropout, mean-variance, sample-to-sample variability).
when_to_use_negative:
- The analysis target is differential state inference on real experimental data rather
  than method benchmarking via simulation.
- The simulation must accommodate an arbitrary number of groups or hierarchical-type
  genes — the authors note the framework does not yet support these extensions.
- Reference data are bulk RNA-seq or lack per-cell sample/group/cluster metadata required
  to anchor simData.
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: muscat
  role: Provides simData (reference-anchored simulation) and aggregateData (pseudobulk
    aggregation) used to generate and evaluate simulated data
  repo: github.com/HelenaLC/muscat
- name: R
  role: Runtime environment for muscat, Bioconductor ExperimentHub access, and countsimQC
    report generation
- name: Snakemake
  role: Workflow engine used to run simulation replicates at scale
- name: countsimQC
  role: Generates the comparative fidelity report characterizing simulated vs. reference
    count distributions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:4a3a10549cbc275abf8691864e330be33cff42a97870f6945a7b865dd9774469
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Snakemake
  - countsimQC
  - muscat
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-simulation-anchored-to-reference-dataset

## Summary

Generates faithful multi-sample multi-group scRNA-seq simulations using muscat's simData, anchored to a reference SingleCellExperiment (e.g., Kang PBMC control samples via ExperimentHub EH2259), and verifies fidelity with countsimQC by comparing mean-dropout and mean-variance relationships plus pseudobulk dispersion-mean trends.

## When to use

Use when benchmarking differential state (DS) methods requires simulated multi-sample scRNA-seq data that faithfully mimic a reference dataset's characteristics (mean-dropout, mean-variance, sample-to-sample variability).

## When NOT to use

- The analysis target is differential state inference on real experimental data rather than method benchmarking via simulation.
- The simulation must accommodate an arbitrary number of groups or hierarchical-type genes — the authors note the framework does not yet support these extensions.
- Reference data are bulk RNA-seq or lack per-cell sample/group/cluster metadata required to anchor simData.

## Inputs

- Reference scRNA-seq SingleCellExperiment with sample_id, group_id, cluster_id colData columns (e.g., Kang PBMC control samples, ExperimentHub EH2259)
- Simulation scenario parameters for simData (e.g., number of samples, groups, subpopulations, percentage of DS/DA/DM genes)

## Outputs

- Simulated multi-sample multi-group scRNA-seq counts (SingleCellExperiment)
- Pseudobulk counts per sample and subpopulation (aggregateData output)
- countsimQC fidelity report (countsimqc_report.html)

## How to apply

Load the reference dataset into R as a SingleCellExperiment with sample_id, group_id, and cluster_id metadata columns (e.g., the Kang PBMC control samples, ExperimentHub EH2259). Run muscat's simData to generate simulated multi-sample counts anchored to the reference's characteristics. Aggregate both simulated and reference counts to pseudobulk per sample and subpopulation with aggregateData. Compute and compare mean-dropout and mean-variance relationships between simulated and reference data, and compare pseudobulk dispersion-mean trends. Assemble the diagnostics into a countsimQC report; the authors verified fidelity this way (Supplementary File 1 and Supplementary Fig. 1a). The full simulation-replicate campaign was orchestrated with a Snakemake workflow.

## Related tools

- **muscat** (Provides simData (reference-anchored simulation) and aggregateData (pseudobulk aggregation) used to generate and evaluate simulated data) — github.com/HelenaLC/muscat
- **R** (Runtime environment for muscat, Bioconductor ExperimentHub access, and countsimQC report generation)
- **Snakemake** (Workflow engine used to run simulation replicates at scale)
- **countsimQC** (Generates the comparative fidelity report characterizing simulated vs. reference count distributions)

## Examples

```
library(muscat); sim <- simData(sce, n_sims = 1); pb <- aggregateData(sim, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id"))
```

## Evaluation signals

- Simulated and reference data show matching mean-dropout relationships in the countsimQC report (Supplementary File 1).
- Simulated and reference data show matching mean-variance relationships in the countsimQC report.
- Pseudobulk dispersion-mean trends of simulated data reproduce the reference sample-to-sample variability (Supplementary Fig. 1a).
- Simulated SingleCellExperiment carries valid sample_id, group_id, and cluster_id colData columns consistent with the anchor dataset.
- Simulation replicates run end-to-end under the Snakemake workflow without errors for all tested scenarios.

## Limitations

- The simulation framework does not yet accommodate an arbitrary number of groups or hierarchical-type genes.
- Other scenarios of interest, such as different percentages of the DM mixtures, were not tested.
- Fidelity is verified against a single anchor dataset; faithful reproduction of that reference does not guarantee faithfulness to all scRNA-seq data characteristics.

## Evidence

- [intro] we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data and used it to evaluate 16 DS analysis methods: "we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data"
- [supplementary] finding: simulation framework reproduces mean-dropout, mean-variance, and pseudobulk dispersion-mean trends verified by countsimQC: [evidence span withheld — anchor status: not_found]
- [methods] workflow step: load Kang control reference SingleCellExperiment, run simData, aggregateData, compare diagnostics: [evidence span withheld — anchor status: not_found]
- [readme] README: aggregateData computes pseudobulks from SingleCellExperiment with sample_id/group_id/cluster_id metadata: "pb <- aggregateData(sce, 
    assay = "counts", fun = "sum",
    by = c("cluster_id", "sample_id"))"
- [intro] All methods tested are available within the muscat R package and a Snakemake workflow was built to run simulation replicates: "All methods tested are available within the muscat R package and a Snakemake35 workflow was built to run simulation replicates."
