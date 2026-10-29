---
name: bam-splice-coordinate-extraction
description: Use when a BAM/GFF3/BED6 long-read RNA alignment must be converted into
  a consensus set of splice-site/exon-boundary coordinates per gene, prior to isoform
  assembly and quantification.
when_to_use_negative:
- Input is raw FASTQ reads that are not yet aligned — coordinates must come from an
  alignment (GMAP or Minimap2) first
- Goal is whole-transcriptome, reference-guided discovery at scale across many samples
  — VIsoQLR/this CEC approach targets close inspection of one gene at a time (e.g.
  aberrant mRNAs from splicing assays)
- A consensus exon/CEC table already exists and only isoform assembly or quantification
  is needed
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: VIsoQLR
  role: Interactive analyzer that implements the CEC frequency filter (2% default),
    padding merge (5 nt) and close-splice-site merge (3 nt) on alignments uploaded
    as BAM/GFF3/BED6
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: Mini-IsoQLR
  role: Command-line pipeline version that defines consensus exon breakpoints present
    in more than 5% of reads and rescues reads whose breakpoints do not exactly coincide
  repo: https://github.com/TBLabFJD/Mini-IsoQLR
- name: GMAP
  role: Aligner producing GFF3 (--format=2) or BAM alignments whose per-read exon
    coordinates are the raw material for CEC extraction
- name: pysam
  role: Python library used to parse the long-read BAM and extract exon start/end
    (splice-site) coordinates of every aligned read
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:1975a8d6423a06b0ee47c660000f33e15e891b0783f77140d678b65cb0c3fe1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - Mini-IsoQLR
  - VIsoQLR
  - pysam
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bam-splice-coordinate-extraction

## Summary

Extracts per-read exon start/end (splice-site) coordinates from a long-read BAM alignment (e.g. SIRV alignments parsed with pysam) and reduces them to consensus exon coordinates (CECs) using frequency-based filtering, as implemented in VIsoQLR for isoform detection.

## When to use

Use when a BAM/GFF3/BED6 long-read RNA alignment must be converted into a consensus set of splice-site/exon-boundary coordinates per gene, prior to isoform assembly and quantification.

## When NOT to use

- Input is raw FASTQ reads that are not yet aligned — coordinates must come from an alignment (GMAP or Minimap2) first
- Goal is whole-transcriptome, reference-guided discovery at scale across many samples — VIsoQLR/this CEC approach targets close inspection of one gene at a time (e.g. aberrant mRNAs from splicing assays)
- A consensus exon/CEC table already exists and only isoform assembly or quantification is needed

## Inputs

- Long-read alignment file in BAM format (with BAI index), or GFF3/BED6 alignments
- Gene/locus over which to collect splice-site coordinates (e.g. SIRV gene from a SIRV spike-in long-read BAM)
- Configurable parameters: Read threshold (%) (default 2), Padding (# of bases) (default 5), Merge close splice sites (# of bases) (default 3)

## Outputs

- List of consensus exon coordinates (CECs) per gene with associated read frequencies
- Tab-separated CEC table, e.g. sirv_cecs.tsv (coordinate + frequency), for downstream isoform assembly and validation

## How to apply

Parse the BAM alignment with pysam and extract the exon start/end (splice-site) coordinates of every aligned read, then compute the proportion of each coordinate among all reads. Select candidate CECs with the Read threshold (%) filter (default 2% of reads), keeping only coordinates occurring above this proportion. Merge below-threshold (non-candidate) CECs into a candidate within the Padding (# of bases) window (default 5 bases on each side), then merge candidate CECs closer than the Merge close splice sites distance (default 3 bases) into the most frequent one. The related Mini-IsoQLR command-line pipeline applies the same logic with a default 5% breakpoint frequency threshold. The rationale is that genuine splice sites are supported by many reads while small coordinate deviations arise from alignment noise, so frequency filtering plus proximity-based merging yields robust consensus boundaries without prior annotation.

## Related tools

- **VIsoQLR** (Interactive analyzer that implements the CEC frequency filter (2% default), padding merge (5 nt) and close-splice-site merge (3 nt) on alignments uploaded as BAM/GFF3/BED6) — https://github.com/TBLabFJD/VIsoQLR
- **Mini-IsoQLR** (Command-line pipeline version that defines consensus exon breakpoints present in more than 5% of reads and rescues reads whose breakpoints do not exactly coincide) — https://github.com/TBLabFJD/Mini-IsoQLR
- **GMAP** (Aligner producing GFF3 (--format=2) or BAM alignments whose per-read exon coordinates are the raw material for CEC extraction)
- **pysam** (Python library used to parse the long-read BAM and extract exon start/end (splice-site) coordinates of every aligned read)

## Examples

```
import pysam; coords=[(r.reference_start, r.reference_end) for r in pysam.AlignmentFile('sirv_reads.bam','rb').fetch(gene)]
```

## Evaluation signals

- The proportion of each exon coordinate among all reads is computed and candidate CECs are exactly those above the configured Read threshold (default 2%)
- Every non-candidate CEC lying within the Padding window (default 5 bases each side) of a candidate is merged, and none survive outside it
- Candidate CECs closer than the Merge close splice sites distance (default 3 bases) are collapsed into the single most frequent coordinate
- Output CEC table (e.g. sirv_cecs.tsv) contains a coordinate plus frequency per entry and can be validated against VIsoQLR's known SIRV CECs
- On SIRV PacBio Iso-Seq data, downstream isoform detection should reach ~99% exon coordinate accuracy (VIsoQLR detects 72% of known SIRV isoforms at 99% coordinate accuracy, outperforming StringTie2 and FLAIR)

## Limitations

- Isoforms whose exon coordinates occur below the default 2% read threshold are missed; lowering the threshold detects more isoforms but at the cost of detecting more isoforms/noise, so parameters may need manual tuning per gene
- The article notes that some isoform descriptions 'require a close inspection and sometimes a manual configuration of certain parameters' rather than fully automatic detection
- A limitation concerning Minimap2 alignments is advised on the Minimap2 GitHub, so aligner choice affects the extracted coordinates

## Evidence

- [methods] The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads.: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [methods] The `Padding (# of bases)` are the number of bases at each side of a candidate CES where other non-candidate CEC (below range in the frequency filter) are merged.: "The `Padding (# of bases)` are the number of bases at each side of a candidate CES where other non-candidate CEC (below range in the frequency filter) are merged."
- [methods] The `Merge close splice sites (# of bases)` is used to merger candidate CESs into the most frequent one if they are closer than the given distance: "The `Merge close splice sites (# of bases)` is used to merger candidate CESs into the most frequent one if they are closer than the given distance"
- [abstract] Our tool takes sequences aligned to a reference, and for each gene, it defines consensus splice sites and quantifies isoforms.: "Our tool takes sequences aligned to a reference, and for each gene, it defines consensus splice sites and quantifies isoforms."
- [readme] defines the consensus breakpoints (start and end of exons which are present in more than 5% of the reads by default): "1. 
defines the consensus breakpoints (start and end of exons which are present in more than 5% of the reads by default), "
