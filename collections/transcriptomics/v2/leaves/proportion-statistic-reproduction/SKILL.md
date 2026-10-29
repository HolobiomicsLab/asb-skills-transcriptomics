---
name: proportion-statistic-reproduction
description: Use when a published paper reports a concordance proportion (e.g. X%
  of annotated mutations showing a significant signal) and you must recompute it from
  stored per-sample calls and public mutation repositories to validate an analysis.
when_to_use_negative:
- Input does not contain per-sample outlier/shape-change calls that can be joined
  to sample-level mutation annotations (e.g. only aggregate gene-level summaries).
- The mutation annotation set lacks a variant class column distinguishing splice-site
  mutations from other TP53 variants.
- The analysis goal is discovery of new shape changes rather than validation/reproduction
  of a reported concordance statistic.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCISSOR
  role: Supplies the ScissorOutput object whose SC sample IDs and GSC$OS / LSC$OS
    outlyingness scores define which samples show statistically significant global
    or local TP53 shape changes
  repo: https://github.com/hyochoi/SCISSOR
- name: R
  role: Language used to load the ScissorOutput object, filter TCGA HNSC mutation
    calls to TP53 splice-site mutations, join by sample ID, compute the proportion,
    and write the CSV outputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:7a45a42729d7546fc1ff638739c16e681890a330662ce0369ffe01ce018b91f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - R
  - SCISSOR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# proportion-statistic-reproduction

## Summary

Reproduce a published proportion statistic (here: the 59% of TP53 splice-site mutations in TCGA HNSC associated with a statistically significant SCISSOR shape change) by intersecting a tool's outlier/shape-change sample calls with publicly reported mutation annotations and computing the concordant fraction.

## When to use

Use when a published paper reports a concordance proportion (e.g. X% of annotated mutations showing a significant signal) and you must recompute it from stored per-sample calls and public mutation repositories to validate an analysis.

## When NOT to use

- Input does not contain per-sample outlier/shape-change calls that can be joined to sample-level mutation annotations (e.g. only aggregate gene-level summaries).
- The mutation annotation set lacks a variant class column distinguishing splice-site mutations from other TP53 variants.
- The analysis goal is discovery of new shape changes rather than validation/reproduction of a reported concordance statistic.

## Inputs

- SCISSOR ScissorOutput results object for TP53 in TCGA HNSC (ScissorOutput$SC sample IDs; ScissorOutput$GSC$OS and ScissorOutput$LSC$OS outlyingness scores)
- Publicly reported TCGA HNSC mutation calls (MAF-style per-sample variant annotations from the TCGA repository)
- Known detector run parameters: 452 analyzed samples, 26 GSCs and 24 LSCs in TP53
- Published reference statistic (59% of annotated splice-site mutations associated with significant shape change)

## Outputs

- Per-mutation flag table splice_site_mutation_association.csv (one row per annotated splice-site mutation with a boolean shape-change association flag)
- Summary proportion file splice_site_summary.csv (computed fraction of splice-site mutations with associated significant shape change)
- Comparison of computed fraction against the published 59% value within rounding tolerance

## How to apply

First recompute (or load) the detector's per-sample calls: from the SCISSOR TP53 run on 452 of the 522 TCGA HNSC samples, extract shape-change sample IDs from ScissorOutput$SC, with outlyingness scores from ScissorOutput$GSC$OS and ScissorOutput$LSC$OS (26 GSCs and 24 LSCs). Then load the publicly reported TCGA HNSC mutation calls and filter to the event class of interest (TP53 variants annotated as splice site mutations). For each annotated splice-site mutation, flag a boolean whether its sample coincides with a SCISSOR-detected global or local shape change in TP53. Compute the proportion of splice-site mutations associated with a significant shape change and compare it against the published statistic (59%) within rounding tolerance; write the per-mutation flag table and the summary proportion to CSV. The rationale is that discordant cases (the remaining ~41%) should be inspected for misannotation or lack of supporting reads, since in nearly every discordant case the called mutation was likely incorrectly annotated or had limited or no supporting reads for abnormal splicing.

## Related tools

- **SCISSOR** (Supplies the ScissorOutput object whose SC sample IDs and GSC$OS / LSC$OS outlyingness scores define which samples show statistically significant global or local TP53 shape changes) — https://github.com/hyochoi/SCISSOR
- **R** (Language used to load the ScissorOutput object, filter TCGA HNSC mutation calls to TP53 splice-site mutations, join by sample ID, compute the proportion, and write the CSV outputs)

## Examples

```
install_github("hyochoi/SCISSOR"); library(SCISSOR); write.csv(flag_table, "splice_site_mutation_association.csv"); write.csv(summary_prop, "splice_site_summary.csv")
```

## Evaluation signals

- The computed fraction of splice-site mutations with an associated significant shape change matches the reported value of 59% within rounding tolerance.
- Every row of splice_site_mutation_association.csv carries a boolean shape-change association flag.
- The number of shape-change samples used (26 GSCs and 24 LSCs from 452 analyzed samples) matches the published SCISSOR TP53 run.
- The mutation set used contains only TP53 variants annotated as splice-site mutations in the public TCGA HNSC repository.
- Discordant cases, when inspected, show likely incorrect annotation or limited/no supporting reads for abnormal splicing, consistent with the paper's finding.

## Limitations

- Nearly every discordant case reflected likely incorrect annotation or limited/no supporting reads for abnormal splicing, so the proportion depends on annotation quality of the public repository.
- SCISSOR tuning parameters (log normalization, normality cutoff) may require optimization for different RNA-seq protocols, affecting which samples count as shape changes.
- LSC detection results can depend on the chosen window size, which may alter the sample sets used in the proportion computation.

## Evidence

- [results] 59% of annotated splice-site mutations show significant shape change: "only 59% of mutations annotated as splice site mutations in the public repository of HNSC mutations were associated with a statistically significant shape change"
- [results] SCISSOR TP53 run call counts: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [methods] Installation of the SCISSOR R package: "install_github("hyochoi/SCISSOR")"
- [readme] SCISSOR scope: unsupervised screening of structural alterations: "aims for unsupervised screening of a range of structural alterations in RNA-seq data"
