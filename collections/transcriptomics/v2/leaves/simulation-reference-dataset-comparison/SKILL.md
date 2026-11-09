---
name: simulation-reference-dataset-comparison
description: Use when simulated single-cell count data (e.g. from dyngen's SSA simulation)
  must be shown to match the technical characteristics of a real transcript-count
  dataset before using it for benchmarking.
when_to_use_negative:
- The input data are already real single-cell measurements, so no protocol emulation
  or reference-dataset calibration is needed
- The goal is benchmarking downstream methods (trajectory inference, RNA velocity,
  network inference) on existing simulations rather than validating the realism of
  the simulated count distribution
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: dyngen
  role: Simulation engine whose simulated counts are calibrated to and compared against
    a real reference dataset
  repo: https://github.com/dynverse/dyngen
- name: countsimQC
  role: Assesses whether characteristic single-cell omics features (library size,
    fraction zeros, mean-variance) of the simulation are consistent with the reference
    dataset
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:a39ee97ad78491706935b942ff04cc5b15ff058c834c7ad742b0d84111d5b96a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - GillespieSSA2
  - countsimQC
  - dyngen
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# simulation-reference-dataset-comparison

## Summary

Validates dyngen's simulated single-cell count matrices against real reference single-cell RNA-seq datasets by emulating profiling protocols (library sizes drawn from a reference transcript-count distribution, capture rates from N(1, 0.05), multinomial molecule drawing) and checking that characteristic omics features (library size, fraction zeros, mean-variance) match the reference as assessed by countsimQC.

## When to use

Use when simulated single-cell count data (e.g. from dyngen's SSA simulation) must be shown to match the technical characteristics of a real transcript-count dataset before using it for benchmarking.

## When NOT to use

- The input data are already real single-cell measurements, so no protocol emulation or reference-dataset calibration is needed
- The goal is benchmarking downstream methods (trajectory inference, RNA velocity, network inference) on existing simulations rather than validating the realism of the simulated count distribution

## Inputs

- dyngen SSA simulation state with per-gene pre-mRNA (xG), mature mRNA (yG), and protein (zG) abundances per cell
- Real reference single-cell RNA-seq dataset with transcript-count distribution (e.g. zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca)

## Outputs

- Simulated count matrix (e.g. dyngen_simulated_counts.csv) with realistic library sizes, zero fraction, and mean-variance behavior
- countsimQC-style comparison of simulated vs reference count characteristics

## How to apply

After dyngen's Gillespie SSA produces molecular abundances (pre-mRNA xG, mature mRNA yG, protein zG), emulate the experiment by first drawing each simulated cell's library size from a real reference dataset's transcript-count distribution (e.g. a dataset named like zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca downloaded from a repository of single-cell RNA-seq datasets). Then draw capture rates from a Normal distribution N(1, 0.05) and draw observed molecules from a multinomial distribution given the library size and capture rates. Save the resulting count matrix (e.g. dyngen_simulated_counts.csv) and validate realism by confirming characteristic single-cell omics features — library size, fraction of zeros, and mean-variance relationship — are consistent with the reference dataset using countsimQC (Supplementary Note 1).

## Related tools

- **dyngen** (Simulation engine whose simulated counts are calibrated to and compared against a real reference dataset) — https://github.com/dynverse/dyngen
- **countsimQC** (Assesses whether characteristic single-cell omics features (library size, fraction zeros, mean-variance) of the simulation are consistent with the reference dataset)

## Examples

```
name_realcounts <- "zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca"
```

## Evaluation signals

- The output contains the requested number of cells and genes
- Library sizes of simulated cells match the reference dataset's transcript-count distribution
- Fraction of zeros and mean-variance relationship of the simulated counts are consistent with the reference dataset as assessed by countsimQC
- Capture rates are drawn from N(1, 0.05) and observed molecules from a multinomial distribution rather than arbitrary noise models

## Limitations

- dyngen assumes cells are well-matched and independent from each other ('cells are assumed to be well-mixed and independent from each other')
- The reference single-cell RNA-seq dataset repository is not explicitly named in the methods ('A real dataset is downloaded from a repository of single-cell RNA-seq datasets49.')
- No software version numbers are reported for the tools used

## Evidence

- [methods] Reference dataset repository: "A real dataset is downloaded from a repository of single-cell RNA-seq datasets49."
- [methods] Named real-counts reference dataset: "name_realcounts <- "zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca""
- [supplementary] countsimQC validation of realism: [evidence span withheld — anchor status: not_found]
- [readme] README comparison-to-reference guide: "Advanced: Comparison to reference
dataset"
