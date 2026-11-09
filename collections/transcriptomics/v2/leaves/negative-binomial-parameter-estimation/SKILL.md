---
name: negative-binomial-parameter-estimation
description: Use when building synthetic multi-sample scRNA-seq count data for benchmarking
  differential state (DS) methods and you have a real reference dataset (e.g. muscat's
  example_sce) to anchor the simulation.
when_to_use_negative:
- You are analyzing real experimental data (e.g. pseudobulk DS analysis via pbDS/mmDS)
  rather than generating synthetic benchmark data
- The simulation requires an arbitrary number of groups or hierarchical-type genes,
  which the muscat simulation framework does not yet accommodate
- The reference data are not scRNA-seq counts with sample_id/group_id/cluster_id metadata
  needed to estimate subpopulation- and sample-specific NB parameters
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: muscat
  role: R package providing the simulation platform (simData) and the NB parameter
    estimation that anchors simulations to a reference dataset
  repo: https://github.com/HelenaLC/muscat
- name: R
  role: Execution environment; muscat is installed from Bioconductor via BiocManager
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:aa6c05abd31b3e3f36e5f8adae5515c9fb12477530c93d00a9860ffc309bc54d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HelenaLC/muscat
  tools:
  - R
  - muscat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# negative-binomial-parameter-estimation

## Summary

Estimates subpopulation- and sample-specific negative binomial means, per-gene dispersions, and sample-specific library sizes from a reference SingleCellExperiment so that muscat's simData can anchor multi-sample, multi-subpopulation scRNA-seq simulations to a real dataset.

## When to use

Use when building synthetic multi-sample scRNA-seq count data for benchmarking differential state (DS) methods and you have a real reference dataset (e.g. muscat's example_sce) to anchor the simulation.

## When NOT to use

- You are analyzing real experimental data (e.g. pseudobulk DS analysis via pbDS/mmDS) rather than generating synthetic benchmark data
- The simulation requires an arbitrary number of groups or hierarchical-type genes, which the muscat simulation framework does not yet accommodate
- The reference data are not scRNA-seq counts with sample_id/group_id/cluster_id metadata needed to estimate subpopulation- and sample-specific NB parameters

## Inputs

- Reference SingleCellExperiment object (e.g. muscat's example_sce, or datasets from Bioconductor ExperimentHub anchor datasets)
- Simulation scenario configuration: number of samples per group, number of subpopulations, fraction of DS genes, target logFC magnitudes

## Outputs

- Simulated multi-sample, multi-subpopulation count matrix stored as simulated_counts.rds
- Ground-truth annotation of embedded DS genes and induced parameters (DE/DP/DM/DB categories, logFC magnitudes)
- Estimated NB anchoring parameters: subpopulation-specific means, per-gene dispersions, sample-specific library sizes

## How to apply

Load the reference dataset (data("example_sce") shipped with muscat) and estimate negative-binomial anchoring parameters: subpopulation-specific means, per-gene dispersions, and sample-specific library sizes, drawn from the subpopulation/sample-specific empirical distributions. Then configure the simulation scenario — number of samples per group, number of cell subpopulations, fraction of genes designated as differential-state, and the logFC magnitude of the induced state changes. Run muscat's simData to generate baseline NB counts and inject differential state following the Korthauer et al. differential distribution approach via DE, DP, DM, or DB changes. Save the simulated counts and the ground-truth DS annotation (e.g. simulated_counts.rds), then verify the simulation recapitulates the reference's library-size and dispersion structure.

## Related tools

- **muscat** (R package providing the simulation platform (simData) and the NB parameter estimation that anchors simulations to a reference dataset) — https://github.com/HelenaLC/muscat
- **R** (Execution environment; muscat is installed from Bioconductor via BiocManager)

## Examples

```
library(muscat); data("example_sce")
```

## Evaluation signals

- The simulated data recapitulates the reference dataset's library-size and dispersion structure
- The simulated DS gene set matches the specified fraction of differential-state genes
- Induced logFC magnitudes in the simulation match the configured scenario values
- Embedded DS genes fall into the intended DE/DP/DM/DB differential distribution categories (Korthauer et al. approach)
- The output file simulated_counts.rds loads and contains both the count matrix and the ground-truth DS annotation

## Limitations

- The simulation framework does not yet accommodate an arbitrary number of groups or hierarchical type genes
- Other simulation scenarios of interest, such as different percentages of DM mixtures, were not tested
- Depending on the scale of the data analyzed, logFC estimates can be attenuated, which may affect simulations and benchmarks

## Evidence

- [intro] we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data: "we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data"
- [full_text] The framework anchors simulations to a reference dataset by first estimating subpopulation- and sample-specific negative binomial means, dispersions, and library sizes: [evidence span withheld — anchor status: not_found]
- [intro] anchor datasets via ExperimentHub: [evidence span withheld — anchor status: not_found]
- [readme] muscat package installation: "`muscat` is available through Bioconductor, and
can be installed using the following commands"
