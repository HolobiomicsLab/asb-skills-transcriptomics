---
name: hidden-factor-associated-gene-discovery
description: Use when an IA-SVA run has produced a significant surrogate variable
  (e.g., SV1 on alpha-cell scRNA-seq counts) and you need the marker genes that explain
  that hidden factor, such as contamination/doublet genes (COL4A1, COL4A2) or cell-cycle
  genes.
when_to_use_negative:
- No surrogate variable has been estimated yet — run IA-SVA first (e.g., fast_iasva
  with permutation-based significance index) to obtain the SV before calling find_markers()
- Genes have not passed the expression filter (genes with >5 reads in at least 3 cells
  are kept); applying find_markers() to unfiltered sparse counts inflates noise
- The goal is to determine the number of hidden factors rather than to characterize
  an already-detected factor's gene set
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: iasva
  role: R/Bioconductor package providing find_markers() to compute per-gene R-squared
    against the SV and study_R2() to tune the R-squared threshold
  repo: https://github.com/UcarLab/iasva
- name: fast_iasva
  role: Fast IA-SVA function that estimates the surrogate variables (SV1, SV2, ...)
    whose associated genes this skill then extracts
  repo: https://github.com/UcarLab/iasva
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:323c5b922d3c4cdc2acdabcc19ab2e7d8f61a96ba0bfa42e4d30d0b53c2577fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/iasva
  tools:
  - fast_iasva
  - iasva
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hidden-factor-associated-gene-discovery

## Summary

Identify the gene set driving a hidden factor by regressing every gene's scRNA-seq expression on an IA-SVA surrogate variable (SV) with the iasva package's find_markers() function, retaining genes with adjusted p-value < 0.05 and R-squared > 0.3. This delivers the genes 'associated with the detected hidden source' for downstream analyses such as outlier-cell identification.

## When to use

Use when an IA-SVA run has produced a significant surrogate variable (e.g., SV1 on alpha-cell scRNA-seq counts) and you need the marker genes that explain that hidden factor, such as contamination/doublet genes (COL4A1, COL4A2) or cell-cycle genes.

## When NOT to use

- No surrogate variable has been estimated yet — run IA-SVA first (e.g., fast_iasva with permutation-based significance index) to obtain the SV before calling find_markers()
- Genes have not passed the expression filter (genes with >5 reads in at least 3 cells are kept); applying find_markers() to unfiltered sparse counts inflates noise
- The goal is to determine the number of hidden factors rather than to characterize an already-detected factor's gene set

## Inputs

- scRNA-seq read count matrix for the cell subset (e.g., iasva_counts_test.Rds, human alpha cells, n = 101, Fluidigm C1)
- Surrogate variable values (e.g., SV1) estimated by an IA-SVA run
- Optionally, known covariates used in the IA-SVA run (e.g., patient identity, geometric library size)

## Outputs

- Marker-gene table/matrix of genes significantly associated with the SV (adjusted p-value < 0.05, R-squared > 0.3), e.g., sv1_marker_genes.csv
- R-squared / adjusted p-value statistics per gene for threshold tuning via study_R2()

## How to apply

Load the read-count matrix (e.g., iasva_counts_test.Rds: alpha cells from healthy non-diabetic subjects, n = 101) together with the surrogate variable values from the IA-SVA run. Regress each gene's expression on the SV using find_markers() from the iasva package, which computes the coefficient of determination (R-squared) for every gene against the SV. Retain genes passing the default significance cutoff (multiple-testing adjusted p-value < 0.05) and R-squared cutoff > 0.3; use the study_R2() function to visualize how different R-squared thresholds influence the number of selected markers before finalizing the cutoff. Save the surviving marker-gene count matrix (e.g., alpha-cell markers such as GCG plus stress/contamination markers) as sv1_marker_genes.csv. Interpret the marker set biologically: fibroblast-expressed genes like COL4A1/COL4A2 signal doublet/contamination, and the corresponding outlier cells should be flagged for exclusion.

## Related tools

- **iasva** (R/Bioconductor package providing find_markers() to compute per-gene R-squared against the SV and study_R2() to tune the R-squared threshold) — https://github.com/UcarLab/iasva
- **fast_iasva** (Fast IA-SVA function that estimates the surrogate variables (SV1, SV2, ...) whose associated genes this skill then extracts) — https://github.com/UcarLab/iasva

## Examples

```
sv1_markers <- iasva::find_markers(alpha_counts, iasva_res$SV1)
```

## Evaluation signals

- Output table sv1_marker_genes.csv exists and contains only genes with R-squared > 0.3 and multiple-testing adjusted p-value < 0.05 for SV1
- Recovered marker set includes biologically expected markers: GCG for alpha cells; fibroblast-expressed COL4A1/COL4A2 when the hidden factor reflects doublet/contamination (paper: 27 genes at FDR < 0.05, R2 > 0.6 separating six outlier alpha cells)
- Marker genes, when visualized (e.g., Spectral tSNE/Cell View), separate the outlier cells driving the SV
- Threshold choice is documented via study_R2() showing the effect of alternative R-squared cutoffs
- Genes had passed the expression filter (>5 reads in at least 3 cells) before marker testing

## Limitations

- IA-SVA detects heterogeneity from compromised alpha cells (cell contamination), and such cells should be excluded from downstream analyses — the marker gene set identifies, but does not itself remove, outlier cells
- The default R-squared cutoff (> 0.3) may need adjustment; users can visualize the influence of different thresholds with study_R2(), but no universally optimal threshold is prescribed
- The significance index (SI) used to decide how many SVs exist is not an empirical p-value; a stringent SI cut-off is recommended when many unknown factors are expected, yet no specific threshold is provided

## Evidence

- [methods] find_markers default cutoffs: "marker genes that are significantly associated with SV1 (multiple testing adjusted p-value < 0.05, default significance cutoff, and R-squared value > 0.3"
- [methods] study_R2 threshold tuning: "With the study_R2() function, users can visualize how different R-squared thresholds influence both factors"
- [methods] alpha-cell input data: "we use read counts of alpha cells from healthy (non-diabetic) subjects (n = 101)"
- [abstract] purpose of associated gene set: "IA- SVA delivers a set of genes associated with the detected hidden source to be used in downstream data analyses"
- [discussion] exclusion of compromised cells: "IA-SVA detects heterogeneity stemming from compromised alpha cells (which may be explained by cell contamination), which should be excluded from the downstream analyses"
- [readme] README framing of hidden factor detection: "identify a hidden factor for unwanted heterogeneity while adjusting for all known factors"
