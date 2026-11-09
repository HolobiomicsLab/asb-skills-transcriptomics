---
name: per-sample-aberration-classification
description: Use when SCISSOR shape-change calls (per-sample GSC/LSC sample IDs) must
  be validated against an external mutation repository such as a TCGA MAF to label
  each call as a known or previously unrecognized aberration.
when_to_use_negative:
- Classifying the mutation repository's entries by shape-change status — the mapping
  is one-directional; only 59% of publicly annotated splice site mutations showed
  a statistically significant shape change
- Genes or cohorts with no SCISSOR shape-change calls, or with no externally reported
  mutation calls available for cross-referencing
- Samples outside the analyzed cohort (e.g., the 70 severely degraded HNSC cases excluded
  from downstream SCISSOR analysis)
tools:
- name: SCISSOR
  role: Produces the per-sample GSC/LSC shape-change calls (via ScissorOutput$GSC
    and ScissorOutput$LSC) that are cross-referenced against mutation calls
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:783871855cfd4002973ed5cbdc745abec00794a5e42ff2c09daec444020baa7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# per-sample-aberration-classification

## Summary

Classifies SCISSOR-detected RNA shape changes (GSCs and LSCs), sample by sample, as 'known' when they coincide with publicly reported mutation calls (e.g., TCGA HNSC MAF) and 'previously unrecognized' otherwise, to quantify what fraction of detected aberrations are novel. Applied to the 26 GSCs and 24 LSCs SCISSOR found at TP53 in 452 TCGA HNSC samples, yielding the 82%/18% known/novel split.

## When to use

Use when SCISSOR shape-change calls (per-sample GSC/LSC sample IDs) must be validated against an external mutation repository such as a TCGA MAF to label each call as a known or previously unrecognized aberration.

## When NOT to use

- Classifying the mutation repository's entries by shape-change status — the mapping is one-directional; only 59% of publicly annotated splice site mutations showed a statistically significant shape change
- Genes or cohorts with no SCISSOR shape-change calls, or with no externally reported mutation calls available for cross-referencing
- Samples outside the analyzed cohort (e.g., the 70 severely degraded HNSC cases excluded from downstream SCISSOR analysis)

## Inputs

- ScissorOutput object with ScissorOutput$GSC and ScissorOutput$LSC (per-sample shape-change calls for a gene)
- Publicly reported TCGA HNSC mutation calls in MAF format (gene, variant classification, genomic position per sample)
- TCGA sample barcodes for the analyzed cohort (452 matched samples)

## Outputs

- Per-sample classification table labeling each shape change (GSC or LSC) as 'known' or 'previously unrecognized'
- Mutation class annotation per coinciding sample (splice site vs other variant classifications)
- Summary fraction of known vs previously unrecognized aberrations (expected ~82% / ~18%)

## How to apply

Load the SCISSOR run results (ScissorOutput) and retrieve the sample IDs called as global shape changes (GSCs) and local shape changes (LSCs) for the target gene, e.g., the 26 GSCs and 24 LSCs at TP53 among the 452 analyzed HNSC cases (522 cohort minus 70 severely degraded samples). Import the publicly reported mutation calls (MAF file), subset to the samples with matched mutation data, and extract gene, variant classification, and genomic position per sample. Match each shape-change sample to its TCGA barcode in the MAF, check for a mutation in the target gene, and record the mutation class (splice site versus other classes such as exon skipping, intron retention, and in-frame indels). Label each detected shape change 'known' when it coincides with an annotated mutation and 'previously unrecognized' otherwise, producing a per-sample classification table. Summarize the fraction of known versus previously unrecognized aberrations and compare to the reported 82%/18% split; note that this direction matters — only 59% of mutations annotated as splice site mutations in the public HNSC repository were associated with a statistically significant shape change, so the cross-reference goes from shape change to mutation, not the reverse.

## Related tools

- **SCISSOR** (Produces the per-sample GSC/LSC shape-change calls (via ScissorOutput$GSC and ScissorOutput$LSC) that are cross-referenced against mutation calls) — https://github.com/hyochoi/SCISSOR

## Examples

```
install_github("hyochoi/SCISSOR"); library(SCISSOR); data(TOY_data)
```

## Evaluation signals

- Reproduced known-aberration fraction matches the reported 82% known / 18% previously unrecognized split
- Classification table contains 452 sample rows (the analyzed HNSC cohort after removing 70 degraded cases)
- All 26 GSCs and 24 LSCs at TP53 receive a 'known' or 'previously unrecognized' label
- Each sample coinciding with a mutation carries a variant-classification annotation, with splice-site class distinguished from others (exon skipping, intron retention, in-frame indels)
- Reverse-direction check: only ~59% of publicly annotated splice-site mutations should coincide with a significant shape change

## Limitations

- Only 59% of mutations annotated as splice site mutations in the public HNSC repository were associated with a statistically significant shape change, so concordance is incomplete in the mutation-to-shape-change direction
- SCISSOR tuning parameters (log normalization, normality cutoff) may require optimization for RNA-seq protocols different from TCGA's, affecting which shape changes are called
- The GSC detection algorithm loses information about the order of base pairs, and LSC results depend on the chosen window size, which can change the set of calls being classified

## Evidence

- [results] In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [results] 82% of TP53 shape changes coincided with mutations: "As expected, many of the shape changes coincided with mutations (82%), most commonly at splice sites"
- [results] Only 59% of splice site mutations in the public HNSC repository had a significant shape change: "only 59% of mutations annotated as splice site mutations in the public repository of HNSC mutations were associated with a statistically significant shape change"
- [readme] SCISSOR identifies known as well as novel aberrations (README): "As a result, `SCISSOR` identifies known as well as novel aberrations including abnormal splicing, intra-/intergenic deletions, small indels, alternative transcription start/termination."
