---
name: dmc-count-validation
description: Use when validating that an Illumina 450K/EPIC differential methylation
  analysis reproduces a published DMC count, especially counts restricted to promoter-proximal
  CpG feature groups such as TSS200, TSS1500, and 5'UTR.
when_to_use_negative:
- No published reference DMC count exists to compare against — this skill validates
  reproduction, not novel DMC discovery
- Differential results have not yet been annotated with genomic features, so TSS200/TSS1500/5'UTR
  membership cannot be assigned
- The upstream analysis is not an Illumina 450K/EPIC array differential methylation
  run (e.g., WGBS or RNA-seq data), for which minfi/ChAMP probe annotations do not
  apply
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3518
tools:
- name: methylR
  role: Graphical pipeline used to run the differential methylation analysis whose
    results are filtered and counted for validation against the published figure
  repo: https://github.com/JD2112/methylr
- name: minfi
  role: Provides the probe annotation used to assign each DMC its genomic feature
    group (TSS200, TSS1500, 5'UTR) prior to subsetting
- name: ChAMP
  role: Alternative differential analysis engine within methylR whose output DMC table
    can be annotated and filtered the same way
- name: R
  role: Environment in which the join, subset, count, and summary report are computed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:81c09f48e76b7dfa38b86106b0d108f4604c6df9de27f27080bebdbbe2dca730
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - ChAMP
  - R
  - methylR
  - minfi
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# dmc-count-validation

## Summary

Reproduces a published differential methylation finding by filtering DMC results to a defined genomic-feature subset (TSS200/TSS1500 + 5'UTR annotated CpGs) and comparing the resulting DMC count against the reported reference value, as demonstrated by methylR on GSE207426.

## When to use

Use when validating that an Illumina 450K/EPIC differential methylation analysis reproduces a published DMC count, especially counts restricted to promoter-proximal CpG feature groups such as TSS200, TSS1500, and 5'UTR.

## When NOT to use

- No published reference DMC count exists to compare against — this skill validates reproduction, not novel DMC discovery
- Differential results have not yet been annotated with genomic features, so TSS200/TSS1500/5'UTR membership cannot be assigned
- The upstream analysis is not an Illumina 450K/EPIC array differential methylation run (e.g., WGBS or RNA-seq data), for which minfi/ChAMP probe annotations do not apply

## Inputs

- Differential methylation results table (CpG IDs with test statistics) for GSE207426 from methylR/ChAMP or minfi
- CpG probe annotation table mapping CpG identifiers to genomic features (TSS200, TSS1500, 5'UTR, gene body, etc.) via minfi or ChAMP
- Published reference DMC counts to compare against (123,510 full-set DMCs; 26,626 TSS200/TSS1500 + 5'UTR subset DMCs)

## Outputs

- Filtered CpG subset restricted to TSS200, TSS1500, and 5'UTR features with their annotations
- Observed vs. reported DMC count comparison
- Summary report file (e.g., filtered_dmc_summary.csv) with the subset, feature annotations used, and pass/fail validation status

## How to apply

Load the differential methylation results table for the dataset (e.g., GSE207426) produced by an upstream methylR/ChAMP or minfi differential analysis. Join the CpG identifiers to their probe annotations to assign each CpG a genomic feature group (TSS200, TSS1500, 5'UTR, etc.) using the annotation facilities of minfi or ChAMP in R. Subset the table to CpGs whose feature annotation is TSS200, TSS1500, or 5'UTR, then count the filtered CpGs and compare against the published reference count (26,626 DMCs in the methylR paper's validation, from a full result of 123,510 DMCs). Write a summary report containing the filtered subset, the feature annotations used, and observed vs. reported counts (e.g., filtered_dmc_summary.csv). Treat a mismatch between the observed and reported counts as a failed validation (output_matches_reference).

## Related tools

- **methylR** (Graphical pipeline used to run the differential methylation analysis whose results are filtered and counted for validation against the published figure) — https://github.com/JD2112/methylr
- **minfi** (Provides the probe annotation used to assign each DMC its genomic feature group (TSS200, TSS1500, 5'UTR) prior to subsetting)
- **ChAMP** (Alternative differential analysis engine within methylR whose output DMC table can be annotated and filtered the same way)
- **R** (Environment in which the join, subset, count, and summary report are computed)

## Examples

```
Rscript -e 'dmc <- read.csv("GSE207426_dmc_results.csv"); ann <- minfi::getAnnotation(minfi::mapToGenome(minfi::read.metharray("."), extend=FALSE)); sub <- dmc[ann[dmc$cpg, "feature"] %in% c("TSS200","TSS1500","5'UTR"),]; write.csv(sub, "filtered_dmc_summary.csv"); nrow(sub) == 26626'
```

## Evaluation signals

- Filtered TSS200/TSS1500/5'UTR subset count equals the published reference of 26,626 DMCs (output_matches_reference)
- Full differential results table count matches the published 123,510 DMCs before subsetting, confirming the upstream analysis reproduced correctly
- Every retained CpG in the filtered subset carries a feature annotation of exactly TSS200, TSS1500, or 5'UTR
- Summary report contains the filtered subset, the feature annotations used, and both observed and reported DMC counts
- A count mismatch explicitly flags the run as failing the published-result comparison rather than silently passing

## Limitations

- methylR supports only AMD64 architecture — ARM64 chipset is not supported, so the validation environment must be x86-64
- After one complete analysis (ChAMP or minfi), the browser should be closed to clear temporary memory before restarting, or memory-residue artifacts may affect results
- Validation is exact-count reproduction; differences in probe annotation versions or upstream parameters can change the subset count and cause a false failure

## Evidence

- [results] methylR reproduced the published results on GSE207426, finding the same number of DMCs in the full dataset (123,510 DMCs) and in the TSS200/TSS1500 + 5'UTRs subset (26,626 DMCs): [evidence span withheld — anchor status: not_found]
- [methods] the acceptance criterion is that the filtered TSS200/TSS1500/5'UTR subset count reproduces the reported value of 26,626 DMCs (output_matches_reference): [evidence span withheld — anchor status: not_found]
- [readme] annotation of the genomic features resulting from the analysis: [evidence span withheld — anchor status: external_doc]
- [abstract] We evaluated a publicly available dataset and compared the published results with those obtained with our tool: "We evaluated a publicly available dataset and compared the published results with those obtained with our tool"
