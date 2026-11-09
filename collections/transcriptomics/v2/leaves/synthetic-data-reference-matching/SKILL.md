---
name: synthetic-data-reference-matching
description: Use when a simulated single-cell dataset must be shown to have count-level
  characteristics (library sizes, dropouts, dispersions, correlations) comparable
  to a real scRNA-seq reference dataset.
when_to_use_negative:
- When the goal is benchmarking downstream methods (trajectory inference, RNA velocity,
  CSNI) rather than validating that simulation output matches real-data characteristics
- When no real reference count dataset with comparable dimensions is available to
  anchor the simulation
- 'When the data are bulk RNA-seq: dyngen''s profiling emulation targets single-cell
  characteristics such as low library sizes and high dropout rates'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: dyngen
  role: Generates the simulated single-cell count matrix, using real reference datasets
    to emulate single-cell omics profiling protocols
  repo: https://github.com/dynverse/dyngen
- name: countsimQC
  role: Produces the unified comparative quality report of reference vs simulated
    count data characteristics
- name: DESeq2
  role: Wraps both count matrices into DESeqDataSet objects (design ~1) and supplies
    dispGeneEst/dispFit/dispFinal dispersion estimates
- name: edgeR
  role: Supplies tagwise/common/trended dispersion and TMM normalisation factors via
    calculateDispersionsddsList
- name: ggplot2
  role: Renders the comparative visualisation panels of the countsimQC report
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:ebda0be9b3822a1414346b212edaf3fddd2341904f21976fef8f5964640e3949
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# synthetic-data-reference-matching

## Summary

A validation skill that anchors simulated single-cell data to real data by wrapping both the reference count matrix and the synthetic (dyngen) count matrix into DESeq2 DESeqDataSet objects and comparing their data characteristics with countsimQC, demonstrating that the simulation emulates single-cell omics profiling protocols.

## When to use

Use when a simulated single-cell dataset must be shown to have count-level characteristics (library sizes, dropouts, dispersions, correlations) comparable to a real scRNA-seq reference dataset.

## When NOT to use

- When the goal is benchmarking downstream methods (trajectory inference, RNA velocity, CSNI) rather than validating that simulation output matches real-data characteristics
- When no real reference count dataset with comparable dimensions is available to anchor the simulation
- When the data are bulk RNA-seq: dyngen's profiling emulation targets single-cell characteristics such as low library sizes and high dropout rates

## Inputs

- Real reference scRNA-seq count matrix from the dyngen realcounts table (e.g. zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca)
- Simulated count matrix produced by dyngen with dimensions matched to the reference dataset

## Outputs

- Rendered countsimQC comparative report (ggplot2 panels) showing data-set dimensions, edgeR BCV and DESeq2 dispersion plots, mean-variance plots, library-size and fraction-zero histograms, sample- and feature-correlation histograms, and library-size vs fraction-zero / mean-expression vs fraction-zero scatter plots
- Computed sample characteristics: library size, fraction of zeros, TMM normalisation factors, effective library sizes
- Computed feature characteristics: tagwise/common/trended dispersion, mean and variance of log2 CPM (prior.count = 2), DESeq2 dispGeneEst/dispFit/dispFinal

## How to apply

Download a real reference scRNA-seq dataset via the dyngen realcounts mechanism (e.g. 'zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca') and run a dyngen simulation (e.g. backbone_bifurcating_loop) with num_cells, num_tfs, num_targets and num_hks matched to the reference dimensions, sampling experiment snapshots from the realcounts. Wrap both count matrices into DESeq2 DESeqDataSet objects with design ~1. Use countsimQC internal helpers calculateDispersionsddsList, calculateSampleCorrs and calculateFeatureCorrs (maxNForCorr = 500) to compute edgeR dispersions and Spearman correlations. Summarize sample characteristics (library size, fraction of zeros, TMM normalisation factors, effective library sizes) and feature characteristics (tagwise/common/trended dispersion; mean and variance of log2 CPM with prior.count = 2; DESeq2 dispGeneEst/dispFit/dispFinal) and render the comparative countsimQC ggplot2 report.

## Related tools

- **dyngen** (Generates the simulated single-cell count matrix, using real reference datasets to emulate single-cell omics profiling protocols) — https://github.com/dynverse/dyngen
- **countsimQC** (Produces the unified comparative quality report of reference vs simulated count data characteristics)
- **DESeq2** (Wraps both count matrices into DESeqDataSet objects (design ~1) and supplies dispGeneEst/dispFit/dispFinal dispersion estimates)
- **edgeR** (Supplies tagwise/common/trended dispersion and TMM normalisation factors via calculateDispersionsddsList)
- **ggplot2** (Renders the comparative visualisation panels of the countsimQC report)

## Examples

```
calculateDispersionsddsList(ddsList = list(real = DESeqDataSet(countMatrix(real_counts), ~1), sim = DESeqDataSet(countMatrix(sim_counts), ~1)), maxNForDisp = 500)
```

## Evaluation signals

- The rendered countsimQC report shows that dyngen produces datasets with highly similar data characteristics to the reference dataset, matching the Supplementary Note 1 result
- Sample characteristics (library size, fraction of zeros, TMM normalisation factors, effective library sizes) distributions overlap between reference and simulated data
- Feature characteristics (edgeR tagwise/common/trended dispersion, DESeq2 dispGeneEst/dispFit/dispFinal) show comparable trends between the two datasets
- Simulated dataset dimensions (num_cells, num_tfs, num_targets, num_hks) match the reference dataset dimensions
- Spearman correlation histograms (sample and feature, computed with maxNForCorr = 500) show similar correlation ranges for both datasets

## Limitations

- dyngen assumes cells are well-mixed and independent from each other, which constrains how well simulated sample-level characteristics can mimic real data
- No software version numbers are reported for any tool, so exact reproducibility of the comparison may depend on package versions
- The comparison is bounded by the specific reference dataset downloaded from the repository of single-cell RNA-seq datasets; matching other tissues/protocols requires selecting an appropriate realcounts entry

## Evidence

- [intro] Finally, real reference datasets are used to emulate single-cell omics profiling protocols.: "Finally, real reference datasets are used to emulate single-cell omics profiling protocols."
- [methods] realcounts name: "name_realcounts <- "zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca""
- [methods] A real dataset is downloaded from a repository of single-cell RNA-seq datasets.: "A real dataset is downloaded from a repository of single-cell RNA-seq datasets49."
- [readme] README advanced vignette: "- [Advanced: Comparison to reference
  dataset](https://dyngen.dynverse.org/articles/advanced_topics/comparison_reference.html)"
