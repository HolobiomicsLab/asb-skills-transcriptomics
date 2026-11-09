---
name: tcga-mutation-call-curation
description: Use when validating or auditing publicly reported TCGA mutation calls
  (MAF) against independent RNA-seq evidence, e.g. checking whether annotated splice-site
  mutations show aberrant transcript shapes in SCISSOR output.
when_to_use_negative:
- The mutation calls are from your own targeted sequencing with splice-junction read
  support already verified, rather than public TCGA MAF annotations needing independent
  RNA-shape validation
- RNA-seq coverage data for the samples is unavailable, so SCISSOR shape-change detection
  cannot provide independent evidence against which to curate the calls
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: SCISSOR
  role: Provides the independent RNA shape-change evidence (GSC/LSC sample sets and
    outlyingness scores) against which public TCGA mutation annotations are cross-referenced
    and curated
  repo: https://github.com/hyochoi/SCISSOR
- name: R
  role: Implementation language for matching TCGA barcodes between MAF calls and SCISSOR
    outputs and computing association proportions
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:98ce8b992a028a7fe35e781bdffcf6681454e4629cf5d9b25c123d772ddb571c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - R
  - SCISSOR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tcga-mutation-call-curation

## Summary

Curates publicly reported TCGA mutation calls (MAF) by cross-referencing them against SCISSOR-detected RNA shape changes, quantifying how many annotated mutations (e.g. splice site) coincide with statistically significant shape changes and flagging likely mis-annotated calls.

## When to use

Use when validating or auditing publicly reported TCGA mutation calls (MAF) against independent RNA-seq evidence, e.g. checking whether annotated splice-site mutations show aberrant transcript shapes in SCISSOR output.

## When NOT to use

- The mutation calls are from your own targeted sequencing with splice-junction read support already verified, rather than public TCGA MAF annotations needing independent RNA-shape validation
- RNA-seq coverage data for the samples is unavailable, so SCISSOR shape-change detection cannot provide independent evidence against which to curate the calls

## Inputs

- SCISSOR results object (ScissorOutput) with ScissorOutput$SC sample IDs, ScissorOutput$GSC$OS and ScissorOutput$LSC$OS outlyingness scores
- Publicly reported TCGA HNSC mutation calls (MAF file) with gene, variant classification, and genomic position per sample
- TCGA sample barcodes linking RNA-seq and mutation-call data

## Outputs

- Per-mutation table with boolean shape-change association flags (e.g. splice_site_mutation_association.csv)
- Summary proportion of annotated mutations associated with a significant shape change (e.g. splice_site_summary.csv)
- Per-sample classification of shape changes as 'known' vs 'previously unrecognized' aberrations

## How to apply

Load the SCISSOR results object and extract shape-change sample IDs from ScissorOutput$SC, with outlyingness scores from ScissorOutput$GSC$OS and ScissorOutput$LSC$OS (26 GSCs and 24 LSCs in the 452-sample TCGA HNSC TP53 analysis). Import the publicly reported TCGA HNSC MAF calls, subset to the analyzed samples, and filter to the mutation class of interest (e.g. TP53 mutations annotated as splice site). For each annotated mutation, flag whether its sample coincides with a SCISSOR-detected global or local shape change, and label each detected shape change as 'known' (coinciding with an annotated TP53 mutation) or 'previously unrecognized'. Compute the association proportion (expected: 59% of annotated splice-site mutations showed a significant shape change; 82%/18% known vs unrecognized split) and inspect discordant cases, since nearly every discordant call was likely incorrectly annotated or had limited or no supporting reads for abnormal splicing.

## Related tools

- **SCISSOR** (Provides the independent RNA shape-change evidence (GSC/LSC sample sets and outlyingness scores) against which public TCGA mutation annotations are cross-referenced and curated) — https://github.com/hyochoi/SCISSOR
- **R** (Implementation language for matching TCGA barcodes between MAF calls and SCISSOR outputs and computing association proportions)

## Examples

```
install_github("hyochoi/SCISSOR"); library(SCISSOR); flag <- samples_splice_maf %in% c(ScissorOutput$GSC$IDs, ScissorOutput$LSC$IDs)
```

## Evaluation signals

- Computed fraction of annotated splice-site mutations associated with a significant shape change matches the reported 59% within rounding tolerance
- Per-mutation flag table contains a boolean shape-change association flag on every row
- Classification table contains 452 sample rows for the TP53 HNSC analysis
- Reproduced known-aberration fraction matches the reported 82% (with 18% previously unrecognized)
- Sample counts reconcile with the article's 26 GSCs and 24 LSCs among the 452 analyzed samples

## Limitations

- Only 59% of publicly annotated splice-site mutations showed a significant shape change; discordant cases were likely incorrectly annotated or had limited or no supporting reads for abnormal splicing
- SCISSOR tuning parameters (log normalization, normality cutoff) require optimization for different RNA-seq protocols, which can affect which mutations appear concordant
- Samples with severe RNA degradation (70 of 522 HNSC cases) must be excluded first; retaining them can confound shape-change calls and false discordance

## Evidence

- [results] only 59% of mutations annotated as splice site mutations in the public repository of HNSC mutations were associated with a statistically significant shape change: "only 59% of mutations annotated as splice site mutations in the public repository of HNSC mutations were associated with a statistically significant shape change"
- [results] In 452 samples, SCISSOR identified 26 global shape changes (GSCs) and 24 local shape changes (LSCs) in TP53: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [results] As expected, many of the shape changes coincided with mutations (82%), most commonly at splice sites: "As expected, many of the shape changes coincided with mutations (82%), most commonly at splice sites"
- [results] we relied on publicly reported mutation calls from TCGA as well as TCGA RNA-seq data: "we relied on publicly reported mutation calls from TCGA as well as TCGA RNA-seq data"
- [methods] SCISSOR is free software and available on Github: "`SCISSOR` is free software and available on [Github](https://github.com/hyochoi/SCISSOR)."
