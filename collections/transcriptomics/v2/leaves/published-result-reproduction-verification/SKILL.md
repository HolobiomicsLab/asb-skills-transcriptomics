---
name: published-result-reproduction-verification
description: Use when a pipeline (e.g., methylR on a public array dataset like GSE207426)
  must be validated by reproducing a published count or statistic from an existing
  differential methylation results table.
when_to_use_negative:
- No published reference value exists for the dataset, so no output_matches_reference
  comparison is possible
- Input is raw IDAT data rather than an already-produced differential methylation
  results table — run the methylR/ChAMP/minfi pipeline first
- The published result uses different genomic-feature filters (not TSS200/TSS1500/5'UTR),
  making the subset count incomparable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: methylR
  role: Shiny-based pipeline that produces the differential methylation results and
    performs genomic-feature annotation of DMCs for the dataset under validation
  repo: https://github.com/JD2112/methylr
- name: minfi
  role: Alternative engine within methylR for differential analysis and probe annotation
    used to assign CpGs to TSS200/TSS1500/5'UTR groups
- name: ChAMP
  role: Alternative engine within methylR for differential analysis and probe annotation
    used to assign CpGs to genomic feature groups
- name: R
  role: Language in which the annotation join, subsetting, counting, and CSV summary
    are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:0dc542b33433ef8251d80e32ffa956e2961452ab46dc0729219edc92d9349ac3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - ChAMP
  - R
  - methylR
  - minfi
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# published-result-reproduction-verification

## Summary

Re-derive a published downstream statistic (e.g., a filtered DMC count such as the 26,626 TSS200/TSS1500 + 5'UTR DMCs from GSE207426) using methylR/ChAMP or minfi differential methylation output joined to probe annotations, and compare the reproduced value against the published figure as an acceptance criterion.

## When to use

Use when a pipeline (e.g., methylR on a public array dataset like GSE207426) must be validated by reproducing a published count or statistic from an existing differential methylation results table.

## When NOT to use

- No published reference value exists for the dataset, so no output_matches_reference comparison is possible
- Input is raw IDAT data rather than an already-produced differential methylation results table — run the methylR/ChAMP/minfi pipeline first
- The published result uses different genomic-feature filters (not TSS200/TSS1500/5'UTR), making the subset count incomparable

## Inputs

- Differential methylation results table for GSE207426 from methylR/ChAMP or minfi (CpG IDs plus statistics)
- CpG probe annotations mapping CpG identifiers to genomic feature groups (TSS200, TSS1500, 5'UTR, etc.) from minfi or ChAMP
- Published reference statistic to compare against (26,626 filtered DMCs; 123,510 total DMCs)

## Outputs

- filtered_dmc_summary.csv containing the filtered TSS200/TSS1500/5'UTR subset, the feature annotations used, and observed vs. reported DMC counts
- Pass/fail flag for the published-result comparison (output_matches_reference)

## How to apply

Load the differential methylation results table produced by the upstream methylR/ChAMP or minfi analysis (e.g., for GSE207426, the full result containing 123,510 DMCs). Join the CpG identifiers to their Illumina probe annotations using the annotation facilities of minfi or ChAMP in R, assigning each CpG a genomic feature group (TSS200, TSS1500, 5'UTR, etc.). Subset the results to CpGs annotated as TSS200, TSS1500, or 5'UTR and count them; the acceptance criterion is output_matches_reference — the filtered count must equal the published value (26,626 DMCs). A mismatch flags the run as failing the published-result comparison, indicating a divergence in upstream normalization, differential testing, or annotation.

## Related tools

- **methylR** (Shiny-based pipeline that produces the differential methylation results and performs genomic-feature annotation of DMCs for the dataset under validation) — https://github.com/JD2112/methylr
- **minfi** (Alternative engine within methylR for differential analysis and probe annotation used to assign CpGs to TSS200/TSS1500/5'UTR groups)
- **ChAMP** (Alternative engine within methylR for differential analysis and probe annotation used to assign CpGs to genomic feature groups)
- **R** (Language in which the annotation join, subsetting, counting, and CSV summary are executed)

## Examples

```
Rscript -e 'd <- read.csv("gse207426_dmc.csv"); a <- getAnnotation(getAnnotationObject(" IlluminaHumanMethylationEPIC")); s <- merge(d, a[,"UCSC_RefGroup_Name"], by=0); n <- sum(s$UCSC_RefGroup_Name %in% c("TSS200","TSS1500","5'UTR")); stopifnot(n == 26626)'
```

## Evaluation signals

- Filtered TSS200/TSS1500/5'UTR subset count equals the reported 26,626 DMCs (output_matches_reference)
- Full-dataset DMC count equals the reported 123,510 DMCs before filtering
- Every CpG in the filtered subset carries one of the expected feature annotations (TSS200, TSS1500, 5'UTR)
- filtered_dmc_summary.csv is written with the observed vs. reported counts and annotation provenance
- A mismatch in either count flags the run as failing the published-result comparison

## Limitations

- ARM64 chipset architecture is not supported by methylR, constraining where the validation pipeline can run
- Only one complete ChAMP or minfi analysis should run per browser session — close the browser to clear temporary memory before rerunning, or reproduced counts may be stale
- The check is specific to datasets with published reference figures; it cannot validate novel datasets without a reported value

## Evidence

- [results] methylR reproduced the published results on GSE207426, finding the same number of DMCs in the full dataset (123,510 DMCs) and in the TSS200/TSS1500 + 5'UTRs subset (26,626 DMCs), verifying the reported subset count.: [evidence span withheld — anchor status: not_found]
- [methods] Subset the results table, retaining only CpGs whose feature annotation is TSS200, TSS1500, or 5'UTR.: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract on evaluation.: "We evaluated a publicly available dataset and compared the published results with those obtained with our tool"
- [readme] README on run engines.: "a complete pipeline for the analysis of both 450K and EPIC Illumina arrays"
