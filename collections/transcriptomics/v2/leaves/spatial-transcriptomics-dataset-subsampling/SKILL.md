---
name: spatial-transcriptomics-dataset-subsampling
description: Use when benchmarking computational scalability of an SRT method — e.g.
  verifying a claim that runtime scales linearly with the number of spatial locations
  on datasets with thousands or more spots or beads.
when_to_use_negative:
- Goal is to identify spatially variable genes (SVGs) rather than to benchmark computational
  scalability — use the nnSVG SVG-detection workflow instead.
- Runtime depends on the number of genes and you change gene-filtering stringency
  across subsamples — timing comparisons become invalid unless filtering is held fixed.
- Input has fewer spatial locations than the smallest grid point, or the question
  is about detection accuracy/power rather than runtime scaling.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: nnSVG
  role: 'Method under benchmark: run per-gene nearest-neighbor Gaussian process fitting
    on each subsampled dataset with n_threads = 1 to time runtime scaling'
  repo: https://bioconductor.org/packages/nnSVG
- name: STexampleData
  role: Provides the Visium_humanDLPFC example dataset used for spot-count downsampling
- name: SpatialExperiment
  role: Bioconductor container holding counts, spatial coordinates, and per-gene results
    (rowData incl. runtime column)
  repo: https://bioconductor.org/packages/SpatialExperiment
- name: BRISC
  role: Underlying algorithm used by nnSVG for nearest-neighbor Gaussian process model
    fitting and parameter estimation
- name: scran
  role: Normalization (computeLibraryFactors, logNormCounts) applied to each subsample
    before timing runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:2cdd4e66ef40f53b5456a2b8724ec2e5355dc660d563121c66c44e20155f3e41
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BRISC
  - Bioconductor
  - STexampleData
  - SpatialExperiment
  - nnSVG
  - scran
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spatial-transcriptomics-dataset-subsampling

## Summary

Benchmarking skill that measures how the runtime of a spatially-resolved transcriptomics method (e.g. nnSVG) scales with the number of spatial locations by running it on downsampled Visium human DLPFC and Slide-seqV2 mouse HPC data and fitting runtime-vs-locations curves.

## When to use

Use when benchmarking computational scalability of an SRT method — e.g. verifying a claim that runtime scales linearly with the number of spatial locations on datasets with thousands or more spots or beads.

## When NOT to use

- Goal is to identify spatially variable genes (SVGs) rather than to benchmark computational scalability — use the nnSVG SVG-detection workflow instead.
- Runtime depends on the number of genes and you change gene-filtering stringency across subsamples — timing comparisons become invalid unless filtering is held fixed.
- Input has fewer spatial locations than the smallest grid point, or the question is about detection accuracy/power rather than runtime scaling.

## Inputs

- Visium human DLPFC spatial transcriptomics dataset (SpatialExperiment object, e.g. Visium_humanDLPFC() from STexampleData, 33538 genes × 4992 spots)
- Slide-seqV2 mouse hippocampus (HPC) spatial transcriptomics dataset (up to 53208 beads)
- Numeric matrix of normalized log-transformed counts (logcounts) and numeric matrix of spatial coordinates per subsample

## Outputs

- Timing table of elapsed runtime per subsample size and repetition (per gene, per platform)
- Runtime-versus-number-of-spatial-locations plot (reproducing Fig. 3a-b) showing the scaling trend for Visium and Slide-seqV2
- Fitted runtime-vs-locations curve supporting or refuting a linear scaling claim

## How to apply

Load the Visium human dorsolateral prefrontal cortex (DLPFC) dataset (e.g. Visium_humanDLPFC from STexampleData: 33538 genes × 4992 spots, 3639 over-tissue spots) and subsample it across a grid of spot counts (n = 200 to 3639); similarly subsample the Slide-seqV2 mouse hippocampus dataset across a grid of bead counts (n = 1000 to 53208). Preprocess each subsample consistently: keep spots over tissue, apply gene filtering (remove mitochondrial genes; retain genes with at least 3 counts in at least 0.5% of spatial locations), compute logcounts with scran (computeLibraryFactors + logNormCounts), since runtime depends on the number of genes and filtering stringency must be held fixed. Run the method on each subsample for a single gene with gene-specific nearest-neighbor Gaussian process fitting, executing on a single processor core and repeating 10 times per gene to obtain stable timing measurements. Record elapsed runtime for each subsample size and repetition into a timing table, then plot runtime versus number of spatial locations for both platforms and confirm the fitted curve is approximately linear (near-linear trend as in Fig. 3a-b of the nnSVG paper).

## Related tools

- **nnSVG** (Method under benchmark: run per-gene nearest-neighbor Gaussian process fitting on each subsampled dataset with n_threads = 1 to time runtime scaling) — https://bioconductor.org/packages/nnSVG
- **STexampleData** (Provides the Visium_humanDLPFC example dataset used for spot-count downsampling)
- **SpatialExperiment** (Bioconductor container holding counts, spatial coordinates, and per-gene results (rowData incl. runtime column)) — https://bioconductor.org/packages/SpatialExperiment
- **BRISC** (Underlying algorithm used by nnSVG for nearest-neighbor Gaussian process model fitting and parameter estimation)
- **scran** (Normalization (computeLibraryFactors, logNormCounts) applied to each subsample before timing runs)

## Examples

```
set.seed(123); spe <- Visium_humanDLPFC(); spe <- spe[, colData(spe)$in_tissue == 1]; spe <- filter_genes(spe); spe <- nnSVG(spe, n_threads = 1)
```

## Evaluation signals

- The fitted runtime-vs-locations curve is approximately linear (near-linear trend) for both the Visium DLPFC and Slide-seqV2 HPC datasets, reproducing Fig. 3a-b.
- Timing is based on 10 repetitions per gene on a single processor core, giving stable elapsed-time measurements.
- nnSVG's per-gene runtime is recorded in the rowData 'runtime' column of the SpatialExperiment output and matches the timing table.
- Gene filtering (mitochondrial removal; ≥3 counts in ≥0.5% of locations) is applied identically to every subsample so runtime differences reflect location count only.
- Subsample sizes span the stated grids (n = 200–3639 spots for Visium; n = 1000–53208 beads for Slide-seqV2).

## Limitations

- Transcriptome-wide runs on datasets with >=10,000 spatial locations still require runtimes of several hours, even with 10 processor cores on a high-performance compute cluster.
- Since runtime depends on the number of genes, more stringent gene filtering is required to reduce runtime — filtering must be fixed across subsamples for a fair scaling benchmark.
- Some small negative values in estimated LR statistics occur mainly for lower-ranked genes and are difficult to interpret (relevant if ranking quality is checked alongside timing).

## Evidence

- [intro] We demonstrate that our method scales linearly with the number of spatial locations, ensuring the method can be applied to datasets with thousands or more spatial locations.: "We demonstrate that our method scales linearly with the number of spatial locations, ensuring the method can be applied to datasets with thousands or more spatial locations."
- [readme] README overview: "The method scales linearly with the number of spatial locations and can be applied to datasets containing thousands or more spatial locations."
- [discussion] Computational cost caveat: "runtimes are on the order of several hours when using 10 processor cores on a high-performance compute cluster"
- [readme] Gene filtering threshold: [evidence span withheld — anchor status: external_doc]
- [discussion] Filtering affects runtime: "Since runtime depends on the number of genes, this can be reduced with more stringent gene filtering"
