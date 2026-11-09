---
name: single-cell-rna-seq-count-matrix-quality-comparison
description: Use when a simulated scRNA-seq count matrix must be shown to have data
  characteristics (library size, zero fraction, dispersion, correlations) matching
  a real reference count matrix.
when_to_use_negative:
- The goal is evaluating dyngen's biological simulation fidelity (trajectory, velocity
  or network-inference ground truth) rather than matching marginal count-data characteristics
  to a reference dataset
- Input data are bulk RNA-seq or non-count matrices, since the skill's edgeR TMM/BCV,
  DESeq2 dispersion and fraction-of-zero statistics are specific to sparse single-cell
  count data
- No real reference scRNA-seq dataset is available to compare against, as the skill
  is defined by dyngen's realcounts-anchored comparison
tools:
- name: dyngen
  role: Generates the simulated count matrix and provides the realcounts table for
    downloading the reference dataset; its parameters are matched to the reference
    dimensions
  repo: https://github.com/dynverse/dyngen
- name: countsimQC
  role: Provides internal helpers calculateDispersionsddsList, calculateSampleCorrs
    and calculateFeatureCorrs and the unified comparative quality report of synthetic
    vs real data characteristics
- name: DESeq2
  role: Wraps reference and simulated count matrices into DESeqDataSet objects (design
    ~1) and supplies dispGeneEst/dispFit/dispFinal dispersion estimates
- name: edgeR
  role: Supplies tagwise/common/trended dispersion, TMM normalisation factors and
    BCV estimates for the comparison
- name: ggplot2
  role: Renders the comparative panels of the countsimQC report
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:117bffd0ff5a867f7eaa3d6cc99c20095ff24bf213d029dad4eb986d98873335
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# single-cell-rna-seq-count-matrix-quality-comparison

## Summary

Validates that simulated single-cell count matrices match a real scRNA-seq reference by computing edgeR dispersions, DESeq2 dispersions, library sizes, fraction of zeros and Spearman correlations, and rendering comparative countsimQC/ggplot2 panels. Anchored in dyngen's emulation of single-cell omics profiling protocols using real reference datasets.

## When to use

Use when a simulated scRNA-seq count matrix must be shown to have data characteristics (library size, zero fraction, dispersion, correlations) matching a real reference count matrix.

## When NOT to use

- The goal is evaluating dyngen's biological simulation fidelity (trajectory, velocity or network-inference ground truth) rather than matching marginal count-data characteristics to a reference dataset
- Input data are bulk RNA-seq or non-count matrices, since the skill's edgeR TMM/BCV, DESeq2 dispersion and fraction-of-zero statistics are specific to sparse single-cell count data
- No real reference scRNA-seq dataset is available to compare against, as the skill is defined by dyngen's realcounts-anchored comparison

## Inputs

- Real reference scRNA-seq count matrix from the dyngen realcounts table (e.g. zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca)
- Simulated dyngen count matrix with dimensions (num_cells, num_tfs, num_targets, num_hks) matched to the reference
- DESeq2 DESeqDataSet objects built from both count matrices with design ~1

## Outputs

- countsimQC comparative report with ggplot2 panels: data-set dimensions, edgeR BCV and DESeq2 dispersion plots, mean-variance plots, library-size and fraction-zero histograms, sample- and feature-correlation histograms, library-size vs fraction-zero and mean-expression vs fraction-zero scatter plots
- Computed edgeR dispersions and Spearman sample/feature correlations (maxNForCorr = 500)
- Summaries of sample characteristics (library size, fraction of zeros, TMM normalisation factors, effective library sizes) and feature characteristics (tagwise/common/trended dispersion, log2 CPM mean/variance with prior.count = 2, DESeq2 dispGeneEst/dispFit/dispFinal)

## How to apply

Download a real reference scRNA-seq dataset via the dyngen realcounts table (e.g. 'zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca') and run a dyngen simulation whose num_cells, num_tfs, num_targets and num_hks match the reference dimensions with experiment_snapshot sampling. Wrap both count matrices into DESeq2 DESeqDataSet objects with a design of ~1. Use countsimQC internal helpers calculateDispersionsddsList, calculateSampleCorrs and calculateFeatureCorrs (maxNForCorr = 500) to compute edgeR dispersions and Spearman correlations. Summarize sample characteristics (library size, fraction of zeros, TMM normalisation factors, effective library sizes) and feature characteristics (tagwise/common/trended dispersion, mean and variance of log2 CPM with prior.count = 2, DESeq2 dispGeneEst/dispFit/dispFinal), then render the comparative ggplot2 panels (BCV and dispersion plots, mean-variance plots, library-size and fraction-zero histograms, correlation histograms, scatter plots). The rationale is that dyngen's third pipeline stage uses real reference datasets to emulate single-cell omics profiling, so the countsimQC report verifies the simulation end-to-end.

## Related tools

- **dyngen** (Generates the simulated count matrix and provides the realcounts table for downloading the reference dataset; its parameters are matched to the reference dimensions) — https://github.com/dynverse/dyngen
- **countsimQC** (Provides internal helpers calculateDispersionsddsList, calculateSampleCorrs and calculateFeatureCorrs and the unified comparative quality report of synthetic vs real data characteristics)
- **DESeq2** (Wraps reference and simulated count matrices into DESeqDataSet objects (design ~1) and supplies dispGeneEst/dispFit/dispFinal dispersion estimates)
- **edgeR** (Supplies tagwise/common/trended dispersion, TMM normalisation factors and BCV estimates for the comparison)
- **ggplot2** (Renders the comparative panels of the countsimQC report)

## Examples

```
name_realcounts <- "zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca"; countsimQC::countsimQCReport(ddsList = list(real = dds_real, sim = dds_sim), outputFile = "comparison.html")
```

## Evaluation signals

- The rendered countsimQC report shows dyngen produces datasets with highly similar data characteristics to the reference dataset, matching the Supplementary Note 1 result
- Simulated matrix dimensions (num_cells, num_tfs, num_targets, num_hks) match the dimensions of the downloaded reference dataset
- Both matrices were processed identically: DESeqDataSet objects with design ~1, dispersions via calculateDispersionsddsList, correlations via calculateSampleCorrs/calculateFeatureCorrs with maxNForCorr = 500
- Feature-characteristic summaries include all three dispersion flavors (tagwise/common/trended from edgeR and dispGeneEst/dispFit/dispFinal from DESeq2)
- Log2 CPM mean/variance summaries used prior.count = 2 as specified

## Limitations

- dyngen assumes cells are well-mixed and independent from each other, which constrains what the comparison can validate about spatial or cell-cell interaction effects
- dyngen's simulations anchor to whatever real dataset is downloaded from the repository of single-cell RNA-seq datasets; matching is only to that reference's characteristics
- The reference dataset repository is referenced generically in the methods ('A real dataset is downloaded from a repository of single-cell RNA-seq datasets') and no software version numbers are reported for any tool

## Evidence

- [intro] dyngen anchors simulations to real data: "Finally, real reference datasets are used to emulate single-cell omics profiling protocols."
- [methods] realcounts reference dataset identifier: "name_realcounts <- "zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca""
- [readme] dyngen published and reproducible: "dyngen is a novel, multi-modal simulation engine for studying dynamic cellular processes at single-cell resolution."
- [methods] reference dataset source named generically: "A real dataset is downloaded from a repository of single-cell RNA-seq datasets49."
