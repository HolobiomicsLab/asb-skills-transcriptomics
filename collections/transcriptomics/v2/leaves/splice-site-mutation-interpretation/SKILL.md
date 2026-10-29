---
name: splice-site-mutation-interpretation
description: Use when SCISSOR (or similar) has called sample-level RNA coverage shape
  changes and you need to determine which aberrations coincide with annotated mutations
  (e.g., splice sites, exon skipping, intron retention, in-frame indels) and which
  are novel.
when_to_use_negative:
- 'You need complete mutation-to-phenotype concordance: only 59% of annotated splice-site
  mutations in the public HNSC repository showed a statistically significant shape
  change, so this skill cannot confirm every annotated mutation.'
- The gene's RNA-seq coverage data has not been preprocessed by SCISSOR (pileup via
  build_pileup, ranges via get_Ranges, Scissor run); without shape-change calls there
  is nothing to cross-reference.
- The cohort is much smaller than ~30 samples, the smallest cohort the SCISSOR authors
  report analyzing successfully.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3676
- http://edamontology.org/topic_2640
tools:
- name: SCISSOR
  role: Detects global (GSC) and local (LSC) shape changes from base-level RNA-seq
    pileup data; its ScissorOutput GSC/LSC sample calls are the aberrations being
    classified against mutation calls
  repo: https://github.com/hyochoi/SCISSOR
- name: Integrative Genomic Viewer (IGV)
  role: Visual inspection of read alignments supporting detected shape changes and
    coinciding mutations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:2733b76c6a597c71d06dc012b0a86e440708d2d23c81072f2d3c32a33a95d931
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - Integrative Genomic Viewer (IGV)
  - SCISSOR
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3676
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# splice-site-mutation-interpretation

## Summary

Cross-references RNA-seq shape changes detected by SCISSOR (global and local shape changes) against publicly reported mutation calls (MAF) to classify each aberration as a known mutation, most commonly splice-site, versus previously unrecognized. Validated on 50 TP53 shape changes (26 GSCs, 24 LSCs) in 452 TCGA HNSC samples, where 82% coincided with known mutations.

## When to use

Use when SCISSOR (or similar) has called sample-level RNA coverage shape changes and you need to determine which aberrations coincide with annotated mutations (e.g., splice sites, exon skipping, intron retention, in-frame indels) and which are novel.

## When NOT to use

- You need complete mutation-to-phenotype concordance: only 59% of annotated splice-site mutations in the public HNSC repository showed a statistically significant shape change, so this skill cannot confirm every annotated mutation.
- The gene's RNA-seq coverage data has not been preprocessed by SCISSOR (pileup via build_pileup, ranges via get_Ranges, Scissor run); without shape-change calls there is nothing to cross-reference.
- The cohort is much smaller than ~30 samples, the smallest cohort the SCISSOR authors report analyzing successfully.

## Inputs

- ScissorOutput object with GSC and LSC slots containing per-gene shape-change sample calls (e.g., 26 GSCs and 24 LSCs at TP53 in 452 samples)
- Publicly reported TCGA mutation calls in MAF format, subset to the samples with matched mutation data
- TCGA sample barcodes linking shape-change calls to mutation records

## Outputs

- Per-sample classification table with one row per analyzed sample (452 rows in the reference run), labeling each shape change as 'known' or 'previously unrecognized' with variant class (e.g., splice site, exon skipping, intron retention, in-frame indel)
- Summary fraction of known versus previously unrecognized aberrations (target: 82% / 18%)

## How to apply

Load the ScissorOutput for the gene of interest and extract sample IDs for global shape changes (ScissorOutput$GSC) and local shape changes (ScissorOutput$LSC) among the analyzed cohort (e.g., 26 GSCs and 24 LSCs at TP53 among 452 HNSC cases after excluding 70 severely degraded samples). Import the publicly reported TCGA mutation calls (MAF file), subset to samples with matched mutation data, and extract gene, variant classification, and genomic position per sample. Match each shape-change sample to its TCGA barcode in the MAF and record whether a mutation exists in the gene and whether it is a splice-site mutation or another class (splice sites were the most common coinciding class). Label each detected shape change as 'known' when it coincides with an annotated mutation and 'previously unrecognized' otherwise, then compute the known fraction; in the reference analysis this reproduced the reported 82% known vs 18% unrecognized split at TP53. Note that concordance is incomplete in both directions: only 59% of splice-site mutations annotated in the public HNSC repository produced a statistically significant shape change, so absence of a shape change does not rule out an annotated mutation.

## Related tools

- **SCISSOR** (Detects global (GSC) and local (LSC) shape changes from base-level RNA-seq pileup data; its ScissorOutput GSC/LSC sample calls are the aberrations being classified against mutation calls) — https://github.com/hyochoi/SCISSOR
- **Integrative Genomic Viewer (IGV)** (Visual inspection of read alignments supporting detected shape changes and coinciding mutations)

## Examples

```
library(SCISSOR); data(TOY_data); pileup <- build_pileup(...); ranges <- get_Ranges(...); out <- Scissor(pileup, ranges); samples_gsc <- out$GSC
```

## Evaluation signals

- Classification table contains exactly one row per analyzed sample (452 rows for the TCGA HNSC TP53 analysis after excluding 70 degraded cases from 522).
- The number of shape-change rows (26 GSCs + 24 LSCs = 50 at TP53) matches the SCISSOR run output.
- Reproduced known-aberration fraction matches the reported 82% (with 18% previously unrecognized).
- Splice-site mutations appear as the most common coinciding variant class among the known aberrations.
- Each shape-change sample barcode resolves to a unique MAF record (or is correctly flagged as having no annotated TP53 mutation).

## Limitations

- Only 59% of splice-site mutations annotated in the public HNSC repository were associated with a statistically significant shape change, so concordance is directionally incomplete.
- SCISSOR tuning parameters (log normalization, normality cutoff) require optimization when a significantly different wet-lab RNA-seq protocol is used, affecting which shape changes are called.
- LSC detection results depend on window size choice, and GSC detection loses information about the order of base pairs, which can change which aberrations are called and thus classified.
- The 18% 'previously unrecognized' class may include artifactual signals from variable fragment length or GC content effects rather than true novel variants.

## Evidence

- [results] In 452 samples, SCISSOR identified 26 global shape changes (GSCs) and 24 local shape changes (LSCs) in TP53: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [results] Many shape changes coincided with mutations: "As expected, many of the shape changes coincided with mutations (82%), most commonly at splice sites"
- [results] Incomplete concordance in the mutation-to-shape direction: "only 59% of mutations annotated as splice site mutations in the public repository of HNSC mutations were associated with a statistically significant shape change"
- [readme] SCISSOR package installation: "install_github("hyochoi/SCISSOR")"
- [readme] SCISSOR detects structural alterations unsupervised: "SCISSOR identifies known as well as novel aberrations including abnormal splicing, intra-/intergenic deletions, small indels, alternative transcription start/termination."
