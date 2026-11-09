---
name: fdr-multiple-testing-thresholding
description: Use when a spatial transcriptomics DE method (C-SIDE, SPARK-X) returns
  per-gene p-values across many genes and cell types, and significant gene calls must
  be tabulated at a controlled FDR level such as 1%.
when_to_use_negative:
- The analysis is a single-hypothesis test or a small pre-specified gene set where
  family-wise error is trivially controlled and FDR correction is unnecessary
- The downstream comparison requires strict family-wise error control (e.g., confirming
  one specific marker gene) rather than discovery of a gene set
- Uncorrected effect sizes (log2FC) alone are the target and no p-value multiplicity
  table exists to threshold
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: C-SIDE (spacexr)
  role: Produces per-cell-type spatial DE p-values (via run.CSIDE.nonparametric with
    fdr parameter) that are then thresholded at FDR 1%
  repo: https://github.com/dmcable/spacexr
- name: SPARK-X
  role: Provides global spatially DE gene p-values whose multiple-testing-adjusted
    calls (243 genes) are tabulated alongside C-SIDE results
- name: Giotto
  role: Example of adjusted-p thresholding in the same paper's pipeline (p.adj < 0.25
    for ligand-receptor signaling selection)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:a40bd9b3a257203a3989294645d2605fa31ed6cbe58c240a1ede9f421304ea3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dmcable/spacexr
  tools:
  - C-SIDE
  - C-SIDE (spacexr)
  - Giotto
  - SPARK-X
  - SpatialScope
  - spacexr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# FDR multiple-testing thresholding

## Summary

Apply false-discovery-rate (Benjamini–Hochberg-style) correction and explicit FDR cutoffs when calling significant genes in spatial differential expression analyses (e.g., C-SIDE cell-type-specific DE and SPARK-X global spatial DE), so that large multiplicity over ~1,938 imputed genes does not inflate false positives.

## When to use

Use when a spatial transcriptomics DE method (C-SIDE, SPARK-X) returns per-gene p-values across many genes and cell types, and significant gene calls must be tabulated at a controlled FDR level such as 1%.

## When NOT to use

- The analysis is a single-hypothesis test or a small pre-specified gene set where family-wise error is trivially controlled and FDR correction is unnecessary
- The downstream comparison requires strict family-wise error control (e.g., confirming one specific marker gene) rather than discovery of a gene set
- Uncorrected effect sizes (log2FC) alone are the target and no p-value multiplicity table exists to threshold

## Inputs

- Per-gene p-values/q-values from C-SIDE (run.CSIDE.nonparametric) on the SpatialScope-imputed MERFISH matrix (~1,938 genes)
- Per-gene p-values from SPARK-X global spatial DE test with cell-type labels as covariates
- Target FDR threshold (0.01 for the headline counts; fdr=0.2 used as the C-SIDE run parameter)

## Outputs

- Per-cell-type table of significant spatially DE genes at FDR 1%
- Tabulated count of significant genes before (63) versus after (293) SpatialScope imputation
- SPARK-X global results (243 spatially DE genes) written to spatial_de_results.csv

## How to apply

Run the DE tool first (e.g., C-SIDE via run.CSIDE.nonparametric on the SpatialScope-imputed MERFISH mouse1_slice180 matrix with gene_threshold=0.001, cell_type_threshold=10, and fdr=0.2), then collect per-cell-type gene p-values and q-values. Threshold the corrected values at the reported significance level — here FDR < 1% — per cell type, and separately for SPARK-X global tests where cell-type labels are used as covariates to exclude genes explained purely by cell-type spatial distribution. Report counts before versus after imputation (63 vs 293 significant cell-type-specific DE genes at FDR 1%; 243 globally spatial DE genes, ~2.3x the MERFISH-only count) so the effect of imputation on power is quantified, and write the per-cell-type C-SIDE results and SPARK-X global results to spatial_de_results.csv. The rationale is that with ~1,938 genes tested across 20 cell types, uncorrected p-value cutoffs would yield hundreds of false positives, so only FDR-adjusted calls are comparable to published figures (Fig. 6e-h).

## Related tools

- **C-SIDE (spacexr)** (Produces per-cell-type spatial DE p-values (via run.CSIDE.nonparametric with fdr parameter) that are then thresholded at FDR 1%) — https://github.com/dmcable/spacexr
- **SPARK-X** (Provides global spatially DE gene p-values whose multiple-testing-adjusted calls (243 genes) are tabulated alongside C-SIDE results)
- **Giotto** (Example of adjusted-p thresholding in the same paper's pipeline (p.adj < 0.25 for ligand-receptor signaling selection))

## Examples

```
Rscript -e 'library(spacexr); cside <- run.CSIDE.nonparametric(puck, cell_type_info, gene_threshold=0.001, cell_type_threshold=10, fdr=0.2); sig <- cside$results[cside$results$qval < 0.01, ]'
```

## Evaluation signals

- Significant-gene count at FDR 1% reproduces the reported 63 (original MERFISH) versus 293 (imputed, 1,938 genes total) cell-type-specific DE genes
- SPARK-X global count at the applied FDR threshold yields 243 genes, ~2.3x the MERFISH-only count
- All reported calls have q-value (adjusted p) below the stated cutoff, not raw p < 0.05
- Recovered significant genes include the expected spatial patterns (Ryr3 in layer-6 excitatory neurons, Lingo2 in upper layers)
- Output table spatial_de_results.csv contains per-cell-type C-SIDE results plus SPARK-X global results with consistent FDR thresholds

## Limitations

- The C-SIDE fdr parameter used in the reproduction run was 0.2, so headline FDR 1% counts require post-hoc tabulation of significant calls rather than reading the tool's default significance calls
- Only ~1,938 genes survive after SpatialScope imputation of the MERFISH panel, so FDR power comparisons are restricted to that gene universe
- Globally spatial DE calls must be interpreted with cell-type covariates, otherwise genes driven purely by cell-type spatial distribution inflate the count

## Evidence

- [results] Applying C-SIDE, 63 cell-type-specific spatially DE genes were detected in the original MERFISH genes at FDR 1%: [evidence span withheld — anchor status: not_found]
- [readme] C-SIDE identifies significant genes as a function of user-defined covariates: [evidence span withheld — anchor status: external_doc]
- [methods] Adjusted p-value thresholding is also used in the paper's ligand-receptor selection: "we selected top and reliable ligand-receptor signaling with the following threshold: p.adj < 0.25, abs(log2fc) > 0.1"
- [abstract] SpatialScope enables identification of spatially differentially expressed genes: "facilitating downstream analysis, including detecting cellular communication through ligand-receptor interactions, localizing cellular subtypes, and identifying spatially differentially expressed"
