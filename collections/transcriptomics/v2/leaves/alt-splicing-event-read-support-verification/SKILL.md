---
name: alt-splicing-event-read-support-verification
description: Use when novel splice junctions (e.g. novel acceptor/donor events) were
  detected by a spliced aligner on unmapped reads and you must confirm that an alternative
  quantifier's refined counts use the identical supporting read set.
when_to_use_negative:
- When no spliced-aligner realignment of unmapped reads (e.g. STAR star.bam) is available
  — the --refine step has no novel splice sites to incorporate.
- When only genome-level alignment counts are needed and no novel or refinement-sensitive
  splicing events are of interest (a plain align-and-count or fortuna --quant run
  suffices).
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: fortuna
  role: Runs --quant to produce cnt.tsv/alt.tsv and then --refine to incorporate STAR-detected
    novel splice sites and emit refined counts, refined events, and ref.txt
  repo: https://github.com/canzarlab/fortuna
- name: STAR
  role: Spliced aligner that realigns previously unmapped reads in a second phase,
    providing the BAM from which novel splice sites are detected
- name: htslib
  role: Bundled dependency used by fortuna for BAM file access during refinement
  repo: https://github.com/samtools/htslib
- name: kallisto
  role: Pseudoaligns reads to transcript fragments; the equivalence-class counts that
    --refine adjusts against STAR realignment
  repo: https://github.com/pachterlab/kallisto
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:390b6901f06d4c7e2f834670206dd647dc227fbb4deff20c6923fd5a4d6a7264
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - STAR
  - fortuna
  - htslib
  - kallisto
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Alternative splicing event read-support verification

## Summary

Verifies that the alignment-free caller fortuna and a spliced aligner (STAR) attribute the exact same set of supporting reads to novel splice-site events, by running fortuna --refine on the STAR realignment of unmapped reads and comparing per-event read support (rebuilding Supplementary Table S4).

## When to use

Use when novel splice junctions (e.g. novel acceptor/donor events) were detected by a spliced aligner on unmapped reads and you must confirm that an alternative quantifier's refined counts use the identical supporting read set.

## When NOT to use

- When no spliced-aligner realignment of unmapped reads (e.g. STAR star.bam) is available — the --refine step has no novel splice sites to incorporate.
- When only genome-level alignment counts are needed and no novel or refinement-sensitive splicing events are of interest (a plain align-and-count or fortuna --quant run suffices).

## Inputs

- Segmented GTF annotation (annotation.rg.gtf, preprocessed with bin/processGTF.sh)
- Per-sample signature-count file from fortuna --quant (cnt.tsv)
- Per-sample novel-event file from fortuna --quant (alt.tsv)
- STAR realignment BAM of previously unmapped reads per sample (star.bam)
- Longest read length in the dataset (for the -rl parameter)

## Outputs

- cnt.refined.tsv (updated signature counts)
- alt.refined.tsv (novel splicing events with per-event read support, e.g. AA/AD events)
- ref.txt (all newly created or refined subexons and introns with IDs and genomic coordinates)
- Per-event comparison table of fortuna vs STAR supporting read sets (Supplementary Table S4 equivalent)

## How to apply

Start from fortuna --quant outputs (cnt.tsv, alt.tsv) built from the preprocessed segmented annotation (annotation.rg.gtf), and the per-sample BAM of previously unmapped reads realigned by STAR (star.bam). Run fortuna --refine with -rl set to the longest read length, -gtf pointing to the segmented GTF, -bam to the STAR BAM, -incnt/-inalt taking the input count and novel-event files, and -outcnt/-outalt/-ref writing cnt.refined.tsv, alt.refined.tsv, and ref.txt. Inspect ref.txt, which lists all newly created or refined subexons and introns with their IDs and genomic coordinates, then extract the novel acceptor (AA) and donor (AD) events from alt.refined.tsv and tabulate read support per event. Finally, cross-check that fortuna and STAR agree on the exact set of supporting reads for each novel event; the rationale is that fortuna's refinement step incorporates novel splice sites identified by any spliced aligner from reads that remained unmapped during pseudoalignment.

## Related tools

- **fortuna** (Runs --quant to produce cnt.tsv/alt.tsv and then --refine to incorporate STAR-detected novel splice sites and emit refined counts, refined events, and ref.txt) — https://github.com/canzarlab/fortuna
- **STAR** (Spliced aligner that realigns previously unmapped reads in a second phase, providing the BAM from which novel splice sites are detected)
- **htslib** (Bundled dependency used by fortuna for BAM file access during refinement) — https://github.com/samtools/htslib
- **kallisto** (Pseudoaligns reads to transcript fragments; the equivalence-class counts that --refine adjusts against STAR realignment) — https://github.com/pachterlab/kallisto

## Examples

```
fortuna --refine -rl 100 -gtf annotation.rg.gtf -bam star.bam -incnt cnt.tsv -inalt alt.tsv -outcnt cnt.refined.tsv -outalt alt.refined.tsv -ref ref.txt
```

## Evaluation signals

- ref.txt is non-empty and lists each new/refined subexon or intron with its ID and genomic coordinates (chromosome, start, end).
- Novel acceptor (AA) and donor (AD) events extracted from alt.refined.tsv match the events reported in Supplementary Table S4 for the eight ASD samples.
- For every novel event, the set of supporting reads attributed by fortuna is identical to STAR's set (exact read-level agreement).
- Throughput is consistent with the paper: ~3M reads realigned in ~25 min, and count adjustment under 2 min per sample.

## Limitations

- Refinement depends entirely on the upstream spliced aligner: only novel splice sites identified by the aligner (e.g. STAR) from reads unmapped in the pseudoalignment step can be incorporated.
- A competing quantifier may fail entirely on such events — in the paper, Whippet was unable to identify any of the reads supporting the novel acceptor/donor sites, so it cannot serve as a cross-check for this comparison.

## Evidence

- [intro] Optionally, fortuna can further incorporate novel splice sites (e.g. segment s2) identified by any spliced aligner from reads that remained unmapped in step (D): [evidence span withheld — anchor status: not_found]
- [methods] Fortuna requires C++11 to run and CMake version >=2.8.12 to compile.: "Fortuna requires C++11 to run and CMake version >=2.8.12 to compile."
- [readme] A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
