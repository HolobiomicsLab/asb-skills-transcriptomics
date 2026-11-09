---
name: runtime-scaling-visualization
description: Use when a paper claims an algorithm scales linearly (or better) with
  input size and you need to empirically reproduce the runtime-versus-size curve on
  downsampled datasets before applying the method to large SRT data.
when_to_use_negative:
- The goal is to identify or rank spatially variable genes rather than to characterize
  computational scalability — use the standard nnSVG SVG-detection workflow instead.
- The claim under test is statistical performance (SVG detection accuracy) rather
  than runtime scaling with the number of spatial locations.
- Timing with multiple threads or parallel cores is intended; the linear-scaling measurement
  requires single-processor-core runs to isolate per-core scaling behavior.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: nnSVG
  role: Method being timed; run per gene with nearest-neighbor Gaussian process fitting
    via BRISC on a single thread, recording the runtime column across subsampled spot/bead
    counts
  repo: https://bioconductor.org/packages/nnSVG
- name: Bioconductor
  role: Framework providing nnSVG, SpatialExperiment, and STexampleData for loading
    and subsampling the Visium DLPFC and Slide-seqV2 datasets
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:c98b85b8c5f4c5acb2815bdc6fa0c711874132a97e15c95fc3d3c9e33583cacd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - nnSVG
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# runtime-scaling-visualization

## Summary

Empirically verifies computational scalability of a method by timing repeated runs of nnSVG (single gene, single processor core) across downsampled spot/bead grids from Visium human DLPFC and Slide-seqV2 mouse HPC datasets, then plotting runtime versus number of spatial locations to reproduce the near-linear trend of Fig. 3a-b.

## When to use

Use when a paper claims an algorithm scales linearly (or better) with input size and you need to empirically reproduce the runtime-versus-size curve on downsampled datasets before applying the method to large SRT data.

## When NOT to use

- The goal is to identify or rank spatially variable genes rather than to characterize computational scalability — use the standard nnSVG SVG-detection workflow instead.
- The claim under test is statistical performance (SVG detection accuracy) rather than runtime scaling with the number of spatial locations.
- Timing with multiple threads or parallel cores is intended; the linear-scaling measurement requires single-processor-core runs to isolate per-core scaling behavior.

## Inputs

- Visium human DLPFC spatial transcriptomics dataset (e.g. STexampleData Visium_humanDLPFC, 33538 genes x 4992 spots, 3639 over-tissue spots) subsampled to spot grids from n = 200 to 3639
- Slide-seqV2 mouse hippocampus (HPC) dataset subsampled to bead grids from n = 1000 to 53208 beads
- SpatialExperiment Bioconductor object (or numeric matrix of logcounts plus matrix of spatial coordinates)
- Choice of a single gene to time repeatedly

## Outputs

- Timing table of elapsed runtime per subsample size and repetition (10 replicates per size)
- Runtime-vs-number-of-spatial-locations plot for both platforms, reproducing Fig. 3a-b
- Fitted runtime-vs-locations trend confirming approximate linearity

## How to apply

Subsample the Visium human dorsolateral prefrontal cortex (DLPFC) dataset across a grid of spot counts (n = 200 to 3639 spots) and the Slide-seqV2 mouse hippocampus (HPC) dataset across a grid of bead counts (n = 1000 to 53208 beads). For each subsample, run nnSVG with gene-specific nearest-neighbor Gaussian process fitting on a single processor core (n_threads = 1), repeating 10 times per gene on a single gene to obtain stable timing measurements, and record elapsed runtime into a timing table. The `runtime` column is returned in rowData of the SpatialExperiment object. Plot runtime versus number of spatial locations for both platforms and fit a trend; validation requires the fitted curve to be approximately linear for both datasets, matching the near-linear trend reported in Fig. 3a-b. Rationale: linearity with the number of spatial locations is the property that lets nnSVG be applied to datasets with thousands or more locations, so the timing experiment directly tests the scalability claim.

## Related tools

- **nnSVG** (Method being timed; run per gene with nearest-neighbor Gaussian process fitting via BRISC on a single thread, recording the runtime column across subsampled spot/bead counts) — https://bioconductor.org/packages/nnSVG
- **Bioconductor** (Framework providing nnSVG, SpatialExperiment, and STexampleData for loading and subsampling the Visium DLPFC and Slide-seqV2 datasets) — https://bioconductor.org/packages/nnSVG

## Examples

```
set.seed(123); spe <- nnSVG(spe, n_threads = 1); timings <- rowData(spe)$runtime
```

## Evaluation signals

- The fitted runtime-vs-locations curve is approximately linear for both the Visium DLPFC (200–3639 spots) and Slide-seqV2 HPC (1000–53208 beads) subsample grids, matching the near-linear trend of Fig. 3a-b.
- Each timing measurement comes from 10 repetitions per gene on a single processor core, giving stable timing rather than single noisy runs.
- Per-run `runtime` values are present in rowData of the SpatialExperiment object after calling nnSVG.
- Timing is performed for a single gene so runtime reflects per-gene NN-GP fitting cost, not multiplicative gene-count effects (runtime depends on the number of genes, so gene sets must be held constant across subsamples).

## Limitations

- Transcriptome-wide runs on datasets with >=10,000 spatial locations still require runtimes on the order of several hours with 10 processor cores on a high-performance compute cluster, so linearity does not imply trivial cost at full scale.
- Runtime depends on the number of genes; transcriptome-wide timing experiments are expensive, so timing is typically done per gene or after gene filtering.
- The linear-scaling demonstration is specific to the measured platforms (Visium DLPFC spots and Slide-seqV2 HPC beads); other platforms or hardware may shift absolute runtimes.

## Evidence

- [abstract] Linear scaling claim: "nnSVG scales linearly with the number of spatial locations"
- [intro] Objective of the demonstration: "We demonstrate that our method scales linearly with the number of spatial locations, ensuring the method can be applied to datasets with thousands or more spatial locations."
- [readme] Runtime measurement recorded per run: [evidence span withheld — anchor status: external_doc]
- [readme] Single-thread invocation for reproducible timing: "set.seed(123)
spe <- nnSVG(spe, n_threads = 1)"
- [discussion] Runtime depends on gene count, enabling filtering-based reduction: "Since runtime depends on the number of genes, this can be reduced with more stringent gene filtering"
- [discussion] Residual computational cost at full scale: "runtimes are on the order of several hours when using 10 processor cores on a high-performance compute cluster"
