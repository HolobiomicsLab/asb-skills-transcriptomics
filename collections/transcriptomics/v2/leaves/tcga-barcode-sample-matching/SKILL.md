---
name: tcga-barcode-sample-matching
description: Use when RNA-seq-derived outlier samples (e.g. SCISSOR shape changes)
  must be validated against an external TCGA mutation repository by matching TCGA
  barcodes to label aberrations as known vs novel.
when_to_use_negative:
- Input is a raw MAF or variant list with no RNA-based outlier calls — there are no
  shape-change samples to cross-reference
- The RNA-seq cohort and the mutation repository are unmatched (no shared TCGA barcodes),
  so barcode matching cannot be performed
- The goal is de novo mutation calling from sequencing data rather than annotation
  of RNA-detected aberrations against existing calls
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SCISSOR
  role: Produces the sample outlier calls (global and local shape changes) whose sample
    IDs are matched to TCGA barcodes for known-vs-novel classification
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:a70d0e22fe6aea1c1b5ae77135a6428e2e0c8c8b2258704f666b37c8e2cb1e93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tcga-barcode-sample-matching

## Summary

Cross-references SCISSOR-detected RNA shape-change samples (GSC/LSC sample IDs) against publicly reported TCGA mutation calls (MAF) by TCGA barcode to classify each detected aberration as a known mutation or a previously unrecognized aberration, e.g. reproducing the 82%/18% TP53 split in TCGA HNSC.

## When to use

Use when RNA-seq-derived outlier samples (e.g. SCISSOR shape changes) must be validated against an external TCGA mutation repository by matching TCGA barcodes to label aberrations as known vs novel.

## When NOT to use

- Input is a raw MAF or variant list with no RNA-based outlier calls — there are no shape-change samples to cross-reference
- The RNA-seq cohort and the mutation repository are unmatched (no shared TCGA barcodes), so barcode matching cannot be performed
- The goal is de novo mutation calling from sequencing data rather than annotation of RNA-detected aberrations against existing calls

## Inputs

- SCISSOR run results (ScissorOutput object) with GSC and LSC sample ID lists (e.g. 26 GSC and 24 LSC samples for TP53)
- Publicly reported TCGA HNSC mutation calls in MAF format
- Cohort of matched samples with RNA-seq and mutation data (e.g. 452 HNSC cases)

## Outputs

- Per-sample classification table (one row per called shape change) labeling each as 'known' or 'previously unrecognized', with variant class
- Summary fraction of known vs previously unrecognized aberrations (target: 82% known, 18% unrecognized for TP53)

## How to apply

Load the tool's per-sample outlier calls (e.g. ScissorOutput$GSC and ScissorOutput$LSC sample IDs for TP53 among the 452 analyzed HNSC cases). Import the public TCGA HNSC MAF, subset it to the matched samples with RNA-seq data, and extract gene, variant classification, and genomic position per Tumor_Sample_Barcode. Match each called sample to its TCGA barcode and test whether the target gene (e.g. TP53) carries an annotated mutation, noting variant classes such as splice-site mutations, exon skipping, intron retention, and in-frame indels. Label each shape change 'known' when it coincides with an annotated mutation and 'previously unrecognized' otherwise, then summarize the fraction and compare against the reported 82% known / 18% unrecognized benchmark.

## Related tools

- **SCISSOR** (Produces the sample outlier calls (global and local shape changes) whose sample IDs are matched to TCGA barcodes for known-vs-novel classification) — https://github.com/hyochoi/SCISSOR

## Examples

```
maf_sub <- maf[maf$Tumor_Sample_Barcode %in% matched_barcodes, ]; maf_sub[maf_sub$Hugo_Symbol == 'TP53' & maf_sub$Tumor_Sample_Barcode %in% c(gsc_samples, lsc_samples), ]
```

## Evaluation signals

- The classification table must contain the full matched cohort (e.g. 452 sample rows for TCGA HNSC)
- The reproduced known-aberration fraction must match the reported 82% known vs 18% previously unrecognized for TP53 shape changes
- Called sample counts must match the source calls: 26 GSCs and 24 LSCs (50 total TP53 shape changes)
- Every shape-change sample ID must resolve to exactly one TCGA barcode in the MAF; unmatched barcodes should be flagged, not silently dropped
- Splice-site mutations should dominate the 'known' class, consistent with the article's report that coincident mutations were most commonly at splice sites

## Limitations

- Only 59% of splice-site mutations annotated in the public HNSC repository were associated with a statistically significant shape change, so absence of a shape change does not imply absence of mutation
- Matching depends on the exact subset of matched samples (452 of 522 HNSC; 70 severely degraded cases were excluded), so barcode mismatches or cohort-version differences will shift the known/unrecognized fractions
- Previously unrecognized aberrations may reflect annotation limitations rather than true novel events

## Evidence

- [results] 82% of TP53 shape changes coincide with known TCGA mutations while 18% were previously unrecognized: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [results] Splice sites are the most common coincident mutation class: "As expected, many of the shape changes coincided with mutations (82%), most commonly at splice sites"
- [readme] SCISSOR is retrieved from GitHub for reproducing the shape-change calls: [evidence span withheld — anchor status: not_found]
- [results] Publicly reported TCGA mutation calls are the annotation source: "we relied on publicly reported mutation calls from TCGA as well as TCGA RNA-seq data"
