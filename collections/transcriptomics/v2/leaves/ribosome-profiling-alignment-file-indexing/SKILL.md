---
name: ribosome-profiling-alignment-file-indexing
description: Use when SAM alignment files from conjoint genome/transcriptome RiboSeq
  alignment (HISAT2 + Bowtie2) must be coordinate-sorted, filtered to uniquely mapped
  reads, converted to BAM, and indexed for downstream htseq-count or riboWaltz steps.
when_to_use_negative:
- Input is still raw FastQ (untrimmed/unfiltered) — adapter trimming with Cutadapt
  and rRNA/tRNA contaminant depletion with Bowtie2 must happen before alignment and
  BAM generation
- BAM files already sorted, flag-filtered, and indexed — proceed directly to read
  counting with htseq-count or P-site offset prediction with riboWaltz/TRiP
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: samtools
  role: Sorts alignments by coordinate, filters with samtools view -F 3844, converts
    to compressed BAM, and generates .bai indices (v1.12)
  repo: https://sourceforge.net/projects/samtools/files/samtools/
- name: RiboDoc
  role: Docker/Snakemake pipeline that orchestrates the sort-filter-index step and
    writes *.25-35.bam/.bai pairs to RESULTS/BAM.25-35
  repo: https://github.com/equipeGST/RiboDoc
- name: HISAT2
  role: Upstream conjoint genome and transcriptome aligner whose merged SAM output
    is the input to sorting/indexing
  repo: http://daehwankimlab.github.io/hisat2/
- name: Bowtie2
  role: Co-aligner with HISAT2 for genome/transcriptome alignment; merged SAM output
    feeds the sort/filter/index step
  repo: https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:47ad220f81cbff2dd85f125e7cd39326f9697c517fdbe7fc6cafbd0694a65530
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  tools:
  - Bowtie2
  - HISAT2
  - RiboDoc
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ribosome-profiling-alignment-file-indexing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sort, flag-filter, and index merged HISAT2/Bowtie2 SAM alignments from a ribosome profiling pipeline into random-accessible BAM files (samtools v1.12), keeping only uniquely mapped RiboSeq reads for downstream counting and P-site analysis.

## When to use

Use when SAM alignment files from conjoint genome/transcriptome RiboSeq alignment (HISAT2 + Bowtie2) must be coordinate-sorted, filtered to uniquely mapped reads, converted to BAM, and indexed for downstream htseq-count or riboWaltz steps.

## When NOT to use

- Input is still raw FastQ (untrimmed/unfiltered) — adapter trimming with Cutadapt and rRNA/tRNA contaminant depletion with Bowtie2 must happen before alignment and BAM generation
- BAM files already sorted, flag-filtered, and indexed — proceed directly to read counting with htseq-count or P-site offset prediction with riboWaltz/TRiP

## Inputs

- Merged SAM alignment files from conjoint HISAT2/Bowtie2 genome and transcriptome alignment, one per RiboSeq sample (mapped reads only)
- RiboDoc Docker/Snakemake project folder with per-sample naming [CONDITION]_[NAME].[REPLICATE]

## Outputs

- Coordinate-sorted, flag-filtered BAM files (one per sample) in RESULTS/BAM.25-35
- Corresponding .bai index files (one per sample), e.g. Wild_Type.1.25-35.bai

## How to apply

Take the merged SAM files produced by the conjoint HISAT2/Bowtie2 genome and transcriptome alignment (keeping only mapped reads) for each sample. Sort each SAM by genomic coordinate with samtools sort (samtools v1.12), then apply samtools view -F 3844 to discard reads carrying the flag combination unmapped (4), mate-unmapped (8), secondary (256), QC-fail (512), and supplementary (2048), thereby retaining uniquely mapped reads. Convert the filtered alignments to compressed BAM and generate the .bai index so files can be randomly accessed by downstream tools (htseq-count, riboWaltz/TRiP). Write one *.25-35.bam/.bai pair per sample to RESULTS/BAM.25-35 (e.g. Wild_Type.1.25-35.bam), reflecting RiboDoc's default 25–35 nt RPF length selection.

## Related tools

- **samtools** (Sorts alignments by coordinate, filters with samtools view -F 3844, converts to compressed BAM, and generates .bai indices (v1.12)) — https://sourceforge.net/projects/samtools/files/samtools/
- **RiboDoc** (Docker/Snakemake pipeline that orchestrates the sort-filter-index step and writes *.25-35.bam/.bai pairs to RESULTS/BAM.25-35) — https://github.com/equipeGST/RiboDoc
- **HISAT2** (Upstream conjoint genome and transcriptome aligner whose merged SAM output is the input to sorting/indexing) — http://daehwankimlab.github.io/hisat2/
- **Bowtie2** (Co-aligner with HISAT2 for genome/transcriptome alignment; merged SAM output feeds the sort/filter/index step) — https://bowtie-bio.sourceforge.net/bowtie2/index.shtml

## Examples

```
samtools view -F 3844 sample.sam | samtools sort -o sample.25-35.bam; samtools index sample.25-35.bam
```

## Evaluation signals

- For every input sample, exactly one sorted, flag-filtered BAM plus its .bai index exists in RESULTS/BAM.25-35 (e.g. Wild_Type.1.25-35.bam and .bai)
- Each BAM is successfully indexable and readable by samtools (e.g. samtools quickcheck and samtools index complete without error)
- No reads with flags 4 (unmapped), 8 (mate-unmapped), 256 (secondary), 512 (QC-fail), or 2048 (supplementary) remain in output BAMs (verifiable with samtools view -c -f <flag>)
- BAM content matches the merged HISAT2/Bowtie2 SAM input after removing multimappers — read counts differ only by the discarded flag classes
- Filenames follow the [CONDITION]_[NAME].[REPLICATE].25-35 convention consistent with RiboDoc's 25–35 nt RPF length selection

## Limitations

- The -F 3844 filter removes unmapped, secondary, QC-fail, and supplementary records but does not itself resolve all multimapping classes; RiboDoc's pipeline relies on merging mapped-only HISAT2/Bowtie2 alignments upstream
- RiboDoc's default RPF selection is 25–35 nt; samples with atypical footprint-length distributions (e.g. the GEO GSE173856 example that 'does not align enough material for a use of the riboWaltz pipeline') may need the offset tool switched to TRiP
- File names must avoid dashes, parentheses, blank spaces, and special characters except underscores, or the Snakemake pipeline breaks

## Evidence

- [other] the SAM files produced by Hisat2 and Bowtie2 are merged with only mapped reads kept, then sorted and filtered to retain uniquely mapped reads: [evidence span withheld — anchor status: not_found]
