---
name: ggplot2-comparative-panel-visualization
description: Use when a synthetic single-cell count matrix must be visually validated
  against a real reference dataset by plotting comparative panels of dispersions,
  library sizes, fraction of zeros, and sample/feature correlations.
when_to_use_negative:
- The goal is quantitative simulation of dynamic cellular processes rather than visual
  quality comparison of count-matrix characteristics
- The two count matrices differ substantially in dimensions (num_cells, num_tfs, num_targets,
  num_hks not matched to the reference), making dimension-comparison panels uninformative
- Data are not count matrices amenable to edgeR/DESeq2 dispersion estimation (e.g.
  already normalised or non-integer data)
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3170
tools:
- name: ggplot2
  role: Renders the comparative panels (dispersion, mean-variance, histogram, and
    scatter plots) comparing simulated and reference data characteristics
- name: countsimQC
  role: Provides internal helpers (calculateDispersionsddsList, calculateSampleCorrs,
    calculateFeatureCorrs) that generate the statistics visualised in the panels
- name: dyngen
  role: Produces the simulated count matrix being compared against the real reference
    dataset
  repo: https://github.com/dynverse/dyngen
- name: DESeq2
  role: Wraps both count matrices as DESeqDataSet objects (design ~1) for dispersion
    estimation
- name: edgeR
  role: Source of tagwise/common/trended dispersion estimates and TMM normalisation
    factors plotted in the BCV panels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:d818c7e64ec83256352cecbe069301e47cd409228ca11ad738e24c05c53eed3d
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
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ggplot2-comparative-panel-visualization

## Summary

Render side-by-side ggplot2 comparison panels that show whether dyngen-simulated single-cell count data match a real reference dataset's characteristics, mirroring the countsimQC-based Supplementary Note 1 result of the dyngen manuscript.

## When to use

Use when a synthetic single-cell count matrix must be visually validated against a real reference dataset by plotting comparative panels of dispersions, library sizes, fraction of zeros, and sample/feature correlations.

## When NOT to use

- The goal is quantitative simulation of dynamic cellular processes rather than visual quality comparison of count-matrix characteristics
- The two count matrices differ substantially in dimensions (num_cells, num_tfs, num_targets, num_hks not matched to the reference), making dimension-comparison panels uninformative
- Data are not count matrices amenable to edgeR/DESeq2 dispersion estimation (e.g. already normalised or non-integer data)

## Inputs

- Reference single-cell RNA-seq count matrix (e.g. zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca from the dyngen realcounts table)
- Simulated dyngen count matrix generated with backbone_bifurcating_loop and dimensions matched to the reference
- DESeq2 DESeqDataSet objects (design ~1) wrapping both count matrices
- Precomputed edgeR dispersions and Spearman correlation vectors from countsimQC helpers

## Outputs

- Rendered comparative ggplot2 report panels: data-set dimensions, edgeR BCV and DESeq2 dispersion plots, mean-variance plots, library-size and fraction-zero histograms, sample- and feature-correlation histograms, and library-size vs fraction-zero and mean-expression vs fraction-zero scatter plots
- Summaries of sample characteristics (library size, fraction of zeros, TMM normalisation factors, effective library sizes) and feature characteristics (tagwise/common/trended dispersion, mean and variance of log2 CPM)

## How to apply

After computing edgeR dispersions (via countsimQC's calculateDispersionsddsList) and Spearman sample/feature correlations (calculateSampleCorrs and calculateFeatureCorrs with maxNForCorr = 500) on DESeq2 DESeqDataSet objects built from the reference and simulated matrices (design ~1), assemble ggplot2 panels covering: data-set dimensions, edgeR BCV and DESeq2 dispersion plots (dispGeneEst/dispFit/dispFinal), mean-variance plots of log2 CPM with prior.count = 2, library-size and fraction-of-zero histograms, sample- and feature-correlation histograms, and scatter plots of library-size vs fraction-of-zeros and mean-expression vs fraction-of-zeros. Summarize sample characteristics (library size, fraction of zeros, TMM normalisation factors, effective library sizes) and feature characteristics (tagwise/common/trended dispersion) alongside the plots. The rationale is that dyngen emulates single-cell omics profiling using real reference datasets, so the rendered panels are the visual evidence that simulated and real data characteristics align.

## Related tools

- **ggplot2** (Renders the comparative panels (dispersion, mean-variance, histogram, and scatter plots) comparing simulated and reference data characteristics)
- **countsimQC** (Provides internal helpers (calculateDispersionsddsList, calculateSampleCorrs, calculateFeatureCorrs) that generate the statistics visualised in the panels)
- **dyngen** (Produces the simulated count matrix being compared against the real reference dataset) — https://github.com/dynverse/dyngen
- **DESeq2** (Wraps both count matrices as DESeqDataSet objects (design ~1) for dispersion estimation)
- **edgeR** (Source of tagwise/common/trended dispersion estimates and TMM normalisation factors plotted in the BCV panels)

## Examples

```
countsimQC::calculateDispersionsddsList(ddsList = list(reference = dds_ref, simulated = dds_sim), maxNForDisp = 500); ggplot2::ggplot(disp_df, aes(mean, disp, colour = dataset)) + geom_line()
```

## Evaluation signals

- The rendered report shows that dyngen produces datasets with highly similar data characteristics to the reference dataset, matching the Supplementary Note 1 countsimQC result
- Data-set dimensions panel shows simulated num_cells, num_tfs, num_targets and num_hks matched to the reference dataset's dimensions
- EdgeR BCV and DESeq2 dispersion (dispGeneEst/dispFit/dispFinal) curves overlap visually between simulated and reference matrices
- Library-size and fraction-of-zero histograms, and their pairwise scatter plots, show comparable distributions for both datasets
- Correlation histograms are computed with maxNForCorr = 500 Spearman correlations for both sample and feature correlations

## Limitations

- Validation is visual/characteristic-based; the panels demonstrate similarity of data characteristics, not biological equivalence
- Requires the specific real reference dataset downloaded via the dyngen realcounts table; results are anchored to that dataset
- The dyngen article assumes cells are well-mixed and independent, which the comparison panels do not capture

## Evidence

- [other] Use countsimQC internal helpers to compute dispersions and correlations: [evidence span withheld — anchor status: not_found]
- [other] Render comparative ggplot2 panels: [evidence span withheld — anchor status: not_found]
- [intro] dyngen anchors simulations to real data: "Finally, real reference datasets are used to emulate single-cell omics profiling protocols."
- [other] Wrap count matrices into DESeq2 objects: [evidence span withheld — anchor status: not_found]
- [readme] Reproduction code is available in the manuscript repository: [evidence span withheld — anchor status: external_doc]
