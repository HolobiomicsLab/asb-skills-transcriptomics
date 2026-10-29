---
name: reference-anchored-count-data-fitting
description: Use when benchmarking differential state (DS) analysis methods on simulated
  scRNA-seq data that must recapitulate real reference dataset characteristics such
  as library-size and dispersion structure across samples and subpopulations.
when_to_use_negative:
- The simulation framework does not accommodate an arbitrary number of groups or hierarchical-type
  genes, so it is the wrong choice for simulations requiring those designs
- Analysis goal is actual differential state inference on real data rather than benchmark
  data generation — use pbDS or mmDS instead of simData
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: muscat
  role: R package providing the simData simulation platform and example_sce reference
    dataset for anchoring the negative-binomial simulation
  repo: https://github.com/HelenaLC/muscat
- name: R
  role: Runtime environment for loading the reference SingleCellExperiment, fitting
    NB parameters, and running simData
- name: Snakemake
  role: Workflow engine used to run simulation replicates across scenarios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:9b08d5c9d5f46f0a36cfe6d3a4b37d5edb5e18f803dd93ba2cbbbff1e202757d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HelenaLC/muscat
  tools:
  - R
  - Snakemake
  - muscat
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# reference-anchored-count-data-fitting

## Summary

Generate synthetic multi-sample, multi-subpopulation scRNA-seq count data by fitting negative-binomial parameters (subpopulation-specific means, per-gene dispersions, sample-specific library sizes) to a real reference SingleCellExperiment (e.g., muscat's example_sce from Kang PBMC control samples), then injecting differential state via DE, DP, DM, or DB changes following Korthauer et al.

## When to use

Use when benchmarking differential state (DS) analysis methods on simulated scRNA-seq data that must recapitulate real reference dataset characteristics such as library-size and dispersion structure across samples and subpopulations.

## When NOT to use

- The simulation framework does not accommodate an arbitrary number of groups or hierarchical-type genes, so it is the wrong choice for simulations requiring those designs
- Analysis goal is actual differential state inference on real data rather than benchmark data generation — use pbDS or mmDS instead of simData

## Inputs

- Reference SingleCellExperiment object with cell metadata columns sample_id, group_id, and cluster_id (e.g., muscat's example_sce from Kang PBMC control samples)
- Simulation scenario configuration: number of samples per group, number of subpopulations, fraction of DS genes, and logFC magnitudes

## Outputs

- Simulated multi-sample, multi-subpopulation scRNA-seq count matrix with embedded differential-state genes
- Ground-truth annotation of DS genes and simulation parameters (e.g., simulated_counts.rds)

## How to apply

Load the reference dataset (e.g., `data("example_sce")` shipped with the muscat R package) and estimate subpopulation- and sample-specific negative-binomial anchoring parameters: per-subpopulation means, per-gene dispersions, and sample-specific library sizes. Configure the simulation scenario by specifying the number of samples per group, the number of cell subpopulations, the fraction of genes designated as differential-state, and the logFC magnitude of the induced state changes. Run muscat's `simData` to draw baseline counts from the NB distributions and inject differential state following the Korthauer et al. differential distribution approach via DE (differential expression), DP (differential proportion), DM (differential modality), or DB (differential both) changes. Save the simulated count matrix together with the ground-truth annotation of DS genes and parameters (e.g., as simulated_counts.rds). Validate that the simulation recapitulates the reference's library-size and dispersion structure and that the embedded DS gene set matches the specified fraction and logFC magnitudes.

## Related tools

- **muscat** (R package providing the simData simulation platform and example_sce reference dataset for anchoring the negative-binomial simulation) — https://github.com/HelenaLC/muscat
- **R** (Runtime environment for loading the reference SingleCellExperiment, fitting NB parameters, and running simData)
- **Snakemake** (Workflow engine used to run simulation replicates across scenarios)

## Examples

```
library(muscat); data("example_sce"); sim <- simData(example_sce, nsamp = 3, ncell = 1000, probs = NULL, p_ds = 0.1, lfc = 1)
```

## Evaluation signals

- Simulated data recapitulates the reference dataset's library-size distribution per sample
- Simulated data recapitulates the reference dataset's per-gene dispersion structure per subpopulation
- The set of simulated DS genes matches the user-specified fraction of differential-state genes
- Observed logFC magnitudes of the injected DS genes match the configured logFC values
- Simulated count matrix is a valid multi-sample, multi-subpopulation matrix whose colData carries consistent sample_id, group_id, and cluster_id labels

## Limitations

- The simulation framework does not yet accommodate an arbitrary number of groups
- Hierarchical-type genes are not supported by the simulation framework
- Other scenarios of interest, such as different percentages of DM mixtures, were not tested
- Framework was designed for a single anchored reference; logFC estimates were attenuated for some combinations depending on the scale of the data analyzed

## Evidence

- [intro] we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data and used it to evaluate 16 DS analysis methods: "we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data and used it to evaluate 16 DS analysis methods"
- [discussion] simulation framework could be extended to arbitrary groups: "our simulation framework could be extended: (i) to accommodate an arbitrary number of groups"
- [readme] README: muscat provides methods for Differential State analyses: "provides methods for *Differential State* (DS) analyses in scRNA-seq data with multiple samples, groups, and (cell)-subpopulations"
