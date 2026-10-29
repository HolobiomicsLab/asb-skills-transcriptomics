---
name: fdr-significance-threshold-filtering
description: Use when a differential expression or enrichment result table contains
  per-feature FDR-adjusted p-values and you must classify features as significant
  using a study-defined threshold (e.g., FDR < 0.1 for the salamander ortholog-level
  results).
when_to_use_negative:
- Input data are raw counts or unadjusted p-values — significance filtering must occur
  after FDR adjustment, not on raw p-values
- The comparison aims at functional pathway interpretation rather than significance
  classification — there a pathway is defined as significantly enriched only if FDR
  < 0.05 AND at least five DEGs are in the gene set
- The study uses a published threshold different from FDR < 0.1 — adopt that study's
  cutoff instead of this case study's to preserve comparability
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: ExpressAnalyst
  role: Web platform whose DE module returns FDR-adjusted p-values used for significance
    thresholding of ortholog-level results
- name: ExpressAnalystR
  role: Local R package (PerformDEAnal / PerformDEAnalMeta) that performs differential
    expression with FDR adjustment so thresholds can be applied on the resulting comp.res
    table
  repo: https://github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:f1332680ceb27b8247107adb13b66ca363fc52b6ca35f45d794be141f62877c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/ExpressAnalystR
  tools:
  - ExpressAnalyst
  - ExpressAnalystR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# FDR significance threshold filtering

## Summary

Defines significant hits (DEGs, enriched pathways, or duplicated ortholog results) by filtering on FDR-adjusted p-values, using context-specific cutoffs — e.g., FDR < 0.05 with |logFC| > 1.5 for DEGs, FDR < 0.1 for the salamander non-model case study, and FDR < 0.05 with ≥5 DEGs for pathway enrichment.

## When to use

Use when a differential expression or enrichment result table contains per-feature FDR-adjusted p-values and you must classify features as significant using a study-defined threshold (e.g., FDR < 0.1 for the salamander ortholog-level results).

## When NOT to use

- Input data are raw counts or unadjusted p-values — significance filtering must occur after FDR adjustment, not on raw p-values
- The comparison aims at functional pathway interpretation rather than significance classification — there a pathway is defined as significantly enriched only if FDR < 0.05 AND at least five DEGs are in the gene set
- The study uses a published threshold different from FDR < 0.1 — adopt that study's cutoff instead of this case study's to preserve comparability

## Inputs

- Ortholog-level or gene-level differential expression results table with FDR-adjusted p-values (and log fold changes where used)
- Per-species expression direction columns (e.g., for salamander ortholog groups)
- Threshold parameters from the source publication (FDR < 0.1 for salamander; FDR < 0.05 and |logFC| > 1.5 elsewhere)

## Outputs

- Filtered table of significant DEGs (FDR-adjusted p-value below the chosen cutoff)
- duplicated_ortholog_pairs.csv listing duplicated DE ortholog pairs (both members DE at FDR < 0.1 with distinct species-specific patterns)
- A single fraction value: proportion of significant results accounted for by duplicated ortholog pairs (~3% in the salamander case)

## How to apply

Load the result table (e.g., ortholog-level differential expression results with per-species FDR-adjusted p-values and expression directions), then apply the threshold adopted from the original publication — for the salamander case study, genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1, whereas the zebrafish/lobster benchmarks used FDR < 0.05 with absolute log fold change greater than 1.5. ExpressAnalyst computes these adjusted p-values using the False Discovery Rate (FDR) method after DE analysis (limma or equivalent via PerformDEAnal). For downstream questions such as quantifying duplicated DEGs (reads from the same gene mapping to similar but different orthologs across species), restrict the duplicated-pair analysis to pairs where both members pass the FDR < 0.1 cutoff and show distinct species-specific expression patterns, then compute the fraction of significant results they account for. Rationale: FDR adjustment controls the expected false-positive fraction across the thousands of ortholog/gene tests, and matching the original publication's threshold keeps results comparable.

## Related tools

- **ExpressAnalyst** (Web platform whose DE module returns FDR-adjusted p-values used for significance thresholding of ortholog-level results)
- **ExpressAnalystR** (Local R package (PerformDEAnal / PerformDEAnalMeta) that performs differential expression with FDR adjustment so thresholds can be applied on the resulting comp.res table) — https://github.com/xia-lab/ExpressAnalystR

## Examples

```
dataSets <- PerformDEAnal("estrogen.txt", "custom", "absent vs. present", "NA", "intonly"); sig <- dataSet$comp.res[dataSet$comp.res$fdr < 0.1, ]
```

## Evaluation signals

- Every retained significant result has FDR-adjusted p-value below the applied threshold (< 0.1 for the salamander case study)
- Where |logFC| > 1.5 is also used, no significant call violates the fold-change cutoff
- The duplicated-pairs table (duplicated_ortholog_pairs.csv) contains only pairs whose members are both DE at FDR < 0.1
- The computed fraction of significant results attributable to duplicated ortholog pairs approximates the reported ~3%
- Adjusted p-values were produced by FDR correction (as performed by ExpressAnalyst/ExpressAnalystR), not left as raw p-values

## Limitations

- Reads from the same gene may map to similar but different orthologs for different species, so care must be taken while interpreting cross-species significance results
- KO-based annotation has limited transcriptome coverage (e.g., only 76.16% of human protein-coding genes are annotated with KOs), so non-annotated genes cannot appear in FDR-filtered ortholog-level results
- The threshold is study-specific: the salamander analysis follows the original publication's FDR < 0.1 cutoff, which differs from the FDR < 0.05 / |logFC| > 1.5 criterion used elsewhere in the paper

## Evidence

- [discussion] Salamander DEG threshold: FDR-adjusted p < 0.1, following the original publication: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [discussion] General DEG significance criterion used in the paper: "Genes were defined as differentially expressed if their false discovery rate (FDR) adjusted p-values were less than 0.05 and the absolute log fold changes greater than 1.5."
- [discussion] Cross-species ortholog mapping caveat motivating the duplicated-DEG check: "it’s possible for reads from the same gene to be mapped to similar but different orthologs for different species, and care must be taken while interpreting the results"
- [readme] FDR adjustment performed by the DE analysis step in ExpressAnalystR: "Fold change is log2 transformed. Adjusted P-value using False Discovery Rate (FDR) method."
