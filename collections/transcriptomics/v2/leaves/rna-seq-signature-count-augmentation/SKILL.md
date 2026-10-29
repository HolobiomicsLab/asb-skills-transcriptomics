---
name: rna-seq-signature-count-augmentation
description: Use when an RNA-seq quantification based on fortuna signature counts
  (cnt.tsv/alt.tsv from --quant) must account for novel splice junctions that an aligner
  like STAR found in previously unmapped reads.
when_to_use_negative:
- The splicing analysis is already complete from annotation-only events and no aligner-derived
  novel splice sites from unmapped reads need to be incorporated (skip --refine and
  use --quant outputs directly).
- Reads are not from a spliced RNA-seq protocol or no spliced-aligner BAM of unmapped
  reads is available, since fortuna --refine consumes a spliced alignment of unmapped
  reads.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: fortuna
  role: alignment-free method whose --refine command incorporates aligner-detected
    novel splice sites into signature counts and annotates novel events
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: pseudoalignment engine used by fortuna to map reads to fragments and derive
    equivalence-class signature counts prior to refinement
  repo: https://github.com/pachterlab/kallisto
- name: htslib
  role: bundled library used by fortuna for reading BAM alignments of realigned unmapped
    reads
  repo: https://github.com/samtools/htslib
- name: STAR
  role: spliced aligner whose realignment of previously unmapped reads (star.bam)
    supplies the novel splice sites to be incorporated via --refine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:1e67780d382f4f61c3b3f839e75aa862c8a3a0a151cb9ba116033357ea8558f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - STAR
  - fortuna
  - htslib
  - kallisto
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rna-seq-signature-count-augmentation

## Summary

Use fortuna's --refine option to incorporate novel splice sites detected by a spliced aligner (e.g. STAR realignment of unmapped reads) into fortuna's pseudoalignment-based signature counts, producing refined subexon/intron counts and annotated novel events (AA/AD/ES) with per-event read support.

## When to use

Use when an RNA-seq quantification based on fortuna signature counts (cnt.tsv/alt.tsv from --quant) must account for novel splice junctions that an aligner like STAR found in previously unmapped reads.

## When NOT to use

- The splicing analysis is already complete from annotation-only events and no aligner-derived novel splice sites from unmapped reads need to be incorporated (skip --refine and use --quant outputs directly).
- Reads are not from a spliced RNA-seq protocol or no spliced-aligner BAM of unmapped reads is available, since fortuna --refine consumes a spliced alignment of unmapped reads.

## Inputs

- Segmented transcript annotation (annotation.rg.gtf, preprocessed by bin/processGTF.sh, with NodeId subexon identifiers)
- Per-sample signature-count file cnt.tsv from fortuna --quant
- Per-sample novel-event file alt.tsv from fortuna --quant
- Spliced-aligner BAM of previously unmapped reads (e.g. STAR star.bam)
- Target read length (longest read length, passed via -rl)

## Outputs

- cnt.refined.tsv: adjusted signature counts over subexon signatures
- alt.refined.tsv: refined novel-event table including novel acceptor (AA) and novel donor (AD) events with read support
- ref.txt: list of all newly created or refined subexons and introns with IDs and genomic coordinates (chromosome, start, end)

## How to apply

Start from the preprocessed segmented annotation (annotation.rg.gtf produced by bin/processGTF.sh) and the per-sample signature-count and novel-event files (cnt.tsv, alt.tsv) emitted by fortuna --quant. Align reads that remained unmapped in the pseudoalignment step with a spliced aligner (STAR produces star.bam of realigned previously unmapped reads), then run fortuna --refine with -rl set to the longest read length, -gtf pointing to the segmented GTF, -bam to the aligner BAM, and -incnt/-inalt taking the input count and event files. fortuna writes cnt.refined.tsv and alt.refined.tsv (adjusted signature counts and events) and ref.txt, which lists all newly created or refined subexons and introns with their IDs and genomic coordinates (chromosome, start, end). Novel acceptor (AA) and novel donor (AD) events can then be extracted from alt.refined.tsv and their supporting-read sets compared against the aligner's read assignments; in the eight ASD samples of the paper, fortuna and STAR agreed on the exact set of supporting reads per event. The rationale is that novel combinations of annotated splice sites guessed by fortuna cannot include junctions absent from the annotation, so unmapped-read realignment is the only route to recover them, at a throughput of ~3M reads realigned in ~25 min and count adjustment under 2 min per sample.

## Related tools

- **fortuna** (alignment-free method whose --refine command incorporates aligner-detected novel splice sites into signature counts and annotates novel events) — https://github.com/canzarlab/fortuna
- **kallisto** (pseudoalignment engine used by fortuna to map reads to fragments and derive equivalence-class signature counts prior to refinement) — https://github.com/pachterlab/kallisto
- **htslib** (bundled library used by fortuna for reading BAM alignments of realigned unmapped reads) — https://github.com/samtools/htslib
- **STAR** (spliced aligner whose realignment of previously unmapped reads (star.bam) supplies the novel splice sites to be incorporated via --refine)

## Examples

```
fortuna --refine -rl 150 -gtf annotation.rg.gtf -bam star.bam -incnt cnt.tsv -inalt alt.tsv -outcnt cnt.refined.tsv -outalt alt.refined.tsv -ref ref.txt
```

## Evaluation signals

- ref.txt enumerates newly created or refined subexons/introns with correct chromosome, start, end coordinates matching the aligner-detected junctions
- Novel acceptor (AA) and novel donor (AD) events extracted from alt.refined.tsv match the aligner's per-event supporting read sets exactly (as reported for the eight ASD samples in Supplementary Table S4)
- Throughput sanity check: ~3M reads realigned in ~25 min and count adjustment under 2 min per sample
- Signature counts in cnt.refined.tsv remain tab-separated signatures of subexon NodeIds (e.g. '223832|223833|223836|\t2274') consistent with the input GTF identifiers

## Limitations

- Refinement only captures novel splice sites that the spliced aligner found in reads remaining unmapped after pseudoalignment; junctions not detected by that aligner are missed
- The annotation must be preprocessed with bin/processGTF.sh into minimal disjoint segments bounded by splice sites before indexing or refinement; raw GENCODE GTF files cannot be used directly
- kallisto pseudoalignment deliberately avoids base-level alignment, so mismatch-bearing reads across novel junctions require the separate aligner realignment step to be counted correctly

## Evidence

- [intro] Optionally incorporate novel splice sites identified by any spliced aligner from unmapped reads: [evidence span withheld — anchor status: not_found]
- [abstract] Alignment-free signature counting via kallisto pseudoalignment: "It then pseudoaligns reads to fragments using kallisto and efficiently derives counts of the most elementary splicing units from kallisto's equivalence classes."
- [readme] GTF preprocessing is required before indexing/refinement: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
- [readme] Counting output format over subexon NodeIds: "The above example implies that a signature comprised of subexons 223832, 223833, and 223836 has a count of 2274. The IDs of the subexons are taken from the input GTF file"
- [abstract] fortuna is faster than align-and-count approaches: "fortuna was around 7× faster than traditional align and count approaches, and was able to analyze almost 300 million reads in just 15 min when using four threads."
