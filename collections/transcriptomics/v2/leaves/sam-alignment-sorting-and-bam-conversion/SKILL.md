---
name: sam-alignment-sorting-and-bam-conversion
description: Use when RiboSeq (or similar) alignment output exists as merged SAM files
  from conjoint genome/transcriptome alignment and indexed, uniquely-mapped BAM files
  are needed for downstream counting, P-site, or QC steps.
when_to_use_negative:
- Input alignments are already sorted, indexed BAM files rather than merged SAM files
  — re-sorting/filtering would duplicate the step
- The analysis requires retaining secondary or supplementary alignments (e.g. multimapping-aware
  quantification), since -F 3844 discards them
edam_operation: http://edamontology.org/operation_3564
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: samtools
  role: Sorts SAM alignments by coordinate, filters reads with view -F 3844 to keep
    uniquely mapped reads, converts to compressed BAM, and builds the .bai index (v1.12
    in RiboDoc)
  repo: https://sourceforge.net/projects/samtools/files/samtools/
- name: RiboDoc
  role: Docker/Snakemake pipeline orchestrating the sort-filter-index step and writing
    the *.25-35.bam/.bai outputs to RESULTS/BAM.25-35
  repo: https://github.com/equipeGST/RiboDoc
- name: HISAT2
  role: Upstream aligner whose merged SAM output (with Bowtie2) is the input to the
    sorting and BAM conversion step
- name: Bowtie2
  role: Upstream aligner (also used for rRNA/tRNA/viral RNA contaminant removal) whose
    merged SAM output feeds the sorting and BAM conversion step
  repo: https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:f1f337a6de7428c735c8091bb6ae4bb12540fd661af86f1423fdbb00b06e4f7e
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
  edam_operation: http://edamontology.org/operation_3564
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# sam-alignment-sorting-and-bam-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the RiboDoc step that sorts the merged HISAT2/Bowtie2 SAM alignments by genomic coordinate, filters out reads with flag combination 3844 to keep uniquely mapped reads, and converts them to indexed BAM files using samtools v1.12.

## When to use

Use when RiboSeq (or similar) alignment output exists as merged SAM files from conjoint genome/transcriptome alignment and indexed, uniquely-mapped BAM files are needed for downstream counting, P-site, or QC steps.

## When NOT to use

- Input alignments are already sorted, indexed BAM files rather than merged SAM files — re-sorting/filtering would duplicate the step
- The analysis requires retaining secondary or supplementary alignments (e.g. multimapping-aware quantification), since -F 3844 discards them

## Inputs

- Merged SAM alignment files per sample from the conjoint HISAT2/Bowtie2 genome and transcriptome alignment step
- samtools v1.12 (sort, view, index)

## Outputs

- Coordinate-sorted, flag-filtered BAM file per sample (e.g. RESULTS/BAM.25-35/Wild_Type.1.25-35.bam)
- Matching .bai index per BAM file (e.g. RESULTS/BAM.25-35/Wild_Type.1.25-35.bai)

## How to apply

Take the merged SAM alignment files produced by the conjoint HISAT2/Bowtie2 genome and transcriptome alignment step for each sample and sort them by genomic coordinate with samtools sort (v1.12). Filter the sorted alignments with samtools view -F 3844, which discards reads carrying the flags unmapped (4), mate-unmapped (8), secondary (256), QC-fail (512), and supplementary (2048), thereby retaining only uniquely mapped reads. Convert the filtered alignments to compressed BAM format and generate the .bai index with samtools index so files can be randomly accessed by downstream tools such as htseq-count and riboWaltz. In RiboDoc the outputs are written as *.25-35.bam and *.25-35.bai pairs in the RESULTS/BAM.25-35 folder, one pair per sample (e.g. Wild_Type.1.25-35.bam), reflecting the prior 25–35 nt RPF length selection.

## Related tools

- **samtools** (Sorts SAM alignments by coordinate, filters reads with view -F 3844 to keep uniquely mapped reads, converts to compressed BAM, and builds the .bai index (v1.12 in RiboDoc)) — https://sourceforge.net/projects/samtools/files/samtools/
- **RiboDoc** (Docker/Snakemake pipeline orchestrating the sort-filter-index step and writing the *.25-35.bam/.bai outputs to RESULTS/BAM.25-35) — https://github.com/equipeGST/RiboDoc
- **HISAT2** (Upstream aligner whose merged SAM output (with Bowtie2) is the input to the sorting and BAM conversion step)
- **Bowtie2** (Upstream aligner (also used for rRNA/tRNA/viral RNA contaminant removal) whose merged SAM output feeds the sorting and BAM conversion step) — https://bowtie-bio.sourceforge.net/bowtie2/index.shtml

## Examples

```
samtools sort -o sample.sorted.sam merged.sam && samtools view -bS -F 3844 sample.sorted.sam > sample.25-35.bam && samtools index sample.25-35.bam
```

## Evaluation signals

- For every input sample exactly one sorted, flag-filtered BAM plus its .bai index exists in RESULTS/BAM.25-35
- Each BAM file is successfully indexable and readable by samtools (e.g. samtools view/samtools quickcheck passes)
- Filtered BAM contains no reads with flags 4, 8, 256, 512, or 2048 — i.e. samtools view -c -f 4 returns 0
- Output file names follow the [CONDITION].[NAME].[REPLICATE].25-35.bam pattern derived from the input FastQ naming (e.g. Wild_Type.1.25-35.bam)
- Downstream steps (htseq-count, riboWaltz) can open the BAMs randomly via the .bai index

## Limitations

- The -F 3844 filter enforces unique mapping by discarding secondary alignments; genuinely multimapping signal is lost rather than probabilistically assigned
- RiboDoc's samtools version is fixed (v1.12) by the Docker container, so behavior may differ from other samtools versions
- The example GEO dataset (GSE173856) does not align enough material for the riboWaltz branch of the pipeline; TRiP should be selected instead

## Evidence
