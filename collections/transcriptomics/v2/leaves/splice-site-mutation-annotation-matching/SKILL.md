---
name: splice-site-mutation-annotation-matching
description: Use when you have SCISSOR shape-change results plus a public mutation-call
  repository and need to check whether annotated splice-site mutations are supported
  by significant coverage-shape changes.
when_to_use_negative:
- Mutation calls are point mutations within coding regions rather than splice-site
  annotations, since the matching logic targets splice-site entries specifically
- The RNA-seq cohort is too small for SCISSOR to fit a reliable gene-shape model (empirically
  ~30 samples is the smallest considered successfully)
- Samples show severe RNA degradation across many genes, as these were empirically
  removed before shape-change detection
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: SCISSOR
  role: Detects global and local shape changes in base-level RNA-seq coverage; its
    output object supplies the shape-change sample IDs and outlyingness scores matched
    against splice-site annotations
  repo: https://github.com/hyochoi/SCISSOR
- name: Integrative Genomic Viewer (IGV)
  role: Used to visually inspect read-level evidence for discordant splice-site annotations
    lacking shape-change support
- name: R
  role: Implementation environment for SCISSOR and for computing the annotation-vs-shape-change
    concordance proportion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:0063b693ba30bf38a2eddb7aa0fa7485746815fa5acbe957736c4a9cc8e0a790
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - Integrative Genomic Viewer (IGV)
  - R
  - SCISSOR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# splice-site-mutation-annotation-matching

## Summary

Cross-reference publicly reported splice-site mutation calls (e.g., TCGA HNSC TP53) against SCISSOR-detected global and local shape changes to quantify what fraction of annotated splice-site mutations have statistically significant RNA-structure support. This exposes discordant annotations that may be miscalled or lack read-level splicing evidence.

## When to use

Use when you have SCISSOR shape-change results plus a public mutation-call repository and need to check whether annotated splice-site mutations are supported by significant coverage-shape changes.

## When NOT to use

- Mutation calls are point mutations within coding regions rather than splice-site annotations, since the matching logic targets splice-site entries specifically
- The RNA-seq cohort is too small for SCISSOR to fit a reliable gene-shape model (empirically ~30 samples is the smallest considered successfully)
- Samples show severe RNA degradation across many genes, as these were empirically removed before shape-change detection

## Inputs

- SCISSOR results object for the gene of interest (ScissorOutput with $SC, $GSC$OS, $LSC$OS)
- Publicly reported mutation calls for the cohort (e.g., TCGA HNSC repository), filtered to splice-site annotations for the target gene
- Cohort sample identifiers linking mutation calls to RNA-seq samples (452 analyzed HNSC samples after excluding 70 degraded cases)

## Outputs

- Per-mutation table (e.g., splice_site_mutation_association.csv) with boolean shape-change association flag per annotated splice-site mutation
- Summary proportion file (e.g., splice_site_summary.csv) reporting the fraction of splice-site mutations associated with significant shape change

## How to apply

Load the SCISSOR results object and extract shape-change sample IDs from ScissorOutput$SC, with outlyingness scores from ScissorOutput$GSC$OS and ScissorOutput$LSC$OS (in the reference analysis, 26 GSCs and 24 LSCs in TP53 across 452 analyzed TCGA HNSC samples). Load the publicly reported mutation calls (e.g., TCGA HNSC MAF) and filter to TP53 mutations annotated as splice-site mutations. For each annotated splice-site mutation, flag whether its sample coincides with a SCISSOR-detected global or local shape change, then compute the proportion associated with a significant shape change (reported as 59%). For discordant cases, inspect the underlying read evidence (e.g., via IGV) since the paper found such calls were likely incorrectly annotated or had limited/no supporting reads for abnormal splicing. Write the per-mutation flag table and summary proportion to CSV for validation.

## Related tools

- **SCISSOR** (Detects global and local shape changes in base-level RNA-seq coverage; its output object supplies the shape-change sample IDs and outlyingness scores matched against splice-site annotations) — https://github.com/hyochoi/SCISSOR
- **Integrative Genomic Viewer (IGV)** (Used to visually inspect read-level evidence for discordant splice-site annotations lacking shape-change support)
- **R** (Implementation environment for SCISSOR and for computing the annotation-vs-shape-change concordance proportion)

## Examples

```
install_github("hyochoi/SCISSOR"); library(SCISSOR); shape_ids <- ScissorOutput$SC
```

## Evaluation signals

- The computed fraction of splice-site mutations with an associated significant shape change matches the published 59% value within rounding tolerance
- Every row of the per-mutation table carries a boolean shape-change association flag with no missing values
- The number of shape-change samples extracted from ScissorOutput$SC matches the reported counts (26 GSCs and 24 LSCs in 452 analyzed samples for TP53)
- Each splice-site mutation sample ID in the flag table maps to a sample in the analyzed RNA-seq cohort
- Nearly all discordant cases, when inspected, show likely incorrect annotation or limited/no supporting reads for abnormal splicing

## Limitations

- SCISSOR tuning parameters (log normalization, normality cutoff) may require optimization for significantly different RNA-seq protocols
- Omission of large intronic portions may miss important alternate splice sites and cryptic exons, limiting what shape changes can corroborate
- LSC detection results can depend on window size choice, which affects which local shape changes are available for matching

## Evidence

- [results] Only 59% of splice-site mutations have significant shape change: "only 59% of mutations annotated as splice site mutations in the public repository of HNSC mutations were associated with a statistically significant shape change"
- [results] In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs in TP53: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [methods] SCISSOR install via GitHub in R: "install_github("hyochoi/SCISSOR")"
- [readme] SCISSOR reads base-level pileup to find anomalous shapes: "SCISSOR considers a novel shape property of aligned short read data through a base-level pileup file."
- [results] 70 severely degraded HNSC cases excluded from analysis: "identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis"
