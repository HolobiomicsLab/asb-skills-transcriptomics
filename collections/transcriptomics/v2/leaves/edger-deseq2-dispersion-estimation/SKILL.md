---
name: edger-deseq2-dispersion-estimation
description: Use when a dyngen simulation must be shown to match a real single-cell
  RNA-seq reference dataset (e.g. the zenodo_1443566 bone-marrow mesenchyme MCA counts)
  in library size, fraction of zeros, and dispersion characteristics.
when_to_use_negative:
- The count matrices are not paired as a dyngen simulation versus its real reference
  dataset — countsimQC comparison is anchored to dyngen's realcounts emulation, not
  a generic count-matrix comparison
- The data are already normalised/log-transformed rather than raw counts, since edgeR
  TMM normalisation factors, log2 CPM with prior.count = 2, and DESeq2 dispersion
  estimation all require raw count input
- The goal is a single-dataset quality check rather than a two-dataset comparison
  of data characteristics
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dyngen
  role: Generates the simulated count matrix and provides the realcounts table used
    to download the reference dataset and match simulation dimensions
  repo: https://github.com/dynverse/dyngen
- name: countsimQC
  role: Provides internal helpers (calculateDispersionsddsList, calculateSampleCorrs,
    calculateFeatureCorrs) and renders the unified comparative report
- name: DESeq2
  role: Wraps the count matrices as DESeqDataSets (design ~1) and supplies dispGeneEst/dispFit/dispFinal
    dispersion estimates
- name: edgeR
  role: Supplies TMM normalisation factors, tagwise/common/trended dispersions and
    BCV plots for the comparison
- name: ggplot2
  role: Rendering engine for the comparative panels in the countsimQC report
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:e6a1c428c54c8ca105756fc8587b574429dce43497182aefde02dcd4d809744e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - DESeq2
  - countsimQC
  - dyngen
  - edgeR
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# edger-deseq2-dispersion-estimation

## Summary

Jointly estimates edgeR dispersions and DESeq2 dispersion estimates (dispGeneEst/dispFit/dispFinal) on a reference single-cell RNA-seq count matrix and a dyngen-simulated count matrix, then uses countsimQC to render a unified comparison report verifying that the synthetic data share the real data's characteristics.

## When to use

Use when a dyngen simulation must be shown to match a real single-cell RNA-seq reference dataset (e.g. the zenodo_1443566 bone-marrow mesenchyme MCA counts) in library size, fraction of zeros, and dispersion characteristics.

## When NOT to use

- The count matrices are not paired as a dyngen simulation versus its real reference dataset — countsimQC comparison is anchored to dyngen's realcounts emulation, not a generic count-matrix comparison
- The data are already normalised/log-transformed rather than raw counts, since edgeR TMM normalisation factors, log2 CPM with prior.count = 2, and DESeq2 dispersion estimation all require raw count input
- The goal is a single-dataset quality check rather than a two-dataset comparison of data characteristics

## Inputs

- Reference single-cell RNA-seq count matrix downloaded via the dyngen realcounts table (zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca)
- dyngen-simulated count matrix generated with backbone_bifurcating_loop matched to the reference dimensions (num_cells, num_tfs, num_targets, num_hks)
- DESeq2 DESeqDataSet objects wrapping both count matrices with design ~1

## Outputs

- Rendered countsimQC report of comparative ggplot2 panels (data-set dimensions, edgeR BCV and DESeq2 dispersion plots, mean-variance plots, library-size and fraction-zero histograms, correlation histograms, scatter plots)
- edgeR dispersions (tagwise/common/trended) and DESeq2 dispGeneEst/dispFit/dispFinal for both datasets
- Sample and feature Spearman correlations computed with maxNForCorr = 500

## How to apply

Download the reference dataset via the dyngen realcounts table, selecting 'zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca', and run a dyngen simulation with backbone_bifurcating_loop whose num_cells, num_tfs, num_targets and num_hks match the reference dimensions, with experiment_snapshot sampling from the realcounts. Wrap both count matrices into DESeq2 DESeqDataSet objects with a design of ~1. Use countsimQC internal helpers calculateDispersionsddsList, calculateSampleCorrs and calculateFeatureCorrs with maxNForCorr = 500 to compute edgeR dispersions and Spearman correlations. Summarize sample characteristics (library size, fraction of zeros, TMM normalisation factors, effective library sizes) and feature characteristics (tagwise/common/trended edgeR dispersion; mean and variance of log2 CPM with prior.count = 2; DESeq2 dispGeneEst, dispFit, dispFinal). Render the comparative ggplot2 panels — data-set dimensions, edgeR BCV and DESeq2 dispersion plots, mean-variance plots, library-size and fraction-zero histograms, sample- and feature-correlation histograms, and library-size vs fraction-zero and mean-expression vs fraction-zero scatter plots — into a countsimQC report.

## Related tools

- **dyngen** (Generates the simulated count matrix and provides the realcounts table used to download the reference dataset and match simulation dimensions) — https://github.com/dynverse/dyngen
- **countsimQC** (Provides internal helpers (calculateDispersionsddsList, calculateSampleCorrs, calculateFeatureCorrs) and renders the unified comparative report)
- **DESeq2** (Wraps the count matrices as DESeqDataSets (design ~1) and supplies dispGeneEst/dispFit/dispFinal dispersion estimates)
- **edgeR** (Supplies TMM normalisation factors, tagwise/common/trended dispersions and BCV plots for the comparison)
- **ggplot2** (Rendering engine for the comparative panels in the countsimQC report)

## Examples

```
name_realcounts <- "zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca"; dds <- DESeqDataSetFromMatrix(counts, colData, design = ~1)
```

## Evaluation signals

- The rendered report shows that dyngen produces datasets with highly similar data characteristics to the reference dataset, matching the Supplementary Note 1 result generated with countsimQC
- Data-set dimensions panel confirms num_cells, num_tfs, num_targets and num_hks of the simulation match the reference dataset dimensions
- edgeR BCV and DESeq2 dispersion plots show tagwise/common/trended and dispGeneEst/dispFit/dispFinal estimates overlapping between simulated and real data
- Sample- and feature-correlation histograms (Spearman, maxNForCorr = 500) show comparable correlation distributions across the two datasets
- Library-size and fraction-of-zero histograms and their scatter plots against TMM normalisation factors and mean log2 CPM (prior.count = 2) show matched distributions

## Limitations

- dyngen assumes cells are well-mixed and independent from each other, which the authors flag as a major assumption in the discussion
- No software version numbers were reported for any of the tools (dyngen, countsimQC, DESeq2, edgeR) used in the comparison
- Validation is anchored to a single named reference dataset (zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca) from a repository of single-cell RNA-seq datasets

## Evidence

- [intro] Emulation of single-cell omics profiling with real reference data: "Finally, real reference datasets are used to emulate single-cell omics profiling protocols."
- [methods] Reference dataset identifier used by the comparison: "name_realcounts <- "zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca""
- [methods] Repository of real datasets and dyngen grounding: "A real dataset is downloaded from a repository of single-cell RNA-seq datasets49."
- [readme] Official vignette for this comparison workflow: "- [Advanced: Comparison to
  reference dataset](https://dyngen.dynverse.org/articles/advanced_topics/comparison_reference.html)"
