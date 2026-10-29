---
name: spliced-read-realignment
description: Use when a pseudoalignment-based quantification (fortuna --quant) left
  reads unmapped and a spliced aligner such as STAR has detected novel acceptor (AA)
  or donor (AD) splice sites in those reads that must be counted and annotated.
when_to_use_negative:
- No spliced-aligner output on unmapped reads is available — the refine step requires
  a BAM of unmapped reads realigned by a tool such as STAR; without it, plain fortuna
  --quant already covers counting and novel-event annotation
- The goal is de novo transcript assembly or full-length alignment of all reads rather
  than incorporating a small set of novel splice sites into fortuna's signature counts
- The annotation has not been preprocessed with bin/processGTF.sh — unsegmented GTFs
  lack the NodeId subexons that cnt.tsv/alt.tsv and the refine step depend on
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: fortuna
  role: Runs the --refine mode that merges novel splice sites into the segmented annotation,
    adjusts signature counts, and annotates the resulting novel events
  repo: https://github.com/canzarlab/fortuna
- name: STAR
  role: Spliced aligner that realigns reads left unmapped by fortuna's pseudoalignment
    step, supplying the BAM from which novel splice sites are incorporated
- name: htslib
  role: Bundled dependency fortuna uses to read the STAR realignment BAM
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:05c2833859f0e7404ca6d0a997d805d188ca8ce67c7049ce14a0320115fed904
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - STAR
  - fortuna
  - htslib
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spliced-read-realignment

## Summary

Use fortuna's --refine mode to incorporate novel splice sites that a spliced aligner (e.g. STAR) detected in reads left unmapped by pseudoalignment, refining subexons/introns and re-deriving signature counts and novel-event annotations from the realigned BAM.

## When to use

Use when a pseudoalignment-based quantification (fortuna --quant) left reads unmapped and a spliced aligner such as STAR has detected novel acceptor (AA) or donor (AD) splice sites in those reads that must be counted and annotated.

## When NOT to use

- No spliced-aligner output on unmapped reads is available — the refine step requires a BAM of unmapped reads realigned by a tool such as STAR; without it, plain fortuna --quant already covers counting and novel-event annotation
- The goal is de novo transcript assembly or full-length alignment of all reads rather than incorporating a small set of novel splice sites into fortuna's signature counts
- The annotation has not been preprocessed with bin/processGTF.sh — unsegmented GTFs lack the NodeId subexons that cnt.tsv/alt.tsv and the refine step depend on

## Inputs

- Preprocessed segmented GTF annotation (annotation.rg.gtf, with NodeId subexon IDs)
- Per-sample signature-count file from fortuna --quant (cnt.tsv)
- Per-sample novel-event file from fortuna --quant (alt.tsv)
- Per-sample STAR realignment BAM of previously unmapped reads (star.bam)
- Target read length (longest read length, e.g. 75/100/150 bp for the H. sapiens indexes)

## Outputs

- Refined signature-count file (cnt.refined.tsv)
- Refined novel splicing-event file including novel acceptor (AA) and novel donor (AD) events with per-event read support (alt.refined.tsv)
- ref.txt listing all newly created or refined subexons and introns with IDs and genomic coordinates (chromosome, start, end)

## How to apply

Start from the preprocessed segmented annotation (annotation.rg.gtf produced by bin/processGTF.sh) and the per-sample outputs of fortuna --quant (cnt.tsv signature counts, alt.tsv novel events). Per sample, take the STAR realignment BAM (star.bam) of reads previously unmapped in step (D), and run fortuna --refine with -rl set to the longest read length, -gtf pointing to the segmented GTF, -bam to the STAR BAM, and -incnt/-inalt as inputs, writing cnt.refined.tsv, alt.refined.tsv, and ref.txt. Inspect ref.txt, which lists all newly created or refined subexons and introns with IDs and genomic coordinates (chromosome, start, end), then extract the novel AA/AD events from alt.refined.tsv and tabulate per-event read support. The rationale is that pseudoalignment cannot represent unannotated splice junctions, so only a spliced realignment of unmapped reads can seed novel segments; fortuna then adjusts the pseudoalignment counts to the refined subexon/intron set. Validate by cross-checking that fortuna attributes the same exact set of supporting reads per novel event as STAR (as reported for the eight ASD samples in Supplementary Table S4).

## Related tools

- **fortuna** (Runs the --refine mode that merges novel splice sites into the segmented annotation, adjusts signature counts, and annotates the resulting novel events) — https://github.com/canzarlab/fortuna
- **STAR** (Spliced aligner that realigns reads left unmapped by fortuna's pseudoalignment step, supplying the BAM from which novel splice sites are incorporated)
- **htslib** (Bundled dependency fortuna uses to read the STAR realignment BAM) — https://github.com/samtools/htslib

## Examples

```
fortuna --refine -rl 150 -gtf annotation.rg.gtf -bam star.bam -incnt cnt.tsv -inalt alt.tsv -outcnt cnt.refined.tsv -outalt alt.refined.tsv -ref ref.txt
```

## Evaluation signals

- ref.txt contains exactly the newly created or refined subexons and introns, each with ID and genomic coordinates (chromosome, start, end)
- For each novel acceptor/donor event, the set of supporting reads in alt.refined.tsv matches the exact read set attributed by STAR (Supplementary Table S4 agreement)
- Throughput is in the reported range: ~3M reads realigned in ~25 min, with count adjustment under 2 min per sample
- Novel events in alt.refined.tsv follow the documented tab-separated schema: chromosome, start, end, transcript IDs, gene/Locus ID, event type (e.g. AA, AD, ES), read count
- As a sanity check of the underlying quantification, fortuna maps over 99% of the example STAT1 reads and finds slightly over 1000 reads supporting the novel exon skipping at chr2:190984393-190986854

## Limitations

- Whippet was unable to identify any supporting read for the novel AA/AD sites in the eight ASD samples, so comparable count-only methods may yield zero support; agreement must be checked against the aligner itself (STAR)
- The refine step depends on reads that remained unmapped in pseudoalignment — splice sites not recovered by the upstream spliced aligner (STAR) cannot be incorporated
- Requires C++11 and CMake >=2.8.12 to build, plus zlib and autoconf 2.69; pre-compiled binaries and Docker images are provided for Linux and Mac OS (x86_64 and Arm64)

## Evidence

- [intro] fortuna can further incorporate novel splice sites (e.g. segment s2) identified by any spliced aligner from reads that remained unmapped in step (D): [evidence span withheld — anchor status: not_found]
- [supplementary] Run fortuna --refine with -rl set to the longest read length, -gtf pointing to the segmented GTF, -bam to the STAR BAM: [evidence span withheld — anchor status: not_found]
- [results] fortuna and STAR agreed on the exact set of supporting reads, while Whippet was unable to identify any such read (Supplementary Table S4).: [evidence span withheld — anchor status: not_found]
- [readme] fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854.: "fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854."
