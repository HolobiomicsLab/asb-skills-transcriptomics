---
name: isoform-coordinate-quantification
description: Use when long-read (PacBio/ONT) alignments in BAM, GFF3, or BED6 format
  for one or a few genes must be converted into consensus splice sites and quantified
  isoforms without prior knowledge of splice sites.
when_to_use_negative:
- You need automated, hands-off genome-wide transcriptome discovery across many samples
  (VIsoQLR is designed for close inspection and fine-tuning of one gene at a time
  in low-throughput splicing assays)
- Input is short-read sequencing data, which the article notes has significant drawbacks
  in addressing the full spectrum of splicing events
- You already have curated isoform annotations and only need quantification against
  them without CEC re-detection
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: VIsoQLR
  role: Implements the CEC frequency filter (Read threshold 2%, Padding, Merge close
    splice sites) and isoform quantification in an interactive analyzer/viewer/editor
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: Mini-IsoQLR
  role: Command-line variant of the same consensus-breakpoint detection approach for
    minigene/ONTO data
  repo: https://github.com/TBLabFJD/Mini-IsoQLR
- name: GMAP
  role: Aligns raw FASTQ long reads to the reference index, producing GFF3/BAM with
    exon coordinates of all reads
- name: minimap2
  role: Alternative aligner used with a FASTA reference when GMAP is not used
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:e727d2a5c66950e9d197bc1b13415039ad5c929198f33f00180e8b94ceae554c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - Mini-IsoQLR
  - VIsoQLR
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# isoform-coordinate-quantification

## Summary

Defines consensus exon coordinates (CECs, i.e., consensus splice sites) from mapped long-read RNA sequencing data and quantifies isoforms, using VIsoQLR's frequency-filter plus coordinate-merge algorithm. It enables semi-automated detection and quantification of known and novel isoforms for one gene at a time without prior annotation.

## When to use

Use when long-read (PacBio/ONT) alignments in BAM, GFF3, or BED6 format for one or a few genes must be converted into consensus splice sites and quantified isoforms without prior knowledge of splice sites.

## When NOT to use

- You need automated, hands-off genome-wide transcriptome discovery across many samples (VIsoQLR is designed for close inspection and fine-tuning of one gene at a time in low-throughput splicing assays)
- Input is short-read sequencing data, which the article notes has significant drawbacks in addressing the full spectrum of splicing events
- You already have curated isoform annotations and only need quantification against them without CEC re-detection

## Inputs

- Long-read alignments in BAM, GFF3, or BED6 format
- Raw reads in FASTQ format plus reference sequence(s) (FASTA) if GMAP/minimap2 alignment is performed first
- Optional reference isoform annotations (known transcripts or isoforms detected by other methods) for comparison

## Outputs

- Consensus exon coordinate (CEC) list with each coordinate and its read frequency (e.g., sirv_cecs.tsv)
- Per-gene isoform structures (combinations of CECs) with abundance/proportion estimates
- Mapped reads (BAM plus BAI index) if alignment was run through the tool

## How to apply

Upload aligned reads (BAM, GFF3, or BED6) — or align raw FASTQ reads with GMAP against a pre-built reference index (or minimap2 with a FASTA reference) — then parse the alignment to extract the exon start/end (splice-site) coordinates of every aligned read (e.g., with pysam). Compute the proportion of each exon coordinate among all reads and keep candidate CECs above the 'Read threshold (%)' (default 2%). Merge below-threshold (non-candidate) CECs lying within the 'Padding (# of bases)' window (default 5 bases on each side) into nearby candidates, then merge candidate CECs closer than the 'Merge close splice sites (# of bases)' distance (default 3 bases) into the most frequent one. Finally, assign reads to isoforms defined by these CECs and quantify their abundances; inspect and fine-tune the parameters interactively when low-abundance isoforms are expected. The rationale is that a frequency filter plus rescue windows balances sensitivity against noise — most missing isoforms sit below the 2% default, but lowering the threshold detects more isoforms at the cost of spurious calls.

## Related tools

- **VIsoQLR** (Implements the CEC frequency filter (Read threshold 2%, Padding, Merge close splice sites) and isoform quantification in an interactive analyzer/viewer/editor) — https://github.com/TBLabFJD/VIsoQLR
- **Mini-IsoQLR** (Command-line variant of the same consensus-breakpoint detection approach for minigene/ONTO data) — https://github.com/TBLabFJD/Mini-IsoQLR
- **GMAP** (Aligns raw FASTQ long reads to the reference index, producing GFF3/BAM with exon coordinates of all reads)
- **minimap2** (Alternative aligner used with a FASTA reference when GMAP is not used) — https://github.com/lh3/minimap2

## Examples

```
docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- Every output CEC either exceeds the Read threshold (default 2% of reads) or was rescued by merging into a candidate within the Padding window (default 5 bases) or within the Merge close splice sites distance (default 3 bases) of a more frequent candidate
- CEC list validated against VIsoQLR's SIRV CECs / known SIRV isoforms — benchmark shows 99% exon coordinate accuracy on 72% of known SIRV isoforms, outperforming StringTie2 and FLAIR
- Per-gene isoform abundances are proportions of all reads and rank-correlate with independent abundance measurements (e.g., electropherogram rankings)
- Coordinate frequencies recomputed from the BAM with pysam reproduce the CEC frequency values written to the output table

## Limitations

- Isoforms below the default 2% Read threshold are missed in the automatic analysis; lowering the threshold detects more isoforms at the cost of noise, so manual parameter tuning is often required for individual genes
- No software version numbers were reported for VIsoQLR or the benchmarked comparators (StringTie2, FLAIR)
- VIsoQLR is designed for single-locus, low-throughput analysis rather than high-throughput whole-transcriptome pipelines

## Evidence

- [methods] CEC read threshold and merging: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [methods] Padding rescue window: "The `Padding (# of bases)` are the number of bases at each side of a candidate CES where other non-candidate CEC (below range in the frequency filter) are merged."
- [methods] Merging close candidates: "The `Merge close splice sites (# of bases)` is used to merger candidate CESs into the most frequent one if they are closer than the given distance"
- [abstract] Core purpose of the tool: "Our tool takes sequences aligned to a reference, and for each gene, it defines consensus splice sites and quantifies isoforms."
- [discussion] Threshold trade-off: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [readme] Command-line variant default: [evidence span withheld — anchor status: external_doc]
