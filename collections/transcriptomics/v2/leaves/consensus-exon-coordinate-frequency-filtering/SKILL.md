---
name: consensus-exon-coordinate-frequency-filtering
description: Use when building isoform models from long-read RNA-seq alignments (BAM/GFF3/BED6)
  and you must define consensus splice-site coordinates from read start/end positions
  before isoform classification, without prior annotation.
when_to_use_negative:
- You already have a reference annotation of exon coordinates and do not need to infer
  splice sites de novo from read alignments
- You are performing whole-transcriptome de-novo discovery (e.g. with wf-transcriptomes/bambu)
  rather than interactive per-gene analysis with manual parameter fine-tuning
- Input reads are unaligned (raw FASTQ) — alignment with GMAP or Minimap2 must be
  done first
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: VIsoQLR
  role: Interactive analyzer implementing the CEC frequency filter, padding, and merge-close-splice-site
    parameters
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: Mini-IsoQLR
  role: Command-line pipeline variant that defines consensus breakpoints present in
    more than 5% of reads by default
  repo: https://github.com/TBLabFJD/Mini-IsoQLR
- name: pysam
  role: Used to parse the BAM alignment and extract exon start/end coordinates of
    every aligned read
- name: GMAP
  role: Aligner producing the GFF3 alignment from which exon coordinates are extracted
    (--format=2)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:df304d6ace600957c585feef014ea036df59c2b028f20aeca2282e2b05fb18a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - Mini-IsoQLR
  - VIsoQLR
  - pysam
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# consensus-exon-coordinate-frequency-filtering

## Summary

VIsoQLR's method for defining consensus exon coordinates (CECs) from mapped long-read sequencing data by extracting per-read splice-site coordinates, keeping positions occurring above a configurable Read threshold (default 2%), and merging nearby non-candidate or close candidate CECs.

## When to use

Use when building isoform models from long-read RNA-seq alignments (BAM/GFF3/BED6) and you must define consensus splice-site coordinates from read start/end positions before isoform classification, without prior annotation.

## When NOT to use

- You already have a reference annotation of exon coordinates and do not need to infer splice sites de novo from read alignments
- You are performing whole-transcriptome de-novo discovery (e.g. with wf-transcriptomes/bambu) rather than interactive per-gene analysis with manual parameter fine-tuning
- Input reads are unaligned (raw FASTQ) — alignment with GMAP or Minimap2 must be done first

## Inputs

- Long-read alignment file (BAM with BAI index, or GFF3, or BED6) of reads aligned to a reference sequence
- Configurable parameters: Read threshold (%), Padding (# of bases), Merge close splice sites (# of bases)

## Outputs

- Consensus exon coordinate (CEC) list with per-coordinate frequency (e.g. sirv_cecs.tsv)
- Consensus splice-site set used for downstream isoform quantification

## How to apply

Parse the SIRV (or gene-level) long-read BAM alignment with pysam to extract the exon start/end (splice-site) coordinates of every aligned read, then compute each coordinate's proportion among all reads. Keep as candidate CECs only coordinates above the configurable Read threshold, default 2% — the article notes most missing isoforms fell below this default and that lowering it detects more isoforms at the cost of extra calls. Merge non-candidate (below-threshold) CECs into a candidate CEC within the Padding window (default 5 bases per side), then merge candidate CECs closer than the Merge close splice sites distance (default 3 bases) into the most frequent one. Rationale: a frequency filter distinguishes true splice sites from alignment noise while padding rescues reads whose breakpoints do not exactly coincide with consensus positions.

## Related tools

- **VIsoQLR** (Interactive analyzer implementing the CEC frequency filter, padding, and merge-close-splice-site parameters) — https://github.com/TBLabFJD/VIsoQLR
- **Mini-IsoQLR** (Command-line pipeline variant that defines consensus breakpoints present in more than 5% of reads by default) — https://github.com/TBLabFJD/Mini-IsoQLR
- **pysam** (Used to parse the BAM alignment and extract exon start/end coordinates of every aligned read)
- **GMAP** (Aligner producing the GFF3 alignment from which exon coordinates are extracted (--format=2))

## Examples

```
docker pull tblabfjd/visoqlr:latest && docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- Validate the resulting CEC list against VIsoQLR's known SIRV consensus exon coordinates
- Verify each retained candidate CEC occurs in a proportion of reads above the Read threshold (default 2%)
- Check that no two candidate CECs remain within the Merge close splice sites distance (default 3 bases) unless merged into the most frequent one
- Confirm below-threshold CECs within Padding (default 5 bases) of a candidate were merged, not dropped

## Limitations

- Missing isoforms in automatic analysis are often reads below the default 2% threshold; lowering it detects more isoforms at the cost of noise, requiring manual parameter configuration
- Small isoform abundances (e.g. 4% and 1.7%) may be hard to distinguish from noise, so threshold choice trades sensitivity against false positives
- The description of individual genes' isoform collections seems to require close inspection and sometimes manual configuration of certain parameters

## Evidence

- [methods] The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads.: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [methods] The `Padding (# of bases)` are the number of bases at each side of a candidate CES where other non-candidate CEC (below range in the frequency filter) are merged.: "The `Padding (# of bases)` are the number of bases at each side of a candidate CES where other non-candidate CEC (below range in the frequency filter) are merged."
- [discussion] most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [readme] defines the consensus breakpoints (start and end of exons which are present in more than 5% of the reads by default): [evidence span withheld — anchor status: external_doc]
- [readme] This pipeline uses GMAP aligner, which aligns cDNA sequences to a genome, using the parameter `--format=2` which generates a GFF3: [evidence span withheld — anchor status: external_doc]
