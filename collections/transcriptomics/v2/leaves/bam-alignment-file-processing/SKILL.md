---
name: bam-alignment-file-processing
description: Use when a BAM produced by a spliced aligner (e.g. STAR realignment of
  unmapped reads) contains novel splice sites that must be incorporated into fortuna's
  cnt.tsv/alt.tsv signature counts and novel-event annotations.
when_to_use_negative:
- Input reads are still raw FASTQ and no spliced-aligner BAM of unmapped reads exists
  yet — run fortuna --quant (and an aligner pass) first
- The task is only signature counting of annotated events from FASTQ without incorporating
  novel splice sites — the base --index/--quant workflow suffices and --refine adds
  nothing
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: fortuna
  role: Consumes the STAR realignment BAM via --refine and adjusts signature counts
    / annotates novel events
  repo: https://github.com/canzarlab/fortuna
- name: STAR
  role: Produces the per-sample BAM of spliced alignments of previously unmapped reads
    that feeds --refine
- name: htslib
  role: Bundled library fortuna uses to read the BAM alignment file
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:5d863bddd637f823b952b9f1647c65d38fb18226f2cdc529c882e3076ec1cfff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - STAR
  - fortuna
  - htslib
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# bam-alignment-file-processing

## Summary

Process a spliced-aligner BAM file (e.g. STAR realignment of previously unmapped reads) through fortuna's --refine option, using htslib to read alignments and adjust pseudoalignment-derived signature counts so that novel splice junctions gain read support.

## When to use

Use when a BAM produced by a spliced aligner (e.g. STAR realignment of unmapped reads) contains novel splice sites that must be incorporated into fortuna's cnt.tsv/alt.tsv signature counts and novel-event annotations.

## When NOT to use

- Input reads are still raw FASTQ and no spliced-aligner BAM of unmapped reads exists yet — run fortuna --quant (and an aligner pass) first
- The task is only signature counting of annotated events from FASTQ without incorporating novel splice sites — the base --index/--quant workflow suffices and --refine adds nothing

## Inputs

- STAR realignment BAM of previously unmapped reads per sample (star.bam)
- Segmented GTF annotation (annotation.rg.gtf, preprocessed with bin/processGTF.sh, containing NodeId subexon IDs)
- fortuna --quant signature-count file (cnt.tsv)
- fortuna --quant novel-event file (alt.tsv)
- Longest read length (for the -rl parameter)

## Outputs

- Refined signature-count file (cnt.refined.tsv)
- Refined novel-event file (alt.refined.tsv) including novel acceptor (AA) and donor (AD) events with per-event read support
- ref.txt listing newly created or refined subexons and introns with IDs and genomic coordinates (chromosome, start, end)

## How to apply

Start from the fortuna --quant outputs for each sample: the segmented annotation (annotation.rg.gtf), signature-count file (cnt.tsv), and novel-event file (alt.tsv). Run fortuna --refine with -rl set to the longest read length, -gtf pointing to the segmented GTF, -bam pointing to the per-sample STAR realignment BAM of previously unmapped reads (star.bam), -incnt/-inalt taking the input count and event files, and -outcnt/-outalt/-ref writing cnt.refined.tsv, alt.refined.tsv, and ref.txt. BAM parsing is handled via the bundled htslib dependency. Inspect ref.txt, which lists all newly created or refined subexons and introns with their IDs and genomic coordinates (chromosome, start, end), then extract novel acceptor (AA) and novel donor (AD) events from alt.refined.tsv and tabulate read support per event. Validate by cross-checking that fortuna and the aligner (STAR) attribute the same exact set of supporting reads to each novel event; reported throughput is ~3M reads realigned in ~25 min with count adjustment under 2 min per sample.

## Related tools

- **fortuna** (Consumes the STAR realignment BAM via --refine and adjusts signature counts / annotates novel events) — https://github.com/canzarlab/fortuna
- **STAR** (Produces the per-sample BAM of spliced alignments of previously unmapped reads that feeds --refine)
- **htslib** (Bundled library fortuna uses to read the BAM alignment file) — https://github.com/samtools/htslib

## Examples

```
fortuna --refine -rl 150 -gtf annotation.rg.gtf -bam star.bam -incnt cnt.tsv -inalt alt.tsv -outcnt cnt.refined.tsv -outalt alt.refined.tsv -ref ref.txt
```

## Evaluation signals

- ref.txt lists all newly created or refined subexons and introns with expected IDs and genomic coordinates (chromosome, start, end)
- Novel acceptor (AA) and donor (AD) events extracted from alt.refined.tsv match the read-support sets reported in Supplementary Table S4
- fortuna and STAR attribute the exact same set of supporting reads to each novel event in the eight ASD samples
- Throughput is in the reported range: ~3M reads realigned in ~25 min, count adjustment under 2 min per sample

## Limitations

- Only reads that remained unmapped in the fortuna pseudoalignment step (D) and were realigned by a spliced aligner can contribute novel splice sites; Whippet was unable to identify any such reads in the ASD samples
- Requires the GTF to be preprocessed by bin/processGTF.sh into minimal disjoint segments bounded by splice sites before refinement
- Novel splice-site incorporation is optional and comes after the main alignment-free quantification; it cannot recover reads that neither pseudoalign nor realign

## Evidence

- [intro] Optionally, fortuna can further incorporate novel splice sites (e.g. segment s2) identified by any spliced aligner from reads that remained unmapped in step (D): [evidence span withheld — anchor status: not_found]
- [readme] htslib bundled dependency: "Additional dependencies are included with fortuna, including
* kallisto (https://github.com/pachterlab/kallisto)
* htslib (https://github.com/samtools/htslib)"
- [readme] novel event annotation alt.tsv format: "which means that on chromosome 2 there exists a novel intron between coordinates 190984393 and 190986854 on gene Locus_18066 that is a novel exon skipping"
