---
name: stranded-rnaseq-bam-preprocessing
description: Use when you have raw paired-end RNA-Seq reads (e.g. firststrand Illumina
  libraries) destined for aptardi polyA-site prediction and need to produce a sorted
  BAM aligned with HISAT2 plus a StringTie-reconstructed input transcriptome.
when_to_use_negative:
- You already have a genome-aligned, sorted BAM and an assembled input transcriptome
  — proceed directly to running aptardi in Mode 1
- You only need to build a custom aptardi model (Mode 2) from gold-standard polyA
  sites and have no sample reads to align
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: FastQC
  role: Recommended assessment of read quality prior to any processing steps
- name: Cutadapt
  role: Trimming/processing of reads based on FastQC results
- name: HISAT2
  role: Aligns stranded paired-end reads to the genome with --rna-strandness RF --dta
    --reorder
- name: SAMtools
  role: Sorts the HISAT2 alignment into the coordinate-sorted BAM required by aptardi
    (v1.9 in the aptardi README; must not be installed via bioconda in the aptardi
    environment)
  repo: https://github.com/luskry/aptardi
- name: StringTie
  role: Reconstructs the input transcriptome (stringtie.gtf) from the sorted BAM using
    --rf for firststrand libraries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:3904df62799941807496d2a173c9c5383be7b63216569a9555fc44fa33983ff5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - BEDtools
  - Cutadapt
  - FastQC
  - HISAT2
  - SAMtools
  - StringTie
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Stranded RNA-Seq BAM Preprocessing for aptardi

## Summary

Prepares stranded paired-end RNA-Seq reads into a coordinate-sorted BAM and a reconstructed transcriptome (GTF) suitable as aptardi inputs, via FastQC/Cutadapt quality processing, HISAT2 alignment with explicit strandness, SAMtools sorting, and StringTie reconstruction.

## When to use

Use when you have raw paired-end RNA-Seq reads (e.g. firststrand Illumina libraries) destined for aptardi polyA-site prediction and need to produce a sorted BAM aligned with HISAT2 plus a StringTie-reconstructed input transcriptome.

## When NOT to use

- You already have a genome-aligned, sorted BAM and an assembled input transcriptome — proceed directly to running aptardi in Mode 1
- You only need to build a custom aptardi model (Mode 2) from gold-standard polyA sites and have no sample reads to align

## Inputs

- Raw paired-end RNA-Seq reads (FASTQ) from a firststrand Illumina paired-end library
- HISAT2 genome index (e.g. for hg38)
- Optional reference guide annotation (GTF) for StringTie -G

## Outputs

- Coordinate-sorted BAM of genome-aligned reads (sorted.bam), produced with SAMtools
- StringTie-reconstructed transcriptome (stringtie.gtf) in GTF format

## How to apply

First assess read quality with FastQC and perform adapter/quality processing with Cutadapt based on the results. Align the trimmed reads to the reference genome with HISAT2 using explicit strandedness (--rna-strandness RF for this firststrand library), --dta for downstream transcript assembly, and --reorder, then sort the resulting SAM/BAM with SAMtools. Reconstruct the transcriptome from the sorted BAM with StringTie (--rf for firststrand) to obtain stringtie.gtf, optionally guided by a reference annotation via -G. These two files (sorted.bam and stringtie.gtf) are exactly the RNA-Seq and transcriptome inputs aptardi consumes in Mode 1 alongside hg38.fa, model.hdf5, and scale.pk. Correct strandness flags matter because aptardi's mate-orientation argument defaults to --a fr for firststrand Illumina paired-end libraries, and mis-specified orientation corrupts coverage profiles over 3' terminal exons.

## Related tools

- **FastQC** (Recommended assessment of read quality prior to any processing steps)
- **Cutadapt** (Trimming/processing of reads based on FastQC results)
- **HISAT2** (Aligns stranded paired-end reads to the genome with --rna-strandness RF --dta --reorder)
- **SAMtools** (Sorts the HISAT2 alignment into the coordinate-sorted BAM required by aptardi (v1.9 in the aptardi README; must not be installed via bioconda in the aptardi environment)) — https://github.com/luskry/aptardi
- **StringTie** (Reconstructs the input transcriptome (stringtie.gtf) from the sorted BAM using --rf for firststrand libraries)

## Examples

```
hisat2 -q -p 5 --reorder -t --rna-strandness RF --dta -x <hisat2_index> -1 R1.fq.gz -2 R2.fq.gz | samtools sort -o sorted.bam; stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>
```

## Evaluation signals

- samtools --version and bedtools --version can be called from the conda environment, and aptardi --version succeeds, while the bioconda samtools/bedtools packages are NOT installed in that same environment
- sorted.bam is coordinate-sorted and was produced with HISAT2 flags matching the library's strandedness (e.g. RF for this firststrand dataset)
- stringtie.gtf parses as valid GTF and was generated with --rf so the transcript reconstruction respects the firststrand library orientation
- aptardi subsequently runs in Mode 1 on sorted.bam, stringtie.gtf, hg38.fa with model.hdf5/scale.pk and --a fr, completing with a name_aptardi.gtf that contains both original and aptardi transcripts

## Limitations

- Strandness must be specified consistently across HISAT2, StringTie, and aptardi; aptardi's BNLx/SHR results were modestly worse and the authors 'cannot rule out the possibility that this is owing to the unstranded RNA-Seq for these data sets and/or different species'
- The aptardi README warns 'The bioconda samtools and bedtools packages may prevent aptardi from working, please make sure these packages are not installed in the same environment as aptardi'
- The input transcriptome quality affects downstream aptardi redundancy; a poorly curated StringTie reconstruction increases false-positive aptardi transcripts sharing 3' terminal exons

## Evidence

- [methods] FastQC and Cutadapt quality processing recommended before alignment: [evidence span withheld — anchor status: not_found]
- [methods] HISAT2 stranded alignment command with --dta and --reorder: "(hisat2 -q -p 5 --reorder -t --rna-strandness RF --dta -x <hisat2_index> -1"
- [methods] StringTie firststrand reconstruction of the input transcriptome: "stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>"
- [readme] README caution about bioconda samtools/bedtools conflicts: "The bioconda samtools and bedtools packages may prevent aptardi from working, please make sure these packages are not installed in the same environment as aptardi"
- [readme] aptardi mate-orientation default ties alignment strandness to the tool: "--a/-a <fr or rf>		Upstream/downstream mate orientations for paired-end alignment against the forward reference strand, fr = firststrand (appropriate for Illumina paired-end library pre"
