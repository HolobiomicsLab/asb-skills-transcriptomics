---
name: marker-gene-identification-via-rsq-regression
description: Use when an IA-SVA run has produced a significant surrogate variable
  (e.g., SV1 for 101 human alpha cells) and you need the marker genes driving that
  hidden factor for downstream interpretation.
when_to_use_negative:
- No surrogate variable has been estimated yet — run IA-SVA first (e.g., fast_iasva)
  to obtain the SV to which genes are regressed
- The putative 'markers' may stem from compromised/doublet cells or contamination
  (e.g., fibroblast genes in alpha cells) — such cells should be excluded from downstream
  analyses rather than treated as biological signal
- The hidden factor was not significant by IA-SVA's permutation-based significance
  index — an SV that does not explain meaningful variation will yield no reliable
  marker set
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: iasva
  role: Provides find_markers() to compute per-gene R-squared and adjusted p-values
    against the SV, and study_R2() to visualize the effect of different R-squared
    thresholds on marker selection
  repo: https://github.com/UcarLab/iasva
- name: fast_iasva
  role: Produces the surrogate variable (SV) values that each gene's expression is
    regressed against when identifying markers
  repo: https://github.com/UcarLab/iasva
- name: iasvaExamples
  role: R data package with worked tutorials (e.g., detecting hidden heterogeneity
    in human islet alpha cells) demonstrating the marker-identification workflow
  repo: https://github.com/dleelab/iasvaExamples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:3478461c871edc53081491298a8121b42b974eda6ad110ad63f7169ea2b5e8ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/iasva
  tools:
  - fast_iasva
  - iasva
  - iasvaExamples
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# marker-gene-identification-via-rsq-regression

## Summary

Identifies the set of genes significantly associated with a hidden factor captured by an IA-SVA surrogate variable by regressing each gene's expression on the SV with the find_markers() function of the iasva R package and retaining genes passing multiple-testing-adjusted p-value < 0.05 and R-squared > 0.3 thresholds.

## When to use

Use when an IA-SVA run has produced a significant surrogate variable (e.g., SV1 for 101 human alpha cells) and you need the marker genes driving that hidden factor for downstream interpretation.

## When NOT to use

- No surrogate variable has been estimated yet — run IA-SVA first (e.g., fast_iasva) to obtain the SV to which genes are regressed
- The putative 'markers' may stem from compromised/doublet cells or contamination (e.g., fibroblast genes in alpha cells) — such cells should be excluded from downstream analyses rather than treated as biological signal
- The hidden factor was not significant by IA-SVA's permutation-based significance index — an SV that does not explain meaningful variation will yield no reliable marker set

## Inputs

- scRNA-seq read count matrix (e.g., iasva_counts_test.Rds, human alpha cells, n = 101)
- Surrogate variable values (e.g., SV1) estimated by IA-SVA
- Optional known covariates already regressed out during IA-SVA (e.g., patient identity, geometric library size)

## Outputs

- Table of genes significantly associated with the SV with adjusted p-value < 0.05 and R-squared > 0.3 (e.g., sv1_marker_genes.csv)
- Optional per-gene R-squared and adjusted p-value statistics for tuning cutoffs via study_R2()

## How to apply

Load the scRNA-seq read count matrix (e.g., iasva_counts_test.Rds, alpha cells from healthy non-diabetic subjects) together with the surrogate variable values from the IA-SVA run. Apply find_markers() from the iasva package, which computes the coefficient of determination (R-squared) for every gene against the SV and a multiple-testing-adjusted p-value; retain genes with adjusted p-value < 0.05 (default significance cutoff) and R-squared > 0.3. If the cutoff choice is uncertain, run study_R2() first to visualize how different R-squared thresholds influence both the number of selected genes and their association strength; a stricter R-squared cutoff (e.g., > 0.6) can be used for a tighter marker set, as in the paper's alpha-cell analysis where 27 genes (FDR < 0.05, R2 > 0.6) including fibroblast-expressed COL4A1 and COL4A2 separated six outlier alpha cells. Export the resulting marker-gene count matrix (e.g., sv1_marker_genes.csv, containing markers such as GCG and related stress/contamination genes) for downstream clustering or visualization.

## Related tools

- **iasva** (Provides find_markers() to compute per-gene R-squared and adjusted p-values against the SV, and study_R2() to visualize the effect of different R-squared thresholds on marker selection) — https://github.com/UcarLab/iasva
- **fast_iasva** (Produces the surrogate variable (SV) values that each gene's expression is regressed against when identifying markers) — https://github.com/UcarLab/iasva
- **iasvaExamples** (R data package with worked tutorials (e.g., detecting hidden heterogeneity in human islet alpha cells) demonstrating the marker-identification workflow) — https://github.com/dleelab/iasvaExamples

## Examples

```
devtools::install_github("UcarLab/iasva"); library(iasva); sv1_marker_genes <- find_markers(alpha_counts, iasva_res$SVs)
```

## Evaluation signals

- Output table exists and contains only genes with R-squared > 0.3 and adjusted p-value < 0.05 for the SV
- Number of retained markers is sensitive to the R-squared cutoff; study_R2() plots should show the threshold used
- Known marker biology is recovered: in the alpha-cell example the marker set includes GCG and stress/contamination genes such as COL4A1 and COL4A2
- Marker scores (R-squared values) separate outlier cells from the rest — the paper's 27-gene set (FDR < 0.05, R2 > 0.6) separated six outlier alpha cells
- Marker set is usable downstream to improve visualization of single cells into their respective clusters

## Limitations

- The default cutoffs (adjusted p < 0.05, R-squared > 0.3) are guidelines; with many unknown factors expected, users must select a stringent significance-index cutoff, and no specific cut-off threshold is provided for that choice
- Markers may reflect technical heterogeneity (e.g., stacked doublet cells, fibroblast contamination) rather than biological cell state; compromised cells detected this way should be excluded from downstream analyses
- The SI is not an empirical p-value and caveats remain in determining the number of hidden factors underlying the marker selection

## Evidence

- [methods] find_markers significance cutoff and R-squared threshold: "marker genes that are significantly associated with SV1 (multiple testing adjusted p-value < 0.05, default significance cutoff, and R-squared value > 0.3"
- [methods] study_R2 for cutoff tuning: "With the study_R2() function, users can visualize how different R-squared thresholds influence both factors"
- [discussion] ia-SVA detects compromised cells whose markers reflect contamination: "IA-SVA detects heterogeneity stemming from compromised alpha cells (which may be explained by cell contamination), which should be excluded from the downstream analyses"
- [readme] iasva package availability: [evidence span withheld — anchor status: external_doc]
- [readme] tutorials demonstrating the workflow: [evidence span withheld — anchor status: external_doc]
