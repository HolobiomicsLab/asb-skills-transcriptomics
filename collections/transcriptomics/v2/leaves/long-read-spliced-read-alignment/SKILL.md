---
name: long-read-spliced-read-alignment
description: Use when long-read transcript sequencing reads (FASTQ, e.g. PacBio HiFi
  or ONT) must be spliced-aligned to a reference genome (FASTA) before isoform detection
  or benchmarking of tools such as VIsoQLR, FLAIR or StringTie2.
when_to_use_negative:
- Input reads are short reads — the article's workflow and the splice:hq preset target
  long-read sequencing data; short-read aligners should be used instead
- A reference-guided annotation-first transcriptome workflow (e.g. epi2me-labs/wf-transcriptomes
  with bambu and a supplied ref_annotation GTF) is preferred over mapper-agnostic
  isoform calling from a fresh BAM
- Only a pre-existing alignment is available — VIsoQLR accepts GFF3, BED6, or BAM
  inputs directly, so re-alignment with this skill is unnecessary
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Minimap2
  role: Splice-aware long-read mapper run with the '-ax splice:hq -uf --secondary=no'
    preset to produce the BAM alignment shared by VIsoQLR, FLAIR and StringTie2 in
    the SIRV benchmark
  repo: https://github.com/lh3/minimap2
- name: GMAP
  role: cDNA-to-genome aligner built into the VIsoQLR Docker image; used via the tool's
    second panel to align raw FASTQ reads using a GMAP reference index built in the
    first panel
- name: VIsoQLR
  role: Docker-based interactive tool hosting the GMAP index-building and alignment
    panels and consuming the resulting BAM (or GFF3/BED6) for isoform identification
    and quantification
  repo: https://github.com/TBLabFJD/VIsoQLR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:c0122d124ea5b9cabedd32da50500ce19938ad680ebf071ab1aafe109efbe09b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - FLAIR
  - GMAP
  - Minimap2
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# long-read-spliced-read-alignment

## Summary

Aligns long-read cDNA/RNA sequencing reads (e.g. PacBio Iso-Seq SIRV reads) to a reference genome using splice-aware mappers — Minimap2 with the high-quality splice preset '-ax splice:hq -uf --secondary=no', or GMAP from within the VIsoQLR Docker container — to produce spliced BAM alignments that downstream isoform detection tools (VIsoQLR, FLAIR, StringTie2) consume.

## When to use

Use when long-read transcript sequencing reads (FASTQ, e.g. PacBio HiFi or ONT) must be spliced-aligned to a reference genome (FASTA) before isoform detection or benchmarking of tools such as VIsoQLR, FLAIR or StringTie2.

## When NOT to use

- Input reads are short reads — the article's workflow and the splice:hq preset target long-read sequencing data; short-read aligners should be used instead
- A reference-guided annotation-first transcriptome workflow (e.g. epi2me-labs/wf-transcriptomes with bambu and a supplied ref_annotation GTF) is preferred over mapper-agnostic isoform calling from a fresh BAM
- Only a pre-existing alignment is available — VIsoQLR accepts GFF3, BED6, or BAM inputs directly, so re-alignment with this skill is unnecessary

## Inputs

- Long-read sequencing reads in FASTQ format (e.g. PacBio SIRV Iso-Seq reads from the UHR data release)
- Reference genome sequence in FASTA format
- Optional known isoform annotation (GTF/GFF3) for post-alignment comparison of detected isoforms

## Outputs

- Spliced read alignment as SAM/BAM file
- BAM index (BAI) for downstream tools
- Per-tool isoform detection counts compared against the 68 known SIRV isoforms (e.g. VIsoQLR 48/68, FLAIR 35/68, StringTie2 15/68)

## How to apply

Build the reference index first: upload the reference FASTA to VIsoQLR's first panel to build the GMAP index, or index the same FASTA directly with Minimap2. Align raw reads in FASTQ format either with GMAP (second VIsoQLR panel, generating BAM plus BAI index) or with Minimap2 using the splice-aware high-quality preset '-ax splice:hq -uf --secondary=no'; note Minimap2's requirement that the reference be in FASTA format (the same file used to build the GMAP index). In the SIRV benchmark, the same minimap2 BAM was then fed to all three tools so that isoform detection differences (VIsoQLR 48/68 = 71%, FLAIR 35 = 51%, StringTie2 15 = 22%) were attributable to the isoform callers, not the mapper. Validate by confirming the BAM/BAI are produced and that reproduced detection counts match the reported figures (~72% for VIsoQLR with ~99% exon coordinate accuracy).

## Related tools

- **Minimap2** (Splice-aware long-read mapper run with the '-ax splice:hq -uf --secondary=no' preset to produce the BAM alignment shared by VIsoQLR, FLAIR and StringTie2 in the SIRV benchmark) — https://github.com/lh3/minimap2
- **GMAP** (cDNA-to-genome aligner built into the VIsoQLR Docker image; used via the tool's second panel to align raw FASTQ reads using a GMAP reference index built in the first panel)
- **VIsoQLR** (Docker-based interactive tool hosting the GMAP index-building and alignment panels and consuming the resulting BAM (or GFF3/BED6) for isoform identification and quantification) — https://github.com/TBLabFJD/VIsoQLR

## Examples

```
minimap2 -ax splice:hq -uf --secondary=no reference.fa SIRV_reads.fq > aln.sam
```

## Evaluation signals

- A mapped reads BAM together with its BAI index is produced after alignment
- The reference used for Minimap2 mapping is in FASTA format and is the same file used to build the GMAP index
- Reproduced isoform detection counts match the reported benchmark: VIsoQLR 48/68 (71%), FLAIR 35 (51%), StringTie2 15 (22%)
- VIsoQLR detects ~72% of known SIRV isoforms with ~99% exon coordinate accuracy, outperforming FLAIR and StringTie2
- Using minimap2 instead of GMAP gives results similar to the GMAP-based mapping, indicating mapper robustness

## Limitations

- Minimap2 mapping limitations are advised in its GitHub documentation
- With the default 2% CEC read threshold in VIsoQLR, low-abundance isoforms (below 2%) are missed; lowering the threshold increases detection but also detects more isoforms, sometimes requiring manual parameter tuning
- Small isoform abundances (e.g. Iso5 at 4% and Iso8 at 1.7%) may be overrepresented by LRS and hard to distinguish from noise in electropherograms

## Evidence

- [methods] In this case the reference must be in FASTA format (the same file used to build the GMAP index in the first panel).: "In this case the reference must be in FASTA format (the same file used to build the GMAP index in the first panel)."
- [methods] The second panel is dedicated to align the raw reads (in FASTQ format) using GMAP.: "The second panel is dedicated to align the raw reads (in FASTQ format) using GMAP."
- [methods] two download bottoms will appear: the mapped reads (BAM) and its index (BAI).: "two download bottoms will appear: the mapped reads (BAM) and its index (BAI)."
- [other] Align the SIRV long reads to the reference FASTA with Minimap2 using the high-quality splice-aware preset '-ax splice:hq -uf --secondary=no': [evidence span withheld — anchor status: not_found]
- [discussion] VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools.: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
