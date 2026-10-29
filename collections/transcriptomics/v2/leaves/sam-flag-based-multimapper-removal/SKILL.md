---
name: sam-flag-based-multimapper-removal
description: Use when merged SAM alignment files from the conjoint HISAT2/Bowtie2
  genome-transcriptome alignment must be reduced to uniquely mapped reads and converted
  to sorted, indexed BAM files.
when_to_use_negative:
- Input alignments are not merged SAM files from the HISAT2/Bowtie2 step (e.g. raw
  FASTQ or already-counted read tables)
- The analysis goal requires retaining multimapping or secondary alignments rather
  than uniquely mapped reads only
- Downstream tools expect unsorted/unindexed alignments rather than coordinate-sorted
  BAM with .bai indexes
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0091
tools:
- name: samtools
  role: Sorts the merged SAM files by coordinate, filters them with samtools view
    -F 3844 to discard multimappers and non-primary records, converts to compressed
    BAM, and generates .bai indexes
  repo: https://sourceforge.net/projects/samtools/files/samtools/
- name: HISAT2
  role: Upstream aligner whose conjoint genome/transcriptome alignment output (merged
    SAM files) is the input to the filtering step
  repo: http://daehwankimlab.github.io/hisat2/
- name: Bowtie 2
  role: Co-aligner used with HISAT2 for the genome and transcriptome alignment whose
    merged SAM output feeds the sort/filter step
  repo: https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:648c92942d4245f1bc16feaa8d1f05b416a88d1a6025c2142a518c3675291837
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  tools:
  - Bowtie 2
  - HISAT2
  - RiboDoc
  - samtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# SAM flag-based multimapper removal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filter merged HISAT2/Bowtie2 alignment SAM files in RiboDoc with samtools (v1.12) to retain only uniquely mapped reads, converting them into coordinate-sorted, flag-filtered BAM files with .bai indexes for downstream RiboSeq analysis.

## When to use

Use when merged SAM alignment files from the conjoint HISAT2/Bowtie2 genome-transcriptome alignment must be reduced to uniquely mapped reads and converted to sorted, indexed BAM files.

## When NOT to use

- Input alignments are not merged SAM files from the HISAT2/Bowtie2 step (e.g. raw FASTQ or already-counted read tables)
- The analysis goal requires retaining multimapping or secondary alignments rather than uniquely mapped reads only
- Downstream tools expect unsorted/unindexed alignments rather than coordinate-sorted BAM with .bai indexes

## Inputs

- Merged SAM alignment files from the conjoint HISAT2/Bowtie2 genome and transcriptome alignment step (one per sample)
- samtools v1.12 (via the RiboDoc Docker container)

## Outputs

- Coordinate-sorted, flag-filtered BAM files (*.25-35.bam) in RESULTS/BAM.25-35, one per sample
- Corresponding BAM index files (*.25-35.bai)

## How to apply

Take the merged SAM files produced by the conjoint HISAT2/Bowtie2 alignment for each sample and sort them by genomic coordinate using samtools sort (v1.12). Then filter with samtools view -F 3844, which discards reads carrying the flag combination unmapped (4), mate-unmapped (8), secondary (256), QC-fail (512), and supplementary (2048), thereby keeping only uniquely mapped reads. Convert the filtered alignments to compressed BAM and generate a .bai index so files can be randomly accessed. Write the outputs as *.25-35.bam / *.25-35.bai pairs into RESULTS/BAM.25-35, one pair per sample (e.g. Wild_Type.1.25-35.bam). Rationale: ribosome profiling read counting (htseq-count) and P-site analysis downstream assume each read is assigned to a single location, so multimappers and secondary/supplementary records must be removed before counting.

## Related tools

- **samtools** (Sorts the merged SAM files by coordinate, filters them with samtools view -F 3844 to discard multimappers and non-primary records, converts to compressed BAM, and generates .bai indexes) — https://sourceforge.net/projects/samtools/files/samtools/
- **HISAT2** (Upstream aligner whose conjoint genome/transcriptome alignment output (merged SAM files) is the input to the filtering step) — http://daehwankimlab.github.io/hisat2/
- **Bowtie 2** (Co-aligner used with HISAT2 for the genome and transcriptome alignment whose merged SAM output feeds the sort/filter step) — https://bowtie-bio.sourceforge.net/bowtie2/index.shtml

## Examples

```
samtools sort merged.sam > sorted.sam && samtools view -F 3844 -b sorted.sam > sample.25-35.bam && samtools index sample.25-35.bam
```

## Evaluation signals

- For every input sample, exactly one sorted, flag-filtered BAM plus its .bai index exists in RESULTS/BAM.25-35
- Each BAM is successfully indexable and readable by samtools (e.g. samtools quickcheck / samtools index succeeds)
- No remaining records carry flag bits 4, 8, 256, 512, or 2048 (verify with samtools view -c -f 256 being zero, etc.)
- Retained reads fall within the selected RPF length window (default 25-35 nt), consistent with the *.25-35.bam naming

## Limitations

- The -F 3844 filter removes all secondary/supplementary records, so multi-locus reads are lost entirely rather than assigned a best location
- Outputs are limited to the 25-35 nt RPF selection; reads outside this range are excluded from these BAM files

## Evidence

- [methods] Flag-filter description: [evidence span withheld — anchor status: not_found]
