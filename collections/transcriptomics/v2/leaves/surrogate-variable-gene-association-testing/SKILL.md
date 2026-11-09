---
name: surrogate-variable-gene-association-testing
description: Use when IA-SVA has returned a significant surrogate variable on scRNA-seq
  count data and you need the marker genes driving that SV (e.g., SV1 marker-gene
  table from alpha-cell counts).
when_to_use_negative:
- You have not yet run IA-SVA (or obtained a significant SV via its permutation-based
  significance index) — there is no SV to test genes against
- The goal is detecting the hidden factor itself rather than characterizing it; use
  the IA-SVA estimation step (iasva/fast_iasva) first
- Genes have not passed expression filtering (e.g., >5 reads in at least 3 cells),
  which the pipeline applies before SV analysis
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: iasva
  role: R package providing find_markers() to regress gene expression on the SV and
    compute R-squared / adjusted p-values, and study_R2() to tune R-squared thresholds
  repo: https://github.com/UcarLab/iasva
- name: fast_iasva
  role: Fast IA-SVA function that produces the surrogate variables (selecting important
    SVs by proportion of variation explained) whose gene associations are then tested
  repo: https://github.com/UcarLab/iasva
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:96bf96096cf5bce5f2f1f79495f44e477ce15d55c7a3a583481da2720bb4bb8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/iasva
  tools:
  - fast_iasva
  - iasva
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# surrogate-variable-gene-association-testing

## Summary

Identify genes significantly associated with a surrogate variable (SV) estimated by IA-SVA by regressing each gene's expression on the SV, retaining markers at adjusted p-value < 0.05 and R-squared > 0.3. This links hidden factors (e.g., doublets, cell-cycle stage) to an interpretable gene set.

## When to use

Use when IA-SVA has returned a significant surrogate variable on scRNA-seq count data and you need the marker genes driving that SV (e.g., SV1 marker-gene table from alpha-cell counts).

## When NOT to use

- You have not yet run IA-SVA (or obtained a significant SV via its permutation-based significance index) — there is no SV to test genes against
- The goal is detecting the hidden factor itself rather than characterizing it; use the IA-SVA estimation step (iasva/fast_iasva) first
- Genes have not passed expression filtering (e.g., >5 reads in at least 3 cells), which the pipeline applies before SV analysis

## Inputs

- scRNA-seq read count matrix (e.g., iasva_counts_test.Rds; genes x cells, alpha cells n = 101)
- Surrogate variable values (SV1) from a prior IA-SVA run
- Optional covariates used in the IA-SVA run (e.g., patient identity, geometric library size)

## Outputs

- Marker-gene table / count matrix for SV1 (e.g., sv1_marker_genes.csv) restricted to genes with adjusted p-value < 0.05 and R-squared > 0.3
- Per-gene R-squared and multiple-testing adjusted p-values against the SV
- Optional study_R2() diagnostic plot of R-squared threshold effects

## How to apply

Load the scRNA-seq read count matrix (e.g., iasva_counts_test.Rds, 101 alpha cells from healthy non-diabetic subjects) together with the SV values from the IA-SVA run. Apply the find_markers() function from the iasva package, which regresses each gene on the SV and computes the coefficient of determination (R-squared) for every gene. Retain genes passing the default significance cutoff (multiple-testing adjusted p-value < 0.05) and the R-squared cutoff (> 0.3). Optionally use the study_R2() function to visualize how different R-squared thresholds influence the number of marker genes before fixing the cutoff. Save the resulting marker-gene count matrix (e.g., alpha-cell markers like GCG and stress/contamination markers such as COL4A1/COL4A2) to a table such as sv1_marker_genes.csv. These genes can then be used for downstream clustering/visualization, since IA-SVA-captured genes improve grouping of cell subsets into distinct clusters.

## Related tools

- **iasva** (R package providing find_markers() to regress gene expression on the SV and compute R-squared / adjusted p-values, and study_R2() to tune R-squared thresholds) — https://github.com/UcarLab/iasva
- **fast_iasva** (Fast IA-SVA function that produces the surrogate variables (selecting important SVs by proportion of variation explained) whose gene associations are then tested) — https://github.com/UcarLab/iasva

## Examples

```
markers <- find_markers(counts, iasva.sv$sv[,1]); sv1_marker_genes <- rownames(markers)[markers$adj.pvals < 0.05 & markers$R2 > 0.3]; write.csv(counts[sv1_marker_genes,], "sv1_marker_genes.csv")
```

## Evaluation signals

- Output table exists and contains only genes with R-squared > 0.3 and multiple-testing adjusted p-value < 0.05 for the SV
- Known marker genes are recovered: alpha-cell markers such as GCG, and contaminant/stress markers such as COL4A1 and COL4A2 (27 genes significant for SV2 at FDR < 0.05, R2 > 0.6 in the published run)
- Genes identified for the SV separate the outlier cells the SV captured (e.g., six outlier alpha cells, stacked doublets)
- Per-gene statistics table includes both an R-squared value and an adjusted p-value for every tested gene
- Input genes were pre-filtered for adequate expression (>5 reads in at least 3 cells) before testing

## Limitations

- Compromised alpha cells detected via the SV (possible cell contamination) should be excluded from downstream analyses rather than interpreted biologically
- The R-squared cutoff is tunable; study_R2() should be used to assess threshold sensitivity since the article shows effects vary (e.g., R2 > 0.6 used for the SV2/FDR < 0.05 finding)
- The IA-SVA significance index is not an empirical p-value and has caveats in determining the number of hidden factors; a stringent SI cut-off is recommended when many unknown factors are expected, though no specific threshold is provided

## Evidence

- [methods] find_markers significance cutoff: "marker genes that are significantly associated with SV1 (multiple testing adjusted p-value < 0.05, default significance cutoff, and R-squared value > 0.3"
- [methods] study_R2 threshold tuning: "With the study_R2() function, users can visualize how different R-squared thresholds influence both factors"
- [results] alpha-cell dataset and SV1 finding: [evidence span withheld — anchor status: not_found]
- [results] contamination marker genes separating outlier cells: [evidence span withheld — anchor status: not_found]
- [methods] expressed-gene filter: "we used genes with >5 reads in at least 3 cells (i.e., a gene has to be above five in at least three cells to be kept)"
