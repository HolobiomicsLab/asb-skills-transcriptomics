---
name: rna-seq-shape-change-annotation
description: Use when SCISSOR has flagged shape-change outliers (e.g., ScissorOutput$GSC
  / $LSC sample IDs) and you need to determine which detected aberrations correspond
  to existing mutation calls versus novel events.
when_to_use_negative:
- The goal is de novo discovery of novel aberrations without comparison to a curated
  mutation repository — this skill only annotates existing SCISSOR calls against known
  mutations
- Input is raw coverage pileup rather than an existing ScissorOutput with called shape
  changes — run the Scissor pipeline (build_pileup, get_Ranges, Scissor) first
- Cohort lacks matched public mutation calls, making known-vs-novel classification
  impossible
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3520
tools:
- name: SCISSOR
  role: Detects global and local RNA shape-change outliers whose sample IDs are then
    annotated against mutation calls
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:b840f41abaa66e009e0eb5151651a7466f957517a294d247770c23581fe88019
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  - get_Ranges
  - plot_ScissorOS
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# rna-seq-shape-change-annotation

## Summary

Cross-reference SCISSOR-detected RNA coverage shape changes (GSCs and LSCs) against publicly reported mutation calls (e.g., TCGA MAF) to classify each aberration as 'known' (coinciding with an annotated mutation) versus 'previously unrecognized'.

## When to use

Use when SCISSOR has flagged shape-change outliers (e.g., ScissorOutput$GSC / $LSC sample IDs) and you need to determine which detected aberrations correspond to existing mutation calls versus novel events.

## When NOT to use

- The goal is de novo discovery of novel aberrations without comparison to a curated mutation repository — this skill only annotates existing SCISSOR calls against known mutations
- Input is raw coverage pileup rather than an existing ScissorOutput with called shape changes — run the Scissor pipeline (build_pileup, get_Ranges, Scissor) first
- Cohort lacks matched public mutation calls, making known-vs-novel classification impossible

## Inputs

- ScissorOutput object with GSC and LSC slots (shape-change sample IDs and outlyingness scores)
- Public mutation calls in MAF format (e.g., TCGA HNSC mutation calls)
- TCGA sample barcodes for the analyzed cohort (e.g., 452 HNSC samples with matched mutation data)

## Outputs

- Per-sample classification table labeling each GSC/LSC as 'known' or 'previously unrecognized' with variant class
- Summary fraction of known versus previously unrecognized aberrations (target: 82% known / 18% unrecognized for TP53)

## How to apply

Retrieve the shape-change sample IDs from the ScissorOutput object via ScissorOutput$GSC and ScissorOutput$LSC (e.g., 26 GSCs and 24 LSCs at TP53 among 452 analyzed TCGA HNSC cases). Import the public mutation calls (MAF file), subset to the samples analyzed by SCISSOR, and extract gene, variant classification, and genomic position per sample. Match each shape-change sample to its TCGA barcode in the MAF and check for a mutation in the target gene, recording variant class (splice site, exon skipping, intron retention, in-frame indel). Label each detected shape change 'known' when it coincides with an annotated mutation and 'previously unrecognized' otherwise, then summarize the fractions. In the SCISSOR TP53 analysis, 82% of shape changes coincided with mutations (most commonly at splice sites) and 18% were previously unrecognized; note that only 59% of splice-site mutations in the public repository produced a significant shape change, so absence of a shape change does not rule out a mutation.

## Related tools

- **SCISSOR** (Detects global and local RNA shape-change outliers whose sample IDs are then annotated against mutation calls) — https://github.com/hyochoi/SCISSOR

## Examples

```
data(TOY_data); pileupData <- build_pileup(Pileup=toygene_coverage, regions=regions, inputType="whole_intron", outputType="part_intron"); ScissorOutput <- Scissor(pileupData=pileupData, Ranges=geneRanges); hits <- ScissorOutput$SC
```

## Evaluation signals

- Classification table contains one row per analyzed sample (e.g., 452 sample rows for the TCGA HNSC TP53 analysis)
- Every GSC/LSC sample ID from ScissorOutput$SC is assigned exactly one label ('known' or 'previously unrecognized')
- Reproduced known-aberration fraction matches the reported 82% known / 18% previously unrecognized split for TP53
- MAF subsetting retains only samples overlapping the SCISSOR-analyzed cohort (452 of the original 522 HNSC cases after excluding 70 degraded samples)
- Variant classes recorded for known aberrations should be dominated by splice-site mutations, exon skipping, intron retention, and in-frame indels

## Limitations

- Only 59% of splice-site mutations annotated in the public HNSC repository were associated with a statistically significant shape change, so mutation-positives may be missed by shape-change detection
- GSC detection loses information about the order of base pairs, limiting mechanistic interpretation of the matched events
- LSC results depend on the window size choice; variable fragment length and GC content may introduce artifactual shape changes that could be mislabeled during annotation

## Evidence

- [results] 82% of TP53 shape changes coincided with mutations: "As expected, many of the shape changes coincided with mutations (82%), most commonly at splice sites"
- [results] In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs in TP53: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [results] Only 59% of annotated splice-site mutations showed a significant shape change: "only 59% of mutations annotated as splice site mutations in the public repository of HNSC mutations were associated with a statistically significant shape change"
- [abstract] SCISSOR recaptures known variants such as splice site mutations: "This enables SCISSOR to independently recapture known variants such as splice site mutations in tumor suppressor genes"
- [results] Mutation calls relied on public TCGA repository: "we relied on publicly reported mutation calls from TCGA as well as TCGA RNA-seq data"
