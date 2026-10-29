---
name: shape-change-outlier-sample-coincidence-analysis
description: Use when you have a SCISSOR ScissorOutput object (sample IDs, GSC/LSC
  outlyingness scores) plus public mutation calls and need to test whether annotated
  variants like splice site mutations explain detected shape changes, or to audit
  mutation-call annotation quality.
when_to_use_negative:
- The goal is discovery of novel shape changes themselves rather than validating/auditing
  existing mutation annotations — run the SCISSOR GSC/LSC detection pipeline instead.
- The gene was excluded from downstream SCISSOR analysis (e.g., lowly expressed gene
  below the RSEM median/MAD percentile cutoffs, or among the 5802 genes (~28%) with
  many non-expressed samples).
- The cohort is too small for SCISSOR's cohort-level outlier model — a cohort of 30
  samples is the smallest considered successful, so coincidence testing on smaller
  panels is not supported.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
tools:
- name: SCISSOR
  role: Produces the ScissorOutput object whose $SC sample IDs and GSC/LSC outlyingness
    scores define the shape-change outliers to intersect with mutation calls; installed
    via install_github("hyochoi/SCISSOR")
  repo: https://github.com/hyochoi/SCISSOR
- name: Integrative Genomic Viewer (IGV)
  role: Used to manually inspect discordant splice site mutation calls for supporting
    reads of abnormal splicing
- name: RSEM
  role: Source of normalized expression values whose median and MAD percentile cutoffs
    define gene filtering before shape-change detection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:b4d13c965329a11e12395a674f92e6823c3dc6ed2443d58f93b8a1f2d83d8015
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - Integrative Genomic Viewer (IGV)
  - R
  - RSEM
  - SCISSOR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# shape-change-outlier-sample-coincidence-analysis

## Summary

Cross-references SCISSOR-detected RNA shape-change outlier samples (GSCs and LSCs, with outlyingness scores) against publicly reported mutation calls to quantify how often annotated mutations (e.g., TP53 splice site mutations in TCGA HNSC) coincide with statistically significant shape changes, and to flag discordant annotations.

## When to use

Use when you have a SCISSOR ScissorOutput object (sample IDs, GSC/LSC outlyingness scores) plus public mutation calls and need to test whether annotated variants like splice site mutations explain detected shape changes, or to audit mutation-call annotation quality.

## When NOT to use

- The goal is discovery of novel shape changes themselves rather than validating/auditing existing mutation annotations — run the SCISSOR GSC/LSC detection pipeline instead.
- The gene was excluded from downstream SCISSOR analysis (e.g., lowly expressed gene below the RSEM median/MAD percentile cutoffs, or among the 5802 genes (~28%) with many non-expressed samples).
- The cohort is too small for SCISSOR's cohort-level outlier model — a cohort of 30 samples is the smallest considered successful, so coincidence testing on smaller panels is not supported.

## Inputs

- SCISSOR ScissorOutput object for a gene (e.g., TP53) from the TCGA HNSC analysis, containing $SC sample IDs and $GSC$OS / $LSC$OS outlyingness scores
- Publicly reported TCGA HNSC mutation calls (MAF-style variant annotations, including Variant_Classification such as splice site)
- Cohort metadata identifying the 452 analyzed samples (522 HNSC tumors minus 70 severely degraded cases)

## Outputs

- Per-mutation flag table (e.g., splice_site_mutation_association.csv) with a boolean shape-change association flag for each annotated mutation
- Summary proportion of annotated mutations associated with a statistically significant shape change (e.g., splice_site_summary.csv, benchmark: 59% for TP53 splice site mutations)
- List of discordant samples for manual read-level review (e.g., in IGV)

## How to apply

Extract the shape-change sample IDs from ScissorOutput$SC and the outlyingness scores from ScissorOutput$GSC$OS and ScissorOutput$LSC$OS, derived from the 26 GSCs and 24 LSCs SCISSOR identified in the 452 analyzed TCGA HNSC samples. Load the publicly reported TCGA HNSC mutation calls and filter to the variant class of interest (e.g., TP53 mutations annotated as splice site mutations). For each annotated mutation, flag whether its sample coincides with a SCISSOR-detected global or local shape change in the same gene, then compute the proportion of mutations associated with a significant shape change. Validate against the published benchmark: only 59% of TP53 splice site mutations in the public HNSC repository were associated with a significant shape change, and in discordant cases the called mutation was likely incorrectly annotated or lacked supporting reads for abnormal splicing (inspect candidates in IGV). Rationale: SCISSOR recaptures known variants independently of mutation annotations, so disagreement between the two views exposes annotation errors rather than method failure.

## Related tools

- **SCISSOR** (Produces the ScissorOutput object whose $SC sample IDs and GSC/LSC outlyingness scores define the shape-change outliers to intersect with mutation calls; installed via install_github("hyochoi/SCISSOR")) — https://github.com/hyochoi/SCISSOR
- **Integrative Genomic Viewer (IGV)** (Used to manually inspect discordant splice site mutation calls for supporting reads of abnormal splicing)
- **RSEM** (Source of normalized expression values whose median and MAD percentile cutoffs define gene filtering before shape-change detection)

## Examples

```
install_github("hyochoi/SCISSOR"); library(SCISSOR); ids <- ScissorOutput$SC; os_g <- ScissorOutput$GSC$OS; os_l <- ScissorOutput$LSC$OS
```

## Evaluation signals

- The computed fraction of splice site mutations with an associated significant shape change matches the reported 59% for TP53 in TCGA HNSC within rounding tolerance.
- Every row of the per-mutation table carries a boolean shape-change association flag.
- The number of shape-change samples retrieved from ScissorOutput$SC is consistent with the published counts (26 GSCs and 24 LSCs among 452 analyzed samples).
- As a sanity check on concordance overall, roughly 82% of TP53 shape changes should coincide with mutations when intersecting in the reverse direction.
- Discordant samples, when reviewed in IGV, show incorrectly annotated calls or limited/no supporting reads for abnormal splicing.

## Limitations

- SCISSOR tuning parameters (log normalization, normality cutoff) may require optimization when a significantly different wet lab RNA-seq protocol is used, affecting which outliers are detected.
- GSC detection loses information about the order of base pairs, and LSC results can change with the choice of window size, so coincidence flags may shift with these settings.
- Omission of large intronic portions can miss alternate splice sites and cryptic exons, so a splice site mutation may be discordant because its effect lies outside the analyzed region; variable fragment length can also introduce artifactual shape-change signals.

## Evidence

- [results] Benchmark proportion of splice site mutations with significant shape change: "only 59% of mutations annotated as splice site mutations in the public repository of HNSC mutations were associated with a statistically significant shape change"
- [results] Shape-change counts in the analyzed cohort: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [results] Overall mutation concordance of shape changes: "As expected, many of the shape changes coincided with mutations (82%), most commonly at splice sites"
- [results] Manual review of discordant calls in IGV: "a subset of those reads are shown using the Integrative Genomic Viewer (IGV)"
- [readme] Package installation for reproducing the analysis: "install_github("hyochoi/SCISSOR")"
