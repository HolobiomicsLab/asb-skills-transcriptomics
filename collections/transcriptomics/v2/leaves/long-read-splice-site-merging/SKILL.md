---
name: long-read-splice-site-merging
description: Reference for defining consensus exon coordinates (splice start/end positions)
  from long-read BAM/GFF3/BED6 alignments when splice sites are unknown a priori,
  e.g. SIRV or minigene splicing assays analyzed one gene at a time.
when_to_use_negative:
- The analysis goal is automated transcriptome-wide discovery with a supplied genome
  and annotation rather than interactive, single-gene curation (VIsoQLR is tailored
  to low-throughput splicing assays, one gene at a time)
- Known isoforms from another method (e.g. StringTie2 or FLAIR output) are being imported
  as references for comparison, so splice sites need not be re-derived from reads
- Reads are unaligned FASTQ; CEC detection requires mapped reads — align first with
  GMAP or Minimap2
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: VIsoQLR
  role: Interactive analyzer that implements the CEC frequency filter, padding merge,
    and close-splice-site merge with user-configurable parameters
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: Mini-IsoQLR
  role: Command-line pipeline variant that defines consensus breakpoints present in
    >5% of reads and rescues reads via lower/higher thresholds around them
  repo: https://github.com/TBLabFJD/Mini-IsoQLR
- name: GMAP
  role: cDNA-to-genome aligner (parameter --format=2) producing the GFF3 exon coordinates
    from which CECs are computed
- name: pysam
  role: Library used to parse the SIRV long-read BAM alignment and extract exon start/end
    splice-site coordinates per read
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:30e700aa744e443d459cd24b4dab8e7140991e6aaf2cb4da32e5ea0d181c81f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - Mini-IsoQLR
  - VIsoQLR
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# long-read-splice-site-merging

## Summary

VIsoQLR's consensus exon coordinate (CEC) detection: splice-site coordinates are extracted from aligned long reads, filtered by read frequency (default 2%), and low-frequency or nearby coordinates are merged via padding (5 nt) and close-splice-site (3 nt) rules to define consensus splice sites per gene.

## When to use

Reference for defining consensus exon coordinates (splice start/end positions) from long-read BAM/GFF3/BED6 alignments when splice sites are unknown a priori, e.g. SIRV or minigene splicing assays analyzed one gene at a time.

## When NOT to use

- The analysis goal is automated transcriptome-wide discovery with a supplied genome and annotation rather than interactive, single-gene curation (VIsoQLR is tailored to low-throughput splicing assays, one gene at a time)
- Known isoforms from another method (e.g. StringTie2 or FLAIR output) are being imported as references for comparison, so splice sites need not be re-derived from reads
- Reads are unaligned FASTQ; CEC detection requires mapped reads — align first with GMAP or Minimap2

## Inputs

- Mapped long-read sequencing alignments (BAM plus BAI index) from long-read RNA-seq
- GFF3 alignment file from GMAP (--format=2) containing exon coordinates of all reads
- BED6-format aligned reads accepted by VIsoQLR

## Outputs

- Consensus exon coordinate (CEC) list with coordinate and read frequency per gene (e.g. sirv_cecs.tsv)
- Consensus splice-site set used downstream to classify reads into isoforms

## How to apply

Parse the aligned reads (BAM via pysam, or GFF3 from GMAP --format=2) to extract the exon start/end coordinates of every aligned read, then compute the proportion of each coordinate among all reads. Keep as candidate CECs only those exon coordinates occurring above the 'Read threshold (%)' filter (default 2%). Merge non-candidate (below-threshold) CECs into a candidate within the 'Padding (# of bases)' window on each side (default 5 bases), and merge candidate CECs closer than the 'Merge close splice sites (# of bases)' distance (default 3 bases) into the most frequent one. The rationale is that most reads sharing a true splice junction will land on the exact coordinate, while sequencing/alignment jitter produces nearby low-frequency coordinates that should be rescued rather than discarded. Write the resulting coordinate+frequency list (e.g. sirv_cecs.tsv) and validate against known SIRV CECs before proceeding to isoform assembly and quantification.

## Related tools

- **VIsoQLR** (Interactive analyzer that implements the CEC frequency filter, padding merge, and close-splice-site merge with user-configurable parameters) — https://github.com/TBLabFJD/VIsoQLR
- **Mini-IsoQLR** (Command-line pipeline variant that defines consensus breakpoints present in >5% of reads and rescues reads via lower/higher thresholds around them) — https://github.com/TBLabFJD/Mini-IsoQLR
- **GMAP** (cDNA-to-genome aligner (parameter --format=2) producing the GFF3 exon coordinates from which CECs are computed)
- **pysam** (Library used to parse the SIRV long-read BAM alignment and extract exon start/end splice-site coordinates per read)

## Examples

```
docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- Every retained candidate CEC has a coordinate frequency above the Read threshold (default 2%) computed over all reads of the gene
- All below-threshold CECs lying within the padding window (default 5 bases per side) of a candidate are merged, leaving no orphan non-candidate coordinates
- Candidate CECs separated by fewer than the 'Merge close splice sites' distance (default 3 bases) collapse to the most frequent one
- The resulting CEC list reproduces known SIRV exon coordinates, benchmarked at 99% exon-coordinate accuracy with 72% of known SIRV isoforms detected
- Output table (sirv_cecs.tsv) contains exactly two columns: coordinate and frequency, one row per consensus splice site

## Limitations

- Isoforms whose splice-site reads fall below the default 2% threshold are missed in the automatic analysis and require manual configuration of parameters (lowering the threshold costs more detected isoforms)
- Very low-abundance isoforms (~4% and 1.7% in the PAX6 minigene case) can be hard to distinguish from noise, so threshold tuning trades sensitivity against spurious calls
- VIsoQLR is designed for close inspection and fine-tuning of one gene at a time, not hands-off transcriptome-wide discovery

## Evidence

- [methods] Read threshold selects candidate CECs by proportion: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [methods] Padding merges non-candidate CECs beside a candidate: "The `Padding (# of bases)` are the number of bases at each side of a candidate CES where other non-candidate CEC (below range in the frequency filter) are merged."
- [methods] Close-splice-site merge rule: "The `Merge close splice sites (# of bases)` is used to merger candidate CESs into the most frequent one if they are closer than the given distance"
- [discussion] Threshold trade-off on missing isoforms: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [readme] Mini-IsoQLR consensus breakpoint definition: [evidence span withheld — anchor status: external_doc]
