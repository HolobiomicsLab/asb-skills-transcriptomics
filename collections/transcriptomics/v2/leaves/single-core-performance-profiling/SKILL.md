---
name: single-core-performance-profiling
description: 'Use when validating a scalability claim such as ''runtime scales linearly
  with the number of spatial locations'': timing a method across a grid of downsampled
  dataset sizes (e.g. 200–3639 Visium spots, 1000–53208 Slide-seqV2 beads) on one
  processor core.'
when_to_use_negative:
- When benchmarking multi-core or cluster performance — the paper's own non-single-core
  numbers (several hours on 10 processor cores on a high-performance compute cluster)
  reflect parallelized runs and are not comparable to single-thread scaling measurements
- When the goal is identifying spatially variable genes rather than characterizing
  computational scaling — use the standard nnSVG SVG-ranking workflow instead
- When profiling runtime as a function of gene count rather than spatial locations
  — note that runtime also depends on the number of genes, which confounds location-scaling
  measurements if the gene set varies
tools:
- name: nnSVG
  role: Method being profiled; run per gene with gene-specific nearest-neighbor Gaussian
    process fitting on a single thread (n_threads = 1), 10 repetitions per subsample
    size
  repo: https://bioconductor.org/packages/nnSVG
- name: BRISC
  role: Backend algorithm nnSVG uses for model fitting and parameter estimation in
    the nearest-neighbor Gaussian process; its fitting cost drives the measured runtimes
- name: Bioconductor
  role: Framework providing the SpatialExperiment input objects and the nnSVG installation
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:44629a545ac8e63ad85aedcc7344b1c5d81c91aa4d8896acd7f0d5a97a8fa096
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BRISC
  - Bioconductor
  - nnSVG
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-core-performance-profiling

## Summary

Benchmarks the runtime of a method (here, nnSVG's gene-specific nearest-neighbor Gaussian process fitting) as a function of dataset size — downsampled numbers of spatial locations — executed on a single processor core and repeated 10 times per gene to obtain stable timing measurements, in order to verify the claimed linear scaling.

## When to use

Use when validating a scalability claim such as 'runtime scales linearly with the number of spatial locations': timing a method across a grid of downsampled dataset sizes (e.g. 200–3639 Visium spots, 1000–53208 Slide-seqV2 beads) on one processor core.

## When NOT to use

- When benchmarking multi-core or cluster performance — the paper's own non-single-core numbers (several hours on 10 processor cores on a high-performance compute cluster) reflect parallelized runs and are not comparable to single-thread scaling measurements
- When the goal is identifying spatially variable genes rather than characterizing computational scaling — use the standard nnSVG SVG-ranking workflow instead
- When profiling runtime as a function of gene count rather than spatial locations — note that runtime also depends on the number of genes, which confounds location-scaling measurements if the gene set varies

## Inputs

- Visium human dorsolateral prefrontal cortex (DLPFC) spatial transcriptomics dataset (33538 genes x 3639 over-tissue spots), e.g. from STexampleData as a SpatialExperiment object
- Slide-seqV2 mouse hippocampus (HPC) dataset (1000–53208 beads)
- Subsampled grids of spot/bead counts (Visium: n = 200 to 3639 spots; Slide-seqV2: n = 1000 to 53208 beads)

## Outputs

- Timing table of elapsed runtime per subsample size and repetition
- Runtime-vs-numbers-of-spatial-locations plots for both platforms (reproducing Fig. 3a-b)
- Fitted runtime-vs-locations curve with a near-linear trend

## How to apply

Load the Visium human DLPFC dataset (4992 spots, 3639 over-tissue) and the Slide-seqV2 mouse hippocampus dataset, then subsample each across the stated grid of spatial-location counts. Run the method (nnSVG with nearest-neighbor Gaussian process fitting via BRISC) on each subsample for a single gene, pinning to a single thread with n_threads = 1, and repeat 10 times to obtain stable timing. Record elapsed runtime for each subsample size and repetition into a timing table (nnSVG stores per-gene runtime in the 'runtime' column of rowData). Plot runtime versus number of spatial locations for both platforms and check that the fitted curve is approximately linear — near-linear trends reproduce Fig. 3a-b. Rationale: single-core timing isolates the method's intrinsic algorithmic scaling from parallel-communication overhead, and the linear trend is what licenses applying the method to datasets with thousands or more spatial locations.

## Related tools

- **nnSVG** (Method being profiled; run per gene with gene-specific nearest-neighbor Gaussian process fitting on a single thread (n_threads = 1), 10 repetitions per subsample size) — https://bioconductor.org/packages/nnSVG
- **BRISC** (Backend algorithm nnSVG uses for model fitting and parameter estimation in the nearest-neighbor Gaussian process; its fitting cost drives the measured runtimes)
- **Bioconductor** (Framework providing the SpatialExperiment input objects and the nnSVG installation) — https://bioconductor.org/packages/nnSVG

## Examples

```
set.seed(123); spe <- nnSVG(spe, n_threads = 1); rowData(spe)$runtime
```

## Evaluation signals

- The fitted runtime-vs-locations curve for both the Visium human DLPFC and Slide-seqV2 mouse HPC datasets is approximately linear (near-linear trend as reported in Fig. 3a-b)
- Each timing measurement comes from 10 repetitions of nnSVG on a single gene on a single processor core, so runtimes are stable across repetitions
- Runtime is recorded for every point in the subsample grid (200–3639 spots for Visium; 1000–53208 beads for Slide-seqV2), not just the full dataset
- Per-gene runtime values match the 'runtime' column stored in rowData of the SpatialExperiment object after running nnSVG

## Limitations

- Single-core timings do not reflect the practical throughput for full transcriptome-wide runs: datasets with >=10,000 spatial locations require runtimes on the order of several hours even when using 10 processor cores on a high-performance compute cluster
- Runtime also depends on the number of genes, so location-scaling benchmarks must control gene filtering; runtime can be reduced with more stringent gene filtering
- Timing variability is mitigated by repeating each run 10 times per gene; single-run timings may be noisy

## Evidence

- [results] linear scaling claim: [evidence span withheld — anchor status: not_found]
- [readme] README linear-scaling statement: "The method scales linearly with the number of spatial locations and can be applied to datasets containing thousands or more spatial locations."
- [discussion] gene-count confound on runtime: "Since runtime depends on the number of genes, this can be reduced with more stringent gene filtering"
- [discussion] practical compute cost caveat: "runtimes are on the order of several hours when using 10 processor cores on a high-performance compute cluster"
- [readme] single-thread example run: "# set seed for reproducibility
# run nnSVG using a single thread for this example workflow
set.seed(123)
spe <- nnSVG(spe, n_threads = 1)"
