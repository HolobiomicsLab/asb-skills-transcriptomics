---
name: bam-alignment-processing
description: Use when you have long-read cDNA/direct-RNA reads (FASTQ) plus a reference
  FASTA and need a splice-aware BAM alignment as input for isoform detection, quantification
  and tool benchmarking against a known isoform annotation.
when_to_use_negative:
- Input is already an isoform set or annotation — this skill only produces alignments
  and per-tool isoform calls, not comparisons of pre-existing isoform lists
- You need short-read splicing analysis or raw-read QC — this skill targets long-read
  BAM generation for isoform detection only
- Your goal is de novo transcript discovery without a reference FASTA — all steps
  here require a reference sequence (FASTA for minimap2 or GMAP index building)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Splice-aware long-read mapper producing the BAM alignment consumed by all
    three isoform-detection tools (preset '-ax splice:hq -uf --secondary=no')
  repo: https://github.com/lh3/minimap2
- name: VIsoQLR
  role: Interactive Docker-based analyzer that takes the uploaded BAM (or GFF3/BED6),
    defines consensus splice sites and quantifies isoforms with a default 2% CEC read
    threshold
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: FLAIR
  role: Isoform-detection tool run on the same minimap2 BAM without prior splice-site
    annotation, as benchmark comparator
- name: StringTie2
  role: Transcriptome assembler run on the same minimap2 BAM as benchmark comparator
- name: GMAP
  role: Alternative cDNA-to-genome aligner whose reference index is built inside VIsoQLR
    and used to produce the BAM mapping
  repo: https://github.com/TBLabFJD/VIsoQLR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:366ad64e60f028ad1a728e41142ba6d2631cb4e435ea5b3de4961f934cc49be2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - FLAIR
  - GMAP
  - Minimap2
  - StringTie2
  - VIsoQLR
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bam-alignment-processing

## Summary

Process long-read RNA-seq alignments to a BAM file (via minimap2 splice-aware mode or GMAP) so that isoform-detection tools such as VIsoQLR, FLAIR and StringTie2 can define consensus splice sites, detect and quantify isoforms, and be compared against a known annotation like the 68 SIRV isoforms.

## When to use

Use when you have long-read cDNA/direct-RNA reads (FASTQ) plus a reference FASTA and need a splice-aware BAM alignment as input for isoform detection, quantification and tool benchmarking against a known isoform annotation.

## When NOT to use

- Input is already an isoform set or annotation — this skill only produces alignments and per-tool isoform calls, not comparisons of pre-existing isoform lists
- You need short-read splicing analysis or raw-read QC — this skill targets long-read BAM generation for isoform detection only
- Your goal is de novo transcript discovery without a reference FASTA — all steps here require a reference sequence (FASTA for minimap2 or GMAP index building)

## Inputs

- Long-read sequencing reads in FASTQ format (e.g. PacBio Iso-Seq or ONT cDNA/direct RNA)
- Reference genome sequence in FASTA format (same file used to build the GMAP index)
- Known isoform annotation (e.g. the 68 SIRV isoform annotation) for detection comparison

## Outputs

- Splice-aware alignment in BAM format with BAI index
- Per-tool detected isoform sets (isoform calls for VIsoQLR, FLAIR and StringTie2)
- Detection counts per tool versus the known annotation (e.g. 48/68 = 71% VIsoQLR, 35 = 51% FLAIR, 15 = 22% StringTie2), tabulated e.g. into benchmark_detection_counts.csv

## How to apply

Align the long reads to the reference FASTA with minimap2 using the high-quality splice-aware preset '-ax splice:hq -uf --secondary=no' (or use GMAP inside VIsoQLR by first uploading the reference to build the GMAP index, then aligning the raw reads in FASTQ format). Retrieve the mapped reads as a BAM plus its BAI index. Launch VIsoQLR via Docker ('docker pull tblabfjd/visoqlr:latest'; 'docker run -it -p 8888:8888 tblabfjd/visoqlr:latest'), upload the BAM (VIsoQLR also accepts GFF3 or BED6), and let it automatically identify and quantify isoforms with default parameters, including the default 2% CEC read threshold. Run FLAIR and StringTie2 on the same BAM with default parameters, then compare each tool's detected isoform set against the known annotation (e.g. the 68 SIRV isoforms) to count correct detections. The rationale is that all three tools consume the identical minimap2 BAM so isoform-detection differences are attributable to the tools, not the mapper.

## Related tools

- **minimap2** (Splice-aware long-read mapper producing the BAM alignment consumed by all three isoform-detection tools (preset '-ax splice:hq -uf --secondary=no')) — https://github.com/lh3/minimap2
- **VIsoQLR** (Interactive Docker-based analyzer that takes the uploaded BAM (or GFF3/BED6), defines consensus splice sites and quantifies isoforms with a default 2% CEC read threshold) — https://github.com/TBLabFJD/VIsoQLR
- **FLAIR** (Isoform-detection tool run on the same minimap2 BAM without prior splice-site annotation, as benchmark comparator)
- **StringTie2** (Transcriptome assembler run on the same minimap2 BAM as benchmark comparator)
- **GMAP** (Alternative cDNA-to-genome aligner whose reference index is built inside VIsoQLR and used to produce the BAM mapping) — https://github.com/TBLabFJD/VIsoQLR

## Examples

```
minimap2 -ax splice:hq -uf --secondary=no reference.fa sirv_reads.fastq > aln.bam && samtools index aln.bam
```

## Evaluation signals

- The BAM file is accompanied by a valid BAI index and contains splice-aligned reads rather than unspliced contig alignments
- VIsoQLR reproduced benchmark detection counts: 48/68 (~71%) SIRV isoforms with ~99% exon coordinate accuracy; FLAIR 35 (51%); StringTie2 15 (22%)
- Detected isoform coordinates agree with the known SIRV annotation (99% exon coordinate accuracy for VIsoQLR)
- StringTie2 outputs showing extended isoform boundaries that are NOT present in the source BAM flag a known StringTie2 artifact
- Most isoforms missed by VIsoQLR's automatic analysis fall below the default 2% CEC read threshold; lowering the threshold increases detected isoforms

## Limitations

- Missing isoforms in VIsoQLR automatic analysis are attributed to reads below the default 2% CEC threshold, and recovering them requires manual parameter tuning (Read threshold (%), Padding (# of bases), Merge close splice sites (# of bases))
- StringTie2 extends the boundaries of some isoforms and does not call isoforms lacking certain exons; the boundary extension is an artifact not present in the BAM files
- No software version numbers are reported for VIsoQLR, StringTie2 or FLAIR, complicating exact reproduction
- minimap2's known limitation is advised in its GitHub documentation

## Evidence

- [methods] Tool auto-detects isoforms from uploaded BAM: "navigate to your file system to upload your aligner reads in `GFF3`, `BED6`, or `BAM` file format."
- [methods] GMAP index and read alignment panels: "The first one is used to build the reference index used by GMAP by uploading the reference sequence(s)."
- [methods] BAM plus index output: "two download bottoms will appear: the mapped reads (BAM) and its index (BAI)."
- [discussion] Benchmark accuracy headline: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [abstract] VIsoQLR takes aligned sequences: "Our tool takes sequences aligned to a reference, and for each gene, it defines consensus splice sites and quantifies isoforms."
- [readme] Benchmark scripts repository: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark input data source: [evidence span withheld — anchor status: external_doc]
