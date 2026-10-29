---
name: library-size-and-dropout-characterization
description: Use when you need to verify that a simulated (synthetic) single-cell
  count matrix matches the data characteristics — library size, dropout/fraction-of-zeros,
  and dispersion — of a real single-cell RNA-seq reference dataset.
when_to_use_negative:
- When the goal is assessing biological correctness of a simulation (trajectory, GRN
  structure, velocities) rather than marginal count-data statistics — countsimQC only
  compares data characteristics
- When the datasets to compare are bulk RNA-seq rather than single-cell count matrices;
  the dispersion and dropout characteristics of interest are single-cell specific
- When you have fewer features/cells than the correlation subsampling assumes (maxNForCorr
  = 500); the helpers' defaults may need adjustment
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: countsimQC
  role: Generates the unified comparative report of data characteristics between simulated
    and reference count matrices
- name: DESeq2
  role: Provides DESeqDataSet containers and dispersion estimates (dispGeneEst/dispFit/dispFinal)
    for the comparison
- name: edgeR
  role: Supplies tagwise/common/trended dispersions, TMM normalisation factors and
    BCV estimates consumed by countsimQC
- name: ggplot2
  role: Renders the comparative plots in the countsimQC report
- name: dyngen
  role: Produces the simulated count matrix anchored to a real reference dataset,
    which is the input being validated
  repo: https://github.com/dynverse/dyngen
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:1cd5fa234d72c63056256f6f685215406a7f7f9004477fd1b0b07a5a25ca14e7
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Library-size and dropout characterization

## Summary

Characterizes and compares single-cell count-matrix characteristics — library sizes, TMM normalisation factors, fraction of zeros, and mean–variance/dispersion behaviour — between a simulated dataset (e.g. from dyngen) and a real single-cell RNA-seq reference using countsimQC with DESeq2 and edgeR statistics.

## When to use

Use when you need to verify that a simulated (synthetic) single-cell count matrix matches the data characteristics — library size, dropout/fraction-of-zeros, and dispersion — of a real single-cell RNA-seq reference dataset.

## When NOT to use

- When the goal is assessing biological correctness of a simulation (trajectory, GRN structure, velocities) rather than marginal count-data statistics — countsimQC only compares data characteristics
- When the datasets to compare are bulk RNA-seq rather than single-cell count matrices; the dispersion and dropout characteristics of interest are single-cell specific
- When you have fewer features/cells than the correlation subsampling assumes (maxNForCorr = 500); the helpers' defaults may need adjustment

## Inputs

- Real single-cell RNA-seq reference count matrix (e.g. downloaded via the dyngen realcounts table, 'zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca')
- Simulated count matrix (e.g. from a dyngen simulation with backbone_bifurcating_loop, dimensions matched to the reference)
- DESeq2 DESeqDataSet objects built from each count matrix with design ~1

## Outputs

- countsimQC comparative report with ggplot2 panels: data-set dimensions, edgeR BCV and DESeq2 dispersion plots, mean-variance plots, library-size and fraction-of-zeros histograms, sample- and feature-correlation histograms
- Sample characteristics table: library sizes, TMM normalisation factors, effective library sizes, fraction of zeros
- Feature characteristics table: tagwise/common/trended edgeR dispersions, DESeq2 dispGeneEst/dispFit/dispFinal, mean and variance of log2 CPM (prior.count = 2)
- Spearman correlation distributions for samples and features (maxNForCorr = 500)

## How to apply

Wrap both the reference and simulated count matrices into DESeq2 DESeqDataSet objects with a design of ~1. Use countsimQC's internal helpers calculateDispersionsddsList, calculateSampleCorrs and calculateFeatureCorrs (maxNForCorr = 500) to compute edgeR dispersions (tagwise/common/trended) and Spearman correlations, plus DESeq2 dispGeneEst/dispFit/dispFinal. Summarize sample characteristics (library size, fraction of zeros, TMM normalisation factors, effective library sizes) and feature characteristics (mean and variance of log2 CPM with prior.count = 2). Render the comparative ggplot2 panels: data-set dimensions, edgeR BCV and DESeq2 dispersion plots, mean-variance plots, library-size and fraction-zero histograms, sample- and feature-correlation histograms, and scatter plots of library-size vs fraction-zero and mean-expression vs fraction-zero. The rationale, per the dyngen paper, is that simulations should emulate single-cell omics profiling protocols anchored in real reference datasets, so the synthetic data must reproduce these statistical characteristics.

## Related tools

- **countsimQC** (Generates the unified comparative report of data characteristics between simulated and reference count matrices)
- **DESeq2** (Provides DESeqDataSet containers and dispersion estimates (dispGeneEst/dispFit/dispFinal) for the comparison)
- **edgeR** (Supplies tagwise/common/trended dispersions, TMM normalisation factors and BCV estimates consumed by countsimQC)
- **ggplot2** (Renders the comparative plots in the countsimQC report)
- **dyngen** (Produces the simulated count matrix anchored to a real reference dataset, which is the input being validated) — https://github.com/dynverse/dyngen

## Examples

```
dds_ref <- DESeq2::DESeqDataSetFromMatrix(ref_counts, data.frame(x=1), ~1); countsimQC::countsimQCReport(ddsList=list(ref=dds_ref, sim=dds_sim), outputFile="comparison.html")
```

## Evaluation signals

- The rendered countsimQC report shows the dyngen dataset has highly similar data characteristics to the reference dataset, matching the Supplementary Note 1 result
- Library-size and fraction-of-zero histograms of simulated and reference datasets overlap substantially
- edgeR BCV and DESeq2 dispersion curves (dispGeneEst/dispFit/dispFinal) of the simulated data track those of the reference data
- Simulated dataset dimensions match the reference (num_cells, num_tfs, num_targets, num_hks matched to reference dimensions)
- Sample- and feature-correlation histograms (Spearman, maxNForCorr = 500) show comparable distributions across the two datasets

## Limitations

- The article notes no software version numbers were reported for any tool, so exact package versions for reproducibility are unrecorded
- The reference dataset must be selected from the dyngen realcounts repository; the paper only states that 'A real dataset is downloaded from a repository of single-cell RNA-seq datasets'
- dyngen assumes cells are well-mixed and independent from each other, which shapes what characteristics can be expected to match

## Evidence

- [intro] Emulation of single-cell omics profiling using real reference datasets anchors simulations to real data characteristics: "Finally, real reference datasets are used to emulate single-cell omics profiling protocols."
- [methods] countsimQC wraps both matrices into DESeq2 objects for comparative statistics: [evidence span withheld — anchor status: not_found]
- [methods] countsimQC helpers compute edgeR dispersions and Spearman correlations with maxNForCorr = 500: [evidence span withheld — anchor status: not_found]
- [methods] Sample characteristics include library size and fraction of zeros: [evidence span withheld — anchor status: not_found]
- [results] Validation is similarity of characteristics between simulated and reference datasets: [evidence span withheld — anchor status: not_found]
- [readme] The dyngen manuscript repository contains all code to reproduce the analyses: [evidence span withheld — anchor status: external_doc]
