---
name: single-cell-rna-seq-count-emulation
description: Use when simulated molecular abundances must be transformed into a count
  matrix whose library sizes, fraction of zeros, and mean-variance behaviour match
  a real single-cell omics reference dataset for benchmarking.
when_to_use_negative:
- The analysis starts from real experimental counts already carrying protocol-level
  noise — no emulation is needed
- The goal is studying biological network dynamics of the simulation itself rather
  than producing benchmark-ready count data
- The application requires ODE-based bulk simulation; dyngen uses Gillespie's SSA
  specifically because ODEs are ill-suited for low-molecule single-cell simulations
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dyngen
  role: 'Multi-modal simulation engine that performs the experiment-emulation step:
    sampling cells from SSA trajectories and drawing counts from a real reference
    dataset''s distribution'
  repo: https://github.com/dynverse/dyngen
- name: GillespieSSA2
  role: Runs Gillespie's SSA (e.g. ssa_etl with a census interval) to produce the
    pre-mRNA/mRNA/protein abundances that count emulation consumes
  repo: https://github.com/dynverse/dyngen
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:ca49b88b280222a78a91ec108571e704cd7663c8a2c0efdfd50febe84acdca27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - GillespieSSA2
  - dyngen
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell RNA-seq count emulation

## Summary

Convert molecule abundances from stochastic single-cell simulations (pre-mRNA xG, mature mRNA yG, protein zG) into realistic single-cell count data by emulating single-cell omics profiling protocols with real reference datasets, as implemented in dyngen.

## When to use

Use when simulated molecular abundances must be transformed into a count matrix whose library sizes, fraction of zeros, and mean-variance behaviour match a real single-cell omics reference dataset for benchmarking.

## When NOT to use

- The analysis starts from real experimental counts already carrying protocol-level noise — no emulation is needed
- The goal is studying biological network dynamics of the simulation itself rather than producing benchmark-ready count data
- The application requires ODE-based bulk simulation; dyngen uses Gillespie's SSA specifically because ODEs are ill-suited for low-molecule single-cell simulations

## Inputs

- SSA simulation output: per-cell abundances of pre-mRNA (xG), mature mRNA (yG), and protein (zG) for every gene
- Real reference single-cell RNA-seq dataset providing the transcript-count distribution (e.g. zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca)
- Cell sampling specification: snapshot (target cell number N) or time series (number of intervals k, gap g)

## Outputs

- Simulated single-cell count matrix (dyngen_simulated_counts.csv) with the requested number of cells and genes
- Ground-truth layers for benchmarking (cell state, reaction propensity, cellwise regulatory network)

## How to apply

First sample the desired number of cells from the Gillespie SSA simulation: in snapshot mode allocate N_i = N/(L_i/sum L_i) cells per transition edge, or in time-series mode use k=8 intervals with a gap of g=0.75. Then emulate the experiment: draw each cell's library size from a real reference dataset's transcript-count distribution, draw capture rates from a normal distribution N(1, 0.05), and draw observed molecules from a multinomial distribution over the sampled abundances. Rationale: SSA simulations alone are only molecule counts; the emulation step is what makes outputs statistically comparable to real scRNA-seq profiles. Save the result (e.g. dyngen_simulated_counts.csv) and validate against the reference dataset with countsimQC (library size, fraction zeros, mean-variance) as described in Supplementary Note 1.

## Related tools

- **dyngen** (Multi-modal simulation engine that performs the experiment-emulation step: sampling cells from SSA trajectories and drawing counts from a real reference dataset's distribution) — https://github.com/dynverse/dyngen
- **GillespieSSA2** (Runs Gillespie's SSA (e.g. ssa_etl with a census interval) to produce the pre-mRNA/mRNA/protein abundances that count emulation consumes) — https://github.com/dynverse/dyngen

## Examples

```
install.packages("dyngen")
```

## Evaluation signals

- Output count matrix contains the requested number of cells and genes
- countsimQC comparison shows the simulated library size, fraction of zeros, and mean-variance relationship are consistent with the chosen real reference dataset (Supplementary Note 1)
- Cell counts per transition edge match snapshot allocation N_i = N/(L_i/sum L_i), or time-series sampling used k=8 intervals with gap g=0.75
- Capture-rate draws fall around the N(1, 0.05) distribution (mean ~1 per cell)
- Observed molecules per cell are multinomial draws over the sampled abundances, i.e. total counts per cell match the library size drawn from the reference transcript-count distribution

## Limitations

- dyngen assumes cells are well-mixed and independent from each other, which count emulation cannot correct
- Realism is bounded by the chosen reference dataset: no software version numbers for the reference repository were reported
- The article notes a repository of single-cell RNA-seq datasets is used for reference counts but does not name the repository explicitly

## Evidence

- [intro] Second, individual cells are simulated using Gillespie's stochastic simulation algorithm (SSA)12, which is designed to work well in low-molecule simulations: "Second, individual cells are simulated using Gillespie’s stochastic simulation algorithm (SSA)12, which is designed to work well in low-molecule simulations"
- [intro] Finally, real reference datasets are used to emulate single-cell omics profiling protocols.: "Finally, real reference datasets are used to emulate single-cell omics profiling protocols."
- [methods] drawing each cell's library size from a real reference dataset's transcript-count distribution, drawing capture rates from N(1, 0.05): [evidence span withheld — anchor status: not_found]
- [readme] dyngen is a novel, multi-modal simulation engine for studying dynamic cellular processes at single-cell resolution.: "dyngen is a novel, multi-modal simulation engine for studying dynamic cellular processes at single-cell resolution."
